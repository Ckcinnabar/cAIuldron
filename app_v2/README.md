# cAIuldron v2.0 🍳🤖

**Cloud-Native AI Recipe Generator** — Upload a photo of your ingredients and get 5 restaurant-quality recipes, complete with nutrition data, AI-generated dish photos, and web-inspired variations.

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Agent%20Orchestration-purple.svg)](https://github.com/langchain-ai/langgraph)
[![Groq](https://img.shields.io/badge/Groq-Vision%20%2B%2070B-orange.svg)](https://console.groq.com/)
[![ChromaDB](https://img.shields.io/badge/ChromaDB-7%2C913%20Recipes-green.svg)](https://www.trychroma.com/)
[![LangSmith](https://img.shields.io/badge/LangSmith-Observability-blue.svg)](https://smith.langchain.com/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen.svg)]()

---

## 📸 Overview

cAIuldron v2.0 is a **cloud-native, multi-agent AI recipe system** that transforms a single ingredient photo into five complete, restaurant-quality recipes in under 60 seconds. The entire pipeline is orchestrated by a **LangGraph StateGraph** — a directed graph of specialized AI nodes that run in sequence and in parallel, passing structured state between stages.

Upload one photo. The system will:

- 🔍 **Detect ingredients** using Groq Llama 4 Scout Vision (~1s, no GPU needed)
- 🥗 **Estimate nutrition** from a 525-ingredient USDA FoodData Central database
- 📚 **Retrieve similar recipes** via ChromaDB RAG over 7,913 indexed recipes
- 🌐 **Search the web** for live recipe inspiration via Tavily API
- 🍳 **Generate 5 complete recipes** with Groq Llama 3.3 70B (≥10 ingredients, ≥8 steps each)
- 🖼️ **Render dish photos** using FLUX.1-schnell via HF Inference API (5 concurrent)
- 📊 **Log full pipeline trace** to LangSmith with per-node timing

**100% Free APIs · Zero Local GPU · No Model Downloads · Runs in Jupyter**

---

## ✨ Key Features

### 🎯 Core Capabilities

- **Groq Vision Ingredient Detection**: Sends photo as base64 JPEG to Llama 4 Scout 17B — returns a parsed JSON ingredient list in ~1s with no local compute
- **7,913-Recipe RAG Knowledge Base**: ChromaDB with cosine similarity search (sentence-transformers `all-MiniLM-L6-v2`); retrieves the 3 most similar recipes to inject as generation context
- **Live Web Inspiration**: Tavily API searches for real-world recipe ideas; top results are merged into the LLM prompt alongside RAG context
- **Michelin-Quality Recipe Generation**: Single Groq Llama 3.3 70B call produces all 5 recipes in structured JSON — each with ≥10 precisely measured ingredients and ≥8 professional cooking steps including temperatures, timing cues, and plating tips
- **Parallel Agent Execution**: RAG search and web search run simultaneously via LangGraph's fork/join — both results are available before recipe generation begins
- **Concurrent Dish Image Generation**: 5 FLUX.1-schnell images are generated in a single async batch — professional food photography prompts, 512×512, ~30–60s on HF free tier
- **USDA Nutrition Lookup**: Per-100g macros (calories, protein, fat, carbs) for each detected ingredient, with fuzzy matching fallback
- **Full Pipeline Observability**: LangSmith traces every LangGraph node — view timing, inputs, and outputs for each stage in the LangSmith dashboard

### 📊 Technical Highlights

- **LangGraph StateGraph**: Typed `RecipeState` TypedDict with `Annotated` reducers for safe parallel state writes
- **Modular Notebook Architecture**: 5 notebooks (`%run` chained), each independently testable
- **ChromaDB Persistent Index**: Built once (~2–3 min), reloaded in ~1s on all subsequent runs
- **Async Image Generation**: `httpx` + `asyncio.gather` inside Jupyter via `nest_asyncio`
- **Zero VRAM**: All AI compute delegated to cloud APIs — local machine only runs ChromaDB and Gradio

---

## 🆕 Upgrade from v1

| Capability | v1 — Local Models | v2 — Cloud-Native |
|---|---|---|
| **Ingredient detection** | CLIP ViT-Base + DETR ResNet-50 (2–3 GB VRAM) | Groq Llama 4 Scout Vision API (~1s, 0 VRAM) |
| **Recipe generation** | GPT-2 / Llama 3.2 1B LoRA / Llama 3.1 8B GGUF | Groq Llama 3.3 70B (single API call, all 5 recipes) |
| **Recipe knowledge base** | None | ChromaDB RAG — 7,913 recipes, cosine search |
| **Web inspiration** | None | Tavily API — live web results injected into LLM context |
| **Dish image generation** | None | HF FLUX.1-schnell — 5 concurrent photos per run |
| **Agent orchestration** | Sequential Python calls | LangGraph StateGraph with parallel fork/join |
| **Observability** | None | LangSmith — per-node timing, full trace dashboard |
| **GPU requirement** | 4–6 GB VRAM minimum | Zero — no local AI model |
| **Model download size** | 10+ GB (GPT-2 + Llama weights) | None |
| **Cold start time** | 30–90s (model loading into VRAM) | ~2s (ChromaDB index load only) |
| **UI tabs** | 2 (ingredients, recipes) | 6 (ingredients, nutrition, RAG, recipes, gallery, stats) |

---

## 🚀 Quick Start

### Prerequisites

```
Python 3.11+
conda (Anaconda or Miniconda)
~500 MB disk (ChromaDB index + embedding model, auto-downloaded on first run)
```

No GPU. No local AI model downloads.

### Installation

```bash
# 1. Navigate to project root
cd "D:\Github\UF PROJECT\cAIuldron"

# 2. Create and activate conda environment
conda create -n caiuldron-v2 python=3.11 -y
conda activate caiuldron-v2

# 3. Install dependencies
pip install -r app_v2/requirements.txt
```

### API Keys (all free)

| Service | Sign-up URL | Used for |
|---|---|---|
| Groq | [console.groq.com](https://console.groq.com) | Vision + recipe generation |
| Tavily | [app.tavily.com](https://app.tavily.com) | Web recipe search |
| HuggingFace | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) | FLUX.1 image generation |
| LangSmith | [smith.langchain.com](https://smith.langchain.com) | Pipeline observability *(optional)* |

```bash
# 4. Set up environment file (Windows)
copy app_v2\.env.example app_v2\.env
# Then edit app_v2/.env with your keys
```

```env
GROQ_API_KEY=gsk_...
TAVILY_API_KEY=tvly-...
HF_TOKEN=hf_...
LANGCHAIN_API_KEY=ls__...        # optional
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=cAIuldron-v2
```

### Running the App

**Step 1 — Build ChromaDB index (first run only, ~2–3 minutes)**

Open `app_v2/3_rag.ipynb` in VS Code or Jupyter and run all cells.
The index persists to `app_v2/.chroma_db/` and reloads in ~1s on all future runs.

**Step 2 — Launch**

```bash
# In VS Code: open app_v2/app.ipynb → Run All Cells
# In Jupyter:
jupyter notebook app_v2/app.ipynb
```

Gradio launches at `http://127.0.0.1:7860`

For a public shareable URL (valid 72h):
```python
# Last cell of app.ipynb — change share=False to:
demo.launch(server_name='127.0.0.1', server_port=7860, share=True, inbrowser=True)
```

**Test individual modules**

```python
# In any notebook, run sub-modules independently:
%run app_v2/1_config.ipynb      # loads all constants and API keys
%run app_v2/2_tools.ipynb       # loads tool functions
%run app_v2/3_rag.ipynb         # loads/builds ChromaDB index
%run app_v2/4_agent.ipynb       # compiles LangGraph recipe_graph
```

---

## 🏗️ System Architecture

### LangGraph Agent Pipeline

```
START
  │
  ▼
detect_ingredients_node          ← Groq Llama 4 Scout 17B (Vision API)
  │   Encodes image as base64 JPEG · Returns JSON ingredient list (max 8)
  │
  ▼
estimate_nutrition_node          ← USDA FoodData Central (local JSON, 525 ingredients)
  │   Exact → case-insensitive → substring fuzzy match · Per-100g macros
  │
  ├─────────────────────────────────┐   [PARALLEL FORK]
  ▼                                 ▼
rag_search_node                  web_search_node
  ChromaDB cosine similarity       Tavily API (basic depth, 5 results)
  query: "Main ingredients: ..."   query: "{ingredients} quick easy dinner"
  top-3 recipes, similarity ≥0.25  live web recipe inspiration
  │                                 │
  └──────────────┬──────────────────┘   [PARALLEL JOIN]
                 │   Annotated reducers merge state safely
                 ▼
        generate_recipes_node    ← Groq Llama 3.3 70B Versatile
          │   Injected context: RAG recipes + Tavily results + nutrition summary
          │   Single API call → 5 recipes in JSON (≥10 ingredients, ≥8 steps each)
          │   Cuisine rotation: Asian · Mediterranean · American · Mexican · Italian
          │
          ▼
        generate_images_node     ← HF Inference API (FLUX.1-schnell)
          │   5 async concurrent HTTP calls via httpx + asyncio.gather
          │   Professional food photography prompts · 512×512 PNG
          │
          ▼
         END  →  Gradio UI (6 tabs)
```

### AI Model Ecosystem

| Component | Model | API | Speed | VRAM |
|---|---|---|---|---|
| **Ingredient Detection** | Llama 4 Scout 17B (Vision) | Groq | ~1–2s | 0 GB |
| **Recipe Generation** | Llama 3.3 70B Versatile | Groq | ~5–15s | 0 GB |
| **Recipe Embedding** | all-MiniLM-L6-v2 | Local / CPU | index once | 0 GB |
| **Dish Image Generation** | FLUX.1-schnell | HF Inference API | ~30–60s (×5) | 0 GB |
| **Nutrition Lookup** | USDA JSON database | Local | < 0.1s | — |

**Total processing time**: 40–80 seconds (dominated by FLUX.1 image generation on HF free tier)

---

## 📁 Project Structure

```
cAIuldron/
│
├── app_v2/                              ← 🌟 v2 Application (this version)
│   ├── 1_config.ipynb                  # API keys, model IDs, path constants, cuisine rotation
│   ├── 2_tools.ipynb                   # Vision / Nutrition / WebSearch / ImageGen tools
│   ├── 3_rag.ipynb                     # ChromaDB index build + cosine similarity query
│   ├── 4_agent.ipynb                   # RecipeState TypedDict · LangGraph nodes · graph compile
│   ├── app.ipynb                       # Gradio 6-tab UI · %run chain · launch
│   ├── .chroma_db/                     # Persistent ChromaDB index (auto-created, ~50 MB)
│   ├── requirements.txt                # Python dependencies
│   ├── .env                            # API keys (not committed to git)
│   └── .env.example                    # Key template
│
├── app_v1/                              ← v1 Application (local model version, archived)
│   ├── README.md                       # v1 documentation
│   ├── CAIuldron Poster.pdf
│   └── FINAL REPORT.pdf
│
├── data/                                ← Shared data (used by both versions)
│   ├── nutrition_lookup_full.json       # USDA macros · 525 ingredients (runtime lookup)
│   ├── ingredients_nutrition_full.csv   # Full USDA raw export
│   ├── ingredients_vocabulary.csv       # 528-ingredient vocabulary
│   ├── processed/
│   │   └── recipes/
│   │       └── full_recipes.json        # 7,913 recipes (ChromaDB source)
│   └── test_images/                     # Sample ingredient photos
│       ├── test.jpeg
│       ├── original.jpg
│       └── eyecatch-4569.jpg
│
└── reports/                             ← Academic deliverables
    ├── Deliverable1_Technical_Blueprint_Full.pdf
    ├── Deliverable2_Technical_Blueprint_Full.pdf
    └── Deliverable3_Technical_Blueprint_Full.pdf
```

---

## 🎯 Module Descriptions

### 1️⃣ Config (`1_config.ipynb`)

**Purpose**: Single source of truth for all constants, paths, and environment variables.

**Features**:
- Loads `.env` from `app_v2/` (supports running from project root or `app_v2/`)
- Resolves `PROJECT_ROOT`, `DATA_DIR`, `RECIPES_JSON`, `NUTRITION_JSON`, `CHROMA_DIR`
- Validates that all required API keys are present
- Defines model IDs, RAG parameters, image generation timeout, cuisine rotation

**Key constants exported**:
```python
GROQ_VISION_MODEL   = 'meta-llama/llama-4-scout-17b-16e-instruct'
GROQ_TEXT_MODEL     = 'llama-3.3-70b-versatile'
EMBEDDING_MODEL     = 'sentence-transformers/all-MiniLM-L6-v2'
IMAGE_GEN_MODEL     = 'black-forest-labs/FLUX.1-schnell'
HF_INFERENCE_URL    = 'https://router.huggingface.co/hf-inference/models/...'
NUM_RECIPES         = 5
GROQ_TEMPERATURE    = 0.85
RAG_TOP_K           = 3
RAG_MIN_SIMILARITY  = 0.25
TAVILY_MAX_RESULTS  = 5
IMAGE_GEN_TIMEOUT   = 90    # seconds
CUISINE_ROTATION    = [Asian, Mediterranean, American, Mexican, Italian]
```

---

### 2️⃣ Tools (`2_tools.ipynb`)

**Purpose**: Four standalone tool functions used by LangGraph agent nodes.

**Tool 1 — Vision** (`detect_ingredients_from_image(image_bytes) → (List[str], str)`):
- Encodes image as base64 JPEG, sends to Groq Vision API
- System prompt: culinary expert role, JSON-only output, max 8 ingredients
- Parses `[...]` from response with regex fallback for robustness
- Returns `(ingredient_list, raw_response_string)`

**Tool 2 — Nutrition** (`build_nutrition_summary`, `lookup_nutrition`):
- Loads `nutrition_lookup_full.json` (525 USDA ingredients) once at import
- `lookup_nutrition(ingredient)`: exact → case-insensitive → substring fuzzy match
- `build_nutrition_summary(ingredients)`: builds per-ingredient macro dict + markdown table
- Returns `(per_ingredient_dict, markdown_summary_string)`

**Tool 3 — Web Search** (`search_recipe_inspiration(ingredients) → (List[Dict], str)`):
- Query: `"recipes with {top 4 ingredients} quick easy dinner"`
- `search_depth='basic'`, returns up to 5 results including Tavily answer summary
- Formats results into a context block injected into the recipe generation prompt

**Tool 4 — Image Generation** (`generate_images_sync`, `generate_images_async`):
- Async concurrent HTTP POST to HF Inference Router API (FLUX.1-schnell)
- 5 image calls dispatched simultaneously via `asyncio.gather`
- `nest_asyncio.apply()` resolves Jupyter event loop conflict automatically
- Prompt template: `"professional food photography, {title}, {cuisine} cuisine, beautifully plated on white background, soft natural lighting, shallow depth of field, appetizing"`
- Returns `List[Optional[bytes]]` — `None` for any failed image

---

### 3️⃣ RAG (`3_rag.ipynb`)

**Purpose**: Build and query a ChromaDB vector store over 7,913 recipes.

**Index build** (`build_or_load_index(force_rebuild=False)`):
- Checks if `col.count() >= 7000` → loads existing index in ~1s (subsequent runs)
- First run: loads `full_recipes.json`, batches 500 recipes per ChromaDB `add()` call
- Embedding format per recipe:
  ```
  "Recipe: {title}. Cuisine: {cuisine}. Main ingredients: {tags}."
  ```
  Instructions are excluded from embeddings to keep semantic similarity focused on ingredient matching
- Metadata stored per document: title, cuisine, difficulty, cooking_time_minutes, servings, ingredient_tags, instructions_preview (first 3 steps, ≤500 chars)
- Distance metric: cosine (`hnsw:space=cosine`)
- Embedding model: `all-MiniLM-L6-v2` (384-dimensional, runs on CPU)

**Query** (`retrieve_similar_recipes(ingredients) → (List[Dict], str)`):
- Query format mirrors index format: `"Main ingredients: {detected_ingredients}."`
- Returns top-3 results with `similarity = 1 - cosine_distance`
- Filters out results with `similarity < RAG_MIN_SIMILARITY (0.25)`
- Builds a formatted context block for LLM injection

**Exports**:
- `_rag_collection` (ChromaDB Collection)
- `retrieve_similar_recipes()` (function)

---

### 4️⃣ Agent (`4_agent.ipynb`)

**Purpose**: Define `RecipeState`, implement all 5 LangGraph nodes, and compile the `StateGraph`.

**RecipeState TypedDict fields**:

| Field | Type | Stage |
|---|---|---|
| `image_bytes` | `bytes` | Input |
| `detected_ingredients` | `List[str]` | Stage 1 |
| `ingredient_detection_raw` | `str` | Stage 1 |
| `nutrition_per_ingredient` | `Dict` | Stage 2 |
| `nutrition_summary` | `str` | Stage 2 |
| `rag_retrieved_recipes` | `List[Dict]` | Stage 3a (parallel) |
| `rag_context_block` | `str` | Stage 3a (parallel) |
| `web_search_results` | `List[Dict]` | Stage 3b (parallel) |
| `web_context_block` | `str` | Stage 3b (parallel) |
| `recipes` | `List[Dict]` | Stage 4 + 5 |
| `errors` | `Annotated[List[str], operator.add]` | All stages |
| `processing_time_seconds` | `Annotated[Dict[str, float], merge_fn]` | All stages |

**Recipe schema** (generated per recipe):
```python
{
  "title": str,
  "cuisine": str,
  "difficulty": "easy" | "medium" | "hard",
  "prep_time_minutes": int,
  "cook_time_minutes": int,
  "total_time_minutes": int,
  "servings": int,
  "ingredients": List[str],   # ≥10 items with precise measurements
  "instructions": List[str],  # ≥8 detailed steps
  "image_bytes": Optional[bytes],   # filled by generate_images_node
  "rag_source_titles": List[str],   # source recipe titles from RAG
}
```

**Recipe generation prompt design** (Groq Llama 3.3 70B):
- **System**: Michelin-star chef persona · strict JSON schema enforcement · minimum 10 ingredients + 8 steps · include at least one flavour-building step (deglazing, blooming spices, sauce building)
- **User context**:
  1. RAG top-3 retrieved recipes (~400 tokens)
  2. Tavily web search results (~300 tokens)
  3. USDA nutrition summary (~100 tokens)
  4. Cuisine + difficulty target for each of the 5 recipes
- Single API call returns all 5 recipes as `{"recipes": [...]}` JSON

**Exports**:
- `RecipeState` (TypedDict)
- `recipe_graph` (compiled LangGraph)

---

### 5️⃣ App (`app.ipynb`)

**Purpose**: Chain all modules via `%run`, implement the Gradio pipeline function, and launch the UI.

**`process_image(image, progress)` function**:
1. Convert PIL Image → JPEG bytes (quality=85)
2. Build initial `RecipeState` with all fields set to empty defaults
3. `recipe_graph.invoke(initial_state)` — runs full LangGraph pipeline
4. Extract `detected_ingredients`, `nutrition_summary`, `rag_retrieved_recipes`, `recipes`, `processing_time_seconds` from final state
5. Format 6 markdown strings + gallery image list → return to Gradio

**Gradio UI — 6 Tabs**:

| Tab | Content |
|---|---|
| 🔍 Ingredients | Detected ingredient list + any pipeline warnings |
| 🥗 Nutrition | Per-ingredient USDA macro table (calories, protein, fat, carbs per 100g) |
| 📚 Similar Recipes (RAG) | Top-3 ChromaDB matches with cuisine, difficulty, cook time, similarity score |
| 🍳 Generated Recipes | 5 complete recipes — all fields, numbered ingredients and steps |
| 🖼️ Dish Images | `gr.Gallery` — up to 5 FLUX.1 photos, 3-column grid |
| 📊 Pipeline Stats | Per-node seconds · total time · LangSmith trace link (when enabled) |

---

## 📊 Performance

### Pipeline Timing

| Stage | Model | Typical Time |
|---|---|---|
| Ingredient detection | Groq Llama 4 Scout Vision | 1–2s |
| Nutrition lookup | USDA local JSON | < 0.1s |
| RAG search (parallel) | ChromaDB cosine | 0.5–1s |
| Web search (parallel) | Tavily API | 1–3s |
| Recipe generation | Groq Llama 3.3 70B | 5–15s |
| Image generation (×5) | FLUX.1-schnell (HF free) | 30–60s |
| **Total** | | **~40–80s** |

### API Usage per Run (Free Tier)

| Service | Free Limit | Calls per Run |
|---|---|---|
| Groq | ~14,400 req/day | 2 (1 vision + 1 text) |
| Tavily | 1,000 searches/month | 1 |
| HF Inference API | Monthly credit allocation | 5 (images) |
| LangSmith | 5,000 traces/month | 1 trace, 6 spans |

---

## 🔧 Configuration

### Changing the number of recipes

In `1_config.ipynb`:
```python
NUM_RECIPES = 3   # or 5 (default)
```

### Adjusting RAG similarity threshold

```python
RAG_MIN_SIMILARITY = 0.20   # lower = more results (default: 0.25)
RAG_TOP_K          = 5      # more context for the LLM (default: 3)
```

### Disabling image generation (faster runs)

In `4_agent.ipynb`, remove the `generate_images` node and edge from the graph:
```python
# Comment out these lines in build_recipe_graph():
# graph.add_node('generate_images', generate_images_node)
# graph.add_edge('generate_recipes', 'generate_images')
# graph.add_edge('generate_images', END)

# Add direct edge instead:
graph.add_edge('generate_recipes', END)
```

### Enabling LangSmith tracing

```env
# in app_v2/.env
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=ls__...
LANGCHAIN_PROJECT=cAIuldron-v2
```

Once enabled, a LangSmith trace link appears in the **📊 Pipeline Stats** tab after each run.

### Force-rebuild ChromaDB index

```python
# In 3_rag.ipynb, run:
_rag_collection = build_or_load_index(force_rebuild=True)
```

---

## 🧪 Testing

### Test individual tools

```python
# Test Groq Vision (requires image bytes)
%run app_v2/1_config.ipynb
%run app_v2/2_tools.ipynb

with open('data/test_images/test.jpeg', 'rb') as f:
    img_bytes = f.read()
ingredients, raw = detect_ingredients_from_image(img_bytes)
print(f'Detected: {ingredients}')

# Test nutrition lookup
per_ing, summary = build_nutrition_summary(['chicken breast', 'garlic', 'lemon'])
print(summary)

# Test web search
results, context = search_recipe_inspiration(['chicken breast', 'garlic', 'lemon'])
print(context)
```

### Test RAG retrieval

```python
%run app_v2/1_config.ipynb
%run app_v2/3_rag.ipynb

recipes, context = retrieve_similar_recipes(['chicken', 'garlic', 'lemon'])
for r in recipes:
    print(f"{r['title']} — similarity: {r['similarity_score']}")
```

### End-to-end test

```python
# Run app.ipynb → Run All Cells
# Upload data/test_images/test.jpeg
# Verify all 6 tabs populate within 80s
```

### Expected output format (one recipe)

```
## 1. Lemon Garlic Roasted Chicken

**Cuisine**: Mediterranean | **Difficulty**: Easy
**Prep**: 15 min | **Cook**: 40 min | **Total**: 55 min | **Serves**: 4

### Ingredients
- 4 chicken thighs, bone-in, skin-on (about 1.2 kg)
- 3 tbsp extra-virgin olive oil
- 4 cloves garlic, minced
- 2 lemons, zested and juiced
...

### Instructions
1. Preheat oven to 200°C (400°F). Pat chicken dry with paper towels...
2. In a small bowl, whisk together olive oil, garlic, lemon zest...
...
```

---

## 🐛 Troubleshooting

### Groq model permission denied (403)

Llama 4 Scout requires explicit activation. Visit [console.groq.com/settings/limits](https://console.groq.com/settings/limits) and enable the model under your account settings.

### NameError: GROQ_API_KEY not defined

You ran `2_tools.ipynb` directly instead of `app.ipynb`. Always start from `app.ipynb` — it chains all notebooks via `%run`:
```python
%run 1_config.ipynb    # must run first to define all constants
%run 2_tools.ipynb
%run 3_rag.ipynb
%run 4_agent.ipynb
```

### HF image generation credits exhausted (402)

Monthly free credits used up. Options:
- **HF PRO** ($9/month) — renewed monthly credit allocation
- **Pollinations.ai** — completely free, no API key, drop-in replacement for FLUX.1

### ChromaDB returns 0 results

Collection missing or count < 7,000. Force a rebuild:
```python
_rag_collection = build_or_load_index(force_rebuild=True)
```
Ensure `data/processed/recipes/full_recipes.json` exists (7,913 recipes).

### LangGraph InvalidUpdateError (parallel nodes)

Both parallel nodes write to the same state key simultaneously. Check that `RecipeState` uses `Annotated` reducers for any field written by multiple nodes:
```python
errors:                  Annotated[List[str], operator.add]
processing_time_seconds: Annotated[Dict[str, float], lambda a, b: {**a, **b}]
```

### Async event loop error in Jupyter

`nest_asyncio` must be installed and applied before async image generation:
```bash
pip install nest_asyncio
```
`2_tools.ipynb` applies it automatically via `nest_asyncio.apply()`.

### Gradio gallery shows grey empty boxes

CSS selector targets empty slots:
```css
.gallery button:not(:has(img)) { display: none !important; }
```
This is included in `app.ipynb`'s `_CSS` block. If boxes still appear, the images failed to generate (check the 🔍 Ingredients tab for pipeline errors).

---

## 🎓 Key Technologies

### AI Models

| Model | Provider | Role |
|---|---|---|
| Llama 4 Scout 17B (Vision) | Meta / Groq API | Ingredient detection from photos |
| Llama 3.3 70B Versatile | Meta / Groq API | Recipe generation (5 recipes in one call) |
| all-MiniLM-L6-v2 | Sentence Transformers | Recipe embedding for ChromaDB (local, CPU) |
| FLUX.1-schnell | Black Forest Labs / HF API | AI dish photo generation |

### Datasets

- **USDA FoodData Central** — 525-ingredient nutrition database (`data/nutrition_lookup_full.json`)
- **RecipeNLG** — 7,913 curated recipes indexed in ChromaDB (`data/processed/recipes/full_recipes.json`)
- **Custom Vocabulary** — 528-ingredient vocabulary (`data/ingredients_vocabulary.csv`)

### Frameworks & Libraries

- **LangGraph** — StateGraph agent orchestration, parallel node execution, typed state
- **LangChain / LangChain-Groq** — `ChatGroq` LLM wrapper with automatic LangSmith integration
- **ChromaDB** — local persistent vector database with HNSW cosine index
- **Sentence Transformers** — `all-MiniLM-L6-v2` embedding model
- **Groq Python SDK** — Vision API calls (raw `groq.Groq`)
- **Tavily Python** — web search client
- **httpx** — async HTTP for concurrent image generation
- **nest_asyncio** — Jupyter async event loop compatibility
- **Gradio** — 6-tab web interface with gallery, progress tracking
- **python-dotenv** — `.env` file loading

---

## 🗺️ Development Status

### ✅ Completed Features

- [x] Groq Vision ingredient detection (Llama 4 Scout 17B)
- [x] LangGraph StateGraph with parallel RAG + web search fork/join
- [x] ChromaDB RAG over 7,913 recipes (cosine similarity)
- [x] Tavily web search integration
- [x] Groq Llama 3.3 70B recipe generation (5 recipes, structured JSON)
- [x] Michelin-quality recipe prompt (≥10 ingredients, ≥8 steps, flavour-building steps)
- [x] HF FLUX.1-schnell dish image generation (5 concurrent async calls)
- [x] USDA nutrition lookup (525 ingredients, fuzzy match)
- [x] LangSmith pipeline observability (automatic via LangChain)
- [x] Gradio 6-tab UI (ingredients, nutrition, RAG, recipes, gallery, stats)
- [x] Modular notebook architecture (`%run` chained)

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Follow modular architecture — each notebook has a single responsibility
4. Test modules independently before integrating into `app.ipynb`
5. Commit changes (`git commit -m 'Add amazing feature'`)
6. Open a Pull Request

---

## 📝 License

MIT License

---

## 🙏 Acknowledgments

### AI & APIs
- **Groq** — Ultra-fast LLM inference (Vision + 70B)
- **Meta AI** — Llama 4 Scout and Llama 3.3 model weights
- **Black Forest Labs** — FLUX.1-schnell image generation
- **LangChain / LangGraph** — Agent orchestration framework
- **Tavily** — Web search API

### Data
- **USDA** — FoodData Central nutrition database
- **RecipeNLG** — Open recipe dataset (2.23M recipes, 7,913 curated for this project)

### Tools
- **ChromaDB**, **Sentence Transformers**, **Gradio**, **httpx**, **Jupyter**
