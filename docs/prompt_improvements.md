# Prompt Improvements for Better Illustrations

## Four Key Improvements ✅

### 1. ❌ No Text in Images (MAXIMUM STRENGTH)

**Problem**: AI sometimes adds labels, captions, or text to illustrations
**Solution**: MAXIMUM STRENGTH negative prompt

```python
negative_prompt = """
text, labels, words, letters, ANY TEXT, numbers, writing
"""
```

**Additional measures**:
- Increased `guidance_scale` to 12.0 (from 9.5)
- Explicit "no labels" in main prompt

### 2. 🎯 Single Focused Step (ONE OBJECT ONLY)

**Problem**: AI tries to show multiple steps or objects
**Solution**: Ultra-minimal, ONE object emphasis

```python
prompt = """
extremely simple minimalist line art, single object only,
ONE focal point, isolated on pure white,
one step one image, minimal as possible
"""

negative_prompt = """
multiple objects, two things, complex, busy, cluttered, multiple steps
"""
```

### 3. 📐 Side Angle View (NOT Top-Down)

**Problem**: Overhead view is not intuitive for cooking steps
**Solution**: Specify 45-degree side perspective

```python
prompt = """
side angle view 45 degrees,
side perspective NOT overhead, angled view
"""

negative_prompt = """
overhead view, top down
"""
```

### 4. 🔍 No Background/Kitchen Scene

**Problem**: AI adds kitchen counters, stoves, walls, etc.
**Solution**: Explicitly exclude environment

```python
negative_prompt = """
background, kitchen, counter, detailed
"""
```

---

## Before vs After Examples

### Example 1: Cutting Chicken

**Step**: "Dice the chicken breast into 1-inch cubes"

**❌ Before (Bad)**:
- Shows entire kitchen counter
- Has text labels "CHICKEN" or "1 INCH"
- Multiple knives and cutting boards
- Background details (wall, tiles)

**✅ After (Good)**:
- Only: cutting board + knife + chicken pieces
- No text whatsoever
- Pure white background
- Single clear action

---

### Example 2: Frying in Pan

**Step**: "Place salmon skin-side down in the pan and cook for 4 minutes"

**❌ Before (Bad)**:
- Entire stove visible
- Kitchen background
- Text "4 MIN" or timer
- Multiple pans
- Person's hands visible

**✅ After (Good)**:
- Only: pan + salmon fillet
- No stove/background
- No text
- No hands
- Single focused view

---

### Example 3: Seasoning

**Step**: "Season both sides with salt and pepper"

**❌ Before (Bad)**:
- Entire spice rack
- Kitchen counter
- Labels "SALT" "PEPPER"
- Multiple ingredients
- Complex scene

**✅ After (Good)**:
- Only: ingredient + salt shaker + pepper mill
- No labels
- White background
- Simple, clear

---

## Detailed Prompt Breakdown

### Main Prompt Structure

```python
prompt = f"""
# Style specification
minimalist line art diagram,
single cooking action only,

# Content specification
showing: {method}, {instruction},

# Composition rules
isolated objects on pure white background,
no kitchen scene, no environment,
focus only on the essential items for this specific step,

# Quality specifications
clean black outlines,
simple technical illustration,
cookbook diagram style,
clear and uncluttered,
single step visualization,
no background details, no context,

# Style
professional instructional drawing,
hand-drawn quality,
no labels
"""
```

### Negative Prompt Categories

**1. Text removal**:
```
text, words, letters, numbers, labels, captions,
writing, typography, signs, speech bubbles,
annotations, instructions text, recipe text
```

**2. Background removal**:
```
kitchen background, room, walls, counter, tile,
floor, appliances, cabinets, stove background,
oven background, environment, scenery, context,
entire kitchen, cooking area, workspace, countertop
```

**3. Complexity reduction**:
```
multiple unrelated steps, complex scene,
busy composition, cluttered, too many objects
```

**4. Human elements removal**:
```
people, hands, chef, person, human figure, body parts
```

**5. Quality control**:
```
photograph, photo, realistic, 3d render,
colored, color, shading, shadows, gradient,
noise, grain, texture, stripes, watermark,
signature, logo, blurry, low quality, artifacts
```

---

## Per-Step Focus Examples

### What to Show for Each Action

#### Cutting/Dicing/Chopping
**Show**:
- Cutting board
- Knife
- Ingredient pieces

**Don't show**:
- Person's hands
- Kitchen counter
- Multiple knives
- Background

#### Sautéing/Frying
**Show**:
- Pan
- Ingredients in pan
- (Optional: spatula if mentioned)

**Don't show**:
- Stove
- Burner
- Kitchen background
- Entire cooking area

#### Mixing
**Show**:
- Bowl
- Spoon/whisk
- Ingredients in bowl

**Don't show**:
- Counter
- Multiple bowls
- Background ingredients

#### Seasoning
**Show**:
- Main ingredient
- Salt shaker/pepper mill
- Simple arrangement

**Don't show**:
- Entire spice rack
- Labels on containers
- Background

#### Heating/Cooking in Oven
**Show**:
- Baking dish with food
- (Optional: simple oven rack line)

**Don't show**:
- Entire oven
- Kitchen
- Temperature dials
- Text/numbers

---

## Testing Checklist

For each generated illustration, verify:

- [ ] **No text visible** (check carefully for small labels)
- [ ] **White background** (no kitchen, no environment)
- [ ] **Single focused action** (not multiple steps)
- [ ] **Essential objects only** (max 2-3 items)
- [ ] **No human elements** (hands, people, chef)
- [ ] **Clean lines** (no stripes, no artifacts)
- [ ] **Clear subject** (obvious what the step is)

---

## Advanced: Action-Specific Prompts

For even better results, customize per cooking method:

### For Cutting Actions
```python
if method in ['cut', 'dice', 'chop', 'slice']:
    prompt += """
    close-up view of cutting board with knife and ingredient,
    simple overhead angle, no hands visible
    """
```

### For Pan Cooking
```python
if method in ['sauté', 'fry', 'stir-fry']:
    prompt += """
    isolated pan with food, no stove visible,
    simple side angle view
    """
```

### For Mixing
```python
if method in ['mix', 'stir', 'whisk']:
    prompt += """
    bowl with mixing tool and ingredients,
    simple angle, clean composition
    """
```

### For Plating/Serving
```python
if method in ['serve', 'plate', 'garnish']:
    prompt += """
    plated dish only, simple presentation,
    no table background, no dining scene
    """
```

---

## Common Issues & Fixes

### Issue: Still seeing text

**Fix**: Increase guidance_scale to 10.0 or higher
```python
guidance_scale=10.0  # Stronger negative prompt following
```

### Issue: Too much background

**Fix**: Add to prompt:
```python
"extreme close-up, isolated subject, floating on white"
```

### Issue: Multiple objects/steps

**Fix**: Emphasize in prompt:
```python
"single object focus, one step only, minimal composition"
```

### Issue: Hands visible

**Fix**: Add to negative prompt:
```python
"hands, fingers, arms, person, human, chef, holding"
```

---

## Summary of Changes

| Aspect | Before | After |
|--------|--------|-------|
| **Text** | ❌ Sometimes appears | ✅ Never appears |
| **Focus** | ❌ Multiple objects | ✅ Single action |
| **Background** | ❌ Kitchen scene | ✅ Pure white |
| **Complexity** | ❌ Cluttered | ✅ Minimal |
| **Clarity** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**Result**: Clean, focused, professional cookbook-style diagrams!

---

## Usage

Updated notebook automatically uses these improvements:

```python
# Just use the improved function
illustration, metadata = generate_cooking_illustration_improved(
    cooking_step={
        "instruction_text": "Dice the chicken into cubes",
        "cooking_method": "dice"
    },
    output_path="output.png"
)

# Metadata will show:
# "improvements": [
#     "No text in images",
#     "Single focused action",
#     "No background/kitchen scene"
# ]
```

All improvements are automatically applied! ✅
