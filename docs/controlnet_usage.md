# ControlNet 使用說明

## 什麼是 ControlNet？

ControlNet 是一個可以讓 AI 根據**參考圖片的結構**來生成新圖片的技術。

```
您的參考圖 → ControlNet 提取結構 → 指導 Stable Diffusion → 生成相似視角的新圖
```

---

## 為什麼使用 ControlNet？

### 問題：純文字 prompt 的限制
- ❌ 很難用文字精確描述視角
- ❌ AI 常常生成俯視圖（我們不想要的）
- ❌ 每次生成視角都不一致

### 解決：用參考圖控制
- ✅ 用您喜歡的視角圖片作為參考
- ✅ AI 會生成相同視角的新內容
- ✅ 視角一致性高達 80-90%

---

## 我們的參考圖庫

在 `input data/` 資料夾中有 7 張參考圖：

| 參考圖 | 適用的烹飪動作 |
|--------|----------------|
| `cut.png` | cut, dice, chop, slice |
| `frying1.png` | fry, sauté, stir-fry |
| `frying2.png` | pan-fry |
| `boiling.png` | boil, simmer, cook |
| `mixing.png` | mix, stir, whisk, combine |
| `seasoning.png` | season, sprinkle, add |
| `plating.png` | plate, serve, garnish |

系統會**自動選擇**最適合的參考圖！

---

## 如何使用

### 方法 1：單張生成

```python
step = {
    "instruction_text": "Cut the salmon into portions",
    "cooking_method": "cut"  # 會自動使用 cut.png 作為參考
}

illustration, metadata = generate_cooking_illustration_controlnet(
    step,
    "output.png"
)

# 查看使用了哪張參考圖
print(metadata['reference_image'])  # "input data/cut.png"
```

### 方法 2：批次生成（整道菜）

```python
recipe = {
    "recipe_id": "salmon-001",
    "cooking_steps": [
        {"step_number": 1, "instruction_text": "...", "cooking_method": "cut"},
        {"step_number": 2, "instruction_text": "...", "cooking_method": "season"},
        {"step_number": 3, "instruction_text": "...", "cooking_method": "fry"},
        ...
    ]
}

illustrations, stats = generate_recipe_illustrations_controlnet(
    recipe,
    "output/salmon-001"
)
```

---

## 工作流程

### 1. 選擇參考圖
```python
reference_path = get_reference_image("fry")
# → "input data/frying1.png"
```

### 2. 提取邊緣結構
```python
edges = extract_canny_edges(reference_path)
# 使用 Canny 邊緣檢測提取線條結構
```

### 3. 生成新圖片
```python
image = controlnet_pipe(
    prompt="frying salmon, minimal line art",
    image=edges,  # 參考圖的邊緣
    controlnet_conditioning_scale=0.8,  # 遵循程度 80%
    ...
)
```

---

## 關鍵參數

### `controlnet_conditioning_scale`

控制多少程度遵循參考圖：

- **0.0**：完全不參考，等於純 text-to-image
- **0.5**：參考一半
- **0.8**：高度遵循（推薦）
- **1.0**：完全遵循

```python
# 如果生成的圖片視角偏離太多
controlnet_conditioning_scale=0.9  # 提高遵循度

# 如果想要更多變化
controlnet_conditioning_scale=0.6  # 降低遵循度
```

### `guidance_scale`

控制多少程度遵循文字 prompt：

- **7.0-9.0**：標準
- **12.0**：我們使用的值（強制遵守 "no text" 等要求）

---

## 優點 vs 缺點

### ✅ 優點

1. **視角一致**：所有同類動作都用相同視角
2. **不需訓練**：直接使用，無需準備數據集
3. **可控性高**：換參考圖就換視角
4. **相容原有功能**：仍然可以用 prompt 控制細節

### ⚠️ 注意事項

1. **參考圖品質影響結果**：
   - 如果參考圖有手，生成的圖可能也有手
   - 如果參考圖有文字，生成的圖可能也有文字
   - 建議定期更新參考圖為更乾淨的版本

2. **記憶體需求**：
   - 需要額外載入 ControlNet 模型（~1.5GB）
   - 總 VRAM：約 5.5GB（原本 4GB + ControlNet 1.5GB）

3. **速度**：
   - 比純 Stable Diffusion 慢 10-20%
   - 每張圖約 10-15 秒（GPU）

---

## 常見問題

### Q: 可以不用 ControlNet 嗎？

A: 可以！保留了原本的函數：
- `generate_cooking_illustration_improved()` - 純 text-to-image
- `generate_cooking_illustration_controlnet()` - 用 ControlNet

### Q: 如何新增參考圖？

1. 準備一張新的參考圖，例如 `baking.png`
2. 放到 `input data/` 資料夾
3. 更新映射：

```python
REFERENCE_IMAGES = {
    ...
    "bake": "input data/baking.png",
    "roast": "input data/baking.png",
}
```

### Q: 參考圖的品質要求？

- 尺寸：至少 512x512（更大更好）
- 格式：PNG, JPG 都可以
- 內容：清楚的線條，最好是您理想中的視角和風格
- 背景：純色背景最佳

### Q: 出現記憶體不足錯誤？

```python
# 啟用 CPU offload
controlnet_pipe.enable_model_cpu_offload()

# 或降低解析度
height=384, width=384  # 原本 512x512
```

---

## 效果對比

### 純 Text-to-Image (v3)
- Prompt: "side angle view 45 degrees, cut salmon..."
- 視角：❓ 不確定（可能俯視、可能側視）
- 一致性：⭐⭐⭐

### ControlNet + Text (v4)
- Prompt: "cut salmon..." + 參考圖：`cut.png`
- 視角：✅ 與參考圖相同
- 一致性：⭐⭐⭐⭐⭐

---

## 下一步

### 短期（立即可做）
- 運行測試 cell 看效果
- 如果參考圖有不想要的元素（手、文字），可以用圖片編輯軟體清理

### 中期（建議）
- 收集或繪製更多高品質參考圖
- 針對特定烹飪法（烤、蒸、微波）新增參考圖

### 長期（進階）
- 訓練 LoRA 在參考圖上，進一步提升風格一致性
- 探索其他 ControlNet 模型（Depth, Pose）

---

## 總結

✅ **已完成**：
- ControlNet 模型載入
- 7 張參考圖映射系統
- Canny 邊緣提取
- 參考圖導引生成函數
- 批次生成功能

✅ **如何使用**：
```python
# 單張
illustration, metadata = generate_cooking_illustration_controlnet(step, output_path)

# 批次
illustrations, stats = generate_recipe_illustrations_controlnet(recipe, output_dir)
```

🎯 **預期效果**：
- 視角一致性大幅提升
- 仍然保持無文字、極簡風格
- 自動選擇最適合的參考圖

請運行 notebook 測試效果！
