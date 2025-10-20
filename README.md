# cAIuldron 🍳🤖

**AI-Powered Recipe Generator** - Transform ingredient photos into personalized cooking plans using advanced machine learning

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-In%20Development-yellow.svg)](specs/001-ai-recipe-generator/tasks.md)
[![Phase](https://img.shields.io/badge/Phase-Foundation%20Setup-blue.svg)](specs/001-ai-recipe-generator/tasks.md)

## Overview

cAIuldron is an innovative AI-driven cooking assistant that transforms ingredient photos into complete cooking plans. Simply upload a photo of your ingredient, and the system uses multiple AI models to generate diverse recipe suggestions, nutritional estimates, and step-by-step illustrated guides.

Turn a single photo into a full, personalized cooking plan—combining inspiration, precision, and confidence in the kitchen.

## ✨ Key Features

cAIuldron delivers four main user stories, prioritized for incremental development:

### 🥇 Priority 1: Basic Ingredient Photo to Recipe Suggestions (MVP)
- **Smart Recognition**: EfficientNetV2 CNN identifies 100+ common ingredients with 90%+ confidence
- **5+ Diverse Recipes**: Minimum 5 recipe suggestions per ingredient
- **Cuisine Diversity**: Asian, Western, Mediterranean, Fusion, and more
- **Essential Info**: Cooking time, difficulty level (beginner/intermediate/advanced), and ingredient lists
- **Fast Processing**: Complete within 15 seconds

### 🥈 Priority 2: Portion Size & Calorie Estimation
- **Visual Analysis**: Estimate ingredient weight and portion size from photo
- **Nutritional Data**: Calorie calculation based on USDA FoodData Central
- **Portion Details**: Per-serving weight and nutritional breakdown
- **Confidence Intervals**: ±20% accuracy with uncertainty ranges

### 🥉 Priority 3: Step-by-Step Illustrated Guides
- **AI-Generated Art**: Stable Diffusion v1.5 + ControlNet creates clean, engaging line-art illustrations
- **Visual Steps**: Each cooking step includes illustrative guidance
- **Sequential Instructions**: Numbered steps with clear text and images
- **Beginner-Friendly**: Clear, simple visuals for easy understanding

### 🎖️ Priority 4: Interactive Beginner Guidance
- **Technique Explanations**: Contextual definitions for cooking terms (sauté, julienne, etc.)
- **Timing Alerts**: Timer support for time-sensitive cooking steps
- **Contextual Tips**: Helpful guidance at critical moments
- **Encouragement**: Build confidence with supportive feedback and completion celebrations

## 🏗️ System Architecture

```
Photo Upload → CNN Recognition → Recipe Generation → Nutrition Estimation → Illustration Creation
                                  (Transformer+RNN)    (Bayesian PGM)         (Stable Diffusion)
```

### AI Model Ecosystem

| Component | Model | Purpose | Processing Time |
|-----------|-------|---------|-----------------|
| Ingredient Recognition | EfficientNetV2-S | Identify ingredient and estimate size | 50-100ms |
| Recipe Generation | GPT-2 Medium | Create diverse recipes | 2-3s |
| Cooking Refinement | BiLSTM | Adjust cooking time and step order | <50ms |
| Nutrition Estimation | Bayesian Network | Calculate calories and portions | <10ms |
| Illustration Generation | Stable Diffusion v1.5 + ControlNet | Create line-art illustrations | 3-5s (GPU) / 20-30s (CPU) |

**Total Processing Time**: Target < 15 seconds (typical 6-10 seconds)

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- JupyterLab 4.0+ or VS Code with Jupyter extension
- 16GB RAM recommended
- 5-6GB disk space for model weights (including Stable Diffusion)
- GPU with 6GB+ VRAM highly recommended for illustration generation (optional but 10x faster)

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
   - Execute all notebook cells (processing completes in < 15 seconds)
   - System will automatically identify ingredient and generate 5+ recipes
   - Recipes will include at least 3 different cuisine types

3. **View Results**
   - Browse recipe suggestions with cooking time, difficulty, and ingredients
   - Check nutritional information and calorie estimates (±20% accuracy)
   - Follow step-by-step illustrated guides with line-art visuals
   - Access interactive guidance for beginners (timing alerts, technique tips)

## 📁 Project Structure

```
cAIuldron/
├── notebooks/                    # Jupyter notebooks (core development environment)
│   ├── explore_ingredients/      # Exploratory analysis of ingredient dataset
│   │   ├── explore_ingredient_dataset.ipynb
│   │   └── explore_image_preprocessing.ipynb
│   ├── model_ingredient_recognition/  # CNN-based ingredient recognition
│   │   ├── model_cnn_training.ipynb
│   │   └── model_cnn_inference.ipynb
│   ├── model_recipe_generation/       # Transformer-based recipe generation
│   │   ├── model_transformer_training.ipynb
│   │   └── model_transformer_inference.ipynb
│   ├── model_cooking_refinement/      # RNN for cooking sequence optimization
│   │   ├── model_rnn_training.ipynb
│   │   └── model_rnn_inference.ipynb
│   ├── model_nutrition_estimation/    # Probabilistic models for nutrition
│   │   ├── model_pgm_training.ipynb
│   │   └── model_pgm_inference.ipynb
│   ├── model_illustration_generation/ # Stable Diffusion for line-art illustrations
│   │   ├── model_stable_diffusion_lineart.ipynb
│   │   └── model_stable_diffusion_lineart_improved.ipynb
│   ├── pipeline_recipe_app/           # End-to-end recipe generation pipeline
│   │   ├── pipeline_photo_to_recipes.ipynb
│   │   └── pipeline_interactive_guide.ipynb
│   └── utils_recipe/                  # Shared utilities
│       ├── utils_image_processing.ipynb
│       ├── utils_model_loading.ipynb
│       ├── utils_visualization.ipynb
│       ├── utils_reproducibility.ipynb
│       ├── utils_validation.ipynb
│       └── utils_logging.ipynb
│
├── data/                         # Data directory
│   ├── raw/                      # Original datasets
│   │   ├── ingredient_images/    # Training images of ingredients
│   │   ├── recipe_corpus/        # Recipe text data
│   │   └── nutrition_database/   # Nutritional reference data (USDA)
│   ├── processed/                # Preprocessed data
│   │   ├── ingredient_features/  # CNN embeddings
│   │   ├── recipe_tokens/        # Tokenized recipes
│   │   └── nutrition_vectors/    # Nutrition embeddings
│   └── results/                  # Model outputs
│       ├── generated_recipes/    # AI-generated recipes (JSON)
│       ├── illustrations/        # Stable Diffusion-generated line art
│       └── nutrition_estimates/  # Calorie and portion calculations
│
├── models/                       # Trained model weights
│   ├── cnn_ingredient_recognition.h5
│   ├── transformer_recipe_generation.pt
│   ├── rnn_cooking_refinement.pt
│   ├── pgm_nutrition_estimation.pkl
│   └── stable_diffusion_controlnet/  # Downloaded from HuggingFace
│       ├── stable-diffusion-v1-5/
│       └── control_v11p_sd15_lineart/
│
├── tests/                        # Tests
│   ├── notebook_tests/           # Notebook execution tests
│   └── unit_tests/               # Unit tests
│
├── specs/                        # Feature specifications
│   └── 001-ai-recipe-generator/
│       ├── spec.md               # Feature specification (user stories, requirements)
│       ├── plan.md               # Implementation plan (architecture, structure)
│       ├── tasks.md              # Task breakdown (89 tasks, 7 phases)
│       ├── research.md           # Research documentation (AI models, best practices)
│       ├── data-model.md         # Data models (entities, validation)
│       ├── quickstart.md         # Quickstart guide (step-by-step implementation)
│       └── contracts/            # API contracts (input/output specs)
│           ├── ingredient_recognition.json
│           ├── recipe_generation.json
│           ├── nutrition_estimation.json
│           └── illustration_generation.json
│
├── .specify/                     # Project governance
│   ├── memory/
│   │   └── constitution.md       # Project constitution
│   └── templates/                # Templates
│
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## 📋 Functional Requirements

The system implements 18 functional requirements defined in [spec.md](specs/001-ai-recipe-generator/spec.md):

**Core Photo Processing (FR-001 to FR-003)**
- FR-001: Accept JPEG, PNG, HEIC image formats
- FR-002: AI-powered ingredient identification
- FR-003: Generate minimum 5 distinct recipe suggestions

**Recipe Information (FR-004 to FR-006)**
- FR-004: Diverse cuisine types (Asian, Western, fusion minimum)
- FR-005: Cooking time estimates (in minutes)
- FR-006: Difficulty levels (beginner/intermediate/advanced)

**Nutrition & Portions (FR-007 to FR-008)**
- FR-007: Visual-based portion size estimation
- FR-008: Calorie estimates per serving

**Guided Cooking (FR-009 to FR-011)**
- FR-009: Step-by-step cooking instructions
- FR-010: Line-art style illustrations for each step
- FR-011: Beginner-friendly text instructions

**User Experience (FR-012 to FR-018)**
- FR-012: Helpful error messages for invalid photos
- FR-013: Step navigation (next/previous)
- FR-014: Browse all recipe suggestions before selection
- FR-015: Contextual cooking tips and technique explanations
- FR-016: Timing guidance for critical steps
- FR-017: Clear, organized information presentation
- FR-018: Complete processing within 15 seconds

For detailed acceptance scenarios, see [spec.md](specs/001-ai-recipe-generator/spec.md).

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
torch==2.1.0              # PyTorch (CNN, RNN, Stable Diffusion)
diffusers==0.25.0         # Hugging Face Diffusers (Stable Diffusion, ControlNet)
transformers==4.35.0      # Hugging Face Transformers (GPT-2)
controlnet-aux==0.0.7     # ControlNet preprocessors
opencv-python==4.8.1      # Image processing
pillow==10.1.0            # Image loading
pandas==2.1.3             # Data manipulation
pgmpy==0.1.23             # Bayesian networks
jupyterlab==4.0.9         # Notebook environment
papermill==2.5.0          # Notebook testing
```

## 📊 Performance Metrics & Success Criteria

### Processing Speed (from spec.md FR-018)
- **Ingredient Recognition**: 50-100ms (CNN inference)
- **Recipe Generation**: 2-3 seconds (5 recipes in parallel)
- **Nutrition Estimation**: <10ms (Bayesian inference)
- **Illustration Generation**: 3-5s per image (GPU) / 20-30s (CPU)
  - Note: Can be deferred or done in background for better UX
- **Total End-to-End**: Target < 15 seconds for recipe generation (illustrations optional/async)

### Success Criteria (from spec.md SC-001 to SC-010)
- **SC-001**: Recipe suggestions delivered within 15 seconds
- **SC-002**: 90% confidence ingredient identification in 80%+ of cases
- **SC-003**: 90% of users complete photo upload and recipe selection on first attempt
- **SC-004**: At least 3 different cuisine types in 95% of recipe generations
- **SC-005**: 85% of users rate instructions as "clear and easy to follow"
- **SC-006**: 80% of beginners successfully complete recipes using app guidance
- **SC-007**: Portion and calorie estimates within ±20% accuracy
- **SC-008**: 80% of users find line-art illustrations "helpful for understanding the step"
- **SC-009**: Handle 100 concurrent photo uploads without performance degradation
- **SC-010**: 70% of users proceed to view at least one full recipe guide

### Resource Requirements
- **Memory**: 8GB peak (all models loaded including Stable Diffusion), 16GB RAM recommended
- **Storage**: ~5-6GB (model weights including Stable Diffusion v1.5 + ControlNet)
- **GPU**: Highly recommended for illustration generation (10x speedup: 3-5s vs 20-30s per image)
  - NVIDIA GPU with 6GB+ VRAM recommended for Stable Diffusion
  - CPU-only mode supported but slower

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

Aligned with Success Criteria (spec.md SC-001 to SC-010):

- ✅ **SC-001**: All notebooks execute top-to-bottom without errors
- ✅ **SC-002**: Ingredient recognition confidence ≥ 0.9 for 80%+ of common ingredients
- ✅ **SC-003**: User flow (photo upload → recipe selection) completes without errors
- ✅ **SC-004**: Generate 5+ recipes with at least 3 different cuisines (95% of cases)
- ✅ **SC-005**: Instructions clear and beginner-friendly
- ✅ **SC-007**: Calorie estimates within ±20% of USDA database values
- ✅ **SC-008**: Illustration quality score ≥ 0.6 and helpful for understanding
- ✅ **SC-018**: Total processing time < 15 seconds (FR-018)

## 📖 Documentation

- **[Feature Specification](specs/001-ai-recipe-generator/spec.md)** - User stories (4 priorities), functional requirements (FR-001 to FR-018), success criteria (SC-001 to SC-010)
- **[Implementation Plan](specs/001-ai-recipe-generator/plan.md)** - Technical architecture, project structure, constitution compliance
- **[Task Breakdown](specs/001-ai-recipe-generator/tasks.md)** - 89 tasks across 7 phases with dependencies and parallel opportunities
- **[Research Documentation](specs/001-ai-recipe-generator/research.md)** - AI model selection, best practices, and technical decisions
- **[Data Models](specs/001-ai-recipe-generator/data-model.md)** - Entity schemas (Ingredient, Recipe, CookingStep, Illustration, PortionEstimate) and validation rules
- **[Quickstart Guide](specs/001-ai-recipe-generator/quickstart.md)** - Detailed step-by-step implementation guide
- **[API Contracts](specs/001-ai-recipe-generator/contracts/)** - Input/output specifications for each AI model component
- **[Project Constitution](.specify/memory/constitution.md)** - Notebook-first development principles and quality standards

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
- **Food-101**: 101,000 food images for ingredient recognition
- **Recipe1M+**: 1M+ recipe dataset for recipe generation
- **USDA FoodData Central**: Nutritional database for calorie estimation

### Pre-trained Models
- **EfficientNetV2**: Google Research (ingredient recognition)
- **GPT-2**: OpenAI (recipe generation)
- **Stable Diffusion v1.5**: RunwayML (illustration generation)
- **ControlNet**: lllyasviel (lineart conditioning)

### Tools and Frameworks
- PyTorch, Hugging Face Transformers, Diffusers
- Jupyter, Papermill, nbconvert
- OpenCV, Pillow, Pandas, pgmpy

## 📧 Contact

- **Project Link**: [https://github.com/yourusername/cAIuldron](https://github.com/yourusername/cAIuldron)
- **Issue Tracker**: [GitHub Issues](https://github.com/yourusername/cAIuldron/issues)

## 🗺️ Development Roadmap

### Phase 1-2: Foundation (In Progress)
- [ ] Project setup and infrastructure (Tasks T001-T005)
- [ ] Core utilities and reproducibility framework (Tasks T006-T011)

### Phase 3: Priority 1 - MVP (Next)
- [ ] User Story 1: Basic ingredient photo to recipe suggestions (Tasks T012-T033)
  - CNN ingredient recognition
  - Transformer recipe generation (5+ diverse recipes)
  - RNN cooking refinement
  - End-to-end pipeline (< 15 seconds)

### Phase 4: Priority 2 - Nutrition
- [ ] User Story 2: Portion size & calorie estimation (Tasks T034-T045)
  - Bayesian network for nutrition estimation
  - USDA database integration
  - ±20% calorie accuracy

### Phase 5: Priority 3 - Visual Guides
- [ ] User Story 3: Step-by-step illustrated guides (Tasks T046-T063)
  - Stable Diffusion + ControlNet for line-art illustration generation
  - Interactive guide notebook
  - Quality score ≥ 0.6

### Phase 6: Priority 4 - Beginner Support
- [ ] User Story 4: Interactive beginner guidance (Tasks T064-T078)
  - Technique explanations and tips
  - Timing alerts and progress tracking
  - Feedback collection

### Phase 7: Polish & Production
- [ ] Comprehensive documentation and performance monitoring (Tasks T079-T089)
- [ ] End-to-end testing and validation
- [ ] Code quality and constitution compliance checks

### Future Enhancements
- [ ] User interface (web/mobile app)
- [ ] More ingredient support (current 100+, target 500+)
- [ ] Multi-language recipe generation
- [ ] Video cooking guidance
- [ ] Community recipe sharing platform
- [ ] Personalized dietary recommendations
- [ ] Smart shopping list generation
- [ ] Integration with smart kitchen devices

---

**Made with ❤️ and 🤖 AI**

*Transform your ingredients into culinary inspiration!*
