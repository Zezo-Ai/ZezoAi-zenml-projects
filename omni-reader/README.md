# OmniReader - Multi-model text extraction comparison

OmniReader is a document processing workflow that ingests unstructured documents (PDFs, images, scans) and extracts text using multiple OCR models. It provides side-by-side comparison of extraction results, highlighting differences in accuracy, formatting, and content recognition. The multi-model approach allows users to evaluate OCR performance across different document types, languages, and formatting complexity. OmniReader delivers reproducible, automated, and cloud-agnostic analysis, with comprehensive metrics on extraction quality, processing time, and confidence scores for each model. It also supports parallel processing for faster batch operations and can compare an arbitrary number of models simultaneously.

## 🚀 Getting Started

### Prerequisites

- Python 3.9+
- Mistral API key (set as environment variable `MISTRAL_API_KEY`)
- OpenAI API key (set as environment variable `OPENAI_API_KEY`)
- ZenML >= 0.80.0

### Installation

```bash
# Install dependencies
pip install -r requirements.txt
```

### Configuration

1. Ensure any Ollama models you want to use are pulled, e.g.:

```bash
ollama pull llama3.2-vision:11b
ollama pull gemma3:12b
```

2. Set the following environment variables:

```bash
OPENAI_API_KEY=your_openai_api_key
MISTRAL_API_KEY=your_mistral_api_key
```

## 📌 Usage

### Using YAML Configuration (Recommended)

```bash
# Use the default config (ocr_config.yaml)
python run.py

# Run with a custom config file
python run.py --config my_config.yaml
```

### Configuration Structure

YAML configuration files organize parameters into logical sections:

```yaml
# Image input configuration
input:
  image_paths: [] # List of specific image paths
  image_folder: "./assets" # Folder containing images

# OCR model configuration
models:
  custom_prompt: null # Optional custom prompt for all models
  # Either specify individual models (for backward compatibility)
  model1: "llama3.2-vision:11b" # First model for comparison
  model2: "mistral/pixtral-12b-2409" # Second model for comparison
  # Or specify multiple models as a list (new approach)
  models: ["llama3.2-vision:11b", "mistral/pixtral-12b-2409"]
  ground_truth_model: "gpt-4o-mini" # Model to use for ground truth when source is "openai"

# Ground truth configuration
ground_truth:
  source: "openai" # Source: "openai", "manual", "file", or "none"
  texts: [] # Ground truth texts (for manual source)
  file: null # Path to ground truth JSON file (for file source)

# Output configuration
output:
  ground_truth:
    save: false # Whether to save ground truth data
    directory: "ocr_results" # Directory for ground truth data

  ocr_results:
    save: false # Whether to save OCR results
    directory: "ocr_results" # Directory for OCR results

  visualization:
    save: false # Whether to save HTML visualization
    directory: "visualizations" # Directory for visualization
```

### Running with Command Line Arguments

You can still use command line arguments for quick runs:

```bash
# Run with default settings (processes all images in assets directory)
python run.py --image-folder assets

# Run with custom prompt
python run.py --image-folder assets --custom-prompt "Extract all text from this image."

# Run with specific images
python run.py --image-paths assets/image1.jpg assets/image2.png

# Run with OpenAI ground truth for evaluation
python run.py --image-folder assets --ground-truth openai --save-ground-truth

# List available ground truth files
python run.py --list-ground-truth-files

# For quicker processing of a single image without metadata or artifact tracking
python run_compare_ocr.py --image assets/your_image.jpg --model both

# Run comparison with multiple specific models in parallel
python run_compare_ocr.py --image assets/your_image.jpg --model "gemma3:12b,llama3.2-vision:11b,moondream"

# Run comparison with all available models in parallel
python run_compare_ocr.py --image assets/your_image.jpg --model all
```

### Using the Streamlit App

For interactive use, the project includes a Streamlit app:

```bash
streamlit run app.py
```

<div align="center">
  <img src="assets/streamlit.png" alt="Streamlit UI Interface" width="800"/>
  <p><em>Interactive Streamlit interface for easy document processing and model comparison</em></p>
</div>

### ### Remote Artifact Storage and Execution

This project supports storing artifacts remotely and executing pipelines on cloud infrastructure. Follow these steps to configure your environment for remote operation:

#### Setting Up Cloud Provider Integrations

Install the appropriate ZenML integrations for your preferred cloud provider:

```bash
# For AWS
zenml integration install aws s3 -y

# For Azure
zenml integration install azure azure-blob -y

# For Google Cloud
zenml integration install gcp gcs -y
```

For detailed configuration options and other components, refer to the ZenML documentation:

- [Artifact Stores](https://docs.zenml.io/stacks/artifact-stores)
- [Orchestrators](https://docs.zenml.io/stacks/orchestrators)

## 📋 Pipeline Architecture

The OCR comparison pipeline consists of the following components:

### Steps

1. **Multi-Model OCR Step**: Processes images with multiple models in parallel
   - Supports any number of models defined in configuration
   - Models run in parallel using ThreadPoolExecutor
   - Each model processes its assigned images with parallelized execution
   - Progress tracking during batch processing
2. **Ground Truth Step**: Optional step that uses a reference model for evaluation (default: GPT-4o Mini)
3. **Evaluation Step**: Compares results and calculates metrics

The pipeline supports configurable models, allowing you to easily swap out the models used for OCR comparison and ground truth generation via the YAML configuration file. It also supports processing any number of models in parallel for more comprehensive comparisons.

### Configuration Management

The new configuration system provides:

- Structured YAML files for experiment parameters
- Parameter validation and intelligent defaults
- Easy sharing and version control of experiment settings
- Configuration generator for quickly creating new experiment setups
- Support for multi-model configuration via arrays
- Flexible model selection and comparison

### Metadata Tracking

ZenML's metadata tracking is used throughout the pipeline:

- Processing times and performance metrics
- Extracted text length and entity counts
- Comparison metrics between models (CER, WER)
- Progress tracking for batch operations
- Parallel processing statistics

### Results Visualization

- Pipeline results are available in the ZenML Dashboard
- HTML visualizations can be automatically saved to configurable directories

<div align="center">
  <img src="assets/html_visualization.png" alt="HTML Visualization of OCR Results" width="800"/>
  <p><em>HTML visualization showing metrics and comparison results from the OCR pipeline</em></p>
</div>

## 📁 Project Organization

```
omni-reader/
│
├── configs/               # YAML configuration files
├── pipelines/             # ZenML pipeline definitions
├── steps/                 # Pipeline step implementations
├── utils/                 # Utility functions
│   ├── config.py          # Configuration utilities
│   └── ...
├── run.py                 # Main script for running the pipeline
├── config_generator.py    # Tool for generating experiment configurations
└── README.md              # Project documentation
```

## 🔗 Links

- [ZenML Documentation](https://docs.zenml.io/)
- [Mistral AI Vision Documentation](https://docs.mistral.ai/capabilities/vision/)
- [Gemma 3 Documentation](https://ai.google.dev/gemma/docs/integrations/ollama)
