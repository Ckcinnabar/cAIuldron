# Implementation Plan: AI Recipe Generator

**Branch**: `001-ai-recipe-generator` | **Date**: 2025-10-16 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/001-ai-recipe-generator/spec.md`

## Summary

Build an AI-powered recipe generator application that transforms ingredient photos into personalized cooking plans. Users upload a photo of a raw ingredient (e.g., chicken breast, salmon), and the system uses computer vision to identify and measure the ingredient, then generates 5-7 diverse recipe suggestions with nutritional information and step-by-step illustrated cooking guides.

**Primary Requirement**: Photo → 5-7 recipes with cuisine diversity, portion/calorie estimates, cooking metadata, and line-art illustrations

**Technical Approach**:
- Vision Transformer (ViT) for ingredient recognition and size estimation
- Fine-tuned T5 transformer for recipe text generation with safety validation
- ControlNet + Stable Diffusion for line-art illustration generation
- USDA FoodData Central API for nutritional calculations
- All models implemented in Python using PyTorch, developed in Jupyter notebooks
- FastAPI service for model serving

## Technical Context

**Language/Version**: Python 3.11 (constitutional requirement: Python 3.8+)
**Primary Dependencies**:
- PyTorch 2.1+ (deep learning framework for all AI models)
- HuggingFace Transformers 4.35+ (ViT for CV, T5 for recipe generation)
- Diffusers 0.24+ (Stable Diffusion + ControlNet for illustrations)
- FastAPI 0.104+ (API serving)
- Jupyter Lab 4.0+ (development environment per constitution)

**Storage**:
- Local filesystem for model weights (~10GB total: ViT 300MB, T5 220MB, SD+ControlNet 3.4GB)
- USDA FoodData Central API (cloud, free tier: 1000 requests/hour)
- Optional: PostgreSQL for user preferences and saved recipes (P4 feature)

**Testing**:
- pytest 7.4+ for Python module testing
- nbval 0.10+ for Jupyter notebook validation (constitutional requirement)
- Custom integration tests for end-to-end pipeline latency validation

**Target Platform**:
- Development: Local/cloud Jupyter notebooks with GPU (CUDA 11.8+)
- Production: Linux server with NVIDIA GPU (11GB+ VRAM recommended)
- API deployment: Docker containers with FastAPI

**Project Type**: Single project with library-first architecture (per constitution)

**Performance Goals**:
- Ingredient recognition: 90%+ accuracy on top 50 common ingredients
- Total latency: <10 seconds from upload to recipe display
- Recipe generation: 5-7 diverse recipes per ingredient
- Nutrition accuracy: ±20% vs USDA standards

**Constraints**:
- Python & Jupyter only (constitutional requirement—all AI code must be in .ipynb or .py)
- CLI-first interface for all libraries (constitutional requirement)
- All models must be locally hostable (no mandatory cloud AI dependencies)
- Safety validation layer required (food safety: min cooking temps/times)

**Scale/Scope**:
- MVP: Support 50-100 common ingredients
- Initial launch: 10K-100K users
- Recipe database: AI-generated on-demand (not pre-computed)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Verify compliance with `.specify/memory/constitution.md`:

- [x] **Principle I: Python & Jupyter Stack** - All components use Python 3.11 with PyTorch/HuggingFace libraries; model development in Jupyter notebooks; no alternative languages
- [x] **Principle II: Library-First Architecture** - Feature designed as standalone libraries: ingredient_recognition, recipe_generation, illustration_generation, nutrition_calculator—each independently testable
- [x] **Principle III: CLI-First Interface** - Each library will expose CLI with JSON/text output (e.g., `recipe-gen --ingredient="chicken" --weight=300 --output=json`)
- [x] **Principle IV: Test-First Development** - TDD approach planned: contract tests for model I/O schemas, integration tests for pipeline latency, unit tests for preprocessing/safety validation
- [x] **Principle V: Simplicity and Minimalism** - Using proven architectures (ViT, T5, ControlNet) over custom research; avoiding PGMs for nutrition (simple lookup sufficient); monolithic deployment for MVP

**Violations**: None—all constitutional principles satisfied.

**Justification**: The AI/ML nature of this feature inherently requires complex models (ViT, T5, diffusion), but we've chosen established, well-documented architectures over custom solutions. Complexity is justified by user requirements (ingredient recognition, recipe generation, illustration generation) and documented in research phase.

## Project Structure

### Documentation (this feature)

```
specs/001-ai-recipe-generator/
├── plan.md                # This file
├── spec.md                # Feature specification
├── research.md            # Phase 0 research output
├── data-model.md          # Phase 1 data entities
├── quickstart.md          # Phase 1 development guide
├── contracts/             # Phase 1 API contracts
│   └── api-schema.yaml    # OpenAPI 3.0 specification
└── checklists/
    └── requirements.md    # Spec quality validation
```

### Source Code (repository root)

```
src/
├── ingredient_recognition/  # Library: CV for ingredient ID + size estimation
│   ├── __init__.py
│   ├── model.py            # ViT model wrapper
│   ├── preprocessing.py     # Image preprocessing
│   ├── cli.py              # CLI interface (constitutional requirement)
│   └── size_estimator.py   # Weight/size estimation from image
│
├── recipe_generation/      # Library: T5-based recipe text generation
│   ├── __init__.py
│   ├── model.py            # T5 model wrapper
│   ├── prompts.py          # Prompt templates for T5
│   ├── safety.py           # Food safety validation rules
│   ├── cli.py              # CLI interface
│   └── cooking_time_rnn.py # RNN for dynamic time adjustment
│
├── illustration_generation/  # Library: ControlNet+SD line-art generation
│   ├── __init__.py
│   ├── model.py            # Diffusion model wrapper
│   ├── prompt_engineering.py  # Line-art prompt templates
│   ├── cli.py              # CLI interface
│   └── style_consistency.py   # LoRA fine-tuning utilities
│
├── nutrition_calculator/   # Library: USDA-based nutrition lookup
│   ├── __init__.py
│   ├── usda_client.py      # USDA API client
│   ├── calculator.py       # Nutrition math
│   ├── cli.py              # CLI interface
│   └── portion_estimator.py  # Serving size calculations
│
└── api/                    # FastAPI service (orchestrates libraries)
    ├── __init__.py
    ├── main.py             # FastAPI app
    ├── routes.py           # API endpoints
    ├── schemas.py          # Pydantic models
    └── pipeline.py         # End-to-end orchestration

notebooks/                  # Jupyter development (constitutional requirement)
├── 01-data-exploration/
│   ├── 01-food101-eda.ipynb
│   └── 02-recipe-datasets.ipynb
├── 02-model-training/
│   ├── 01-ingredient-recognition-vit.ipynb
│   ├── 02-recipe-generation-t5.ipynb
│   ├── 03-cooking-time-rnn.ipynb
│   └── 04-illustration-controlnet.ipynb
├── 03-evaluation/
│   ├── 01-accuracy-validation.ipynb
│   └── 02-latency-benchmarks.ipynb
└── 04-deployment/
    └── 01-model-export.ipynb

tests/
├── contract/              # Model I/O schema tests
│   ├── test_ingredient_model.py
│   ├── test_recipe_model.py
│   └── test_illustration_model.py
├── integration/           # End-to-end pipeline tests
│   ├── test_full_pipeline.py
│   └── test_latency_requirements.py
├── unit/                  # Component unit tests
│   ├── test_preprocessing.py
│   ├── test_safety_validation.py
│   └── test_nutrition_calculator.py
└── notebooks/             # Notebook validation (nbval)
    └── test_notebooks.py
```

**Structure Decision**: Single project with library-first architecture (per constitution Principle II). Each AI component (ingredient recognition, recipe generation, illustration generation, nutrition) is a standalone library with CLI interface, orchestrated by FastAPI service. Jupyter notebooks for model development/training, Python modules for production code.

## Complexity Tracking

*Fill ONLY if Constitution Check has violations that must be justified*

No violations—constitution check passed. However, documenting inherent complexity:

| Complexity | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Multiple AI models (ViT, T5, ControlNet) | User requirements demand ingredient recognition, recipe generation, AND illustrations—each requires different model architecture | Single model cannot handle computer vision + text generation + image generation; attempted multi-modal models (GPT-4V) rejected due to cost ($400K/month) and latency (5s per API call) |
| PyTorch + Transformers + Diffusers libraries | Industry-standard implementations of ViT, T5, Stable Diffusion; proven and well-documented | Custom implementations would take 6-12 months and unlikely to match quality; using established libraries reduces risk |
| GPU requirement | AI models require GPU for acceptable latency (10s target) | CPU inference: 60-180s (6-18x slower), violates latency requirement |
| Fine-tuning required | Pre-trained models insufficient for food domain specificity (Food-101: 85% accuracy vs 90% required) | Generic models don't meet accuracy targets; fine-tuning on food datasets necessary for 90%+ accuracy |

## Phase 0: Research & Decision Documentation

**Objective**: Resolve all technical unknowns and document technology choices.

**Research Areas**:
1. ML framework selection (PyTorch vs TensorFlow)
2. Computer vision architecture for ingredient recognition
3. Text generation approach for recipes
4. Image generation approach for line-art illustrations
5. Nutritional estimation methodology
6. Model serving architecture
7. Development workflow in Jupyter notebooks

**Status**: ✅ COMPLETED (see research.md for full details)

**Key Decisions**:
- **ML Framework**: PyTorch primary (best Jupyter integration, HuggingFace ecosystem)
- **Ingredient Recognition**: Vision Transformer (ViT) fine-tuned on Food-101 dataset
- **Recipe Generation**: T5-base fine-tuned on Recipe1M+ dataset
- **Illustration Generation**: ControlNet + Stable Diffusion 1.5 with line-art control
- **Nutrition**: USDA FoodData Central API with simple lookup (no ML needed)
- **Architecture**: Monolithic FastAPI service for MVP, microservices for scale
- **Development**: Jupyter notebooks + MLflow + pytest

**Artifacts**:
- `research.md`: Comprehensive technical research (see agent output above)
- Documents alternatives considered and rationale for each decision

## Phase 1: Design & Data Model

### Data Model

See `data-model.md` for complete entity definitions. Key entities:

**IngredientRecognitionResult**:
- ingredient_id (USDA FDC ID)
- ingredient_name (common name, e.g., "chicken breast")
- confidence_score (0.0-1.0)
- estimated_weight_grams
- bounding_box (x, y, width, height)
- quality_assessment (clear/unclear/too_far/multiple_items)

**Recipe**:
- recipe_id (UUID)
- source_ingredient_id
- title
- cuisine_type (asian/western/fusion/mediterranean/etc.)
- difficulty (beginner/intermediate/advanced)
- total_cooking_time_minutes
- prep_time_minutes
- servings
- ingredients[] (list of {name, quantity, unit})
- cooking_steps[] (list of CookingStep)
- nutritional_info (NutritionalInformation)
- safety_validated (boolean, TRUE if passes safety checks)

**CookingStep**:
- step_number (1-indexed)
- instruction_text
- estimated_time_minutes
- illustration_url (path to generated line-art image)
- cooking_techniques[] (e.g., ["dicing", "sautéing"])
- temperature_fahrenheit (optional)

**NutritionalInformation**:
- total_calories
- calories_per_serving
- protein_grams
- carbohydrates_grams
- fat_grams
- servings
- portion_size_grams

**UserPreference** (P4 feature, optional for MVP):
- user_id
- dietary_restrictions[] (vegetarian/vegan/gluten-free/dairy-free/etc.)
- equipment_available[] (stovetop/oven/microwave/air_fryer/etc.)
- max_cooking_time_minutes
- saved_recipe_ids[]

### API Contracts

See `contracts/api-schema.yaml` for full OpenAPI 3.0 specification.

**Key Endpoints**:

```
POST /api/v1/recipes/generate
Content-Type: multipart/form-data
Request:
  - image: File (JPEG/PNG/HEIC, max 10MB)
  - preferences: JSON (optional: {cuisine_filter[], max_time, dietary_restrictions[]})

Response (200 OK):
{
  "request_id": "uuid",
  "ingredient": {
    "name": "chicken breast",
    "confidence": 0.94,
    "estimated_weight_grams": 285,
    "usda_id": "171077"
  },
  "recipes": [
    {
      "recipe_id": "uuid",
      "title": "Teriyaki Chicken Stir-Fry",
      "cuisine_type": "asian",
      "difficulty": "beginner",
      "total_time_minutes": 25,
      "prep_time_minutes": 10,
      "cook_time_minutes": 15,
      "servings": 2,
      "ingredients": [
        {"name": "chicken breast", "quantity": "285", "unit": "g"},
        {"name": "soy sauce", "quantity": "2", "unit": "tbsp"},
        ...
      ],
      "steps": [
        {
          "step_number": 1,
          "instruction": "Cut chicken breast into bite-sized pieces",
          "time_minutes": 3,
          "illustration_url": "/illustrations/abc123-step1.png",
          "techniques": ["dicing"]
        },
        ...
      ],
      "nutrition": {
        "total_calories": 680,
        "calories_per_serving": 340,
        "protein_grams": 48,
        "carbs_grams": 32,
        "fat_grams": 12,
        "servings": 2
      },
      "safety_validated": true
    },
    ... // 4-6 more recipes
  ],
  "generation_time_seconds": 9.2
}

Error Response (400 Bad Request):
{
  "error": "ingredient_not_recognized",
  "message": "We couldn't identify a food ingredient in this photo. Please upload a clearer photo of a single ingredient.",
  "suggestions": [
    "Ensure good lighting",
    "Use a plain background",
    "Photograph only one ingredient"
  ]
}
```

**CLI Interfaces** (Constitutional Requirement - Principle III):

```bash
# Ingredient Recognition CLI
$ ingredient-recognize --image chicken.jpg --output json
{
  "ingredient": "chicken breast",
  "confidence": 0.94,
  "weight_grams": 285,
  "usda_id": "171077"
}

# Recipe Generation CLI
$ recipe-generate --ingredient "chicken breast" --weight 285 --cuisine asian --count 5 --output json
[
  {"title": "Teriyaki Chicken", "cuisine": "asian", "time": 25, ...},
  ...
]

# Illustration Generation CLI
$ illustration-generate --prompt "dice chicken breast into cubes" --style lineart --output step1.png
Generated: step1.png (line-art style, 512x512)

# Nutrition Calculator CLI
$ nutrition-calc --ingredient "chicken breast" --weight 285 --output text
Chicken Breast (285g):
  Calories: 471 kcal
  Protein: 88g
  Carbs: 0g
  Fat: 10g
```

### Quickstart Guide

See `quickstart.md` for full setup instructions. Overview:

**Prerequisites**:
- Python 3.11+
- NVIDIA GPU with 11GB+ VRAM (recommended: RTX 3080, RTX 4080, or cloud V100/A100)
- CUDA 11.8+
- 50GB disk space (models + datasets)

**Setup**:
```bash
# 1. Clone repository
git clone <repo-url>
cd cAIuldron

# 2. Create virtual environment
python3.11 -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download pre-trained models (first-time setup)
python scripts/download_models.py

# 5. Set up USDA API key (free)
export USDA_API_KEY="your-key-here"

# 6. Start Jupyter Lab (for development)
jupyter lab

# 7. Run API server (for testing)
python src/api/main.py
# Access at http://localhost:8000
```

**Development Workflow**:
1. Open Jupyter Lab
2. Navigate to `notebooks/02-model-training/`
3. Run training notebooks to fine-tune models
4. Export trained models to `models/` directory
5. Test via CLI interfaces
6. Run integration tests: `pytest tests/integration/`
7. Start API server for end-to-end testing

**Testing**:
```bash
# Run all tests
pytest

# Run specific test categories
pytest tests/unit/                # Unit tests (fast)
pytest tests/contract/            # Model I/O schema tests
pytest tests/integration/         # End-to-end pipeline (slow, requires GPU)

# Validate notebooks (constitutional requirement)
pytest --nbval notebooks/
```

## Phase 2: Task Generation

**Note**: Task generation happens via `/speckit.tasks` command (NOT part of this plan command).

The tasks.md file will break down implementation into dependency-ordered tasks organized by user story:
- P1 (MVP): Photo upload + ingredient recognition + basic recipe generation
- P2: Portion/nutrition analysis
- P3: Step-by-step illustrated cooking guide
- P4: Recipe customization/preferences

Each task will specify:
- Exact file paths
- Test requirements (TDD: tests written first)
- Parallel execution opportunities ([P] marker)
- Dependencies between tasks

## Implementation Notes

### Constitutional Compliance

**Principle I (Python & Jupyter)**: ✅
- All code in Python 3.11
- Model development in Jupyter notebooks (`notebooks/02-model-training/*.ipynb`)
- Production code extracted to Python modules (`src/**/*.py`)
- No alternative languages used

**Principle II (Library-First)**: ✅
- Four standalone libraries: ingredient_recognition, recipe_generation, illustration_generation, nutrition_calculator
- Each library independently testable
- Clear boundaries and purposes
- Reusable across different contexts (CLI, API, notebooks)

**Principle III (CLI-First)**: ✅
- Every library includes `cli.py` module
- CLI accepts JSON or text input/output
- Composable via Unix pipes (e.g., `ingredient-recognize --image x.jpg | recipe-generate --stdin`)
- Built-in help with `--help` flag

**Principle IV (Test-First)**: ✅
- TDD approach documented in tasks.md
- Three test categories: contract (model schemas), integration (pipeline), unit (components)
- Coverage gate: all code must have tests
- Notebook validation via nbval

**Principle V (Simplicity)**: ✅
- Using established architectures (ViT, T5, ControlNet) over custom research
- Rejecting unnecessary complexity (PGMs for nutrition → simple lookup)
- Monolithic deployment for MVP (microservices only at scale)
- Complexity justified in "Complexity Tracking" section above

### Technology Stack Summary

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Language | Python | 3.11 | Core language (constitutional) |
| ML Framework | PyTorch | 2.1+ | Deep learning models |
| Computer Vision | ViT (HuggingFace) | transformers 4.35+ | Ingredient recognition |
| Text Generation | T5 | transformers 4.35+ | Recipe generation |
| Image Generation | Stable Diffusion + ControlNet | diffusers 0.24+ | Line-art illustrations |
| API Framework | FastAPI | 0.104+ | HTTP API serving |
| Notebook Environment | Jupyter Lab | 4.0+ | Development (constitutional) |
| Testing | pytest + nbval | 7.4+ / 0.10+ | Test framework |
| Experiment Tracking | MLflow | 2.8+ | Model versioning |
| Nutrition Data | USDA FoodData Central API | v2 | Nutritional information |

### Key Risk Mitigations

**Risk 1: Total latency exceeds 10-second target**
- Mitigation: Batch illustration generation (5 images in 6s vs 10s sequential); reduce diffusion inference steps (20 → 15); optionally stream illustrations asynchronously (user gets recipes immediately, illustrations load progressively)
- Fallback: Use faster Pix2Pix GAN for illustrations (0.5s per image)

**Risk 2: Ingredient recognition accuracy < 90%**
- Mitigation: Fine-tune ViT on custom dataset of real user photos; implement confidence thresholds (show top-3 for 60-80% confidence); collect user corrections for continuous improvement
- Fallback: Manual ingredient selection UI when confidence < 60%

**Risk 3: Recipe quality inconsistent (low ratings)**
- Mitigation: Human-in-the-loop validation during training; GPT-3.5/4 fallback for unusual ingredients; active learning from user ratings; safety validation layer for food safety
- Monitoring: Track user ratings per recipe, retrain monthly with best-rated examples

**Risk 4: GPU resource constraints at scale**
- Mitigation: Start with monolithic service (1 GPU handles 100K users); scale to microservices with dedicated GPUs per component; implement caching for common ingredients/recipes
- Cost: Self-hosted GPUs ($1500-2000/month for 3 servers) vs cloud APIs ($5K-10K/month)

### Development Timeline (Estimated)

**Phase 0: Research** (COMPLETED) - 3 days
- Framework selection, architecture decisions, dataset identification

**Phase 1: Data Preparation** - 1 week
- Download Food-101, Recipe1M+ datasets
- Prepare training/validation splits
- Set up MLflow experiment tracking

**Phase 2: Model Training** - 3-4 weeks
- Week 1: Fine-tune ViT on Food-101 (ingredient recognition)
- Week 2: Fine-tune T5 on Recipe1M+ (recipe generation)
- Week 3: Fine-tune ControlNet on cooking illustrations (line-art generation)
- Week 4: Train RNN for cooking time adjustment; implement safety validation

**Phase 3: Library Development** - 2 weeks
- Implement CLI interfaces for all libraries
- Extract notebook code to Python modules
- Write unit and contract tests

**Phase 4: API Integration** - 1 week
- Build FastAPI service
- Implement end-to-end pipeline
- Integration testing and latency optimization

**Phase 5: Testing & Validation** - 1 week
- End-to-end accuracy and latency validation
- Safety testing (food safety rules)
- User acceptance testing with sample recipes

**Total: 8-9 weeks to MVP**

### Success Metrics Tracking

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Ingredient recognition accuracy | 90%+ | Validate on held-out test set (500 images) |
| Total latency | <10s | Integration tests measure full pipeline time |
| Recipe diversity | 5-7 recipes, 3+ cuisines | Validate cuisine distribution in generated output |
| Nutrition accuracy | ±20% | Compare against USDA standard portions |
| User recipe completion | 85% within 3 uses | Track via analytics (post-MVP) |
| Recipe ratings | 4.0+/5.0 | Collect user ratings (post-MVP) |
| Beginner confidence | 80% improved | Pre/post surveys (post-MVP) |

---

**Next Steps**:
1. ✅ Complete this implementation plan
2. → Run `/speckit.tasks` to generate dependency-ordered task list
3. → Begin Phase 0 research (download datasets, set up environment)
4. → Start Phase 1 model training in Jupyter notebooks

**Files to Review Next**:
- `research.md` - Detailed technology research and alternatives analysis
- `data-model.md` - Complete entity definitions and relationships
- `contracts/api-schema.yaml` - Full OpenAPI 3.0 API specification
- `quickstart.md` - Detailed setup and development workflow guide
