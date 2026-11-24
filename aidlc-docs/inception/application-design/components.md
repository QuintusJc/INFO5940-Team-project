# ReceiptChef - Components

**Sources**: `application-design-plan.md` answers (Q1 B, Q2 A, Q3 A, Q4 A, Q5 A, Q6 A, Q7 A, Q8 A, Q9 A, Q10 B, Q11 B, Q12 A), `requirements.md`, `stories.md`

## 1. Package Layout (Q1: B - Layered)

```text
src/receiptchef/
+-- app.py                    # Entry point: uv run streamlit run src/receiptchef/app.py
+-- ui/                       # Layer 1: Streamlit only
|   +-- pages/
|   |   +-- onboarding.py
|   |   +-- generator.py
|   |   +-- planner.py
|   |   +-- settings.py
|   +-- widgets/
|   |   +-- ingredient_table.py
|   |   +-- preference_form.py
|   |   +-- plan_view.py
|   |   +-- followup_panel.py
|   |   +-- shopping_list_view.py
|   |   +-- notices.py
|   +-- session.py            # SessionStore (Q3: A)
+-- services/                 # Layer 2: orchestration, no Streamlit imports
|   +-- extraction_service.py
|   +-- recipe_pipeline.py
|   +-- followup_service.py
|   +-- planner_service.py
|   +-- prompts.py            # Q4: A
+-- domain/                   # Layer 3: models and pure functions, no I/O
|   +-- models.py
|   +-- results.py            # Result objects (Q11: B)
|   +-- ingredients.py        # merge, duplicate flags
|   +-- calories.py
|   +-- checks.py
|   +-- dietary.py
|   +-- reference_search.py
|   +-- shopping.py
+-- infra/                    # Layer 4: external I/O
    +-- config.py
    +-- llm_client.py         # LLMClient protocol + AnthropicLLMClient (Q12: A)
    +-- image_prep.py
    +-- preferences_store.py
    +-- data_loader.py
    +-- logging_setup.py

data/
+-- reference_recipes.json    # about 30 dishes per cuisine
+-- dietary_terms.json        # Q8: A term list per restriction

tests/
+-- fakes/fake_llm_client.py  # FakeLLMClient with recorded responses
+-- strategies.py             # Hypothesis domain generators (PBT-07)
+-- fixtures/receipts/ ...

.receiptchef/preferences.json # Q10: B, created at runtime, listed in .gitignore
```

**Layer rule**: each layer may only import from layers below it. `domain/` imports nothing from the project except itself. Only `infra/llm_client.py` imports `anthropic`. Only `ui/` and `app.py` import `streamlit`.

---

## 2. Component Catalog

### Layer 1 - UI

| ID | Component | Purpose | Responsibilities | Interface | Stories |
|---|---|---|---|---|---|
| C-01 | App shell (`app.py`) | Entry point and navigation | Load settings; show setup message if API key missing; route to onboarding on first launch; register pages with `st.navigation` (Q2: A); build services once per session | Streamlit entry point | TRU-01, ONB-01 |
| C-02 | Onboarding page | First-launch explanation and dietary form | Show intro, checklist, free text (200 chars), "I have none", "Skip" | Page function | ONB-01 |
| C-03 | Generator page | Receipt-to-recipe flow | Upload, extract, review table, confirm, preferences, show plan / Plan B, follow-up panel, handoff to Planner | Page function | UPL, REV, GEN, RES, PLB, FUP |
| C-04 | Planner page | Description-to-shopping-list flow | Description (500 chars), preferences, plan, follow-up panel, shopping list with "already have" | Page function | PLN, FUP |
| C-05 | Settings page | Manage preferences | View, edit, clear; notice when current plan used old preferences | Page function | ONB-03 |
| C-06 | UI widgets | Reusable display and input pieces | Ingredient table (highlights, "On receipt" column, limits), preference form, plan view (cards, totals, labels), follow-up panel (buttons with structured inputs, chat), shopping list view, notices | Render functions taking domain objects | REV, GEN-01, RES, PLB, FUP, PLN, TRU-03 |
| C-07 | SessionStore | Typed access to session data (Q3: A) | Hold images (in memory), extraction results, confirmed list, recipe preferences, current plan and plan history, conversation, planner state; no raw `st.session_state` keys in pages | Class with typed getters / setters | UPL-03, FUP-06, FUP-08, ERR-01 |

### Layer 2 - Services

| ID | Component | Purpose | Responsibilities | Interface | Stories |
|---|---|---|---|---|---|
| C-08 | ReceiptExtractionService | Turn images into one ingredient list | One vision call per image, sequentially (Q6: A); schema validation with one retry (FR-2.8); per-image status; merge and duplicate flags via domain | `extract(...) -> ExtractionResult` | UPL-02, UPL-03, REV-01, TRU-02 |
| C-09 | RecipePipeline | Shared generate-check-revise loop (Q5: A) | Reference lookup, prompt build, generate, calculate, check, revise (max 2), Plan B / no-safe-plan / insufficient outcomes | `run(PlanRequest) -> PipelineResult` | GEN-02, PLB-01 to 03, FUP-09, PLN-01 |
| C-10 | FollowUpService | Buttons and chat (Q7: A) | Route fixed buttons to explain-only or plan-change; chat call returns `answer` or `plan_change`; plan changes go through RecipePipeline; local computation for "what else to buy" | `handle_action(...)`, `handle_chat(...)` | FUP-01 to 09, PLB-03 |
| C-11 | PlannerService | Planner entry and handoff | Build Planner requests for RecipePipeline; build shopping list; Generator-to-Planner handoff without a new model call | `propose(...)`, `shopping_list(...)`, `from_generator(...)` | PLN-01 to 04 |
| C-12 | Prompts (`prompts.py`) | All prompt text in one place (Q4: A) | One function per prompt; user and receipt text placed in clearly delimited data sections (NFR-2.4) | Pure functions returning prompt parts | GEN-02, FUP-08, TRU-03 |

### Layer 3 - Domain

| ID | Component | Purpose | Responsibilities | Interface | Stories |
|---|---|---|---|---|---|
| C-13 | Models (`models.py`) | All structured data (Pydantic v2) | Field limits (60 chars, 60 items, 200 / 500 chars) enforced by validators | Pydantic classes | All |
| C-14 | Results (`results.py`) | Outcome objects (Q11: B) | `ExtractionResult`, `PipelineResult`, `FollowUpResult`, `LLMResponse`, `PreferencesLoadResult`, `PrepareResult` with status enums | Pydantic classes | ERR-01, PLB |
| C-15 | Ingredient merge (`ingredients.py`) | Combine receipts | Merge per-image items, drop non_food, flag duplicates, apply user edits | Pure functions | UPL-02, REV-02, REV-03 |
| C-16 | Calorie calculator (`calories.py`) | Arithmetic in Python | Per-dish and plan totals from per-ingredient estimates; reject impossible values; whole-number display | Pure functions | GEN-02, RES-01, RES-02 |
| C-17 | Constraint checker (`checks.py`) | Plan checks | Time, per-dish calories, cuisine, dish count, dietary (via C-18), unconfirmed ingredients; produce `CheckProblem` list and the "changes from request" list for Plan B | Pure functions | GEN-02, PLB-01, PLB-03 |
| C-18 | Dietary rules (`dietary.py`) | Dietary conflict detection (Q8: A) | Match ingredient names against `dietary_terms.json`; combine with model allergen tags; conflict from either source fails | Pure functions | ONB-02, PLB-03, PLN-01 |
| C-19 | Reference search (`reference_search.py`) | Pick reference dishes | Rank reference dishes by cuisine and ingredient overlap | Pure function | GEN-02 |
| C-20 | Shopping list builder (`shopping.py`) | Combined list | Combine quantities across dishes, attach model store-unit hints (Q9: A), auto-mark "already have" from confirmed list, manual marks | Pure functions | PLN-02, PLN-03, PLN-04 |

### Layer 4 - Infrastructure

| ID | Component | Purpose | Responsibilities | Interface | Stories |
|---|---|---|---|---|---|
| C-21 | Config (`config.py`) | Settings from environment | Load `.env`; `ANTHROPIC_API_KEY` (optional at load, reported missing), `RECEIPTCHEF_MODEL`, preferences path | `load_settings() -> Settings` | TRU-01 |
| C-22 | LLM client (`llm_client.py`) | Only place that calls Anthropic (NFR-1.4, Q12: A) | `LLMClient` protocol; `AnthropicLLMClient` with one automatic retry on API failure, 4,000 output-token cap, JSON parsing and Pydantic validation | Protocol + class | All model stories, ERR-01 |
| C-23 | Image preparation (`image_prep.py`) | Safe images | Validate type and size, resize to 1568 px, strip EXIF, keep in memory | `prepare_image(...) -> PrepareResult` | UPL-01, TRU-02 |
| C-24 | Preferences store (`preferences_store.py`) | Local JSON file (Q10: B) | Read / write / delete `.receiptchef/preferences.json` in the project folder; report missing or corrupt | Class | ONB-01 to 03 |
| C-25 | Data loader (`data_loader.py`) | Bundled data | Load and validate `reference_recipes.json` and `dietary_terms.json` once | Functions | GEN-02, PLB-03 |
| C-26 | Logging setup (`logging_setup.py`) | Safe logging | Structured log events with allowed fields only (NFR-2.6) | `log_event(...)` | TRU-02 |

### Test Support (outside `src/`)

| ID | Component | Purpose |
|---|---|---|
| T-01 | `FakeLLMClient` | Implements `LLMClient` with recorded responses; counts calls for "at most 3 generation calls" tests |
| T-02 | Hypothesis strategies (`tests/strategies.py`) | Reusable domain generators (PBT-07) |

---

## 3. Design Decisions Made During This Stage

These follow from the answers but were not asked directly. Please review them.

| ID | Decision | Reason |
|---|---|---|
| D-1 | **Plan B stays within the 3-call cap.** The generation response schema lets the model return either a plan that meets the request or a best-effort plan with stated conflicts. If the plan still fails non-dietary checks after the last allowed call, the app shows that plan **as Plan B**, and the "Changes from your request" list comes from the checker's problem list, not from the model. If dietary or confirmed-ingredient checks still fail, no plan is shown (`no_safe_plan`). | Keeps FR-5.6 (max 3 calls) and FR-7.4 (never mislabeled): unmet conditions are computed by Python. Dietary problems can never become a "change" (US-PLB-03). |
| D-2 | Multipage navigation uses `st.navigation` with page files in `ui/pages/`, so the layered layout (Q1: B) works with Streamlit pages (Q2: A). | Streamlit's legacy `pages/` folder must sit next to the entry file; `st.navigation` removes that constraint. |
| D-3 | `prompts.py` sits in `services/`. | Prompts are part of orchestration, not pure domain logic. |
| D-4 | Free-text dietary restrictions ("Other restrictions") are checked by model tags only, because the term list covers checklist items only. The UI notes that free-text items are checked by AI. | Q8: A term list cannot cover arbitrary text. |
| D-5 | "What else do I need to buy?" (US-FUP-07) is computed from the current plan without a model call. | The plan already lists additional and pantry items; saves cost. |
| D-6 | The LLM client path changes from `src/receiptchef/llm_client.py` (NFR-1.4) to `src/receiptchef/infra/llm_client.py`. The single-client rule is unchanged. Added as C-13 in `technical-environment-proposed-changes.md`. | Follows Q1: B. |
