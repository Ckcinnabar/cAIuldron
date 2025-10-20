# Cooking Illustration Generation Guide

**Purpose**: Generate clean, engaging line-art illustrations for cooking steps

## Overview of Methods

| Method | Difficulty | Quality | Speed | Cost | Best For |
|--------|-----------|---------|-------|------|----------|
| 1. Stable Diffusion + ControlNet | Medium | ⭐⭐⭐⭐⭐ | Medium | Free | Best overall solution |
| 2. DALL-E 3 API | Easy | ⭐⭐⭐⭐⭐ | Fast | $$ | Quick prototyping |
| 3. Template Library | Easy | ⭐⭐⭐ | Very Fast | Free | MVP/Fallback |
| 4. Image-to-Sketch Conversion | Easy | ⭐⭐⭐ | Fast | Free | Converting existing photos |
| 5. Custom Pix2Pix GAN Training | Hard | ⭐⭐⭐⭐ | Fast | Free | Custom style needed |

---

## Method 1: Stable Diffusion + ControlNet ⭐ (RECOMMENDED)

### Why This Method?

✅ **FREE**: Open-source, runs locally
✅ **High Quality**: Consistent line-art style
✅ **Controllable**: Use sketches to guide output
✅ **No API Limits**: Generate unlimited illustrations

### Setup

```bash
pip install diffusers transformers accelerate controlnet_aux
```

### Implementation

See notebook: `notebooks/model_illustration_generation/model_stable_diffusion_lineart.ipynb`

### Prompt Engineering Tips

**Good Prompt Structure**:
```
simple line art drawing, black and white, clean lines, minimalist style,
cooking instruction: [ACTION], [DETAILS],
single object, white background, technical illustration,
no shading, outline only, beginner-friendly visual guide
```

**Negative Prompt**:
```
photo, photograph, realistic, colored, shading, gradient,
complex, cluttered, text, watermark, multiple views
```

### Example Results

- **Generation Time**: 3-5 seconds (GPU) / 20-30 seconds (CPU)
- **Quality**: Consistent, clean lines
- **Customization**: High (can control via sketches)

---

## Method 2: DALL-E 3 API

### Why This Method?

✅ **Easiest**: Just call API
✅ **Excellent Quality**: State-of-the-art results
✅ **Fast**: ~10 seconds per image
❌ **Costs Money**: ~$0.04 per image (1024x1024)

### Setup

```bash
pip install openai
```

### Implementation

```python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def generate_with_dalle(cooking_step: dict):
    """Generate illustration using DALL-E 3."""

    prompt = f"""
    Simple black and white line art illustration:
    Cooking action: {cooking_step['cooking_method']}
    Instruction: {cooking_step['instruction_text']}

    Style: Clean minimalist line drawing, white background,
    no shading, simple outlines only, cooking instruction diagram
    """

    response = client.images.generate(
        model="dall-e-3",
        prompt=prompt,
        size="1024x1024",
        quality="standard",  # or "hd" for better quality
        n=1,
    )

    image_url = response.data[0].url

    # Download and save
    import requests
    from PIL import Image
    from io import BytesIO

    img_data = requests.get(image_url).content
    img = Image.open(BytesIO(img_data))

    return img
```

### Cost Estimate

- **Per illustration**: $0.04 (1024x1024 standard quality)
- **Per recipe** (5 steps): $0.20
- **1000 recipes**: $200

**Good for**: Prototyping, small-scale deployment

---

## Method 3: Template Library (Fallback)

### Why This Method?

✅ **Instant**: No generation needed
✅ **Reliable**: Guaranteed quality
✅ **Free**: Create once, use forever
❌ **Limited**: Only ~20-50 common actions

### Create Template Library

```python
# Create templates for common cooking actions

COOKING_TEMPLATES = {
    "chop": "templates/chop_vegetables.svg",
    "dice": "templates/dice_ingredients.svg",
    "slice": "templates/slice_thinly.svg",
    "sauté": "templates/saute_pan.svg",
    "fry": "templates/frying_pan.svg",
    "bake": "templates/oven_baking.svg",
    "boil": "templates/pot_boiling.svg",
    "mix": "templates/mixing_bowl.svg",
    "stir": "templates/stirring.svg",
    "season": "templates/seasoning.svg",
    "marinate": "templates/marinating.svg",
    "grill": "templates/grill.svg",
    "steam": "templates/steaming.svg",
    "roast": "templates/roasting.svg",
    "serve": "templates/plating.svg",
}

def get_template_illustration(cooking_method: str):
    """Get pre-made template for cooking method."""
    template_path = COOKING_TEMPLATES.get(cooking_method.lower())

    if template_path and Path(template_path).exists():
        return Image.open(template_path)
    else:
        # Return generic template
        return Image.open("templates/generic_cooking.svg")
```

### Creating Templates

**Option A**: Hire illustrator on Fiverr ($5-20 per illustration)

**Option B**: Use AI to generate, then manually refine in:
- **Inkscape** (Free SVG editor)
- **Adobe Illustrator**
- **Figma**

**Option C**: Use free icon libraries:
- **The Noun Project**: https://thenounproject.com/
- **Flaticon**: https://www.flaticon.com/
- Search for "cooking steps icons"

---

## Method 4: Photo-to-Sketch Conversion

### Why This Method?

✅ **Simple**: Convert existing photos
✅ **Fast**: <1 second per image
✅ **Good for**: Real cooking photos → line art

### Implementation

```python
import cv2
import numpy as np
from PIL import Image

def photo_to_lineart(image_path: str, output_path: str):
    """
    Convert cooking photo to line-art style.
    """
    # Read image
    img = cv2.imread(image_path)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

    # Apply edge detection
    edges = cv2.Canny(gray, threshold1=50, threshold2=150)

    # Invert (black lines on white)
    lineart = cv2.bitwise_not(edges)

    # Clean up with morphological operations
    kernel = np.ones((2,2), np.uint8)
    lineart = cv2.morphologyEx(lineart, cv2.MORPH_CLOSE, kernel)

    # Save
    cv2.imwrite(output_path, lineart)

    return Image.fromarray(lineart)

# Advanced: Use AI-powered conversion
from controlnet_aux import LineartDetector

processor = LineartDetector.from_pretrained("lllyasviel/Annotators")

def advanced_photo_to_lineart(image_path: str):
    """Better quality using AI-based line detection."""
    img = Image.open(image_path)
    lineart = processor(img)
    return lineart
```

### Use Case

Convert stock cooking photos from:
- Unsplash: https://unsplash.com/s/photos/cooking
- Pexels: https://www.pexels.com/search/cooking/

---

## Method 5: Train Custom Pix2Pix GAN

### Why This Method?

✅ **Custom Style**: Train on your specific art style
✅ **Fast Inference**: Once trained, <300ms per image
❌ **Requires Data**: Need 10k+ paired images (photo → line art)
❌ **Training Time**: Several days on GPU

### Dataset Creation

**Option 1**: Manually trace photos
- Take 100+ cooking photos
- Trace in Inkscape/Illustrator
- Labor intensive but high quality

**Option 2**: Semi-automated
1. Use photo-to-sketch conversion (Method 4)
2. Manually refine in vector editor
3. Save as training pairs

**Option 3**: Synthetic data
- Generate photos with Stable Diffusion
- Convert to line-art with ControlNet
- Use as training data

### Training Code (Simplified)

```python
# Using PyTorch + pix2pix implementation
from pix2pix import Pix2PixModel

model = Pix2PixModel()

# Load paired dataset (photo, lineart)
dataset = load_paired_dataset('data/cooking_pairs/')

# Train
model.train(
    dataset,
    epochs=100,
    batch_size=4,
    learning_rate=0.0002
)

model.save('models/gan_illustration_generation.pt')
```

---

## Hybrid Approach (RECOMMENDED for Production)

Combine multiple methods for best results:

```python
def generate_illustration_smart(cooking_step: dict, output_path: str):
    """
    Smart illustration generation with fallbacks.
    """
    method = cooking_step['cooking_method']

    # 1. Try template first (instant)
    if method in COOKING_TEMPLATES:
        template = get_template_illustration(method)
        if template:
            template.save(output_path)
            return template, "template"

    # 2. Try Stable Diffusion (local, free)
    try:
        img = generate_with_stable_diffusion(cooking_step)
        quality = validate_quality(img)

        if quality['quality_score'] >= 0.7:
            img.save(output_path)
            return img, "stable_diffusion"
    except Exception as e:
        print(f"Stable Diffusion failed: {e}")

    # 3. Fallback to DALL-E (if API key available)
    if os.getenv("OPENAI_API_KEY"):
        try:
            img = generate_with_dalle(cooking_step)
            img.save(output_path)
            return img, "dalle"
        except Exception as e:
            print(f"DALL-E failed: {e}")

    # 4. Final fallback: generic template
    generic = Image.open("templates/generic_cooking.svg")
    generic.save(output_path)
    return generic, "generic_fallback"
```

---

## Performance Comparison

### Generation Speed

| Method | GPU | CPU | Notes |
|--------|-----|-----|-------|
| Template | <1ms | <1ms | Instant |
| Photo→Sketch | 100ms | 200ms | Simple OpenCV |
| Stable Diffusion | 3-5s | 20-30s | Depends on steps |
| DALL-E API | 10s | 10s | Network latency |
| Custom GAN | 200-300ms | 500ms | Once trained |

### Quality Comparison

**Stable Diffusion**: ⭐⭐⭐⭐⭐
- Most flexible and high quality
- Can be inconsistent without good prompts

**DALL-E 3**: ⭐⭐⭐⭐⭐
- Excellent quality out of box
- Very consistent

**Templates**: ⭐⭐⭐⭐
- Professional if well-designed
- Limited variety

**Photo→Sketch**: ⭐⭐⭐
- Good for realistic photos
- Can be messy for complex scenes

---

## Recommended Implementation Strategy

### Phase 1: MVP (Use Templates)
- Create 20-30 templates for common actions
- Fast, reliable, no AI needed
- Cost: $100-300 one-time (hire illustrator)

### Phase 2: Add Stable Diffusion
- Set up local Stable Diffusion + ControlNet
- Generate for less common actions
- Templates as fallback

### Phase 3: Optimize (Optional)
- Train custom GAN if need specific style
- Or use DALL-E API if budget allows

---

## Resources

### Pre-trained Models
- **Stable Diffusion 1.5**: `runwayml/stable-diffusion-v1-5`
- **ControlNet Lineart**: `lllyasviel/control_v11p_sd15_lineart`
- **ControlNet Canny**: `lllyasviel/control_v11p_sd15_canny`

### Icon/Illustration Sources
- **The Noun Project**: Cooking icons
- **Flaticon**: Free cooking illustrations
- **unDraw**: Customizable illustrations
- **Freepik**: Cooking vectors (with attribution)

### Tools
- **Inkscape**: Free SVG editor
- **GIMP**: Free image editor
- **Figma**: Design tool (free tier)
- **Adobe Illustrator**: Professional (paid)

---

## Next Steps

1. **For MVP**: Create template library first
2. **For Quality**: Set up Stable Diffusion + ControlNet
3. **For Scale**: Consider DALL-E API
4. **For Custom**: Train Pix2Pix GAN

Start with the notebook I created:
`notebooks/model_illustration_generation/model_stable_diffusion_lineart.ipynb`
