# Proposed Changes to technical-environment.md

**Status**: Proposal for team review (Q6: C). `technical-environment.md` has **not** been edited. Zhiqu (document owner) and the team decide which changes to apply.
**Basis**: `aidlc-docs/inception/requirements/requirements.md`

Each change lists the current text, the proposed text, and the requirement it comes from.

---

## C-1 Intro paragraph

- **Current**: "The user uploads one photo ... chooses servings, maximum total time, a calorie range per serving ... A recipe agent then suggests up to three recipes ... approximate calories per serving"
- **Proposed**: "The user completes a short onboarding for dietary preferences, then either (Generator) uploads one to three receipt photos, confirms a combined ingredient list, and chooses the number of dishes (1-3), a maximum total time for the whole set (15/30/45/60 min), a per-dish calorie range, and a cuisine; or (Planner) describes what they want to eat and gets recipes plus a combined shopping list. An app-coordinated pipeline generates the dishes, checks them, and asks the model to revise when needed. Users can refine the plan with action buttons or a chat box."
- **Source**: FR-1, FR-2.1, FR-4, FR-5, FR-8, FR-9

## C-2 Frameworks: Streamlit bullets

- **Add**: onboarding / preferences screen; read-only "on receipt" quantity column and highlighted unclear rows in `st.data_editor`; preference form with dish count (1-3), time (15/30/45/60), calorie range (4 options), cuisine; follow-up action buttons and `st.chat_input`; Planner page with shopping list.
- **Remove**: "preference form (servings, max time, calorie range, cuisine)".
- **Source**: FR-1, FR-3, FR-4, FR-8, FR-9

## C-3 Anthropic SDK bullet

- **Current**: "... and **tool use** (the recipe agent)"
- **Proposed**: "... **vision** (receipts), and **structured JSON output** (ingredient list, recipe plans, follow-up responses). Tool use is not required."
- **Source**: FR-5.1 (Q5: B)

## C-4 Pydantic model names

- **Proposed list**: `ReceiptItem` (add `explanation`, keep `quantity`/`unit` as on-receipt info), `ConfirmedIngredient`, `DietaryPreferences`, `RecipePreferences` (dish_count, max_total_minutes, calorie_range, cuisine), `Recipe`, `MealPlan` (replaces `RecipeSet`; includes total time, total calories, assumptions, Plan B flags), `ShoppingListItem`, `FollowUpRequest`.
- **Source**: FR-1, FR-2.4, FR-4, FR-6, FR-7, FR-9

## C-5 Stage 1: Receipt interpretation

- **Current**: "Input: one receipt photo." / "one vision call"
- **Proposed**: "Input: one to three receipt photos (one vision call per image, results merged into one list). Each item also includes a short plain-language `explanation` for unclear or unfamiliar products. Repeated items are flagged for review. Quantity and unit are shown read-only as 'on receipt' and never treated as the amount available. If one image is unreadable, the app asks for a clearer copy of that image only."
- **Source**: FR-2.1, FR-2.4, FR-2.6, FR-2.7, FR-3.4

## C-6 Stage 2: replace "Recipe agent (tool-using agent)" with "Recipe pipeline (app-coordinated)"

- **Proposed text**:
  - **Input**: confirmed ingredients, `RecipePreferences` (dish count 1-3; max total time 15/30/45/60 for all dishes; per-dish calorie range Below 400 / 400-below 600 / 600-800 / No preference; cuisine), and `DietaryPreferences`.
  - **Output**: a `MealPlan` with exactly the requested number of dishes, or a conflict explanation plus Plan B.
  - **Pipeline** (fixed order, run by the app):
    1. `search_reference_recipes` - app looks up matching reference dishes and adds them to the prompt.
    2. Model generates the plan with per-ingredient calorie estimates.
    3. `calculate_calories` - Python totals per dish and for the plan (no servings division).
    4. `check_recipe_constraints` - total time, per-dish calorie range, cuisine, dish count, dietary preferences, unconfirmed ingredients.
    5. If checks fail, the app sends the problems back and asks for a revision: **at most 2 revisions (3 generation calls)**. Still failing -> explain the conflict and offer Plan B.
- **Remove**: "Claude decides which tools to call", "The loop stops after 10 tool calls", servings 1-6, 10-120 slider, min/max kcal per serving.
- **Tool table**: rename column "Tool" to "Pipeline function"; `calculate_calories` returns per-dish and plan totals (no servings); `check_recipe_constraints` adds dish count and dietary checks.
- **Source**: FR-4, FR-5, FR-7 (Q5: B, CQ2: A, CQ3: A)

## C-7 New sections to add

- **Plan B**: conflict explanation first; changes clearly identified; labeled "uses confirmed ingredients" or "requires additional purchases"; dietary rules still apply. (FR-7)
- **Follow-up**: action buttons with structured inputs plus a chat box (500 char limit); changes re-run steps 3-5 of the pipeline. (FR-8)
- **Planner**: free-text description (500 chars), same pipeline, combined shopping list with recipe units plus suggested store unit, auto "already have" from Generator list (undoable). (FR-9)

## C-8 Session data

- **Current**: "No database ... live in Streamlit `st.session_state` for the current session only."
- **Proposed**: "No database or accounts. Ingredient list, plans, receipt images (in memory), and conversation live in `st.session_state` for the session only. **Exception**: dietary preferences are saved to a local, git-ignored JSON file until the user clicks 'Clear preferences'."
- **Source**: FR-1.4, FR-1.6, FR-2.10, NFR-1.7

## C-9 Testing

- **Add**: Hypothesis for property-based testing (pure functions and round-trips; seed logging). Receipt fixtures cover Wegmans, Tops, Aldi, Walmart / Target, and an Asian grocery store. Live test reports the percentage found with no pass/fail threshold.
- **Update example test**: replace `test_calories_divided_by_servings` and the `servings` arguments with per-dish / plan-total examples; replace `test_agent_returns_at_most_three_recipes` with a pipeline test asserting at most 3 generation calls.
- **Source**: NFR-5.5, NFR-5.6, NFR-6 (Q7, Q21, Q25)

## C-10 Do NOT Use table

- **Update reason** for LangChain row: "one vision call and an app-coordinated pipeline" instead of "one agent with three tools".
- **Update** database row "Use Instead": "`st.session_state`; local JSON file for dietary preferences only".

## C-11 Security Basics

- **Uploads**: "one to three images per request" instead of "one image at a time".
- **Prompt injection**: add chat messages and Planner descriptions (500 chars) and dietary free text (200 chars) as data-not-instructions inputs.
- **Limits**: replace "10 tool calls" with "at most 3 generation calls (2 revisions)"; keep 4,000 output tokens per call.
- **Logging**: also never log dietary preferences.
- **API failures**: one automatic retry, then friendly error with "Try again"; user data kept.
- **Source**: FR-2.1, FR-5.6, FR-10, NFR-2

## C-12 Example code patterns

- Update the Streamlit example to use dish count, 15/30/45/60 time options, calorie-range radio, and `run_recipe_pipeline` instead of `run_recipe_agent`.
- Update `calculate_calories` example to return per-dish totals with no `servings` parameter.
