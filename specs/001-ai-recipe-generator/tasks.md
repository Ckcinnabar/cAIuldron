# Tasks: AI-Powered Recipe Generator

**Input**: Design documents from `/specs/001-ai-recipe-generator/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Tests are NOT explicitly requested in the specification, so test tasks are excluded.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `- [ ] [ID] [P?] [Story?] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- All tasks include exact file paths

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan: notebooks/, data/raw/, data/processed/, data/results/, models/, tests/
- [ ] T002 Initialize Python project with requirements.txt including torch==2.1.0, transformers==4.35.0, opencv-python==4.8.1, pillow==10.1.0, pgmpy==0.1.23, numpy, pandas, matplotlib, jupyter, papermill, pytest
- [ ] T003 [P] Configure linting tools: create .flake8 config, setup black formatter, install nbstripout for notebook output management
- [ ] T004 [P] Create .gitignore for notebook checkpoints (*.ipynb_checkpoints/), large model files (models/*.h5, models/*.pt), data files (data/raw/*, data/results/*), and Python cache (__pycache__/, *.pyc)
- [ ] T005 [P] Create data directories structure: data/raw/ingredient_images/, data/raw/recipe_corpus/, data/raw/nutrition_database/, data/processed/ingredient_features/, data/processed/recipe_tokens/, data/processed/nutrition_vectors/, data/results/generated_recipes/, data/results/nutrition_estimates/

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 Create base utility notebook notebooks/utils_recipe/utils_image_processing.ipynb with image loading, resizing to 384x384, normalization (ImageNet mean/std), and tensor conversion functions
- [ ] T007 [P] Create notebook notebooks/utils_recipe/utils_model_loading.ipynb with model checkpoint loading, device selection (cuda/cpu), and memory optimization functions
- [ ] T008 [P] Create notebook notebooks/utils_recipe/utils_visualization.ipynb with plot formatting functions, progress bar setup (tqdm), and result display utilities
- [ ] T009 Create reproducibility setup notebook notebooks/utils_recipe/utils_reproducibility.ipynb with random seed setting for NumPy (42), PyTorch (42), and TensorFlow (42), plus environment variable configuration
- [ ] T010 Create data validation notebook notebooks/utils_recipe/utils_validation.ipynb with entity validation functions for Ingredient, Recipe, CookingStep, and PortionEstimate per data-model.md specifications
- [ ] T011 [P] Create logging utility notebook notebooks/utils_recipe/utils_logging.ipynb with timestamp logging, model metadata tracking, and performance metric logging functions

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Basic Ingredient Photo to Recipe Suggestions (Priority: P1) 🎯 MVP

**Goal**: User uploads ingredient photo and receives 5+ diverse recipe suggestions with cuisine type, cooking time, and difficulty level within 5 seconds

**Independent Test**: Upload a chicken breast photo → receive 5 recipes with Asian, Western, and fusion options, each showing cooking time and difficulty

### Implementation for User Story 1

**Roboflow Ingredient Recognition (Required for US1)**

- [x] T012-T014 [P] [US1] ✅ **COMPLETED** - Create Roboflow model setup notebook notebooks/model_ingredient_recognition/setup_roboflow_model.ipynb to download and configure pre-trained food ingredients detection model (v4) from Roboflow Universe, supporting both API and ONNX local inference
- [x] T014-T017 [US1] ✅ **COMPLETED** - Create ingredient recognition notebook notebooks/model_ingredient_recognition/model_cnn_inference.ipynb implementing Roboflow model inference, bounding box size estimation for weight calculation, Ingredient entity creation per data-model.md schema, and validation/error handling for low confidence (<0.7) and detection failures per ingredient_recognition.json contract

**Transformer Recipe Generation (Required for US1)**

- [ ] T018 [P] [US1] Create recipe generation inference notebook notebooks/model_recipe_generation/model_transformer_inference.ipynb loading GPT-2 Medium fine-tuned model with prompt template: "Ingredient: {name}, Cuisine: {type}, Difficulty: {level}, Recipe:"
- [ ] T019 [US1] Implement parallel recipe generation in notebooks/model_recipe_generation/model_transformer_inference.ipynb to generate 5 recipes with diverse cuisines (Asian, Western, Fusion, 2×Any) using batch size 5, temperature 0.8, top_p 0.9
- [ ] T020 [US1] Add recipe parsing and structuring cells to notebooks/model_recipe_generation/model_transformer_inference.ipynb to extract title (max 100 chars), ingredients_list (min 2 items), cooking_steps (min 3 steps), cooking_time_minutes, difficulty_level, and serving_count per recipe_generation.json contract
- [ ] T021 [US1] Create Recipe entity instances in notebooks/model_recipe_generation/model_transformer_inference.ipynb following data-model.md schema with recipe_id (UUID), ingredient_id reference, ai_metadata tracking, and timestamp
- [ ] T022 [US1] Add recipe validation cells to notebooks/model_recipe_generation/model_transformer_inference.ipynb ensuring cooking_time > 0 and < 480 minutes, source ingredient in ingredients_list, and minimum 3 cooking steps

**RNN Cooking Refinement (Required for US1)**

- [ ] T023 [US1] Create RNN refinement notebook notebooks/model_cooking_refinement/model_rnn_inference.ipynb loading BiLSTM model for step sequence optimization and timing adjustment
- [ ] T024 [US1] Implement step tokenization and feature extraction in notebooks/model_cooking_refinement/model_rnn_inference.ipynb extracting cooking method features (sauté, bake, boil) and ingredient complexity scores
- [ ] T025 [US1] Add step reordering logic to notebooks/model_cooking_refinement/model_rnn_inference.ipynb to optimize cooking sequence and validate no circular dependencies
- [ ] T026 [US1] Implement timing refinement in notebooks/model_cooking_refinement/model_rnn_inference.ipynb to adjust estimated_time_minutes per step, validate min 1 minute max 240 minutes, and update total cooking_time_minutes
- [ ] T027 [US1] Create CookingStep entities in notebooks/model_cooking_refinement/model_rnn_inference.ipynb following data-model.md schema with step_id (UUID), recipe_id reference, step_number (sequential), instruction_text, cooking_method, estimated_time_minutes, and dependencies array

**End-to-End Pipeline (Integration for US1)**

- [ ] T028 [US1] Create end-to-end pipeline notebook notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb orchestrating: (1) CNN ingredient recognition, (2) parallel Transformer recipe generation (5 recipes), (3) RNN refinement per recipe, validating total time < 5 seconds
- [ ] T029 [US1] Add recipe diversity validation to notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb ensuring minimum 3 different cuisine types across 5 recipes per FR-004 requirement
- [ ] T030 [US1] Implement recipe output formatting in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb to display recipes with title, cuisine_type, cooking_time_minutes, difficulty_level, and ingredients_list preview
- [ ] T031 [US1] Add error handling and user feedback cells to notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb for unrecognizable photos (helpful message per FR-011), low confidence ingredients (user confirmation), and generation failures (fallback message)
- [ ] T032 [US1] Create recipe storage cells in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb saving Recipe entities as JSON to data/results/generated_recipes/{recipe_id}.json with timestamps and metadata
- [ ] T033 [US1] Add notebook execution validation in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb to verify top-to-bottom execution without errors, test with sample chicken breast image, and validate 5 recipes generated within 5 seconds

**Checkpoint**: At this point, User Story 1 should be fully functional - users can upload photos and receive 5 diverse recipe suggestions with cooking time and difficulty

---

## Phase 4: User Story 2 - Portion Size & Calorie Estimation (Priority: P2)

**Goal**: Users receive portion size (servings) and calorie estimates per serving based on ingredient size from photo, helping with meal planning and dietary tracking

**Independent Test**: Upload chicken breast photo → receive recipes with "200g, serves 2" and "450 calories per serving" estimates

### Implementation for User Story 2

**Bayesian Network Nutrition Estimation (Required for US2)**

- [ ] T034 [P] [US2] Create nutrition database loading notebook notebooks/model_nutrition_estimation/load_usda_database.ipynb to fetch USDA FoodData Central data, extract ingredient-calorie mappings, macronutrient values (protein, carbs, fat), and save to data/raw/nutrition_database/ as Parquet files
- [ ] T035 [P] [US2] Create Bayesian network structure notebook notebooks/model_nutrition_estimation/model_pgm_training.ipynb defining nodes (Ingredient type, Visual size, Portion count, Calories, Macros) and edges per research.md, using pgmpy library
- [ ] T036 [US2] Create PGM inference notebook notebooks/model_nutrition_estimation/model_pgm_inference.ipynb loading Bayesian network, querying USDA database for base calories, and adjusting for estimated ingredient size from CNN
- [ ] T037 [US2] Implement portion calculation in notebooks/model_nutrition_estimation/model_pgm_inference.ipynb using ingredient weight_grams from CNN to calculate servings (int 1-10), portion_size_per_serving string (e.g., "200g"), and total_weight_grams
- [ ] T038 [US2] Add calorie estimation logic to notebooks/model_nutrition_estimation/model_pgm_inference.ipynb calculating calories_per_serving with confidence interval (min, max, 0.8 confidence level), factoring cooking method adjustments (oil, butter), and validating calories > 0 and < 5000 per serving
- [ ] T039 [US2] Implement macronutrient breakdown in notebooks/model_nutrition_estimation/model_pgm_inference.ipynb calculating protein_grams, carbs_grams, fat_grams per serving, validating sum approximately equals total calories (protein×4 + carbs×4 + fat×9)
- [ ] T040 [US2] Create PortionEstimate entity instances in notebooks/model_nutrition_estimation/model_pgm_inference.ipynb following data-model.md schema with estimate_id (UUID), recipe_id and ingredient_id references, servings matching Recipe.serving_count, calorie_range, macronutrients, estimation_method, data_sources, and uncertainty_factors

**Integration with Recipe Pipeline (Connect US2 to US1)**

- [ ] T041 [US2] Update pipeline notebook notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb to call PGM inference for each generated recipe after RNN refinement, passing ingredient size and recipe ingredients_list
- [ ] T042 [US2] Add portion estimate display to notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb showing servings, portion_size_per_serving, calories_per_serving with confidence interval, and optional macro breakdown in recipe output
- [ ] T043 [US2] Implement error handling for nutrition estimation in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb handling missing ingredients (use category average), displaying confidence intervals to users, and providing fallback estimates with uncertainty warnings
- [ ] T044 [US2] Update recipe storage in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb to save PortionEstimate entities linked to recipes in data/results/nutrition_estimates/{estimate_id}.json with metadata
- [ ] T045 [US2] Validate portion estimates in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb ensuring calorie_range min ≤ calories_per_serving ≤ max, servings match recipe, and macros sum reasonably per data-model.md validation rules

**Checkpoint**: User Stories 1 AND 2 should both work independently - US1 provides recipes, US2 adds portion/calorie information

---

## Phase 5: User Story 3 - Interactive Beginner Guidance (Priority: P3)

**Goal**: Beginners receive interactive guidance with cooking tips, technique explanations, and timing alerts to build confidence while cooking

**Independent Test**: View recipe steps → tap technique terms for explanations, set timers for critical steps, see contextual tips

### Implementation for User Story 3

**Technique Explanations & Tips (Required for US3)**

- [ ] T046 [P] [US3] Create technique database notebook notebooks/pipeline_recipe_app/build_technique_database.ipynb with cooking term definitions (sauté, julienne, marinate, etc.) and visual reference links
- [ ] T047 [P] [US3] Add contextual tips generation to notebooks/model_recipe_generation/model_transformer_inference.ipynb using GPT-2 to generate helpful tips per cooking_method (e.g., "Don't overcrowd the pan for better browning" for sauté)
- [ ] T048 [US3] Update CookingStep creation in notebooks/model_cooking_refinement/model_rnn_inference.ipynb to add tips array (optional) and technique_explanations dict mapping technique names to explanation text
- [ ] T049 [US3] Implement interactive tips display in notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb with clickable technique terms using ipywidgets, showing popup explanations with visual references
- [ ] T050 [US3] Add tip highlighting to notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb displaying contextual tips in styled boxes for critical steps (high heat, marinating, timing-sensitive actions)

**Timing Alerts & Progress Tracking (Required for US3)**

- [ ] T051 [US3] Implement timing alert detection in notebooks/model_cooking_refinement/model_rnn_inference.ipynb setting CookingStep.timing_alert = true for time-sensitive steps (marinate X minutes, cook until Y, etc.) and ensuring estimated_time_minutes is set
- [ ] T052 [US3] Create timer widget in notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb using ipywidgets countdown timer, allowing users to start/pause/reset for steps with timing_alert = true
- [ ] T053 [US3] Add progress tracking to notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb showing completed steps checkboxes, total progress bar, and estimated time remaining based on uncompleted CookingStep.estimated_time_minutes sum
- [ ] T054 [US3] Implement cooking session state management in notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb tracking step completion status, timer states, and allowing resume from interruptions

**Feedback & Encouragement (Required for US3)**

- [ ] T055 [US3] Add completion celebration to notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb showing encouragement message when all steps marked complete, with recipe photo upload option
- [ ] T056 [US3] Create feedback collection cells in notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb with rating widgets (1-5 stars), difficulty assessment, and optional text feedback
- [ ] T057 [US3] Implement feedback storage in notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb saving user ratings to data/results/feedback/{recipe_id}_{timestamp}.json for quality monitoring

**Integration & Final Validation (Connect US3 to US1+US2)**

- [ ] T058 [US3] Update pipeline notebook notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb to generate technique explanations and tips during RNN refinement phase
- [ ] T059 [US3] Add beginner mode toggle to notebooks/pipeline_recipe_app/pipeline_interactive_guide.ipynb enabling/disabling interactive features (tips, explanations, timers) based on user preference
- [ ] T060 [US3] Validate complete user journey in notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb: photo upload → 5 recipes with nutrition → select recipe → interactive cooking → completion feedback, ensuring all FR-001 through FR-017 requirements met

**Checkpoint**: All 3 user stories fully functional - complete beginner-friendly recipe generation and guided cooking experience

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and final quality checks

- [ ] T061 [P] Create comprehensive README.md at repository root documenting project overview, installation instructions (requirements.txt), usage examples for each notebook, and performance benchmarks (5-second generation time)
- [ ] T062 [P] Add notebook execution documentation in docs/notebook_guide.md explaining cell organization, reproducibility setup, and troubleshooting common issues (model loading, GPU/CPU selection, memory limits)
- [ ] T063 [P] Create performance monitoring notebook notebooks/pipeline_recipe_app/monitor_performance.ipynb tracking inference times for each model component (CNN: 50-100ms, Transformer: 2-3s, RNN: <50ms, PGM: <10ms), memory usage, and quality metrics (confidence scores, recipe diversity)
- [ ] T064 Code cleanup: run black formatter on all .py utility modules, flake8 linting, and nbstripout to clear notebook outputs before committing
- [ ] T065 [P] Create model checkpoint documentation in models/README.md listing each model file, version, training date, performance metrics, and download links for large files (GPT-2 Medium 1.4GB)
- [ ] T066 [P] Add data versioning manifest in data/processed/manifest.json with dataset checksums (MD5), version dates, train/val/test split ratios, and data source attributions
- [ ] T067 Create end-to-end test script tests/test_full_pipeline.py using papermill to execute notebooks/pipeline_recipe_app/pipeline_photo_to_recipes.ipynb with sample ingredient photos, validating 5 recipes generated, timing < 5 seconds, and cuisine diversity ≥ 3
- [ ] T068 [P] Add error handling improvements across all notebooks: validate image formats before CNN, check model file existence before loading, handle GPU OOM gracefully with CPU fallback, and provide helpful error messages per FR-011
- [ ] T069 Validate quickstart.md instructions by executing each code example, verifying notebook paths, and confirming all dependencies installed correctly
- [ ] T070 Create demo notebook notebooks/demo_recipe_generator.ipynb with pre-loaded sample images, step-by-step execution cells, and expected outputs for quick demonstration and onboarding
- [ ] T071 [P] Add constitution compliance verification: check all notebooks execute top-to-bottom without errors, confirm reproducibility with seed=42, validate markdown documentation cells present, ensure PEP 8 compliance with type hints in extracted functions

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-5)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed) or sequentially by priority (P1 → P2 → P3)
- **Polish (Phase 6)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories - MVP target
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Integrates with US1 but independently testable (works without US1 if given ingredient)
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Enhances US1+US2 but independently testable (can test tips/timers separately)

### Within Each User Story

**User Story 1 (P1) Internal Flow**:
1. T012-T013 (Exploration) can run in parallel [P]
2. T014-T017 (CNN) sequential, then
3. T018-T022 (Transformer) can start in parallel with CNN
4. T023-T027 (RNN) depends on Transformer completing
5. T028-T033 (Pipeline) depends on CNN + Transformer + RNN all complete

**User Story 2 (P2) Internal Flow**:
1. T034-T035 (Database + PGM training) can run in parallel [P]
2. T036-T040 (PGM inference) sequential
3. T041-T045 (Integration) depends on US1 pipeline existing

**User Story 3 (P3) Internal Flow**:
1. T046-T047 (Tips database) can run in parallel [P]
2. T048-T057 (Interactive features) mostly sequential
3. T058-T060 (Integration) depends on US1+US2

### Parallel Opportunities

- All Setup tasks (T003-T005) marked [P] can run in parallel
- All Foundational tasks (T006-T008, T010-T011) marked [P] can run in parallel within Phase 2
- Once Foundational completes, all user stories can START in parallel (different developers)
- Within US1: T012-T013 exploration parallel, T018-T022 Transformer can overlap with CNN
- Within US2: T034-T035 database work parallel
- Within US3: T046-T047 tips database parallel
- Phase 6 polish tasks: T061-T063, T065-T066, T068, T071 all marked [P] can run in parallel

---

## Parallel Example: User Story 1

```bash
# After Foundational phase completes, launch US1 exploration tasks together:
Task T012: "Create exploration notebook notebooks/explore_ingredients/explore_ingredient_dataset.ipynb"
Task T013: "Create preprocessing exploration notebook notebooks/explore_ingredients/explore_image_preprocessing.ipynb"

# While CNN development (T014-T017) progresses, Transformer team can start in parallel:
Task T018: "Create recipe generation inference notebook notebooks/model_recipe_generation/model_transformer_inference.ipynb"
Task T019: "Implement parallel recipe generation ..."
Task T020: "Add recipe parsing and structuring cells ..."

# RNN must wait for Transformer recipes, but integration can be prepared in parallel with RNN development
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T005)
2. Complete Phase 2: Foundational (T006-T011) - CRITICAL blocker
3. Complete Phase 3: User Story 1 (T012-T033)
4. **STOP and VALIDATE**: Test with multiple ingredient photos (chicken, salmon, vegetables), verify 5 recipes generated in <5 seconds, confirm cuisine diversity ≥ 3
5. Deploy/demo MVP

**MVP Deliverable**: Users can upload ingredient photos and receive 5 diverse recipe suggestions with cooking time and difficulty - core value proposition delivered

### Incremental Delivery

1. Complete Setup + Foundational (T001-T011) → Foundation ready
2. Add User Story 1 (T012-T033) → Test independently → **Deploy MVP!**
3. Add User Story 2 (T034-T045) → Test independently → Deploy with nutrition
4. Add User Story 3 (T046-T060) → Test independently → Deploy full beginner experience
5. Polish (T061-T071) → Final quality improvements

**Each story adds value without breaking previous stories**

### Parallel Team Strategy

With multiple developers after Foundational phase (T011) completes:

- **Developer A**: User Story 1 (CNN + Transformer + RNN) - Critical path
- **Developer B**: User Story 2 (PGM nutrition) - Can work independently
- **Developer C**: User Story 3 (Interactive guidance) - Can work independently
- **Developer D**: Foundational utilities refinement

Stories integrate at pipeline level (notebooks/pipeline_recipe_app/) after independent development

---

## Notes

- **[P] tasks**: Different files, no dependencies - safe to parallelize
- **[Story] labels**: Map tasks to user stories for traceability and independent testing
- **No test tasks**: Tests not explicitly requested in spec.md, so excluded per template guidance
- **Notebook-first**: All implementations in Jupyter notebooks per constitution
- **5-second budget**: Total pipeline time for US1 must be < 5 seconds (CNN 100ms + Transformer 3s + RNN 5×50ms + overhead)
- **Model sizes**: Total ~2GB models, peak 4GB RAM usage within 8GB constraint
- **Reproducibility**: All notebooks use seed=42 for NumPy, PyTorch, TensorFlow
- **Data versioning**: Track dataset versions, checksums, and splits in manifest
- **Commit strategy**: Commit after each task or logical group, verify notebook executes top-to-bottom
- **Checkpoints**: Stop at each user story phase end to validate independently before proceeding
