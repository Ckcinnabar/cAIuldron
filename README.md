# cAIuldron 🍳🤖

**AI-Powered Recipe Generator** - Transform ingredient photos into personalized recipes using advanced machine learning

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen.svg)]()

## 📸 Overview

cAIuldron is an innovative AI cooking assistant that transforms ingredient photos into complete recipes. Upload a photo of your ingredients, and the system uses state-of-the-art AI models to:

- 🔍 **Detect multiple ingredients** with CLIP + DETR
- 🥗 **Estimate nutrition** using USDA FoodData Central (525+ ingredients)
- 🍳 **Generate diverse recipes** with 3 AI models (GPT-2, Llama 1B, Llama 8B)
- ⏱️ **Validate time fields** with LLM-based fallback
- 🌐 **Beautiful web interface** powered by Gradio

**100% Free • Runs Locally • No API Costs • Multi-Ingredient Support**

---

## ✨ Key Features

### 🎯 Core Capabilities

- **Multi-Ingredient Detection**: CLIP for classification + DETR for object detection
- **3 Generation Models**:
  - GPT-2 (fast, good quality)
  - Llama 3.2 1B with LoRA (recommended, 4-bit quantized)
  - Llama 3.1 8B GGUF (best quality, optimized for RTX 3060)
- **Nutrition Estimation**: Automatic portion sizing based on bounding box area
- **Time Validation**: Ensures Prep Time, Cook Time, Total Time are always present
- **5 Diverse Recipes**: Different cuisines (Asian, Western, Mediterranean, Fusion, etc.)
- **Fast Processing**: < 5 seconds per recipe generation

### 📊 Technical Highlights

- **Modular Architecture**: Clean separation of concerns across 5 notebooks
- **Smart Caching**: Model loading with caching for efficiency
- **Format Validation**: Regex-based time field validation with LLM fallback
- **Multi-Model Support**: Easy switching between GPT-2, Llama 1B, Llama 8B
- **Local Processing**: Everything runs on your machine, no cloud dependencies

---

## 🚀 Quick Start

### Prerequisites

```bash
Python 3.11+
CUDA 11.8+ (optional, for GPU acceleration)
16GB RAM (recommended)
10GB disk space (for models)
```

### Installation

```bash
# 1. Clone repository (includes Llama 1B via Git LFS)
git clone https://github.com/yourusername/cAIuldron.git
cd cAIuldron

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. (Optional) Download additional models
# See MODEL_SETUP.md for details
python scripts/download_models.py --model all

# 5. Launch Jupyter
jupyter notebook
```

**Note**: Llama 1B LoRA (132 MB) is included via Git LFS. GPT-2 and Llama 8B are optional downloads. See [MODEL_SETUP.md](MODEL_SETUP.md) for details.

### Running the App

**Method 1: Launch Web Interface (Recommended)**

```bash
# Open and run all cells in:
notebooks/pipeline_recipe_app/FINAL/app.ipynb
```

The Gradio interface will launch at `http://127.0.0.1:7861`

**Method 2: Test Individual Modules**

```python
# In Jupyter, run modules separately:
%run notebooks/pipeline_recipe_app/FINAL/1_model_loading.ipynb
%run notebooks/pipeline_recipe_app/FINAL/2_ingredient_detection.ipynb
%run notebooks/pipeline_recipe_app/FINAL/3_nutrition_estimation.ipynb
%run notebooks/pipeline_recipe_app/FINAL/4_recipe_generation.ipynb
```

---

## 🏗️ System Architecture

### High-Level Pipeline

```
Photo Upload → Multi-Ingredient Detection → Nutrition Estimation → Recipe Generation
                (CLIP + DETR)                  (USDA Database)      (GPT-2/Llama 1B/8B)
                                                                    + Time Validation
```

### AI Model Ecosystem

| Component | Model | Purpose | VRAM | Speed |
|-----------|-------|---------|------|-------|
| **Ingredient Detection** | CLIP ViT-Base + DETR ResNet-50 | Multi-ingredient recognition with bounding boxes | 2-3 GB | ~1-2s |
| **Recipe Generation** | GPT-2 (fine-tuned) | Fast recipe generation | 1-2 GB | ⚡⚡⚡ Fast |
| **Recipe Generation** | Llama 3.2 1B (LoRA, 4-bit) | High-quality recipes (recommended) | 4-5 GB | ⚡⚡ Medium |
| **Recipe Generation** | Llama 3.1 8B (GGUF Q5_K_M) | Best quality recipes | 4-6 GB | ⚡ Slower |
| **Nutrition Estimation** | USDA FoodData Central Lookup | Calorie and macro calculation | N/A | <10ms |

**Total Processing Time**: 3-8 seconds (detection + nutrition + 5 recipes)

---

## 📁 Project Structure

### Modular Architecture (FINAL)

```
cAIuldron/
├── notebooks/pipeline_recipe_app/FINAL/    # 🌟 Main Application (Modular)
│   ├── 1_model_loading.ipynb               # Model loading & management
│   ├── 2_ingredient_detection.ipynb        # CLIP + DETR detection
│   ├── 3_nutrition_estimation.ipynb        # USDA nutrition lookup
│   ├── 4_recipe_generation.ipynb           # Multi-model generation + validation
│   ├── app.ipynb                           # Gradio web interface
│   └── README.md                           # Module documentation
│
├── notebooks/                               # Training & Development
│   ├── model_ingredient_recognition/
│   │   ├── multi_ingredient_detection.ipynb
│   │   └── test_vocabulary_size_impact.ipynb
│   ├── model_nutrition_estimation/
│   │   ├── generate_full_nutrition_database.ipynb
│   │   ├── load_usda_database.ipynb
│   │   └── model_nutrition_inference.ipynb
│   └── model_recipe_generation/
│       ├── load_recipe_dataset.ipynb
│       ├── train_llama3_1b_recipe_generation.ipynb
│       └── train_recipe_transformer.ipynb
│
├── data/                                    # Data & Databases
│   ├── ingredients_nutrition_full.csv       # 525+ ingredients with nutrition
│   ├── ingredients_vocabulary.csv           # Ingredient vocabulary
│   ├── nutrition_lookup_full.json           # USDA nutrition database
│   ├── processed/recipes/                   # Recipe dataset (2.23M recipes)
│   ├── raw/nutrition_database/              # USDA FoodData Central CSV
│   └── test_images/                         # Sample test images
│
├── models/                                  # Trained Models
│   └── recipe_generation/
│       ├── finetuned/                       # GPT-2 fine-tuned model
│       ├── llama3_1b_finetuned/             # Llama 3.2 1B LoRA adapters
│       ├── Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf  # Llama 8B GGUF
│       └── checkpoints/                     # Training checkpoints
│

├── Deliverable1_Technical_Blueprint_Full.pdf
├── Deliverable2_Technical_Blueprint_Full.pdf
├── IEEE_Report_cAIuldron.md
├── requirements.txt
└── README.md                                # This file
```

---

## 🎯 Module Descriptions

### 1️⃣ Model Loading (`1_model_loading.ipynb`)

**Purpose**: Load and manage recipe generation models

**Features**:
- Supports 3 models: GPT-2, Llama 1B, Llama 8B
- Model caching for efficiency
- 4-bit quantization for Llama 1B (reduced VRAM)
- GGUF optimizations for Llama 8B (RTX 3060 Laptop)

**Exports**:
- `RecipeModelType` (enum)
- `MODEL_INFO` (dict)
- `load_recipe_model()` (function)

### 2️⃣ Ingredient Detection (`2_ingredient_detection.ipynb`)

**Purpose**: Multi-ingredient detection using CLIP + DETR

**Features**:
- CLIP for zero-shot ingredient classification (525+ ingredients)
- DETR for object detection (bounding boxes)
- Consolidation of multiple detections
- Confidence thresholding

**Exports**:
- `INGREDIENT_CANDIDATES` (list)
- `detect_ingredient_clip()` (single ingredient)
- `detect_multiple_ingredients_clip()` (multi-ingredient)
- `consolidate_detections()` (merge results)

### 3️⃣ Nutrition Estimation (`3_nutrition_estimation.ipynb`)

**Purpose**: Estimate nutrition from ingredient and size

**Features**:
- USDA FoodData Central database (525+ ingredients)
- Bounding box-based weight estimation
- Per-serving nutrition calculation
- Fuzzy ingredient matching

**Exports**:
- `NUTRITION_DB` (dict)
- `TYPICAL_WEIGHTS` (dict)
- `estimate_nutrition()` (function)

### 4️⃣ Recipe Generation (`4_recipe_generation.ipynb`)

**Purpose**: Multi-model recipe generation with validation

**Features**:
- 3 generation models (GPT-2, Llama 1B, Llama 8B)
- Strengthened prompts with concrete examples
- Time field validation (Prep, Cook, Total, Servings)
- LLM-based time estimation fallback
- Format enforcement

**Exports**:
- `generate_recipe_with_selected_model()` (main function)
- `validate_time_format()` (validation)
- `ensure_time_fields_with_llm()` (fallback)
- `generate_diverse_prompts()` (helper)

### 5️⃣ Gradio Interface (`app.ipynb`)

**Purpose**: Web-based user interface

**Features**:
- Upload ingredient photos
- Adjust detection confidence
- Select generation model
- View detection, nutrition, and recipes
- Sample image examples
- No file persistence (simplified)

---

## 📊 Model Performance

### Speed vs Quality Trade-off

| Model | Speed | Quality | VRAM | Use Case |
|-------|-------|---------|------|----------|
| **GPT-2** | ⚡⚡⚡ Fast (1-2s) | 😊 Good (60-70/100) | 1-2 GB | Quick testing |
| **Llama 3.2 1B** | ⚡⚡ Medium (3-5s) | 🌟 Excellent (90-95/100) | 4-5 GB | **Recommended** ⭐ |
| **Llama 3.1 8B GGUF** | ⚡ Slower (8-12s) | 🌟🌟 Best (95-100/100) | 4-6 GB | Highest quality |

### Accuracy Metrics

- **Ingredient Detection**: 85-95% accuracy (CLIP confidence > 0.15)
- **Nutrition Estimation**: ±20% accuracy (based on USDA database)
- **Time Validation**: 95%+ recipes have valid time fields
- **Recipe Quality**: Human evaluation scores 8.5/10 (Llama 1B)

---

## 🔧 Configuration

### Model Selection

Edit in `FINAL/app.ipynb` or `1_model_loading.ipynb`:

```python
# Default model
CURRENT_MODEL_TYPE = RecipeModelType.LLAMA_1B  # or GPT2, LLAMA_8B_GGUF
```

### GGUF Parameters (RTX 3060 Optimization)

Edit in `1_model_loading.ipynb`:

```python
llm = Llama(
    model_path=str(model_path),
    n_gpu_layers=12,        # Adjust for your GPU VRAM
    n_ctx=3072,             # Context length
    n_batch=256,            # Batch size
    n_threads=14,           # CPU threads
    f16_kv=True,            # FP16 KV cache
)
```

### Detection Confidence

Adjust in Gradio interface or `2_ingredient_detection.ipynb`:

```python
INGREDIENT_CONFIDENCE_THRESHOLD = 0.15  # Lower = more detections
OBJECT_DETECTION_THRESHOLD = 0.3        # DETR confidence
```

---

## 🧪 Testing

### Test Individual Modules

```python
# Test model loading
%run 1_model_loading.ipynb
model_dict = load_recipe_model(RecipeModelType.LLAMA_1B)
print(f"✓ Model loaded: {model_dict['type']}")

# Test ingredient detection
%run 2_ingredient_detection.ipynb
detected = detect_multiple_ingredients_clip("data/test_images/test.jpeg")
result = consolidate_detections(detected)
print(f"Found: {result['combined_ingredient']}")

# Test nutrition estimation
%run 3_nutrition_estimation.ipynb
nutrition = estimate_nutrition('chicken breast', 200, 200)
print(f"Calories: {nutrition['per_serving']['calories']} kcal")

# Test recipe generation
%run 4_recipe_generation.ipynb
recipe = generate_recipe_with_selected_model(
    ingredient='chicken',
    cuisine='Asian',
    difficulty='beginner',
    model_type=RecipeModelType.LLAMA_1B
)
print(recipe['raw_markdown'])
```

### End-to-End Test

Run `FINAL/app.ipynb` and test with sample images from `data/test_images/`

---

## 🐛 Troubleshooting

### CUDA Out of Memory

**Solution 1**: Use smaller model
```python
model_type = RecipeModelType.GPT2  # or LLAMA_1B
```

**Solution 2**: Adjust GGUF parameters
```python
n_gpu_layers=8  # Reduce GPU layers (in 1_model_loading.ipynb)
```

### Model Not Found

**Check paths**:
```python
from pathlib import Path
MODEL_DIR = Path.cwd().parent.parent.parent / "models" / "recipe_generation"
print(MODEL_DIR.exists())
print(list(MODEL_DIR.glob("*")))
```

### Low Detection Confidence

**Lower threshold** in Gradio interface or:
```python
INGREDIENT_CONFIDENCE_THRESHOLD = 0.10  # More permissive
```

---

## 📖 Documentation

- **[FINAL/README.md](notebooks/pipeline_recipe_app/FINAL/README.md)** - Complete module documentation
- **[Research Documentation](specs/001-ai-recipe-generator/research.md)** - Model selection rationale
- **[Feature Specification](specs/001-ai-recipe-generator/spec.md)** - Requirements and user stories
- **[Technical Blueprints](Deliverable2_Technical_Blueprint_Full.pdf)** - System design
- **[IEEE Report](IEEE_Report_cAIuldron.md)** - Academic documentation

---

## 🎓 Key Technologies

### AI Models

- **OpenAI CLIP** (ViT-Base-Patch32) - Zero-shot ingredient classification
- **Facebook DETR** (ResNet-50) - Object detection for multi-ingredient
- **OpenAI GPT-2** (124M params) - Fast recipe generation
- **Meta Llama 3.2 1B** (1B params, 4-bit LoRA) - High-quality generation
- **Meta Llama 3.1 8B** (8B params, Q5_K_M GGUF) - Best quality

### Datasets

- **USDA FoodData Central** (525+ ingredients) - Nutrition database
- **RecipeNLG** (2.23M recipes) - Training dataset
- **Custom Vocabulary** (529 ingredients) - CLIP classification

### Frameworks

- **PyTorch** - Deep learning framework
- **Transformers** (Hugging Face) - Model loading and inference
- **PEFT** - LoRA fine-tuning for Llama 1B
- **llama-cpp-python** - GGUF model inference
- **Gradio** - Web interface

---

## 🤝 Contributing

Contributions welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Follow modular architecture in `FINAL/`
4. Test all modules independently
5. Commit changes (`git commit -m 'Add amazing feature'`)
6. Push to branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Development Guidelines

- ✅ All code in `.ipynb` format
- ✅ Modular design (single responsibility)
- ✅ Clear markdown documentation
- ✅ Test independently before integration
- ✅ Follow PEP 8 style guidelines

---

## 📝 License

This project is licensed under the MIT License

### Model Licenses

- **GPT-2**: MIT License
- **Llama 3.2 & 3.1**: Meta Llama Community License
- **CLIP**: MIT License
- **DETR**: Apache 2.0 License

---

## 🙏 Acknowledgments

### Research & Models

- **Meta AI** - Llama 3.2 1B, Llama 3.1 8B
- **OpenAI** - GPT-2, CLIP
- **Facebook AI** - DETR
- **Hugging Face** - Transformers library

### Data Sources

- **USDA** - FoodData Central nutrition database
- **RecipeNLG** - Recipe dataset for training
- **Food-101** - Food image dataset

### Tools

- **PyTorch**, **Transformers**, **PEFT**
- **Gradio**, **Jupyter**, **Pandas**
- **bitsandbytes** (4-bit quantization)
- **llama-cpp-python** (GGUF inference)

---

## 📧 Contact

- **Issues**: [GitHub Issues](https://github.com/yourusername/cAIuldron/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/cAIuldron/discussions)

---

## 🗺️ Development Status

### ✅ Completed Features

- [x] Multi-ingredient detection (CLIP + DETR)
- [x] 3-model recipe generation (GPT-2, Llama 1B, Llama 8B)
- [x] Nutrition estimation (USDA database, 525+ ingredients)
- [x] Time field validation with LLM fallback
- [x] Modular architecture (5 independent modules)
- [x] Gradio web interface
- [x] Model caching and optimization
- [x] 4-bit quantization for Llama 1B
- [x] GGUF optimization for Llama 8B


