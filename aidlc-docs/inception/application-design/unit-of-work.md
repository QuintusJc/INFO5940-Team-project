# ReceiptChef - Units of Work

**Stage**: INCEPTION - Units Generation (Part 2)
**Approved plan**: `aidlc-docs/inception/plans/unit-of-work-plan.md` (Q1 A, Q2 A, Q3 A -> Follow-Up 1 B, Q4 A, Q5 C -> Follow-Up 2 A, Q6 A, Q7 A)
**Related**: `unit-of-work-dependency.md`, `unit-of-work-story-map.md`, `components.md`

ReceiptChef is **one deployable application** (a local Streamlit app). The five units below are **modules** inside it, used to divide, order, and review Construction work. Each unit goes through Functional Design, NFR Requirements, NFR Design, and Code Generation in Construction; Build and Test runs once after all units.

---

## 1. Unit Summary

| Unit | Purpose | Stories | Owner(s) | Build order |
|---|---|---|---|---|
| U1 Foundation | Shared models, LLM client, config, preferences, session, app shell, onboarding | 5 | *To be assigned (Thursday meeting)* | 1 |
| U2 Receipt Intake | Images to a confirmed ingredient list | 7 | *To be assigned (Thursday meeting)* | 2 (parallel with U3) |
| U3 Recipe Engine | Preferences, pipeline, checks, results, Plan B | 8 | *To be assigned (Thursday meeting)* | 2 (parallel with U2) |
| U4 Follow-Up | Action buttons and chat | 9 | *To be assigned (Thursday meeting)* | 3 |
| U5 Planner | Planner flow and shopping list | 4 | *To be assigned (Thursday meeting)* | 4 |
| **Total** | | **33** | | |

**Ownership** (Follow-Up 1 B): owners are left blank on purpose. The whole team will agree on owners at the Thursday meeting (2026-11-26), before Construction starts, and ask the agent to record them through the workflow.

---

## 2. Unit Definitions

### U1 Foundation

- **Purpose**: Everything other units build on: the shared contracts, the only LLM client, configuration, preferences, session state, and the app shell.
- **Components**: C-01 app shell; C-02 onboarding page; C-05 settings page; C-06 `notices.py` widget; C-07 SessionStore; C-13 models (initial set); C-14 results (initial set); C-21 config; C-22 LLM client (`LLMClient` protocol and `AnthropicLLMClient`); C-24 preferences store; C-25 data loader (generic load-and-validate helper and path handling); C-26 logging; T-01 `FakeLLMClient`; T-02 Hypothesis strategies (initial set)
- **Responsibilities**:
  - Define the shared contracts: `models.py`, `results.py`, `LLMClient` protocol (Q4 A)
  - Call the Anthropic API with one automatic retry and the 4,000-token cap; return `LLMResponse` results
  - Load settings; show the setup message when the API key is missing
  - Save, load, and clear `.receiptchef/preferences.json`; add `.receiptchef/` to `.gitignore`
  - Register pages with `st.navigation`; route first launch to onboarding
  - Provide safe logging and the standard notices
  - Project setup: `pyproject.toml`, `uv.lock`, ruff, mypy, pytest, pytest-cov, Hypothesis, `.env.example`
- **Provides to other units**: models and results, `LLMClient` and `FakeLLMClient`, SessionStore, PreferencesStore, `log_event`, notices, data loading helper, base Hypothesis strategies
- **Uses**: nothing from other units
- **Definition of done**: app starts; setup message shown without a key; onboarding saves, reloads, edits, and clears preferences; `FakeLLMClient` and retry behavior tested; PBT round-trip for preferences JSON and model serialization; integration checkpoint 1 passes (Section 5)

### U2 Receipt Intake

- **Purpose**: Turn 1-3 receipt photos into a reviewed, confirmed ingredient list.
- **Components**: C-23 image preparation; C-08 ReceiptExtractionService; C-15 ingredient merge (`ingredients.py`); C-06 `ingredient_table.py` widget; C-03 Generator page (creates the page with its upload, review, and confirm sections); C-12 `extraction_prompt()` in `prompts.py`
- **Responsibilities**:
  - Validate, resize, and strip EXIF from images; keep them in memory only
  - One vision call per image in order; validation retry once; per-image status
  - Merge items, drop non_food, flag duplicates; review table with highlights and "On receipt" column
  - Block confirmation until review is complete; produce `ConfirmedIngredientList`
- **Provides**: `ConfirmedIngredientList` in SessionStore; the Generator page skeleton
- **Uses**: U1 (models, `LLMClient`, SessionStore, logging)
- **Prerequisite**: receipt fixtures collected before U2 Construction begins (Q7 A, Section 6)
- **Definition of done**: Journey A steps 1-5 pass with `FakeLLMClient` and one manual real-API run; live receipt test reports the extraction percentage; PBT invariants for merging (for example no non_food rows; every food item appears)

### U3 Recipe Engine

- **Purpose**: Create checked meal plans and handle Plan B, used by Generator, Planner, and follow-ups.
- **Components**: C-09 RecipePipeline; C-12 `prompts.py` (owner; generation and revision prompts); C-16 calories; C-17 checks; C-18 dietary rules; C-19 reference search; C-06 `preference_form.py` and `plan_view.py` widgets; C-03 Generator page (adds preferences and results sections); typed loaders for `reference_recipes.json` and `dietary_terms.json` (added to C-25); data files `data/reference_recipes.json` and `data/dietary_terms.json`
- **Responsibilities**:
  - Reference lookup, generation, Python calorie totals, all checks, up to 2 revisions (3 calls)
  - Plan B, `insufficient_ingredients`, and `no_safe_plan` outcomes (design decision D-1)
  - Dietary conflict detection from the team's term list plus model tags; dietary problems are never a "change"
  - Dish cards, plan totals, Plan B explanation and labels, notices on results
- **Provides**: `RecipePipeline.run(PlanRequest) -> PipelineResult`; plan view widget
- **Uses**: U1; U2's confirmed list at integration (a sample confirmed list until U2 is ready, Q2 A)
- **Data gate** (Follow-Up 2 A): see Section 6. Code Generation for the dietary check (C-18 and the dietary part of C-17) starts only after the team's `dietary_terms.json` is written and reviewed. **No placeholder dietary file is used.**
- **Definition of done**: Journey A steps 6-7, Journey B steps 1-5 and 7, Journey C step 1 pass; "at most 3 generation calls" tested with `FakeLLMClient`; PBT invariants for calorie totals and checks; team has reviewed the AI-drafted `reference_recipes.json`

### U4 Follow-Up

- **Purpose**: Let users change or ask about the current plan with buttons or chat.
- **Components**: C-10 FollowUpService; C-06 `followup_panel.py` widget; C-12 `explain_prompt()` and `chat_prompt()` in `prompts.py`; C-03 Generator page (adds the follow-up panel); C-07 SessionStore additions (plan history, conversation, if not already in U1)
- **Responsibilities**:
  - Fixed routing of the 7 buttons (explain-only vs plan-change); "What else do I need to buy?" with no model call
  - Chat with `answer` / `plan_change` / `declined` replies; plan changes go through RecipePipeline
  - Keep the previous plan for "Other meals"; conversation is session only
- **Provides**: follow-up panel reused by the Planner page
- **Uses**: U1, U3 (RecipePipeline, plan view); `domain.shopping.additional_items` (see note in Section 3)
- **Definition of done**: Journey A steps 8-9, Journey B step 6, Journey C step 2 pass; chat cannot override dietary preferences (US-PLB-03 Scenario 5)

### U5 Planner

- **Purpose**: Turn a meal description, or a Generator plan, into a combined shopping list.
- **Components**: C-11 PlannerService; C-20 shopping list builder (`shopping.py`); C-04 Planner page; C-06 `shopping_list_view.py` widget
- **Responsibilities**:
  - Planner requests through RecipePipeline; Generator-to-Planner handoff with no model call
  - Combine quantities, attach store-unit hints, auto-mark and manually mark "already have"
  - Recalculate the list whenever the plan changes
- **Uses**: U1, U3 (RecipePipeline, plan view), U4 (follow-up panel on the Planner page)
- **Definition of done**: Journey C steps 3-4 and Journey D pass; PBT invariants for shopping lists (combined totals equal the sum across dishes; marking never adds to-buy items)

---

## 3. Code Organization Strategy (Greenfield)

**Source layout**: the layered layout approved in Application Design (Q1 B) is used:

```text
src/receiptchef/
+-- app.py
+-- ui/        (pages, widgets, session.py)
+-- services/  (extraction_service, recipe_pipeline, followup_service, planner_service, prompts)
+-- domain/    (models, results, ingredients, calories, checks, dietary, reference_search, shopping)
+-- infra/     (config, llm_client, image_prep, preferences_store, data_loader, logging_setup)
```

**Deviation from the default monolith pattern**: `code-generation.md` suggests `src/{unit-name}/` for multi-unit monoliths. ReceiptChef keeps the approved **layered source layout** instead, because units cut across layers (for example U3 has domain, service, and UI parts). Units are tracked by **file ownership** (table below). Tests follow the unit pattern:

```text
tests/
+-- fakes/fake_llm_client.py      (U1)
+-- strategies.py                 (U1 creates; each unit adds its strategies)
+-- u1_foundation/
+-- u2_receipt_intake/
+-- u3_recipe_engine/
+-- u4_follow_up/
+-- u5_planner/
+-- integration/                  (journey checkpoints, Section 5)
+-- fixtures/receipts/            (Q7 A)
```

**File ownership**:

| File | Owner unit | Other units that add to it |
|---|---|---|
| `app.py`, `ui/session.py`, `ui/pages/onboarding.py`, `ui/pages/settings.py`, `ui/widgets/notices.py` | U1 | U4 (session additions if needed) |
| `domain/models.py`, `domain/results.py` | U1 | All (new models; field changes per Q4 A rule) |
| `infra/*` (all) | U1 | U3 (typed loaders in `data_loader.py`) |
| `ui/pages/generator.py` | U2 | U3 (preferences, results), U4 (follow-up panel) |
| `ui/widgets/ingredient_table.py`, `services/extraction_service.py`, `domain/ingredients.py` | U2 | - |
| `services/prompts.py` | U3 | U2 (`extraction_prompt`), U4 (`explain_prompt`, `chat_prompt`) |
| `services/recipe_pipeline.py`, `domain/calories.py`, `domain/checks.py`, `domain/dietary.py`, `domain/reference_search.py`, `ui/widgets/preference_form.py`, `ui/widgets/plan_view.py`, `data/*.json` | U3 | - |
| `services/followup_service.py`, `ui/widgets/followup_panel.py` | U4 | - |
| `services/planner_service.py`, `domain/shopping.py`, `ui/pages/planner.py`, `ui/widgets/shopping_list_view.py` | U5 | U4 (`additional_items`, see note) |

**Note on `domain/shopping.py`**: U4 needs `additional_items(plan)` for "What else do I need to buy?" before U5 starts. U4 creates `domain/shopping.py` with that one function; U5 owns the file afterwards and adds the shopping-list functions.

**Shared-file rule**: when a unit adds to a file owned by another unit, it adds new functions or sections only and does not change existing ones without the owner's agreement (same rule as Q4 A).

---

## 4. Shared Contracts (Q4 A)

| Contract | Defined in | Change rule |
|---|---|---|
| `domain/models.py`, `domain/results.py` | U1 (initial set: preferences, ingredient rows, recipe preferences, plan, results) | Later units may **add** models. Changing an existing field requires a short note in the changing unit's Functional Design and agreement from the owners of affected units. |
| `LLMClient` protocol, `FakeLLMClient` | U1 | Same rule |
| `PlanRequest` / `PipelineResult` | U3 | Same rule; U4 and U5 are the affected units |

---

## 5. Integration Checkpoints (Q6 A)

After each unit, run the journey steps that unit enables: automated with `FakeLLMClient` (in `tests/integration/`) and **one manual run with the real API**.

| After | Checkpoint |
|---|---|
| U1 | App starts; setup message without key; onboarding save / reload / edit / clear |
| U2 | Journey A steps 1-5 |
| U3 | Journey A steps 6-7; Journey B steps 1-5 and 7; Journey C step 1 |
| U4 | Journey A steps 8-9; Journey B step 6; Journey C step 2 |
| U5 | Journey C steps 3-4; Journey D (all steps) |

---

## 6. Prerequisites and Gates

| Item | Who | When | Blocks |
|---|---|---|---|
| Unit owners agreed and recorded | Whole team | Thursday meeting (2026-11-26), before Construction | Construction start |
| Receipt fixtures: each member collects 1-2 receipts (Wegmans, Tops, Aldi, Walmart / Target, Asian grocery store), blurs card numbers and loyalty IDs, writes the expected-ingredients JSON for their own receipts | Each team member | Before U2 Construction begins | U2 Construction |
| `data/dietary_terms.json` written by hand in the format defined by U3 Functional Design | Team | Between U3 Functional Design and U3 Code Generation | U3 Code Generation for the dietary check |
| `data/dietary_terms.json` reviewed | Team | Before dietary-check code generation | Same |
| `data/reference_recipes.json` drafted | AI | During U3 Code Generation | - |
| `data/reference_recipes.json` reviewed | Team | Before U3 is marked complete | U3 completion |

**U3 Code Generation ordering**: the U3 code generation plan puts the dietary-check steps (C-18 and the dietary part of C-17, with their tests) after an explicit gate step "Team `dietary_terms.json` written and reviewed". Other U3 steps may run before the gate. No placeholder dietary file is created at any point.

---

## 7. Estimated Schedule

Estimates only, based on the December 16 deadline (see `execution-plan.md`).

| Window | Work |
|---|---|
| 2026-11-26 | Thursday meeting: owners agreed and recorded |
| 2026-11-27 to 12-01 | U1 Foundation |
| 2026-12-01 to 12-07 | U2 Receipt Intake and U3 Recipe Engine in parallel (U3 dietary gate inside this window) |
| 2026-12-07 to 12-09 | U4 Follow-Up |
| 2026-12-09 to 12-12 | U5 Planner |
| 2026-12-12 to 12-15 | Build and Test |
| 2026-12-16 | Final deadline |
