# cAIuldron v2.0 🍳🤖

**Cloud-Native AI Recipe Generator** — Upload a photo of your ingredients and get 5 restaurant-quality recipes, complete with nutrition info, dish images, and web-inspired variations.

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Agent%20Orchestration-purple.svg)](https://github.com/langchain-ai/langgraph)
[![Groq](https://img.shields.io/badge/Groq-Vision%20%2B%2070B-orange.svg)](https://console.groq.com/)
[![ChromaDB](https://img.shields.io/badge/ChromaDB-7%2C913%20Recipes-green.svg)](https://www.trychroma.com/)
[![LangSmith](https://img.shields.io/badge/LangSmith-Observability-blue.svg)](https://smith.langchain.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

cAIuldron v2.0 is a complete rebuild of the v1 AI recipe system into a **cloud-native, agent-based architecture**. The core pipeline is orchestrated by a **LangGraph StateGraph** that coordinates six specialized AI stages — from visual ingredient detection through dish image generation — using only free-tier cloud APIs. No local GPU required.

Upload one ingredient photo. In under 60 seconds you receive:

- Detected ingredients (Groq Llama Vision)
- Per-ingredient nutrition data (USDA database, 525 ingredients)
- Top 3 similar recipes retrieved from 7,913-recipe ChromaDB (RAG)
- 5 complete, cuisine-diverse recipes with ≥10 ingredients and ≥8 cooking steps
- 5 AI-generated dish photos (FLUX.1-schnell via HF Inference API)
- Full pipeline timing breakdown + LangSmith trace link

---

## What Changed from v1

| Capability | v1 — Local Models | v2 — Cloud-Native |
|---|---|---|
| **Ingredient detection** | CLIP ViT-Base + DETR ResNet-50 (local, 2–3 GB VRAM) | Groq Llama 4 Scout 17B Vision API (~1s, no VRAM) |
| **Recipe generation** | GPT-2 / Llama 3.2 1B LoRA / Llama 3.1 8B GGUF | Groq Llama 3.3 70B Versatile (single API call, 5 recipes) |
| **Recipe knowledge base** | None | ChromaDB RAG — 7,913 recipes, cosine similarity search |
| **Web inspiration** | None | Tavily API — live web search injected into generation context |
| **Dish image generation** | None | HF Inference API FLUX.1-schnell — 5 concurrent photos |
| **Agent orchestration** | Sequential Python function calls | LangGraph StateGraph with parallel fork/join |
| **Observability** | None | LangSmith — per-node timing, full trace dashboard |
| **GPU requirement** | Required (4–6 GB VRAM minimum) | Zero — all compute on cloud APIs |
| **Model downloads** | 10+ GB (GPT-2 + Llama weights) | None |
| **Startup time** | 30–90s (model loading) | ~2s (index load only) |
| **Recipe quality** | 60–95/100 depending on model | Michelin-chef system prompt, structured JSON output |
| **Output tabs** | 2 (ingredients, recipes) | 6 (ingredients, nutrition, RAG, recipes, gallery, stats) |
| **Architecture** | 5 notebooks, `%run` chained | 5 notebooks, `%run` chained + LangGraph agent |

---

## Agent Architecture

```
START
  │
  ▼
detect_ingredients_node          ← Groq Llama 4 Scout 17B Vision
  │   Sends photo as base64; extracts JSON ingredient list
  │
  ▼
estimate_nutrition_node          ← USDA FoodData Central (local JSON, 525 ingredients)
  │   Per-100g macros: calories, protein, fat, carbs
  │
  ├──────────────────────────────┐
  ▼                              ▼
rag_search_node [parallel]     web_search_node [parallel]
  │  ChromaDB cosine query        │  Tavily API, 5 results
  │  top-3 similar recipes        │  live web recipe inspiration
  └──────────────┬───────────────┘
                 │  both results merged into state
                 ▼
        generate_recipes_node    ← Groq Llama 3.3 70B Versatile
          │   Context: RAG recipes + Tavily web results + nutrition summary
          │   Output: 5 complete recipes (JSON), ≥10 ingredients, ≥8 steps each
          │
          ▼
        generate_images_node     ← HF Inference API (FLUX.1-schnell)
          │   5 concurrent async HTTP calls, professional food photography prompts
          │
          ▼
         END
```

**Parallel execution**: `rag_search_node` and `web_search_node` run simultaneously. LangGraph's `Annotated` reducers safely merge their outputs before recipe generation begins.

**State type**: `RecipeState` is a `TypedDict` with typed `Annotated` fields for safe parallel writes:

```python
errors:                  Annotated[List[str], operator.add]
processing_time_seconds: Annotated[Dict[str, float], lambda a, b: {**a, **b}]
```

---

## Tech Stack

| Component | Model / Library | Notes |
|---|---|---|
| **Ingredient vision** | Groq Llama 4 Scout 17B | `meta-llama/llama-4-scout-17b-16e-instruct` |
| **Recipe generation** | Groq Llama 3.3 70B Versatile | `llama-3.3-70b-versatile`, temp 0.85 |
| **Embedding model** | `sentence-transformers/all-MiniLM-L6-v2` | Local, CPU-only, for ChromaDB indexing |
| **Vector store** | ChromaDB (persistent) | `hnsw:space=cosine`, 7,913 recipes |
| **Web search** | Tavily API | `search_depth=basic`, top-5 results |
| **Image generation** | FLUX.1-schnell | HF Inference Router API, 5 concurrent calls |
| **Agent orchestration** | LangGraph StateGraph | Parallel fork/join, typed state |
| **LLM framework** | LangChain + LangChain-Groq | `ChatGroq` for automatic LangSmith tracing |
| **Observability** | LangSmith | Automatic via `LANGCHAIN_TRACING_V2=true` |
| **UI** | Gradio 4.x | 6-tab layout, `gr.Gallery`, `show_progress='minimal'` |
| **Async HTTP** | httpx + nest_asyncio | Concurrent image generation inside Jupyter |

All AI compute runs on **free-tier cloud APIs**. No local GPU, no model downloads.

---

## Data Sources

### Nutrition Database — USDA FoodData Central

- **File**: `data/nutrition_lookup_full.json` (525 ingredients)
- **Content**: Per-100g macros — calories, protein (g), fat (g), carbohydrates (g)
- **Matching strategy**: Exact match → case-insensitive → substring fuzzy match
- **Coverage**: `data/ingredients_nutrition_full.csv` (full raw data, 525 rows)

```
data/
├── nutrition_lookup_full.json          ← runtime lookup (525 ingredients, keyed by name)
├── ingredients_nutrition_full.csv      ← raw USDA export
└── ingredients_vocabulary.csv          ← 528-ingredient vocabulary (also used in v1)
```

### Recipe Dataset — RecipeNLG

- **Indexed**: 7,913 recipes in ChromaDB (`app_v2/.chroma_db/`)
- **Source file**: `data/processed/recipes/full_recipes.json`
- **Fields per recipe**: `recipe_title`, `cuisine`, `difficulty`, `cooking_time_minutes`, `servings`, `ingredient_tags`, `instructions`
- **Embedding format**: `"Recipe: {title}. Cuisine: {cuisine}. Main ingredients: {tags}."` — title + cuisine + tags only, not full instructions, to keep semantic similarity focused
- **Index metadata stored**: title, cuisine, difficulty, cooking_time_minutes, servings, ingredient_tags, instructions_preview (first 3 steps, ≤500 chars)

### Test Images

```
data/test_images/
├── test.jpeg
├── original.jpg
└── eyecatch-4569.jpg
```

---

## Gradio UI — 6 Tabs

| Tab | What It Shows |
|---|---|
| **🔍 Ingredients** | Groq Vision detection result — bullet list of identified ingredients |
| **🥗 Nutrition** | Per-100g macros table from USDA database for each detected ingredient |
| **📚 Similar Recipes (RAG)** | Top 3 ChromaDB matches — title, cuisine, difficulty, cook time, cosine similarity score |
| **🍳 Generated Recipes** | 5 complete recipes — ingredients (≥10 items with precise measurements), instructions (≥8 steps with temperatures and timing), cuisine and difficulty label |
| **🖼️ Dish Images** | `gr.Gallery` — up to 5 FLUX.1-generated food photos, 512×512, professional food photography style |
| **📊 Pipeline Stats** | Per-node processing time breakdown + LangSmith trace link (when tracing is enabled) |

---

## Project Structure

```
cAIuldron/
├── app_v2/                              ← v2 application (this README)
│   ├── 1_config.ipynb                  # API keys, model names, path constants
│   ├── 2_tools.ipynb                   # Vision / Nutrition / WebSearch / ImageGen
│   ├── 3_rag.ipynb                     # ChromaDB index build + similarity query
│   ├── 4_agent.ipynb                   # RecipeState, LangGraph nodes, graph compile
│   ├── app.ipynb                       # Main — %run chain + Gradio UI + launch
│   ├── .chroma_db/                     # ChromaDB persistent index (auto-created)
│   ├── requirements.txt
│   ├── .env                            # Your API keys (not committed)
│   └── .env.example                    # Key template
│
├── app_v1/                              ← v1 application (local model version)
│   ├── README.md
│   ├── CAIuldron Poster.pdf
│   └── FINAL REPORT.pdf
│
├── data/                                ← Shared data across both versions
│   ├── nutrition_lookup_full.json       # 525 ingredients, USDA macros
│   ├── ingredients_nutrition_full.csv   # Raw USDA export
│   ├── ingredients_vocabulary.csv       # 528-ingredient vocabulary
│   ├── processed/
│   │   ├── recipes/
│   │   │   └── full_recipes.json        # 7,913 recipes (ChromaDB source)
│   │   └── nutrition_lookup.json        # Processed nutrition subset
│   └── test_images/                     # Sample photos for testing
│
└── reports/                             ← Academic deliverables
    ├── Deliverable1_Technical_Blueprint_Full.pdf
    ├── Deliverable2_Technical_Blueprint_Full.pdf
    └── Deliverable3_Technical_Blueprint_Full.pdf
```

---

## Module Descriptions

### 1. Config (`1_config.ipynb`)

Loads `.env`, resolves project paths, defines all constants used by the pipeline.

**Key constants**:
```python
GROQ_VISION_MODEL  = 'meta-llama/llama-4-scout-17b-16e-instruct'
GROQ_TEXT_MODEL    = 'llama-3.3-70b-versatile'
EMBEDDING_MODEL    = 'sentence-transformers/all-MiniLM-L6-v2'
IMAGE_GEN_MODEL    = 'black-forest-labs/FLUX.1-schnell'
HF_INFERENCE_URL   = 'https://router.huggingface.co/hf-inference/models/...'
NUM_RECIPES        = 5
GROQ_TEMPERATURE   = 0.85
RAG_TOP_K          = 3
RAG_MIN_SIMILARITY = 0.25
CUISINE_ROTATION   = [Asian, Mediterranean, American, Mexican, Italian]
```

### 2. Tools (`2_tools.ipynb`)

Four standalone tool functions used by agent nodes.

**Tool 1 — Vision** (`detect_ingredients_from_image`):
- Encodes image as base64 JPEG
- Sends to Groq Llama 4 Scout Vision API
- Returns parsed JSON ingredient list (max 8 items)

**Tool 2 — Nutrition** (`build_nutrition_summary`, `lookup_nutrition`):
- Exact → case-insensitive → substring matching against 525-ingredient USDA JSON
- Returns per-ingredient macro dict + markdown summary string

**Tool 3 — Web Search** (`search_recipe_inspiration`):
- Queries Tavily with `"recipes with {top 4 ingredients} quick easy dinner"`
- Returns top-5 results + formatted context block for LLM injection

**Tool 4 — Image Generation** (`generate_images_sync`, `generate_images_async`):
- Async concurrent HTTP POST to HF Inference Router (FLUX.1-schnell)
- 5 calls run in parallel via `asyncio.gather`
- `nest_asyncio.apply()` handles Jupyter event loop conflict
- Prompt template: `"professional food photography, {title}, {cuisine} cuisine, beautifully plated on white background, soft natural lighting, shallow depth of field, appetizing"`

### 3. RAG (`3_rag.ipynb`)

Builds and queries the ChromaDB recipe vector store.

**Index build** (`build_or_load_index`):
- First run: loads `full_recipes.json`, embeds 7,913 recipes in batches of 500
- Embedding text per recipe: `"Recipe: {title}. Cuisine: {cuisine}. Main ingredients: {tags}."` (semantic-focused, no instructions)
- Subsequent runs: checks `col.count() >= 7000`, skips rebuild (~1s)
- Distance metric: cosine (`hnsw:space=cosine`)

**Query** (`retrieve_similar_recipes`):
- Query format: `"Main ingredients: {detected_ingredients}."` (mirrors index format)
- Returns top-3 results with similarity score ≥ 0.25
- Similarity = `1 - cosine_distance`

### 4. Agent (`4_agent.ipynb`)

Defines `RecipeState`, all 5 LangGraph nodes, and compiles the `StateGraph`.

**Nodes**:
| Node | Input state keys | Output state keys |
|---|---|---|
| `detect_ingredients_node` | `image_bytes` | `detected_ingredients`, `ingredient_detection_raw` |
| `estimate_nutrition_node` | `detected_ingredients` | `nutrition_per_ingredient`, `nutrition_summary` |
| `rag_search_node` | `detected_ingredients` | `rag_retrieved_recipes`, `rag_context_block` |
| `web_search_node` | `detected_ingredients` | `web_search_results`, `web_context_block` |
| `generate_recipes_node` | `detected_ingredients` + both context blocks + `nutrition_summary` | `recipes` |
| `generate_images_node` | `recipes` | `recipes` (updated with `image_bytes` per recipe) |

**Recipe generation prompt design**:
- System: Michelin-star chef persona, strict JSON schema enforcement, minimum 10 ingredients + 8 steps per recipe
- User context layers: RAG top-3 context (~400 tokens) + Tavily web results (~300 tokens) + nutrition summary (~100 tokens)
- Single API call produces all 5 recipes at once
- Output validated with `json.loads`; markdown fences stripped via regex

### 5. App (`app.ipynb`)

Chains all modules via `%run`, defines the Gradio `process_image()` pipeline function, builds the 6-tab UI, and launches the server.

**Pipeline flow in `process_image()`**:
1. Convert PIL image → JPEG bytes
2. Build initial `RecipeState` dict
3. `recipe_graph.invoke(initial_state)` — runs full LangGraph pipeline
4. Extract outputs and format 6 tab markdown strings
5. Return to Gradio

---

## Setup

### Prerequisites

```
Python 3.11+
conda (Anaconda or Miniconda)
~500 MB disk (ChromaDB index + sentence-transformers model, auto-downloaded)
```

No GPU required. No local AI model downloads.

### 1. Create conda environment

```bash
conda create -n caiuldron-v2 python=3.11 -y
conda activate caiuldron-v2
pip install -r app_v2/requirements.txt
```

### 2. Get API keys (all free)

| Service | Where to get | Purpose |
|---|---|---|
| **Groq** | [console.groq.com](https://console.groq.com) | Vision + recipe generation |
| **Tavily** | [app.tavily.com](https://app.tavily.com) | Web recipe search |
| **HuggingFace** | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) | FLUX.1 image generation |
| **LangSmith** | [smith.langchain.com](https://smith.langchain.com) | Pipeline observability (optional) |

### 3. Configure environment

```bash
# Windows
copy app_v2\.env.example app_v2\.env
```

Edit `app_v2/.env`:

```env
GROQ_API_KEY=gsk_...
TAVILY_API_KEY=tvly-...
HF_TOKEN=hf_...
LANGCHAIN_API_KEY=ls__...        # optional, for LangSmith tracing
LANGCHAIN_TRACING_V2=true        # set false to disable tracing
LANGCHAIN_PROJECT=cAIuldron-v2
```

### 4. Build ChromaDB index (first run only, ~2–3 minutes)

Open `app_v2/3_rag.ipynb` in VS Code or Jupyter and run all cells. The index persists to `app_v2/.chroma_db/` and loads in ~1 second on subsequent runs.

### 5. Launch

Open `app_v2/app.ipynb` and click **Run All Cells**.

```
✅ All modules loaded — ready to launch Gradio
✅ Gradio ready
✅ process_image() ready
✅ Gradio UI built
Running on local URL: http://127.0.0.1:7860
```

For a public shareable link (valid 72 hours), change `share=False` → `share=True` in the last cell.

---

## Performance

| Stage | Typical Time |
|---|---|
| Ingredient detection (Groq Vision) | 1–2s |
| Nutrition lookup (local USDA JSON) | < 0.1s |
| RAG search + Web search (parallel) | 1–3s |
| Recipe generation (Groq 70B, 5 recipes) | 5–15s |
| Image generation (5× FLUX.1, concurrent) | 30–60s (HF free tier cold start) |
| **Total** | **~40–80s** |

Pipeline timing is shown per-node in the **📊 Pipeline Stats** tab after each run.

---

## API Usage (Free Tier Limits)

| Service | Free Limit | v2 Usage per Run |
|---|---|---|
| Groq | ~14,400 req/day | 2 calls (1 vision + 1 text) |
| Tavily | 1,000 searches/month | 1 call |
| HF Inference API | Monthly credit allocation | 5 image calls |
| LangSmith | 5,000 traces/month | 1 trace (6 spans) |

---

## Troubleshooting

### Groq model permission denied (403)

Enable the model at [console.groq.com/settings/limits](https://console.groq.com/settings/limits) — Llama 4 Scout requires explicit activation.

### HF image generation fails (402 credits exhausted)

Monthly HF free credits are consumed. Options:
- Subscribe to HF PRO ($9/month) for renewed credits
- Replace `generate_images_node` with [Pollinations.ai](https://pollinations.ai) (completely free, no API key)

### ChromaDB index empty or corrupt

Force a full rebuild:
```python
_rag_collection = build_or_load_index(force_rebuild=True)
```

### Async error in Jupyter (`cannot run nested event loop`)

`nest_asyncio` is applied automatically in `2_tools.ipynb`. Ensure it's installed:
```bash
pip install nest_asyncio
```

### LangGraph `InvalidUpdateError` on parallel nodes

Both `rag_search_node` and `web_search_node` write to `processing_time_seconds` simultaneously. This is handled in `RecipeState` with:
```python
processing_time_seconds: Annotated[Dict[str, float], lambda a, b: {**a, **b}]
```
If you add new parallel nodes, use the same `Annotated` reducer pattern.

---

## Key Technologies

### AI Models

| Model | Provider | Role |
|---|---|---|
| Llama 4 Scout 17B (Vision) | Meta / Groq API | Ingredient detection from photos |
| Llama 3.3 70B Versatile | Meta / Groq API | Recipe generation |
| all-MiniLM-L6-v2 | Sentence Transformers | Recipe embedding (local, CPU) |
| FLUX.1-schnell | Black Forest Labs / HF API | Dish image generation |

### Frameworks

- **LangGraph** — StateGraph agent orchestration, parallel node execution
- **LangChain / LangChain-Groq** — `ChatGroq` wrapper with automatic LangSmith tracing
- **ChromaDB** — local persistent vector store with HNSW cosine index
- **Gradio** — 6-tab web interface with gallery, progress tracking
- **httpx + asyncio** — concurrent async HTTP for parallel image generation

### Datasets

- **USDA FoodData Central** — 525-ingredient nutrition database
- **RecipeNLG** — 7,913 curated recipes indexed in ChromaDB

---

## Acknowledgments

- **Groq** — Ultra-fast LLM inference API (Vision + 70B)
- **Meta AI** — Llama 4 Scout and Llama 3.3 model weights
- **Black Forest Labs** — FLUX.1-schnell image generation model
- **LangChain / LangGraph** — Agent orchestration framework
- **USDA** — FoodData Central nutrition database
- **RecipeNLG** — Open recipe dataset

---

## License

MIT License
