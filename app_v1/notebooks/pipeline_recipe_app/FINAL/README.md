# 🍳 cAIuldron - AI Recipe Generator (Modular Version)

**Transform ingredient photos into delicious recipes with AI!**

## 📁 模組化架構

重新整理後的專案採用模組化設計，每個功能獨立成一個 notebook，方便維護和擴展。

```
FINAL/
├── 1_model_loading.ipynb          # 模型載入與管理
├── 2_ingredient_detection.ipynb   # 食材偵測 (CLIP + DETR)
├── 3_nutrition_estimation.ipynb   # 營養估算
├── 4_recipe_generation.ipynb      # 食譜生成邏輯
├── app.ipynb                      # Gradio 介面 (主程式)
└── README.md                      # 本文件
```

---

## 🚀 快速開始

### 方法 1：直接執行主程式

```bash
# 在 Jupyter 中打開並執行
jupyter notebook app.ipynb
```

或使用 JupyterLab:
```bash
jupyter lab app.ipynb
```

### 方法 2：分別測試各模組

```python
# 在 Jupyter 中依序執行
%run 1_model_loading.ipynb
%run 2_ingredient_detection.ipynb
%run 3_nutrition_estimation.ipynb
%run 4_recipe_generation.ipynb
```

---

## 📚 模組說明

### 1️⃣ `1_model_loading.ipynb` - 模型載入

**功能：**
- 載入 GPT-2 (fine-tuned)
- 載入 Llama 3.2 1B (fine-tuned with LoRA)
- 載入 Llama 3.1 8B GGUF (量化模型)

**主要函數：**
```python
load_recipe_model(RecipeModelType.LLAMA_1B)
```

**匯出變數：**
- `RecipeModelType` (enum)
- `MODEL_INFO` (dict)
- `RECIPE_MODELS` (cache)
- `load_recipe_model()` (function)

---

### 2️⃣ `2_ingredient_detection.ipynb` - 食材偵測

**功能：**
- 使用 CLIP 進行食材分類
- 使用 DETR 進行多物件偵測
- 整合多個偵測結果

**主要函數：**
```python
# 單一食材偵測
detect_ingredient_clip(image_path)

# 多食材偵測
detected = detect_multiple_ingredients_clip(image_path)

# 整合結果
result = consolidate_detections(detected)
```

**匯出變數：**
- `INGREDIENT_CANDIDATES` (list)
- `detect_ingredient_clip()` (function)
- `detect_multiple_ingredients_clip()` (function)
- `consolidate_detections()` (function)

---

### 3️⃣ `3_nutrition_estimation.ipynb` - 營養估算

**功能：**
- 根據食材類型估算營養
- 使用 USDA FoodData Central 資料庫
- 根據邊界框大小估算重量

**主要函數：**
```python
nutrition = estimate_nutrition(
    ingredient='chicken breast',
    bbox_width=200,
    bbox_height=200
)
```

**匯出變數：**
- `NUTRITION_DB` (dict)
- `TYPICAL_WEIGHTS` (dict)
- `estimate_nutrition()` (function)

---

### 4️⃣ `4_recipe_generation.ipynb` - 食譜生成

**功能：**
- 支援多模型生成 (GPT-2, Llama 1B, Llama 8B)
- 強化的提示詞與具體範例
- 時間欄位格式驗證
- LLM-based 時間估算備援

**主要函數：**
```python
recipe = generate_recipe_with_selected_model(
    ingredient='chicken',
    cuisine='Asian',
    difficulty='beginner',
    model_type=RecipeModelType.LLAMA_1B,
    nutrition_data=nutrition
)
```

**匯出變數：**
- `generate_recipe_with_selected_model()` (main function)
- `generate_recipe_llama()` (Llama generation)
- `generate_recipe_gguf()` (GGUF generation)
- `generate_recipe_gpt2()` (GPT-2 generation)
- `validate_time_format()` (validation)
- `ensure_time_fields_with_llm()` (fallback)

---

### 5️⃣ `app.ipynb` - 主程式 (Gradio 介面)

**功能：**
- 整合所有模組
- 提供 Gradio Web 介面
- 簡化的工作流程（移除 JSON 儲存）

**執行流程：**
1. 載入所有模組
2. 設定 Gradio 介面
3. 啟動 Web 伺服器 (http://127.0.0.1:7861)

---

## ✨ 主要改進

### 相較於原始 `pipeline_photo_to_recipes_v3.ipynb`：

✅ **已移除的功能：**
- JSON 檔案儲存（簡化流程）
- 自動修復函數（不需要）
- 冗餘的 GPT-2 解析邏輯
- 複雜的錯誤處理（簡化）

✅ **保留的核心功能：**
- 食材偵測與整合
- 營養估算
- 多模型食譜生成
- 時間欄位驗證與格式強制執行

✅ **架構改進：**
- 模組化設計（單一職責原則）
- 更好的程式碼組織
- 更容易維護和測試
- 可重複使用的模組

---

## 🎯 使用範例

### 範例 1：獨立使用模型載入模組

```python
%run 1_model_loading.ipynb

# 載入 Llama 1B 模型
model_dict = load_recipe_model(RecipeModelType.LLAMA_1B)
print(f"Model type: {model_dict['type']}")
```

### 範例 2：獨立使用食材偵測模組

```python
%run 2_ingredient_detection.ipynb

# 偵測食材
detected = detect_multiple_ingredients_clip("path/to/image.jpg")
result = consolidate_detections(detected)
print(f"Found: {result['combined_ingredient']}")
```

### 範例 3：獨立使用營養估算模組

```python
%run 3_nutrition_estimation.ipynb

# 估算營養
nutrition = estimate_nutrition('chicken breast', 200, 200)
if nutrition['success']:
    print(f"Calories: {nutrition['per_serving']['calories']} kcal")
```

### 範例 4：完整流程

```python
# 載入所有模組
%run 1_model_loading.ipynb
%run 2_ingredient_detection.ipynb
%run 3_nutrition_estimation.ipynb
%run 4_recipe_generation.ipynb

# 1. 偵測食材
detected = detect_multiple_ingredients_clip("test.jpg")
result = consolidate_detections(detected)

# 2. 估算營養
nutrition = estimate_nutrition(
    result['primary_ingredient'],
    int(result['primary_area'] ** 0.5),
    int(result['primary_area'] ** 0.5)
)

# 3. 生成食譜
recipe = generate_recipe_with_selected_model(
    ingredient=result['combined_ingredient'],
    cuisine='Asian',
    difficulty='beginner',
    model_type=RecipeModelType.LLAMA_1B,
    nutrition_data=nutrition
)

print(recipe['raw_markdown'])
```

---

## 🔧 系統需求

### 硬體需求：
- **GPU**: NVIDIA GPU with 4GB+ VRAM (建議)
- **RAM**: 16GB+
- **Storage**: 10GB+ (模型檔案)

### 軟體需求：
```bash
pip install torch transformers peft llama-cpp-python
pip install gradio pillow pandas numpy
pip install bitsandbytes accelerate
```

### 模型檔案位置：
```
models/recipe_generation/
├── finetuned/                    # GPT-2 fine-tuned
├── llama3_1b_finetuned/          # Llama 3.2 1B LoRA adapters
└── Meta-Llama-3.1-8B-Instruct-Q5_K_M.gguf  # Llama 3.1 8B GGUF
```

---

## 📊 效能比較

| 模型 | 速度 | 品質 | VRAM | 推薦情境 |
|------|------|------|------|---------|
| GPT-2 | ⚡⚡⚡ 快 | 😊 良好 (60-70/100) | 1-2 GB | 快速測試 |
| Llama 3.2 1B | ⚡⚡ 中等 | 🌟 優秀 (90-95/100) | 4-5 GB | **推薦使用** ⭐ |
| Llama 3.1 8B GGUF | ⚡ 較慢 | 🌟🌟 最佳 (95-100/100) | 4-6 GB | 最高品質 |

---

## 🐛 疑難排解

### 問題 1：模組載入失敗
```python
# 確認路徑正確
import sys
from pathlib import Path
print(Path.cwd())

# 確認模組存在
%ls 1_model_loading.ipynb
```

### 問題 2：CUDA 記憶體不足
```python
# 使用較小的模型
model_type = RecipeModelType.GPT2  # 或 LLAMA_1B

# 或調整 GGUF 參數 (在 1_model_loading.ipynb 中)
n_gpu_layers=8  # 減少 GPU 層數
```

### 問題 3：模型檔案找不到
```python
# 檢查模型路徑
from pathlib import Path
MODEL_DIR = Path.cwd().parent.parent.parent / "models" / "recipe_generation"
print(MODEL_DIR.exists())
print(list(MODEL_DIR.glob("*")))
```

---

## 📝 維護建議

### 更新單一模組：
1. 修改對應的 `.ipynb` 檔案
2. 在 `app.ipynb` 中重新載入該模組
3. 測試是否正常運作

### 新增功能：
1. 在對應模組中新增函數
2. 在檔案末尾更新「Module exports」說明
3. 在 `README.md` 中更新文件

### 效能優化：
1. 模型載入：調整量化參數
2. 偵測：調整信心度閾值
3. 生成：調整溫度和 top_p 參數

---

## 📄 授權

本專案為教育用途，請遵守相關模型的使用條款：
- GPT-2: MIT License
- Llama 3.2 & 3.1: Meta Llama License
- CLIP: MIT License
- DETR: Apache 2.0 License

---

## 🙏 致謝

- **Hugging Face** - 模型與 Transformers 函式庫
- **Meta AI** - Llama 模型
- **OpenAI** - CLIP 模型
- **Facebook AI** - DETR 模型
- **USDA** - FoodData Central 營養資料庫

---

## 📧 聯絡方式

如有問題或建議，請在 GitHub Issues 中提出。

**Happy Cooking! 🍳**
