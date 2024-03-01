# Language Model Diversification Toolkit

## Overview
The Language Model Diversification Toolkit empowers the development of more adaptable and robust language models by facilitating the creation of diversified training datasets. Leveraging advanced temperature-controlled text generation and incorporating mechanisms for the validation of complex data structures such as Mermaid diagrams, this toolkit is pivotal in enhancing model generalization capabilities across a broad spectrum of NLP tasks.

## Key Features
- **Temperature-Controlled Text Generation**: Dynamically generates text over a range of temperatures, ensuring a richly diverse dataset.
- **Mermaid Diagram Integration**: Validates and generates Mermaid diagrams, adding a structural and visual dimension to text-based datasets.
- **Unique Output Assurance**: Implements retry mechanisms to guarantee the uniqueness of generated outputs, thereby improving dataset quality and consistency.
- **Incremental Dataset Construction**: Writes outputs to the dataset file in real-time, ensuring no data loss and enabling immediate review and use of generated data.

## Installation
This toolkit requires Python 3.8 or later and npm for installing the Mermaid CLI tool. Follow these steps to set up your environment:

1. **Clone the Repository**:
    ```bash
    git clone <repository-url>
    ```

2. **Install Python Dependencies**:
    Navigate to the toolkit's directory and install the required Python packages:
    ```
    requests==2.25.1
    argparse==1.4.0
    ```

3. **Install Mermaid CLI**:
    The Mermaid CLI tool is necessary for diagram generation and validation:
    ```bash
    npm install -g @mermaid-js/mermaid-cli
    ```

## Usage
Execute the toolkit script with your chosen input source. The toolkit accepts inputs in various formats: plain text, `.txt`, or `.json` files containing multiple prompts. Optional streaming can be enabled for real-time response generation.

```bash
python main_script.py "path/to/your_input_file.json"
```

```bash
python main_script.py "path/to/your_input_file.txt"
```

```bash
python main_script.py "input as plain text"
```

Outputs, including diversified text entries and associated Mermaid diagrams (if enabled), are saved in an `output.json` file and /Entries.

## Contribution to NLP Research and Development
The Language Model Diversification Toolkit is designed not just as a utility for dataset enhancement but as a contribution to the broader NLP research community. It embodies the practical application of theoretical insights from cognitive linguistics and computational modeling, providing a scalable solution for one of the most pressing challenges in NLP: developing language models that truly understand and adapt to the dynamic nature of human language.