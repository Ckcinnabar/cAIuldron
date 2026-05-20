# cAIuldron v2.0 — AI Recipe Generator

> Upload a photo of your ingredients → get 5 restaurant-quality recipes with nutrition data, dish images, and web-inspired variations.

---

## What's New in v2

| Capability | v1 (local models) | v2 (cloud-native) |
|---|---|---|
| **Ingredient detection** | Local CLIP + DETR (CPU-bound, slow) | Groq Llama 4 Scout Vision (API, ~1s) |
| **Recipe generation** | Local GPT-2 / Llama 1B–8B | Groq Llama 3.3 70B (API, Michelin-quality output) |
| **Recipe knowledge base** | None | ChromaDB RAG — 7,913 recipes with cosine similarity search |
| **Web inspiration** | None | Tavily API — live web search for recipe ideas |
| **Dish images** | None | HF Inference API FLUX.1-schnell — 5 concurrent AI-generated photos |
| **Agent orchestration** | Sequential Python calls | LangGraph StateGraph with parallel fork/join |
| **Observability** | None | LangSmith — per-node timing + full trace dashboard |
| **GPU usage** | Required (VRAM-intensive) | Zero — all compute on cloud APIs |
| **UI tabs** | 2 (ingredients + recipes) | 6 (ingredients, nutrition, RAG, recipes, gallery, stats) |

---

## Architecture

```
START
  └── detect_ingredients_node     ← Groq Llama 4 Scout Vision API
        └── estimate_nutrition_node   ← USDA JSON lookup (525 ingredients, local)
              ├── rag_search_node       ← ChromaDB (7,913 recipes) [parallel]
              └── web_search_node       ← Tavily web search         [parallel]
                    └── generate_recipes_node  ← Groq Llama 3.3 70B
                          └── generate_images_node  ← HF FLUX.1-schnell (5 images concurrent)
                                └── END
```

**Parallel execution**: `rag_search` and `web_search` run simultaneously via LangGraph's parallel fork — both results feed into recipe generation.

---

## Tech Stack

| Component | Technology |
|---|---|
| Ingredient vision | Groq Llama 4 Scout 17B (Vision) |
| Recipe generation | Groq Llama 3.3 70B Versatile |
| RAG vector store | ChromaDB + `sentence-transformers/all-MiniLM-L6-v2` |
| Web search | Tavily API |
| Dish image generation | HF Inference API — `black-forest-labs/FLUX.1-schnell` |
| Agent orchestration | LangGraph StateGraph |
| UI | Gradio 4.x (6-tab layout) |
| Observability | LangSmith (automatic via LangGraph) |
| Nutrition data | USDA FoodData Central (525 ingredients, local JSON) |

All AI compute runs on free-tier cloud APIs — no local GPU required.

---

## Gradio UI — 6 Tabs

| Tab | Content |
|---|---|
| 🔍 Ingredients | Groq Vision detection results |
| 🥗 Nutrition | Per-100g macros from USDA database |
| 📚 Similar Recipes (RAG) | Top 3 matches from 7,913-recipe ChromaDB with similarity scores |
| 🍳 Generated Recipes | 5 complete recipes with ingredients (≥10 items) and instructions (≥8 steps) |
| 🖼️ Dish Images | Gallery of 5 FLUX.1-generated food photos |
| 📊 Pipeline Stats | Per-node timing + LangSmith trace link |

---

## Setup

### 1. Create conda environment

```bash
conda create -n caiuldron-v2 python=3.11 -y
conda activate caiuldron-v2
pip install -r app_v2/requirements.txt
```

### 2. Configure API keys

```bash
cp app_v2/.env.example app_v2/.env
```

Edit `app_v2/.env`:

```env
GROQ_API_KEY=your_key_here          # console.groq.com (free)
TAVILY_API_KEY=your_key_here        # app.tavily.com (free)
HF_TOKEN=your_key_here              # huggingface.co/settings/tokens (free)
LANGCHAIN_API_KEY=your_key_here     # smith.langchain.com (free, optional)
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=cAIuldron-v2
```

### 3. Build the ChromaDB index (first run only, ~2 minutes)

Open `app_v2/3_rag.ipynb` in VS Code or Jupyter and run all cells. The index is saved to `app_v2/.chroma_db/` and reused on subsequent runs.

### 4. Launch

Open `app_v2/app.ipynb` and **Run All Cells**.

Gradio starts at `http://127.0.0.1:7860`.

For a public shareable link (72h), change `share=False` → `share=True` in the last cell.

---

## Notebook Structure

```
app_v2/
├── 1_config.ipynb      # API keys, model names, constants
├── 2_tools.ipynb       # Vision / Nutrition / WebSearch / ImageGen functions
├── 3_rag.ipynb         # ChromaDB index build + similarity search
├── 4_agent.ipynb       # RecipeState TypedDict, LangGraph nodes, graph compile
├── app.ipynb           # Main app — %run chains above, then Gradio launch
├── requirements.txt
├── .env                # Your API keys (not committed)
└── .env.example        # Key template
```

`app.ipynb` loads all modules via `%run` then launches Gradio — only this file needs to be executed end-to-end.

---

## Data Sources

- **Recipes**: RecipeNLG dataset — 7,913 recipes indexed in ChromaDB
- **Nutrition**: USDA FoodData Central — 525 ingredients, `data/nutrition_lookup_full.json`

---

## API Costs

All services used on free tiers:

| Service | Free limit |
|---|---|
| Groq | ~14,400 req/day (Vision + Text) |
| Tavily | 1,000 searches/month |
| HF Inference API | Monthly credit allocation |
| LangSmith | 5,000 traces/month |
