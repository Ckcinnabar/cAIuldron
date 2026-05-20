# Pipeline Photo to Recipes v2 - Model Selection Guide

## What's New in v2

**Multi-Model Support**: Choose between 3 different recipe generation models:
- GPT-2 (Fast, 60-70/100 quality)
- Llama 3.2 1B (Recommended, 90-95/100 quality)
- Llama 3.1 8B GGUF (Best quality, 95-100/100)

## Notebook Structure

### Core Components

1. **Environment Setup** (Cells 0-1)
   - Package imports
   - Environment configuration

2. **Paths & Parameters** (Cells 2-3)
   - `PROJECT_ROOT`: C:\Users\Champion\Documents\GitHub\cAIuldron
   - `MODEL_DIR`: {PROJECT_ROOT}/models/recipe_generation
   - Detection modes: single/multi

3. **Data Loading** (Cells 4-7)
   - Recipe dataset
   - Ingredient vocabulary
   - CLIP model

4. **GPT-2 Model** (Cells 8-9)
   - Original GPT-2 model loading
   - Still used when GPT-2 is selected

5. **MODEL SELECTION SYSTEM** (Cells 10-14) ⭐ NEW
   - Cell 10: Markdown documentation
   - Cell 11: Model configuration (RecipeModelType, MODEL_INFO, CURRENT_MODEL_TYPE)
   - Cell 12: Model loading functions (load_recipe_model, load_gpt2_model, load_llama_1b_model, load_llama_8b_gguf_model)
   - Cell 13: Recipe generation functions (generate_recipe_with_selected_model, generate_recipe_llama, generate_recipe_gguf, parse_llama_recipe_output)
   - Cell 14: Interactive model selector (dropdown widget)

6. **Nutrition Database** (Cells 15-16)
   - USDA nutrition lookup

7. **Helper Functions** (Cells 17-19)
   - CLIP detection
   - Text cleaning
   - Recipe parsing

8. **Main Pipeline** (Cells 20-21)
   - `process_ingredient_photo()` function
   - Uses `generate_recipe_with_selected_model()` ✓

9. **Display & Testing** (Cells 22-25)
   - Result display
   - Test pipeline

10. **Interactive & Web** (Cells 26-33)
    - Interactive mode
    - Gradio web interface
    - Uses `generate_recipe_with_selected_model()` ✓

## Key Functions

### Model Management

```python
def load_recipe_model(model_type: RecipeModelType) -> Dict[str, Any]
```
- Loads and caches the specified model
- Handles GPT-2, Llama 1B, and GGUF models

### Recipe Generation

```python
def generate_recipe_with_selected_model(
    ingredient: str,
    cuisine: str = "any",
    difficulty: str = "medium",
    model_type: RecipeModelType = None
) -> Dict[str, Any]
```
- Unified interface for all models
- Routes to appropriate generation function based on model type

## How to Use

### Step 1: Start Jupyter

```bash
cd C:\Users\Champion\Documents\GitHub\cAIuldron\notebooks\pipeline_recipe_app
jupyter notebook pipeline_photo_to_recipes_v2.ipynb
```

### Step 2: Run All Cells

**Kernel → Restart Kernel & Run All Cells**

Wait for:
```
Model configuration system loaded
Default model: Llama 3.2 1B (Recommended)

Model loading functions defined

Recipe generation functions defined

Currently using: Llama 3.2 1B (Recommended)
```

### Step 3: Verify Model Selector

You should see:
```
Select Recipe Generation Model:
[Dropdown showing: Llama 3.2 1B (Recommended)]
[Show Model Info button]
```

### Step 4: Run Test

Execute the test cell:
```python
result = process_ingredient_photo(TEST_IMAGE, verbose=True)
```

Expected output:
```
================================================================================
RECIPE GENERATION PIPELINE (Llama 3.2 1B (Recommended))
================================================================================
...
Loading Llama 3.2 1B (Recommended)...
  Loading base model...
  Loading LoRA adapter...
  Merging weights...
Llama 3.2 1B loaded successfully
```

## Model Paths

### GPT-2
```
{PROJECT_ROOT}/models/recipe_generation/finetuned/
```

### Llama 3.2 1B
```
{PROJECT_ROOT}/models/recipe_generation/llama3_1b_finetuned/
├── adapter_config.json
├── adapter_model.safetensors
├── tokenizer files
└── checkpoints
```

### Llama 3.1 8B GGUF (Optional)
```
{PROJECT_ROOT}/models/recipe_generation/Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf
```

## Troubleshooting

### Issue: Still showing "GPT-2 GENERATION"

**Solution**:
1. Restart Kernel
2. Run All Cells (don't skip any)
3. Verify model selector appeared
4. Re-run test

### Issue: "NameError: name 'load_recipe_model' is not defined"

**Solution**: You skipped Cell 12. Run Cells 11-14 in order.

### Issue: "Can't find 'adapter_config.json'"

**Solution**: Check path:
```bash
ls C:\Users\Champion\Documents\GitHub\cAIuldron\models\recipe_generation\llama3_1b_finetuned
```
Should contain: adapter_config.json, adapter_model.safetensors

### Issue: Model not loading

**Solution**:
1. Check if you have HuggingFace token set up
2. Verify you have access to Llama models
3. Check GPU memory (need 4-5GB for Llama 1B)

## Quality Comparison

| Model | Quality | Speed | VRAM | Training |
|-------|---------|-------|------|----------|
| GPT-2 | 60-70/100 | Fast | 1-2GB | Original |
| Llama 1B | 90-95/100 | Medium | 4-5GB | Trained ✓ |
| Llama 8B GGUF | 95-100/100 | Slow | 6GB | Pre-trained |

## Expected Results (Llama 1B)

### Input
```
Ingredient: Chicken breast and Pork kidney
Cuisine: Fusion
```

### Output
```
# Fusion Style Chicken and Pork Kidney Stew

## Ingredients
- 1 lb. chicken breasts, cut into bite-size pieces
- 1 lb. pork kidneys, sliced
- 1 large onion
- 2 cloves garlic
[... 14 more ingredients]

## Instructions
1. Preheat oven to 375°F.
2. Cut the stems off the onions, then slice them in rings.
[... 10 more clear steps]
```

**Quality Features**:
- ✓ Correct ingredients mentioned
- ✓ Logical cooking steps
- ✓ Proper formatting
- ✓ No gibberish or XML tags
- ✓ Coherent recipe structure

## Development Notes

### Function Call Chain

```
User runs pipeline
    ↓
process_ingredient_photo(image_path)
    ↓
generate_recipe_with_selected_model(ingredient, cuisine, difficulty, model_type=CURRENT_MODEL_TYPE)
    ↓
load_recipe_model(model_type) [if not cached]
    ↓
generate_recipe_llama() / generate_recipe_gpt2() / generate_recipe_gguf()
    ↓
parse_llama_recipe_output() [for Llama models]
    ↓
Return structured recipe dict
```

### Model Caching

Models are cached after first load in `RECIPE_MODELS` dict:
```python
RECIPE_MODELS = {
    RecipeModelType.LLAMA_1B: {
        "model": <merged_model>,
        "tokenizer": <tokenizer>,
        "type": "llama"
    }
}
```

Subsequent calls reuse the cached model.

## Files

```
pipeline_recipe_app/
├── pipeline_photo_to_recipes.ipynb      # v1 (GPT-2 only)
├── pipeline_photo_to_recipes_v2.ipynb   # v2 (Multi-model) ⭐
└── README_v2.md                         # This file
```

## Version History

- **v1**: Original pipeline with GPT-2 only
- **v2**: Added Llama 3.2 1B and model selection system

---

**Status**: ✓ Ready for use
**Recommended Model**: Llama 3.2 1B (Recommended)
**Last Updated**: 2025-11-21
