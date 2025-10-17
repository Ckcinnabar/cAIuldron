# cAIuldron 🍳🤖

**AI-Powered Recipe Generator** - Transform ingredient photos into personalized cooking plans using advanced machine learning

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Overview

cAIuldron is an innovative AI-driven cooking assistant that transforms ingredient photos into complete cooking plans. Simply upload a photo of your ingredient, and the system uses multiple AI models to generate diverse recipe suggestions, nutritional estimates, and step-by-step illustrated guides.

Turn a single photo into a full, personalized cooking plan—combining inspiration, precision, and confidence in the kitchen.

## ✨ Key Features

### 🔍 Smart Ingredient Recognition
- **Deep Learning CNN**: Uses EfficientNetV2 to identify 100+ common ingredients
- **Size Estimation**: Visual analysis-based weight and portion estimation
- **High Accuracy**: 90%+ confidence scores for reliable predictions

### 📝 Diverse Recipe Generation
- **AI Creativity**: Transformer models generate creative recipes
- **Cuisine Diversity**: Asian, Western, Mediterranean, Fusion, and more
- **At Least 5 Suggestions**: Minimum 5 different cooking options per ingredient
- **Difficulty Levels**: Beginner, intermediate, and advanced classifications

### 🥗 Nutritional Information
- **Calorie Calculation**: Accurate estimates based on USDA database
- **Portion Details**: Per-serving weight and nutritional breakdown
- **Confidence Intervals**: Uncertainty ranges for estimates

### 🎨 Step-by-Step Illustrated Guides
- **AI-Generated Art**: GAN creates clean, engaging line-art illustrations
- **Visual Steps**: Each cooking step includes illustrative guidance
- **Beginner-Friendly**: Clear, simple visuals for easy understanding

### 🎓 Interactive Beginner Guidance
- **Technique Explanations**: Real-time definitions for cooking terms
- **Timing Alerts**: Time management for critical steps
- **Encouragement**: Build confidence with supportive feedback

## 🏗️ System Architecture

```
Photo Upload → CNN Recognition → Recipe Generation → Nutrition Estimation → Illustration Creation
                                  (Transformer+RNN)    (Bayesian PGM)         (GAN)
```

### AI Model Ecosystem

| Component | Model | Purpose | Processing Time |
|-----------|-------|---------|-----------------|
| Ingredient Recognition | EfficientNetV2-S | Identify ingredient and estimate size | 50-100ms |
| Recipe Generation | GPT-2 Medium | Create diverse recipes | 2-3s |
| Cooking Refinement | BiLSTM | Adjust cooking time and step order | <50ms |
| Nutrition Estimation | Bayesian Network | Calculate calories and portions | <10ms |
| Illustration Generation | Pix2Pix GAN | Create line-art illustrations | 200-300ms |

**Total Processing Time**: 4-8 seconds (well under 15-second requirement)

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- JupyterLab 4.0+ or VS Code with Jupyter extension
- 16GB RAM (GPU recommended for faster GAN generation)
- 2GB disk space for model weights

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/cAIuldron.git
cd cAIuldron

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download pre-trained models (requires configuration)
# See docs/model_setup.md for model download links

# 5. Launch JupyterLab
jupyter lab
```

### Basic Usage

1. **Upload Ingredient Photo**
   - Open `notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb`
   - Upload a clear photo of your ingredient (recommended: good lighting, single primary ingredient)

2. **Generate Recipes**
   - Execute all notebook cells
   - System will automatically identify ingredient and generate 5+ recipes

3. **View Results**
   - Browse recipe suggestions across different cuisines
   - Check nutritional information and calorie estimates
   - Follow step-by-step illustrated guides for cooking

## 📁 Project Structure

```
cAIuldron/
├── notebooks/                    # Jupyter notebooks (core development environment)
│   ├── explore_ingredients/      # Ingredient data exploration
│   ├── model_ingredient_recognition/  # CNN model
│   ├── model_recipe_generation/       # Transformer model
│   ├── model_cooking_refinement/      # RNN model
│   ├── model_nutrition_estimation/    # Bayesian network
│   ├── model_illustration_generation/ # GAN model
│   ├── pipeline_recipe_app/           # End-to-end pipeline
│   └── utils_recipe/                  # Shared utilities
│
├── data/                         # Data directory
│   ├── raw/                      # Original datasets
│   ├── processed/                # Preprocessed data
│   └── results/                  # Model outputs
│
├── models/                       # Trained model weights
│   ├── cnn_ingredient_recognition.h5
│   ├── transformer_recipe_generation.pt
│   ├── rnn_cooking_refinement.pt
│   ├── pgm_nutrition_estimation.pkl
│   └── gan_illustration_generation.pt
│
├── tests/                        # Tests
│   ├── notebook_tests/           # Notebook execution tests
│   └── unit_tests/               # Unit tests
│
├── specs/                        # Feature specifications
│   └── 001-ai-recipe-generator/
│       ├── spec.md               # Feature specification
│       ├── plan.md               # Implementation plan
│       ├── research.md           # Research documentation
│       ├── data-model.md         # Data models
│       ├── quickstart.md         # Quickstart guide
│       └── contracts/            # API contracts
│
├── .specify/                     # Project governance
│   ├── memory/
│   │   └── constitution.md       # Project constitution
│   └── templates/                # Templates
│
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## 🔬 Technical Details

### Notebook-First Development Principles

This project follows a **Jupyter Notebook-first** development philosophy:

✅ **All code in .ipynb format**
✅ **Cell-level modularity**: Each cell has a single, well-defined purpose
✅ **Reproducibility**: Random seeds, version control, data checksums
✅ **Documentation-driven**: Markdown cells explain logic and decisions
✅ **Python best practices**: PEP 8, type hints, error handling

### Core Dependencies

```
torch==2.1.0              # PyTorch (CNN, RNN, GAN)
tensorflow==2.14.0        # TensorFlow (alternative)
transformers==4.35.0      # Hugging Face (GPT-2)
opencv-python==4.8.1      # Image processing
pillow==10.1.0            # Image loading
pandas==2.1.3             # Data manipulation
pgmpy==0.1.23             # Bayesian networks
jupyterlab==4.0.9         # Notebook environment
papermill==2.5.0          # Notebook testing
```

## 📊 Performance Metrics

### Processing Speed
- **Ingredient Recognition**: 50-100ms (CNN inference)
- **Recipe Generation**: 2-3 seconds (5 recipes in parallel)
- **Nutrition Estimation**: <10ms (Bayesian inference)
- **Illustration Generation**: 200-300ms per image
- **Total**: 4-8 seconds (end-to-end)

### Accuracy Targets
- **Ingredient Recognition**: 90% top-1 accuracy
- **Nutrition Estimation**: ±20% calorie accuracy
- **User Satisfaction**: 85% rate instructions as "clear and easy to follow"

### Resource Requirements
- **Memory**: 6GB peak (all models loaded)
- **Storage**: ~1.7GB (model weights)
- **GPU**: Optional (10x speedup for GAN)

## 🧪 Testing

### Run Tests

```bash
# Notebook execution tests
papermill notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb \
  output.ipynb \
  -p test_image "data/raw/ingredient_images/chicken_breast.jpg"

# Unit tests
pytest tests/unit_tests/

# End-to-end validation
python tests/notebook_tests/test_end_to_end_pipeline.py
```

### Validation Checklist

- ✅ All notebooks execute top-to-bottom without errors
- ✅ Ingredient recognition confidence ≥ 0.7 for 80% of common ingredients
- ✅ Generate 5 recipes with at least 3 different cuisines
- ✅ Calorie estimates within ±20% of expected values
- ✅ Illustration quality score ≥ 0.6
- ✅ Total processing time < 15 seconds

## 📖 Documentation

- **[Feature Specification](specs/001-ai-recipe-generator/spec.md)** - User requirements and acceptance criteria
- **[Implementation Plan](specs/001-ai-recipe-generator/plan.md)** - Technical architecture and project structure
- **[Research Documentation](specs/001-ai-recipe-generator/research.md)** - AI model selection and best practices
- **[Data Models](specs/001-ai-recipe-generator/data-model.md)** - Entity relationships and validation rules
- **[Quickstart Guide](specs/001-ai-recipe-generator/quickstart.md)** - Detailed implementation guide
- **[API Contracts](specs/001-ai-recipe-generator/contracts/)** - Input/output specifications for each model
- **[Project Constitution](.specify/memory/constitution.md)** - Development principles and quality standards

## 🤝 Contributing

We welcome all forms of contributions!

### Contribution Guidelines

1. Fork the project
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Follow the Notebook development principles in the project constitution
4. Ensure all tests pass
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Development Principles

- All code must be in `.ipynb` format
- Follow PEP 8 style guidelines
- Set random seeds for reproducibility
- Document design decisions in markdown cells
- Clear output cells before committing (unless essential for documentation)

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## 🙏 Acknowledgments

### Datasets
- **Food-101**: 101,000 food images
- **Recipe1M+**: 1M+ recipe dataset
- **USDA FoodData Central**: Nutritional database

### Pre-trained Models
- **EfficientNetV2**: Google Research
- **GPT-2**: OpenAI
- **Pix2Pix**: UC Berkeley

### Tools and Frameworks
- PyTorch, TensorFlow, Hugging Face Transformers
- Jupyter, Papermill, nbconvert
- OpenCV, Pillow, Pandas

## 📧 Contact

- **Project Link**: [https://github.com/yourusername/cAIuldron](https://github.com/yourusername/cAIuldron)
- **Issue Tracker**: [GitHub Issues](https://github.com/yourusername/cAIuldron/issues)

## 🗺️ Roadmap

### Coming Soon
- [ ] User interface (web/mobile app)
- [ ] More ingredient support (current 100+, target 500+)
- [ ] Multi-language recipe generation
- [ ] User rating and feedback system
- [ ] Offline mode (local model inference)

### Future Plans
- [ ] Video cooking guidance
- [ ] Community recipe sharing
- [ ] Personalized dietary recommendations
- [ ] Smart shopping list generation
- [ ] Integration with smart kitchen devices

---

**Made with ❤️ and 🤖 AI**

*Transform your ingredients into culinary inspiration!*
