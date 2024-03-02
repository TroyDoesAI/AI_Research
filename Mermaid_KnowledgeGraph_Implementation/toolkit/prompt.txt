import argparse
import json
import os
import requests
import subprocess
import tempfile

# This script integrates the MermaidDiagramGenerator into the main pipeline, generating and validating Mermaid diagrams for each unique response. 
# If a diagram generation fails, it adjusts the temperature and retries until a valid diagram is produced or all attempts are exhausted. 
# Each valid diagram is saved as a PNG in the "Entries" folder with a filename reflecting its entry and output number. 
# The script writes to the JSON file after processing each input, ensuring that the data is incrementally saved.

##### MermaidDiagramGenerator Class #####
class MermaidDiagramGenerator:
    def __init__(self, theme='dark', background='transparent'):
        self._theme = theme
        self._background = background
        self._entries_dir = os.path.join(os.getcwd(), 'Entries')
        os.makedirs(self._entries_dir, exist_ok=True)

    def convert_to_image(self, mermaid_code, entry_number, output_number):
        clean_code = self._remove_mermaid_block_markers(mermaid_code)
        output_filename = f"entry_{entry_number}_{output_number}.png"
        output_path = os.path.join(self._entries_dir, output_filename)
        self._generate_image_from_code(clean_code, output_path)
        return output_path

    def _remove_mermaid_block_markers(self, code):
        code_lines = code.strip().splitlines()
        if code_lines[0].startswith("```mermaid") and code_lines[-1] == "```":
            return "\n".join(code_lines[1:-1]).strip()
        return code

    def _generate_image_from_code(self, mermaid_code, output_path):
        with tempfile.NamedTemporaryFile(delete=False, mode='w', suffix='.mmd') as temp_file:
            temp_file.write(mermaid_code)
            input_path = temp_file.name
        result = subprocess.run(["mmdc", "-i", input_path, "-o", output_path, "-t", self._theme, "-b", self._background], shell=True, check=False)
        os.remove(input_path)
        if result.returncode != 0:
            raise ValueError("Mermaid diagram generation failed.")

##### Script to Generate Responses and Validate Mermaid Diagrams #####
def read_input(input_source):
    if os.path.isfile(input_source):
        filename, file_extension = os.path.splitext(input_source)
        if file_extension == '.json':
            with open(input_source, 'r') as file:
                return json.load(file)
        elif file_extension == '.txt':
            with open(input_source, 'r') as file:
                return [{"input": file.read()}]
    else:
        return [{"input": input_source}]

def generate_unique_responses(prompt, base_temperatures, stream, generator, entry_number):
    prompt_template = """
    Below is an instruction that describes a task, paired with an input that provides further context.
    Write a response that appropriately completes the request.

    ### Instruction:
    Create the mermaid diagram for the following input:

    ### Input:
    {input}

    ### Response:
    ```mermaid
    graph TD;

    """.format(input=prompt)
    url = "http://127.0.0.1:5000/v1/completions"
    headers = {"Content-Type": "application/json"}
    dataset_entries = []
    unique_outputs = set()

    for output_number, temp in enumerate(base_temperatures, start=1):
        while True:
            data = {
                "prompt": prompt_template,
                "max_tokens": 4096,
                "temperature": temp,
                "top_p": 1.0,
                "seed": -1,
                "top_k": 40,
                "repetition_penalty": 1.0,
                "typical_p": 1.0,
                "stream": stream,
            }

            response = requests.post(url, headers=headers, json=data, verify=False)
            response_text = response.json()['choices'][0]['text'].strip()

            # Remove ```mermaid and ``` markers if present
            if response_text.startswith("```mermaid"):
                response_text = response_text[len("```mermaid"):].strip()
            if response_text.endswith("```"):
                response_text = response_text[:-len("```")].strip()

            # Ensure the Mermaid code starts with 'graph TD;' for both image generation and dataset entry
            mermaid_code_for_image = "graph TD;\n" + response_text
            formatted_response_text = "'''mermaid\ngraph TD;\n" + response_text + "\n'''"

            if mermaid_code_for_image not in unique_outputs:
                try:
                    # Convert the Mermaid code to an image
                    image_path = generator.convert_to_image(mermaid_code_for_image, entry_number, output_number)
                    print(f"Mermaid diagram generated at: {image_path}")

                    unique_outputs.add(mermaid_code_for_image)  # Add the Mermaid code to the set to avoid duplicates
                    break
                except ValueError as e:
                    print(f"Validation failed, retrying... Error: {e}")
                    temp += 0.1  # Adjust temperature and retry if Mermaid diagram is invalid
            else:
                temp += 0.1  # Adjust temperature if output is not unique

            dataset_entry = {
                "input": prompt,
                "output": formatted_response_text,  # Use the formatted Mermaid code for the dataset entry
                "temperature": temp
            }
            dataset_entries.append(dataset_entry)

    return dataset_entries

def main(input_source, stream=False):
    generator = MermaidDiagramGenerator()
    input_data = read_input(input_source)
    base_temperatures = [i / 10 for i in range(5, 11)]  # Adjusted for batch of unique outputs per input
    output_file = "output.json"
    all_entries = []  # Initialize an empty list to store all entries

    for entry_number, entry in enumerate(input_data, start=1):
        prompt = entry.get("input", "")
        if prompt:
            entries = generate_unique_responses(prompt, base_temperatures, stream, generator, entry_number)
            all_entries.extend(entries)  # Extend the list with new entries

    # Write all entries to the JSON file at once
    with open(output_file, "w") as f:
        json.dump(all_entries, f, indent=4)  # Dump the entire list of entries into the file

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Generate unique responses and validate Mermaid diagrams.")
    parser.add_argument('input_source', type=str, help='A multi-line string, path to a .txt file, or a .json file with prompts.')
    parser.add_argument('--stream', action='store_true', help='Use streaming responses.')
    args = parser.parse_args()
    
    main(args.input_source, args.stream)