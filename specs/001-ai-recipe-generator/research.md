# Research: AI Model Architecture for Recipe Generator

**Feature**: AI-Powered Recipe Generator
**Date**: 2025-10-16
**Purpose**: Document AI model selection, implementation approach, and best practices for each component

## Overview

The recipe generator uses a multi-model AI ecosystem with 5 specialized components working together to transform ingredient photos into complete cooking guides.

## 1. Roboflow API: Ingredient Recognition & Measurement

### Model Selection

**Chosen**: Roboflow Inference SDK (Serverless API)
- **Model**: `food-ingredients-dataset/2` (pre-trained object detection)
- **Rationale**:
  - No local model training required
  - Serverless deployment (auto-scaling, no infrastructure management)
  - Fast inference (<200ms via HTTP API)
  - Pre-trained on diverse food dataset
  - Includes bounding box detection for portion estimation
- **Cost**: Pay-per-use (free tier available)
- **Inference time**: ~100-200ms per image (including network latency)

**Alternatives Considered**:
- EfficientNetV2-S: Requires local training, model management, and GPU resources
- Custom CNN: High development cost, longer time-to-market
- Cloud Vision APIs: More expensive, less specialized for ingredients

### Implementation Approach

**API Integration Strategy**:
- Use `inference-sdk` Python package (lightweight HTTP client)
- Serverless inference via `https://serverless.roboflow.com`
- No model download or local deployment needed

**Input Specifications**:
- Supported formats: JPEG, PNG, WebP
- Image size: Flexible (API handles resizing)
- Preprocessing: Handled automatically by Roboflow API

**Response Structure**:
```json
{
  "predictions": [{
    "class": "chicken breast",
    "confidence": 0.87,
    "x": 320, "y": 240,
    "width": 200, "height": 150
  }],
  "image": {"width": 640, "height": 480}
}
```

**Size Estimation Logic**:
- Use bounding box dimensions from API response
- Calculate ingredient area percentage: `(bbox_width * bbox_height) / (image_width * image_height)`
- Estimate weight using ingredient-specific density lookup table
- Provide confidence intervals based on bbox confidence score

**Data Source**:
- Roboflow Universe: `food-ingredients-dataset/2`
- Pre-trained on 1000+ ingredient images
- Regular updates and improvements by Roboflow team

### Performance & Constraints

- **Inference Time**: ~100-200ms per image (HTTP + inference)
- **Memory**: ~50MB RAM (SDK + HTTP client only)
- **Network**: Requires internet connection
- **Accuracy Target**: 85%+ confidence for common ingredients
- **Rate Limits**: Free tier: 1000 requests/month, Paid: unlimited

### Best Practices

**API Usage Pipeline**:
```python
# Standard inference in notebook cells
from inference_sdk import InferenceHTTPClient

1. Initialize client with API key
2. Send image path or bytes to API
3. Parse JSON response for predictions
4. Extract class, confidence, bbox coordinates
5. Handle low-confidence or no-detection cases
```

**Reproducibility**:
- Pin `inference-sdk>=0.9.0` in requirements.txt
- Document API key management (environment variables)
- Log API version and model ID in config

**Error Handling**:
- Validate image format before API call
- Handle network errors with retry logic
- Threshold confidence at 0.7 for reliable predictions
- Fallback: Request user confirmation if confidence < 0.7

**Monitoring Metrics**:
- API response time (p50, p95, p99)
- Confidence score distribution
- Detection rate (% of images with successful detection)
- Cost per 1000 requests

## 2. Transformer: Recipe Generation

### Model Selection

**Chosen**: GPT-2 Medium (355M parameters) fine-tuned on recipe corpus
- **Rationale**: Good balance of generation quality and speed; proven for text generation tasks
- **Size**: ~1.4GB model file
- **Inference time**: ~2-3 seconds for 5 recipes (parallel generation)

**Alternatives Considered**:
- GPT-J (6B): Too large for 16GB RAM constraint
- T5-base: Better for translation tasks, less suitable for creative generation
- BLOOM-560M: Similar performance but less recipe-specific pre-training available

### Implementation Approach

**Fine-tuning Strategy**:
- Start with GPT-2 Medium pre-trained weights
- Fine-tune on 100k+ recipe dataset
- Condition generation on: [INGREDIENT] + [CUISINE TYPE] + [DIFFICULTY]

**Input/Output Specifications**:
- Input prompt: "Ingredient: {name}, Cuisine: {type}, Difficulty: {level}, Recipe:"
- Max tokens: 512 per recipe
- Temperature: 0.8 for diversity, top_p: 0.9 for quality
- Generate 5 recipes with different cuisine types

**Training Data Sources**:
- Recipe1M+ dataset (~1M recipes with images)
- Tasty recipe dataset (~20k structured recipes)
- AllRecipes.com scraped data (with attribution)

### Performance & Constraints

- **Inference Time**: 2-3 seconds for 5 parallel recipes (batch size 5)
- **Memory**: ~3GB RAM for model + generation
- **GPU**: Highly recommended for faster generation (10x speedup)
- **Quality Target**: 85% human-rated "clear and followable"

### Best Practices

**Generation Pipeline**:
```python
# Generation cell structure
1. Load fine-tuned GPT-2 model
2. Prepare prompts with ingredient + cuisine diversity
3. Generate with temperature sampling
4. Post-process: extract steps, ingredients list
5. Validate recipe structure (has steps, ingredients, timing)
```

**Reproducibility**:
- Set generation seed for consistent outputs during testing
- Log prompt templates and generation parameters
- Version control for fine-tuned model checkpoints

**Error Handling**:
- Validate generated recipe structure (must have steps)
- Filter inappropriate/unsafe cooking instructions
- Fallback: Use template-based recipes if generation fails

**Monitoring Metrics**:
- BLEU score vs. human recipes
- Recipe diversity (unique ingredients across 5 suggestions)
- Human evaluation: clarity, creativity, feasibility

## 3. RNN: Cooking Time & Step Sequence Refinement

### Model Selection

**Chosen**: Bidirectional LSTM (2 layers, 256 hidden units)
- **Rationale**: Understands temporal dependencies in cooking steps; lightweight and fast
- **Size**: ~10M parameters, ~40MB model file
- **Inference time**: <50ms per recipe

**Alternatives Considered**:
- Transformer encoder: Overkill for sequence adjustment task
- Simple feedforward: Cannot capture step dependencies
- GRU: Similar performance but LSTM more stable for training

### Implementation Approach

**Training Strategy**:
- Train on recipe dataset with timing annotations
- Input: Recipe steps (tokenized), Output: Adjusted times and reordered steps
- Use cross-entropy loss for step reordering, MSE for time prediction

**Input/Output Specifications**:
- Input: Sequence of cooking steps (max 20 steps, 128 tokens each)
- Output: Refined step order + time estimate per step
- Features: Ingredient complexity, cooking method, step dependencies

**Training Data Sources**:
- Recipe timing dataset (extracted from Recipe1M+)
- Cooking video datasets with step timestamps (YouCook2)
- Manually annotated recipe refinements

### Performance & Constraints

- **Inference Time**: <50ms per recipe
- **Memory**: ~200MB RAM for model + inference
- **GPU**: Not required; CPU inference sufficient
- **Accuracy Target**: ±15% timing accuracy, 90% correct step order

### Best Practices

**Sequence Processing**:
```python
# LSTM refinement cell
1. Tokenize recipe steps
2. Extract cooking method features (sauté, bake, boil)
3. Feed through BiLSTM
4. Decode step reordering and timing adjustments
5. Validate: no circular dependencies in step order
```

**Reproducibility**:
- Set PyTorch LSTM dropout seed
- Document tokenization vocabulary
- Version control training data splits

**Error Handling**:
- Validate step dependencies (no step depends on future steps)
- Cap timing estimates (min 1 min, max 240 min per step)
- Fallback: Use original GPT-2 generated times if refinement fails

**Monitoring Metrics**:
- Mean Absolute Error (MAE) for timing predictions
- Step reordering accuracy (Kendall's Tau correlation)
- User feedback: "Was the timing accurate?"

## 4. Probabilistic Graphical Models: Nutrition Estimation

### Model Selection

**Chosen**: Bayesian Network with Gaussian distributions for portion/calorie estimation
- **Rationale**: Handles uncertainty in visual size estimation; incorporates prior nutritional knowledge
- **Size**: ~1MB model parameters
- **Inference time**: <10ms per recipe

**Alternatives Considered**:
- Deep neural network regressor: Less interpretable, harder to incorporate nutritional priors
- Simple lookup table: Doesn't handle size variations from photos
- Markov Random Field: Overcomplicated for this task

### Implementation Approach

**Model Structure**:
- Nodes: Ingredient type, Visual size, Portion count, Calories, Macros
- Edges: Ingredient → Calories (nutrition DB), Visual size → Portion count
- Inference: Variational inference for posterior estimation

**Input/Output Specifications**:
- Input: Ingredient ID, estimated size (grams), recipe servings
- Output: Calorie range (mean ± std), portion size, macro breakdown
- Prior: USDA nutritional database for ingredient-calorie mapping

**Training Data Sources**:
- USDA FoodData Central (nutrient database)
- Recipe1M+ with portion annotations
- Visual portion estimation studies (plate/hand references)

### Performance & Constraints

- **Inference Time**: <10ms per recipe
- **Memory**: ~50MB RAM including nutritional database
- **GPU**: Not required
- **Accuracy Target**: ±20% calorie accuracy vs. measured portions

### Best Practices

**Estimation Pipeline**:
```python
# Bayesian network inference cell
1. Load nutritional database (USDA)
2. Query base calories for ingredient
3. Adjust for estimated size (from CNN)
4. Factor in cooking method (oil, butter, etc.)
5. Output: calorie range with confidence interval
```

**Reproducibility**:
- Version control nutritional database (date snapshot)
- Document Bayesian network structure (DAG)
- Log inference parameters (convergence threshold)

**Error Handling**:
- Handle missing ingredients: use category average (e.g., "fish" avg if "salmon" missing)
- Validate output ranges (no negative calories, reasonable maximums)
- Display confidence intervals to users

**Monitoring Metrics**:
- Mean Absolute Percentage Error (MAPE) for calories
- Coverage probability of confidence intervals (should be ~80%)
- User feedback: "Was the portion size realistic?"

## 5. GAN: Line-Art Illustration Generation

### Model Selection

**Chosen**: Pix2Pix GAN (U-Net generator + PatchGAN discriminator)
- **Rationale**: Excellent for paired image-to-image translation (photo → line art)
- **Size**: ~55M parameters, ~220MB model file
- **Inference time**: ~200-300ms per illustration

**Alternatives Considered**:
- CycleGAN: Unpaired translation; less control over line-art style
- StyleGAN2: Better for faces, overkill for cooking illustrations
- Diffusion models: Too slow for real-time generation (5-10s per image)

### Implementation Approach

**Training Strategy**:
- Generator: U-Net architecture (encoder-decoder with skip connections)
- Discriminator: PatchGAN (classifies image patches as real/fake line art)
- Loss: L1 pixel loss + adversarial loss (70:30 ratio)

**Input/Output Specifications**:
- Input: Text description of cooking step → intermediate sketch (using CLIP text-to-sketch)
- Output: Clean line-art illustration (512x512 pixels)
- Style: Simple, clean lines; minimal shading; beginner-friendly

**Training Data Sources**:
- Cooking illustration dataset (scraped from recipe sites with line drawings)
- Quick, Draw! dataset (for basic shapes: knife, pan, vegetables)
- Custom paired dataset: cooking photos → manually drawn line art

### Performance & Constraints

- **Inference Time**: 200-300ms per illustration (5-10 per recipe = 2-3s total)
- **Memory**: ~1.5GB RAM for model + generation
- **GPU**: Highly recommended (10x faster than CPU)
- **Quality Target**: 80% rated "helpful for understanding step"

### Best Practices

**Generation Pipeline**:
```python
# GAN generation cell
1. Extract cooking action from step text (chop, stir, bake)
2. Retrieve or generate base illustration for action
3. Feed through Pix2Pix generator
4. Post-process: clean edges, ensure consistent line weight
5. Save as SVG for scalability
```

**Reproducibility**:
- Set GAN noise seed for deterministic generation during testing
- Version control generator/discriminator checkpoints
- Document training hyperparameters (learning rate, batch size)

**Error Handling**:
- Validate illustration quality (not blank, not too cluttered)
- Fallback: Use template illustrations for common actions if generation fails
- Human review queue for safety (no dangerous illustrations)

**Monitoring Metrics**:
- Fréchet Inception Distance (FID) for image quality
- Human evaluation: clarity, helpfulness, style consistency
- Generation failure rate (blank or corrupted images)

## Integration & Orchestration

### End-to-End Pipeline

**Execution Flow** (in `pipeline_photo_to_recipes.ipynb`):
1. **Image Upload** → CNN ingredient recognition (100ms)
2. **Parallel Recipe Generation** → Transformer generates 5 recipes (2-3s)
3. **Sequential Refinement** → RNN adjusts each recipe (5 × 50ms = 250ms)
4. **Parallel Nutrition** → PGM estimates for 5 recipes (5 × 10ms = 50ms)
5. **Parallel Illustrations** → GAN generates 5-10 per recipe (2-3s per recipe)
6. **Total Time**: ~4-8 seconds (well within 15-second requirement)

### Performance Optimization

**Parallelization Strategy**:
- Recipe generation: Batch size 5 (parallel Transformer inference)
- Illustration generation: Async GPU queue for multiple illustrations
- Nutrition estimation: Vectorized computation for all recipes

**Memory Management**:
- Load models lazily (only when needed)
- Clear GPU cache between steps
- Use model quantization for Transformer (reduce to 8-bit)

**Caching Strategy**:
- Cache common ingredient embeddings (from CNN)
- Cache nutrition DB queries
- Cache generated illustrations for common actions

## Reproducibility Checklist

✅ **Random Seeds**:
- NumPy: `np.random.seed(42)`
- PyTorch: `torch.manual_seed(42)`, `torch.cuda.manual_seed_all(42)`
- TensorFlow: `tf.random.set_seed(42)`

✅ **Version Pinning** (requirements.txt):
```
torch==2.1.0
tensorflow==2.14.0
transformers==4.35.0
opencv-python==4.8.1
pillow==10.1.0
pgmpy==0.1.23
```

✅ **Data Versioning**:
- Document dataset versions with dates
- Compute and store data checksums (MD5)
- Track train/val/test splits

✅ **Model Checkpointing**:
- Save models with timestamp and version
- Document training configuration
- Store evaluation metrics with checkpoint

## Technology Stack Summary

| Component | Technology | Version | Size | Inference Time |
|-----------|-----------|---------|------|----------------|
| CNN | EfficientNetV2-S | PyTorch 2.1 | 84MB | 50-100ms |
| Transformer | GPT-2 Medium | HF 4.35 | 1.4GB | 2-3s (batch 5) |
| RNN | BiLSTM | PyTorch 2.1 | 40MB | <50ms |
| PGM | Bayesian Network | pgmpy 0.1.23 | 1MB | <10ms |
| GAN | Pix2Pix | PyTorch 2.1 | 220MB | 200-300ms |

**Total Storage**: ~1.7GB for all models
**Total RAM**: ~6GB peak usage (within 16GB constraint)
**Total Inference**: 4-8 seconds (within 15-second requirement)

## Next Steps

1. **Phase 1**: Create data models and contracts for each component
2. **Validate**: Ensure all models fit performance and memory constraints
3. **Implement**: Build notebooks following constitution principles
4. **Test**: Execute end-to-end pipeline with papermill
