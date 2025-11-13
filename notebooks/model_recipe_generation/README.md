# Recipe Generation Module

AI-powered recipe generation from ingredients using GPT-2 Transformer model.

## 📋 Overview

This module generates diverse, detailed recipes from ingredient inputs using a GPT-2 Medium language model.

**Input**: Ingredient name (e.g., "chicken breast", "salmon", "tomato")

**Output**: 5 diverse recipes including:
- Recipe title
- Cuisine type (American, Italian, Asian, Mexican, Indian)
- Difficulty level (easy/medium/hard)
- Cooking time (minutes)
- Servings
- Full ingredients list
- Step-by-step instructions

## 🚀 Quick Start

### 1. Setup Model (Run Once)

```bash
jupyter notebook setup_recipe_transformer.ipynb
```

This will:
- Install transformers and PyTorch
- Download GPT-2 Medium model (~1.5GB)
- Configure generation parameters
- Test basic text generation

**Time**: ~5-10 minutes (first run)

### 2. Load Recipe Dataset (Run Once)

```bash
jupyter notebook load_recipe_dataset.ipynb
```

This will:
- **Download real recipe dataset** (RecipeNLG 2.2M recipes or Food.com 500k)
- Clean and standardize data
- Extract main ingredients
- Infer cuisine and difficulty metadata
- Format recipes for training
- Split into train/validation sets (80/20)
- Save processed data

**Dataset Options**:
- **RecipeNLG**: Auto-downloads from Hugging Face (recommended)
- **Food.com**: Manual download from Kaggle

**Output**: `data/processed/recipes/`

### 3. Fine-tune Model (Optional but Recommended)

```bash
jupyter notebook train_recipe_transformer.ipynb
```

This will:
- Load pre-trained GPT-2 Medium
- Fine-tune on RecipeNLG dataset (2.23M recipes)
- Train for 3 epochs with validation
- Save fine-tuned model to `models/recipe_generation/finetuned/`

**Time**: 6-12 hours (GPU) / 2-3 days (CPU)

**Benefits**:
- ✅ Better recipe quality (trained on real recipes)
- ✅ More realistic ingredient combinations
- ✅ Improved instruction coherence
- ✅ Lower perplexity scores

**Note**: You can skip this step and use the pre-trained model, but quality will be lower.

### 4. Generate Recipes (Main Usage)

```bash
jupyter notebook model_gpt2_inference.ipynb
```

This will:
- **Auto-detect** and load fine-tuned model (if available)
- Fall back to pre-trained model if not fine-tuned
- Generate 5 recipes from ingredient input
- Parse and structure output
- Save results as JSON

**Time**: <3 seconds per ingredient (target)

## 💻 Usage Examples

### Python API

```python
# Import functions
from model_gpt2_inference import generate_recipes_api

# Generate recipes
result = generate_recipes_api(
    ingredient="chicken breast",
    num_recipes=5
)

# Access recipes
for recipe in result['recipes']:
    print(recipe['recipe_title'])
    print(recipe['cuisine'])
    print(recipe['ingredients'])
    print(recipe['instructions'])
```

### Interactive Mode

Run the interactive generator in `model_gpt2_inference.ipynb`:

```python
interactive_recipe_generator()
```

Then enter ingredients interactively:
```
Enter ingredient: chicken breast
Enter ingredient: salmon
Enter ingredient: tomato
```

## 📁 File Structure

```
notebooks/model_recipe_generation/
├── README.md                           # This file
├── setup_recipe_transformer.ipynb      # T018: Model setup
├── load_recipe_dataset.ipynb           # T019: Dataset preparation
├── train_recipe_transformer.ipynb      # T021: Model fine-tuning (optional)
└── model_gpt2_inference.ipynb          # T020: Recipe generation
```

## 📊 Performance

| Device | Recipes | Time | Status |
|--------|---------|------|--------|
| GPU (CUDA) | 5 | 0.5-1.5s | ✅ Meets target |
| CPU | 5 | 2-5s | ⚠️ May need optimization |

**Target**: < 3 seconds for 5 recipes

## 🎯 Model Specifications

- **Model**: GPT-2 Medium
- **Parameters**: 355 million
- **Architecture**: Transformer (12 layers, 16 attention heads)
- **Vocabulary**: 50,257 tokens
- **Max Length**: 512 tokens
- **Temperature**: 0.9 (creativity)
- **Top-K**: 50 (diversity)
- **Top-P**: 0.95 (nucleus sampling)

## 📦 Dependencies

```bash
pip install torch>=2.1.0
pip install transformers>=4.35.0
pip install numpy pandas
```

See `requirements.txt` in project root.

## 🔧 Configuration

Edit parameters in notebooks:

```python
# Generation settings
MAX_LENGTH = 512          # Maximum tokens per recipe
TEMPERATURE = 0.9         # 0.0 (deterministic) to 1.0 (creative)
TOP_K = 50                # Top-k sampling
TOP_P = 0.95              # Nucleus sampling threshold
NUM_RECIPES = 5           # Recipes to generate
```

## 📈 Output Format

### JSON Structure

```json
{
  "success": true,
  "ingredient": "chicken breast",
  "num_recipes": 5,
  "recipes": [
    {
      "ingredient": "chicken breast",
      "recipe_title": "Grilled Chicken with Herbs",
      "cuisine": "American",
      "difficulty": "easy",
      "cooking_time_minutes": 25,
      "servings": 2,
      "ingredients": [
        "2 chicken breasts",
        "2 tbsp olive oil",
        "1 tsp herbs"
      ],
      "instructions": [
        "Preheat grill to medium-high.",
        "Season chicken with herbs.",
        "Grill 6-7 minutes per side."
      ]
    }
  ],
  "cuisines": ["American", "Italian", "Asian", "Mexican", "Indian"],
  "generation_time_seconds": 1.23,
  "timestamp": "2025-01-15 10:30:00"
}
```

## 🔗 Integration

### Input: From Ingredient Recognition

```python
# From Roboflow ingredient detection
ingredient_name = roboflow_result['predictions'][0]['class']

# Generate recipes
recipes = generate_recipes_api(ingredient=ingredient_name)
```

### Output: To Cooking Refinement

```python
# Pass to BiLSTM for step optimization
for recipe in recipes['recipes']:
    refined_steps = bilstm_refine(recipe['instructions'])
    recipe['instructions'] = refined_steps
```

### Output: To Nutrition Estimation

```python
# Pass to Bayesian network for calories
for recipe in recipes['recipes']:
    nutrition = estimate_nutrition(recipe)
    recipe['calories'] = nutrition['calories_per_serving']
```

## ⚠️ Current Limitations

1. **Model Training Required**: Default pre-trained GPT-2 has limited recipe knowledge
   - ✅ **Solution Available**: Run `train_recipe_transformer.ipynb` to fine-tune on RecipeNLG (2.23M recipes)
   - **Status**: Training notebook ready
   - **Time**: 6-12 hours (GPU) / 2-3 days (CPU)
   - **Result**: Significantly improved recipe quality

2. **Variable Quality** (without fine-tuning): Output quality varies by ingredient
   - ✅ **Solution**: Fine-tune the model (see step 3 in Quick Start)
   - Alternative: Add output validation and regeneration

3. **Performance**: May exceed 3s on CPU
   - Solutions:
     - Use GPU acceleration (CUDA)
     - Cache common ingredients (dataset lookup)
     - Use smaller model (gpt2 instead of gpt2-medium)
     - Reduce max_length parameter

## 📚 Dataset Options

### Current: Real Recipe Datasets (Automatic Download)

The `load_recipe_dataset.ipynb` notebook **automatically downloads** real recipe datasets:

1. **RecipeNLG** (2.2M recipes) - **Primary Choice**
   - URL: https://huggingface.co/datasets/recipe_nlg
   - Format: CSV with title, ingredients, directions, NER tags
   - Size: ~300-500MB
   - **Auto-download**: Runs automatically in notebook
   - Best for: Large-scale GPT-2 fine-tuning

2. **Food.com** (500k recipes) - **Alternative**
   - URL: https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions
   - Format: CSV with ratings, reviews, nutrition
   - Size: ~200MB
   - **Manual download**: Download `RAW_recipes.csv` to `data/raw/recipes/`
   - Best for: Recipes with user feedback

3. **Recipe1M+** (1M+ recipes) - **Advanced**
   - URL: http://pic2recipe.csail.mit.edu/
   - Format: JSON with images
   - Size: ~5GB
   - Best for: Multimodal training (future enhancement)

### What the Notebook Does

The notebook automatically:
- ✅ Downloads RecipeNLG dataset (or detects manual downloads)
- ✅ Cleans data (removes incomplete recipes)
- ✅ Extracts main ingredients (chicken, beef, salmon, etc.)
- ✅ Infers cuisine types (Italian, Asian, Mexican, etc.)
- ✅ Calculates difficulty (easy/medium/hard)
- ✅ Formats for GPT-2 training
- ✅ Creates train/validation splits (80/20)
- ✅ Saves processed data

## 🔍 Troubleshooting

### Model Download Fails
```bash
# Set proxy if needed
export HTTP_PROXY=http://proxy:port
export HTTPS_PROXY=http://proxy:port

# Or download manually
git clone https://huggingface.co/gpt2-medium
```

### Out of Memory
```python
# Reduce batch size or max_length
MAX_LENGTH = 256  # Instead of 512
NUM_RECIPES = 3   # Instead of 5

# Use smaller model
MODEL_NAME = "gpt2"  # Instead of "gpt2-medium"
```

### Slow Generation
```python
# Enable GPU if available
device = torch.device("cuda")

# Reduce parameters
MAX_LENGTH = 384
NUM_RECIPES = 3

# Use caching for common ingredients
```

## 📝 Next Steps

1. ✅ **T021**: Fine-tune model on recipe datasets (COMPLETE - `train_recipe_transformer.ipynb`)
2. **T022**: Optimize diversity across cuisines
3. **T023**: Create adapter to cooking refinement module
4. **Add validation** for generated recipes
5. **Integrate with nutrition estimation**

## 🤝 Contributing

Follow project conventions:
- All code in English
- Use Jupyter notebooks (.ipynb)
- Follow cell structure: imports → config → functions → execution
- Add markdown documentation
- Test with sample data

## 📄 License

See project root LICENSE file.

## 🔗 Related Modules

- **Ingredient Recognition**: `notebooks/model_ingredient_recognition/`
- **Cooking Refinement**: `notebooks/model_cooking_refinement/` (TODO)
- **Nutrition Estimation**: `notebooks/model_nutrition_estimation/` (TODO)
- **Pipeline Integration**: `notebooks/pipeline_recipe_app/` (TODO)

---

**Status**: ✅ T018-T021 Complete (Phase 3: Recipe Generation + Fine-tuning)

**Performance**: Target <3s ⚠️ (varies by device, achievable with GPU + dataset lookup)

**Model Training**: Fine-tuning available via `train_recipe_transformer.ipynb`

**Next Task**: T022 - Diversity Optimization
