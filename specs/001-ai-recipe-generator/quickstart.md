# Quickstart Guide: AI Recipe Generator

**Branch**: 001-ai-recipe-generator
**Last Updated**: 2025-10-16

## Prerequisites

Before starting development, ensure you have:

- **Python 3.11+** (constitutional requirement: 3.8+, but 3.11 recommended)
- **NVIDIA GPU** with 11GB+ VRAM (recommended: RTX 3080, RTX 4080, or cloud V100/A100)
- **CUDA 11.8+** and cuDNN installed
- **Git** for version control
- **50GB free disk space** (models + datasets)
- **USDA FoodData Central API key** (free, register at https://fdc.nal.usda.gov/api-key-signup.html)

## Initial Setup

### 1. Clone Repository

```bash
git clone <repository-url>
cd cAIuldron
git checkout 001-ai-recipe-generator
```

### 2. Create Virtual Environment

```bash
# Create venv (constitutional requirement: isolated environments)
python3.11 -m venv venv

# Activate
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate  # Windows
```

### 3. Install Dependencies

```bash
# Install core dependencies
pip install --upgrade pip
pip install -r requirements.txt

# Expected packages:
# - torch>=2.1.0, torchvision, torchaudio
# - transformers>=4.35.0
# - diffusers>=0.24.0
# - fastapi>=0.104.0
# - jupyterlab>=4.0.0
# - pytest>=7.4.0, nbval>=0.10.0
# - mlflow>=2.8.0
# - pillow, numpy, pandas
```

### 4. Download Pre-trained Models

```bash
# Run model download script (first-time setup only)
python scripts/download_models.py

# This downloads:
# - ViT model for ingredient recognition (~300MB)
# - T5-base for recipe generation (~220MB)
# - Stable Diffusion + ControlNet for illustrations (~3.4GB)
# Total: ~4GB, takes 10-20 minutes depending on connection
```

### 5. Set Up API Keys

```bash
# USDA FoodData Central API (free tier: 1000 requests/hour)
export USDA_API_KEY="your-api-key-here"

# Optional: HuggingFace token for gated models
export HF_TOKEN="your-huggingface-token"  # Only if using gated models
```

### 6. Verify Installation

```bash
# Test imports
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
python -c "import transformers; import diffusers; print('All imports successful')"

# Expected output:
# PyTorch: 2.1.x, CUDA: True
# All imports successful
```

## Development Workflow

### Option A: Jupyter Notebook Development (Recommended for ML Work)

```bash
# Start Jupyter Lab
jupyter lab

# Open browser to http://localhost:8888
# Navigate to notebooks/02-model-training/
# Start with 01-ingredient-recognition-vit.ipynb
```

**Jupyter Best Practices**:
- Use markdown cells to document experiments
- Log all experiments with MLflow
- Clear outputs before committing (`jupyter nbconvert --clear-output --inplace notebook.ipynb`)
- Validate notebooks with nbval: `pytest --nbval notebooks/`

### Option B: CLI Development (For Testing Libraries)

```bash
# Test ingredient recognition CLI
python -m src.ingredient_recognition.cli --image test_images/chicken.jpg --output json

# Test recipe generation CLI
python -m src.recipe_generation.cli --ingredient "chicken breast" --weight 300 --count 5 --output json

# Test nutrition calculator CLI
python -m src.nutrition_calculator.cli --ingredient "chicken breast" --weight 300 --output text
```

### Option C: API Development (For Integration)

```bash
# Start FastAPI server
python src/api/main.py

# Or with auto-reload
uvicorn src.api.main:app --reload --host 0.0.0.0 --port 8000

# Access at:
# - API: http://localhost:8000
# - Docs: http://localhost:8000/docs (Swagger UI)
# - OpenAPI spec: http://localhost:8000/openapi.json
```

## Running Tests

```bash
# Run all tests
pytest

# Run specific test categories
pytest tests/unit/                # Unit tests (fast, no GPU needed)
pytest tests/contract/            # Model schema validation
pytest tests/integration/         # End-to-end pipeline (slow, GPU required)

# Run notebook validation (constitutional requirement)
pytest --nbval notebooks/

# Run with coverage
pytest --cov=src tests/

# Run specific test file
pytest tests/integration/test_full_pipeline.py -v
```

## Model Training Workflow

### Phase 1: Data Preparation (Week 1)

```bash
# Download datasets
python scripts/download_datasets.py

# This downloads:
# - Food-101 (101K images, ~5GB)
# - Recipe1M+ sample (100K recipes, ~2GB)
# - Cooking illustration dataset (5K images, ~500MB)
```

### Phase 2: Model Fine-tuning (Weeks 2-4)

```bash
# Start Jupyter and open training notebooks in sequence:
# 1. notebooks/02-model-training/01-ingredient-recognition-vit.ipynb
# 2. notebooks/02-model-training/02-recipe-generation-t5.ipynb
# 3. notebooks/02-model-training/03-cooking-time-rnn.ipynb
# 4. notebooks/02-model-training/04-illustration-controlnet.ipynb

# Each notebook:
# 1. Loads dataset
# 2. Fine-tunes model with MLflow tracking
# 3. Evaluates on validation set
# 4. Exports trained model to models/ directory
```

### Phase 3: Model Export

```python
# In Jupyter or Python script
import torch
import mlflow

# Load best model from MLflow
model = mlflow.pytorch.load_model("models:/ingredient-recognition/Production")

# Export to TorchScript for production
scripted_model = torch.jit.script(model)
scripted_model.save("models/ingredient_recognition.pt")

# Or export to ONNX
torch.onnx.export(model, dummy_input, "models/ingredient_recognition.onnx")
```

## Experiment Tracking with MLflow

```bash
# Start MLflow UI
mlflow ui --backend-store-uri sqlite:///mlflow.db --port 5000

# Access at http://localhost:5000
# View experiments, compare models, check metrics
```

**MLflow Usage in Notebooks**:
```python
import mlflow

mlflow.set_experiment("ingredient-recognition")
mlflow.start_run(run_name="vit-fine-tune-20250116")

# Log parameters
mlflow.log_params({
    'model': 'vit-base',
    'learning_rate': 3e-4,
    'batch_size': 32
})

# Training loop...

# Log metrics
mlflow.log_metrics({
    'train_accuracy': 0.92,
    'val_accuracy': 0.89
})

# Log model
mlflow.pytorch.log_model(model, "model")
mlflow.end_run()
```

## Docker Deployment (Optional)

```bash
# Build Docker image
docker build -t ai-recipe-generator:latest .

# Run with GPU support
docker run --gpus all -p 8000:8000 ai-recipe-generator:latest

# Or use Docker Compose
docker-compose up
```

## Common Issues & Solutions

### Issue: CUDA out of memory
**Solution**: Reduce batch size in training notebooks or use gradient accumulation

### Issue: Models downloading slowly
**Solution**: Use HuggingFace mirror or download models manually

### Issue: Import errors
**Solution**: Ensure virtual environment is activated: `which python` should show venv path

### Issue: API returns 500 errors
**Solution**: Check logs, ensure models are loaded: `curl http://localhost:8000/health`

### Issue: Notebook validation fails
**Solution**: Clear outputs before committing: `jupyter nbconvert --clear-output --inplace *.ipynb`

## Project Structure Reference

```
cAIuldron/
├── src/                          # Python libraries (production code)
│   ├── ingredient_recognition/   # ViT-based ingredient detection
│   ├── recipe_generation/        # T5-based recipe generation
│   ├── illustration_generation/  # ControlNet+SD illustrations
│   ├── nutrition_calculator/     # USDA API nutrition lookup
│   └── api/                      # FastAPI service
├── notebooks/                    # Jupyter development (constitutional requirement)
│   ├── 01-data-exploration/
│   ├── 02-model-training/
│   ├── 03-evaluation/
│   └── 04-deployment/
├── tests/                        # pytest test suite
│   ├── unit/
│   ├── contract/
│   └── integration/
├── models/                       # Trained model weights
├── data/                         # Datasets
├── specs/001-ai-recipe-generator/  # Feature documentation
│   ├── spec.md
│   ├── plan.md
│   ├── research.md
│   ├── data-model.md
│   ├── quickstart.md (this file)
│   └── contracts/
└── requirements.txt
```

## Next Steps

1. ✅ Complete setup (this guide)
2. → Read `plan.md` for implementation overview
3. → Read `research.md` for technology decisions
4. → Start with `notebooks/01-data-exploration/` to understand datasets
5. → Begin model training in `notebooks/02-model-training/`
6. → Extract stable code to `src/` Python modules
7. → Write tests in `tests/`
8. → Build FastAPI integration in `src/api/`
9. → Run end-to-end validation

## Getting Help

- **Specification**: `specs/001-ai-recipe-generator/spec.md`
- **Implementation Plan**: `specs/001-ai-recipe-generator/plan.md`
- **Technical Research**: `specs/001-ai-recipe-generator/research.md`
- **Data Model**: `specs/001-ai-recipe-generator/data-model.md`
- **API Contracts**: `specs/001-ai-recipe-generator/contracts/api-schema.yaml`
- **Constitution**: `.specify/memory/constitution.md` (project principles)

## Development Checklist

- [ ] Setup complete (Python, GPU, dependencies)
- [ ] USDA API key configured
- [ ] Models downloaded (ViT, T5, ControlNet)
- [ ] Jupyter Lab running
- [ ] MLflow UI accessible
- [ ] Datasets downloaded (Food-101, Recipe1M+)
- [ ] First notebook executed successfully
- [ ] CLI interfaces tested
- [ ] Tests passing (`pytest`)
- [ ] API server running
- [ ] End-to-end pipeline validated

---

**Ready to start?** Open Jupyter Lab and begin with `notebooks/01-data-exploration/01-food101-eda.ipynb`
