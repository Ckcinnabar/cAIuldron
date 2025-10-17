# Quickstart Guide: AI-Powered Recipe Generator

**Feature**: AI-Powered Recipe Generator
**Audience**: Developers implementing the notebook-based recipe generation system
**Prerequisites**: Python 3.11+, JupyterLab 4.0+, 16GB RAM

## Overview

This guide walks you through the end-to-end workflow of the AI-powered recipe generator, from setting up your environment to generating illustrated recipes from ingredient photos.

## System Architecture

```
Photo Upload → CNN Recognition → Recipe Generation → Nutrition Estimation → Illustration Creation
                                  (Transformer + RNN)    (Bayesian PGM)        (GAN)
```

**Total Processing Time**: 4-8 seconds
**Models Used**: 5 specialized AI components working in concert

## Setup

### 1. Environment Setup

Create and activate a Python environment:

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

**Key Dependencies** (requirements.txt):
```
torch==2.1.0
tensorflow==2.14.0
transformers==4.35.0
opencv-python==4.8.1
pillow==10.1.0
pandas==2.1.3
matplotlib==3.8.2
pgmpy==0.1.23
jupyterlab==4.0.9
papermill==2.5.0
nbformat==5.9.2
tqdm==4.66.1
```

### 2. Download Pre-trained Models

```bash
# Create models directory
mkdir -p models

# Download model weights (examples - adjust URLs to actual model locations)
# CNN: EfficientNetV2-S
wget -O models/cnn_ingredient_recognition.h5 [MODEL_URL]

# Transformer: GPT-2 Medium (fine-tuned)
wget -O models/transformer_recipe_generation.pt [MODEL_URL]

# RNN: BiLSTM
wget -O models/rnn_cooking_refinement.pt [MODEL_URL]

# PGM: Bayesian Network
wget -O models/pgm_nutrition_estimation.pkl [MODEL_URL]

# GAN: Pix2Pix
wget -O models/gan_illustration_generation.pt [MODEL_URL]
```

### 3. Download Nutritional Database

```bash
# USDA FoodData Central
mkdir -p data/raw/nutrition_database
wget -O data/raw/nutrition_database/usda_fooddata_2024.csv [DB_URL]
```

### 4. Create Directory Structure

```bash
# Create all necessary directories
mkdir -p notebooks/explore_ingredients
mkdir -p notebooks/model_ingredient_recognition
mkdir -p notebooks/model_recipe_generation
mkdir -p notebooks/model_cooking_refinement
mkdir -p notebooks/model_nutrition_estimation
mkdir -p notebooks/model_illustration_generation
mkdir -p notebooks/pipeline_recipe_app
mkdir -p notebooks/utils_recipe
mkdir -p data/raw/{ingredient_images,recipe_corpus,nutrition_database}
mkdir -p data/processed/{ingredient_features,recipe_tokens,nutrition_vectors}
mkdir -p data/results/{generated_recipes,illustrations,nutrition_estimates}
mkdir -p tests/{notebook_tests,unit_tests}
```

## Workflow

### Phase 1: Ingredient Recognition (CNN)

**Notebook**: `notebooks/model_ingredient_recognition/model_cnn_inference.ipynb`

```python
# Cell 1: Imports and setup
import torch
import numpy as np
from PIL import Image
import json
from pathlib import Path

# Set random seeds for reproducibility
np.random.seed(42)
torch.manual_seed(42)

# Cell 2: Load CNN model
from torchvision import models, transforms

model = models.efficientnet_v2_s(weights=None)
model.load_state_dict(torch.load('models/cnn_ingredient_recognition.h5'))
model.eval()

# Cell 3: Preprocess image
def preprocess_image(image_path):
    transform = transforms.Compose([
        transforms.Resize((384, 384)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406],
                           std=[0.229, 0.224, 0.225])
    ])
    img = Image.open(image_path).convert('RGB')
    return transform(img).unsqueeze(0)

# Cell 4: Run inference
image_tensor = preprocess_image('data/raw/ingredient_images/chicken_breast.jpg')
with torch.no_grad():
    output = model(image_tensor)
    confidence, prediction = torch.max(torch.softmax(output, dim=1), dim=1)

# Cell 5: Extract ingredient info
ingredient = {
    "ingredient_id": str(uuid.uuid4()),
    "name": ingredient_classes[prediction.item()],
    "confidence_score": confidence.item(),
    "estimated_quantity": {
        "weight_grams": estimate_weight(image_tensor),  # Custom function
        "weight_confidence": 0.75
    }
}

# Cell 6: Save result
output_path = f"data/processed/ingredients/{ingredient['ingredient_id']}.json"
with open(output_path, 'w') as f:
    json.dump(ingredient, f, indent=2)
```

**Expected Output**: Ingredient JSON with 90%+ confidence

### Phase 2: Recipe Generation (Transformer + RNN)

**Notebook**: `notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb`

```python
# Cell 1: Load ingredient
with open('data/processed/ingredients/[INGREDIENT_ID].json') as f:
    ingredient = json.load(f)

# Cell 2: Load Transformer model
from transformers import GPT2LMHeadModel, GPT2Tokenizer

tokenizer = GPT2Tokenizer.from_pretrained('gpt2-medium')
model = GPT2LMHeadModel.from_pretrained('models/transformer_recipe_generation.pt')

# Cell 3: Generate 5 diverse recipes
cuisines = ['Asian', 'Western', 'Mediterranean', 'Fusion', 'Latin']
recipes = []

for cuisine in cuisines:
    prompt = f"Ingredient: {ingredient['name']}, Cuisine: {cuisine}, Recipe:"
    inputs = tokenizer(prompt, return_tensors='pt')

    outputs = model.generate(
        inputs['input_ids'],
        max_length=512,
        temperature=0.8,
        top_p=0.9,
        do_sample=True
    )

    recipe_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
    recipes.append(parse_recipe(recipe_text))  # Custom parser

# Cell 4: Refine with RNN
from model_cooking_refinement import refine_steps  # Custom module

for recipe in recipes:
    recipe['cooking_steps'] = refine_steps(
        recipe['cooking_steps'],
        model_path='models/rnn_cooking_refinement.pt'
    )
    recipe['cooking_time_minutes'] = sum(
        step.get('estimated_time_minutes', 0)
        for step in recipe['cooking_steps']
    )

# Cell 5: Save recipes
for recipe in recipes:
    output_path = f"data/results/generated_recipes/{recipe['recipe_id']}.json"
    with open(output_path, 'w') as f:
        json.dump(recipe, f, indent=2)
```

**Expected Output**: 5 recipes with diverse cuisines, refined timing

### Phase 3: Nutrition Estimation (Bayesian PGM)

**Notebook**: `notebooks/model_nutrition_estimation/model_pgm_inference.ipynb`

```python
# Cell 1: Load nutritional database
import pandas as pd

usda_db = pd.read_csv('data/raw/nutrition_database/usda_fooddata_2024.csv')

# Cell 2: Load Bayesian Network
from pgmpy.models import BayesianNetwork
from pgmpy.inference import VariableElimination
import pickle

with open('models/pgm_nutrition_estimation.pkl', 'rb') as f:
    bn_model = pickle.load(f)

inference = VariableElimination(bn_model)

# Cell 3: Estimate nutrition for each recipe
for recipe in recipes:
    # Query USDA database for each ingredient
    total_calories = 0
    for ing in recipe['ingredients_list']:
        ing_data = usda_db[usda_db['name'] == ing['name']]
        calories = ing_data['calories_per_100g'].values[0] * (ing['quantity'] / 100)
        total_calories += calories

    # Run Bayesian inference for confidence interval
    evidence = {
        'ingredient_type': ingredient['category'],
        'estimated_weight': ingredient['estimated_quantity']['weight_grams']
    }
    result = inference.query(['calories'], evidence=evidence)

    # Create portion estimate
    portion = {
        "estimate_id": str(uuid.uuid4()),
        "recipe_id": recipe['recipe_id'],
        "calories_per_serving": total_calories / recipe['serving_count'],
        "calorie_range": {
            "min": result.values[0] * 0.8,
            "max": result.values[0] * 1.2,
            "confidence_level": 0.8
        }
    }

    recipe['portion_estimate'] = portion
```

**Expected Output**: Calorie estimates with ±20% accuracy

### Phase 4: Illustration Generation (GAN)

**Notebook**: `notebooks/model_illustration_generation/model_gan_inference.ipynb`

```python
# Cell 1: Load GAN model
import torch
from pix2pix_model import Pix2PixGenerator  # Custom model class

generator = Pix2PixGenerator()
generator.load_state_dict(torch.load('models/gan_illustration_generation.pt'))
generator.eval()

# Cell 2: Generate illustrations for each recipe
for recipe in recipes:
    for step in recipe['cooking_steps']:
        # Extract cooking action
        action = step['cooking_method']

        # Check template library first
        template_path = f"data/raw/illustration_templates/{action}.svg"
        if Path(template_path).exists() and step.get('use_template', True):
            illustration_path = template_path
        else:
            # Generate with GAN
            text_embedding = encode_text(step['instruction_text'])  # Custom
            with torch.no_grad():
                generated = generator(text_embedding)

            # Save illustration
            illustration_path = f"data/results/illustrations/{recipe['recipe_id']}/step_{step['step_number']}.svg"
            save_as_svg(generated, illustration_path)

        # Add to step
        step['illustration'] = {
            "illustration_id": str(uuid.uuid4()),
            "file_path": illustration_path,
            "quality_score": calculate_quality(generated)  # Custom
        }
```

**Expected Output**: Line-art illustrations for each cooking step

### Phase 5: End-to-End Pipeline

**Notebook**: `notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb`

```python
# Complete pipeline execution
def generate_recipes_from_photo(image_path):
    # Step 1: Ingredient Recognition (CNN)
    ingredient = recognize_ingredient(image_path)

    if ingredient['confidence_score'] < 0.7:
        return {"error": "LOW_CONFIDENCE", "ingredient": ingredient}

    # Step 2: Recipe Generation (Transformer + RNN)
    recipes = generate_recipes(ingredient, num_recipes=5)
    recipes = [refine_recipe_steps(r) for r in recipes]

    # Step 3: Nutrition Estimation (Bayesian PGM)
    for recipe in recipes:
        recipe['portion_estimate'] = estimate_nutrition(recipe, ingredient)

    # Step 4: Illustration Generation (GAN)
    for recipe in recipes:
        recipe['cooking_steps'] = add_illustrations(recipe['cooking_steps'])

    return {
        "ingredient": ingredient,
        "recipes": recipes,
        "processing_time_ms": calculate_time()
    }

# Execute pipeline
result = generate_recipes_from_photo('data/raw/ingredient_images/salmon.jpg')

# Display results
print(f"Identified: {result['ingredient']['name']}")
print(f"Confidence: {result['ingredient']['confidence_score']:.2%}")
print(f"Generated {len(result['recipes'])} recipes in {result['processing_time_ms']}ms")
```

## Testing

### 1. Notebook Execution Tests

```bash
# Run notebook tests with papermill
papermill notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb \
  output.ipynb \
  -p test_image "data/raw/ingredient_images/chicken_breast.jpg"

# Validate output
python tests/notebook_tests/test_end_to_end_pipeline.py
```

### 2. Unit Tests

```bash
# Run unit tests
pytest tests/unit_tests/
```

### 3. Manual Validation

**Checklist**:
- ✅ All notebooks execute top-to-bottom without errors
- ✅ Ingredient recognition confidence ≥ 0.7 for 80% of common ingredients
- ✅ 5 recipes generated with at least 3 different cuisines
- ✅ Calorie estimates within ±20% of expected values
- ✅ Illustrations generated with quality score ≥ 0.6
- ✅ Total processing time < 15 seconds

## Troubleshooting

### Issue: CNN confidence too low

**Solution**: Improve photo quality - ensure good lighting, clear focus, single ingredient

### Issue: Recipe generation fails

**Solution**: Check Transformer model loaded correctly, verify GPU/CPU compatibility

### Issue: RNN refinement slow

**Solution**: Use CPU inference (RNN is lightweight), check for LSTM optimization

### Issue: Nutrition database missing ingredient

**Solution**: Fallback to category average (e.g., "fish" avg if "salmon" missing)

### Issue: GAN generates low-quality illustrations

**Solution**: Use template library as fallback, retry generation with different seed

## Performance Optimization

### 1. Parallel Processing

```python
# Generate recipes in parallel
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=5) as executor:
    recipes = list(executor.map(generate_recipe_for_cuisine, cuisines))
```

### 2. Model Quantization

```python
# Reduce Transformer model size
from transformers import GPT2LMHeadModel
import torch

model = GPT2LMHeadModel.from_pretrained('gpt2-medium')
model = torch.quantization.quantize_dynamic(model, {torch.nn.Linear}, dtype=torch.qint8)
```

### 3. Caching

```python
# Cache CNN embeddings
from functools import lru_cache

@lru_cache(maxsize=100)
def get_ingredient_embedding(ingredient_name):
    # Return cached embedding if available
    pass
```

## Next Steps

1. **Implement User Interface**: Create interactive widgets for photo upload and recipe display
2. **Add User Feedback**: Collect ratings to improve model performance
3. **Expand Ingredient Database**: Support more ingredients and regional variations
4. **Optimize for Production**: Deploy models with ONNX or TensorRT for faster inference
5. **Create API Endpoints**: Expose notebook logic via REST API for app integration

## Resources

- **Documentation**: `/specs/001-ai-recipe-generator/`
- **Data Models**: `data-model.md`
- **API Contracts**: `contracts/`
- **Research**: `research.md`
- **Implementation Plan**: `plan.md`

## Support

For issues or questions, refer to:
- Constitution: `.specify/memory/constitution.md`
- Notebook best practices: Follow PEP 8, use type hints, document with markdown cells
- Reproducibility: Set all random seeds, version dependencies, checkpoint models
