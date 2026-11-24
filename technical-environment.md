# Technical Environment: ReceiptChef

ReceiptChef is a local web app that helps international students turn groceries they have already bought into practical meals, and turn meal ideas into shopping lists. On first launch the user completes a short onboarding for allergies and dietary restrictions. Then the user either:

- **Generator:** uploads one to three English-language grocery receipt photos, reviews and confirms a combined ingredient list, and chooses the number of dishes (1-3), a maximum total time for the whole set of dishes (15 / 30 / 45 / 60 minutes), an estimated calorie range for each dish, and a cuisine (Chinese, Italian, Mexican, or No preference); or
- **Planner:** describes what they want to eat, chooses the same preferences, and gets recipes plus a combined shopping list.

An app-coordinated pipeline generates the dishes, checks them in Python, and asks the model to revise when needed. Each dish has ingredient quantities, steps, approximate calories for the dish, and any additional ingredients needed; the plan shows the total time and total calories for all dishes. If the conditions cannot all be met, the app explains the conflict and offers a Plan B, and dietary restrictions are never relaxed. Users refine the plan with follow-up action buttons or a chat box.

Requirements are defined in `aidlc-docs/inception/requirements/requirements.md`; component design is in `aidlc-docs/inception/application-design/`.

## Language and Package Manager

- **Python 3.12+**
- **uv** for all package management (no pip, poetry, or conda)
- `pyproject.toml` for all project and tool configuration
- `uv.lock` committed to Git
- Application code lives in `src/receiptchef/` in four layers, tests in `tests/`, bundled data in `data/`. AI-DLC artifacts stay in `aidlc-docs/`, and no application code goes there.

```text
src/receiptchef/
+-- app.py      # entry point
+-- ui/         # Streamlit pages, widgets, SessionStore (only layer that imports streamlit)
+-- services/   # extraction, recipe pipeline, follow-up, planner, prompts
+-- domain/     # Pydantic models, result objects, pure functions (no I/O)
+-- infra/      # config, llm_client, image_prep, preferences_store, data_loader, logging
data/
+-- reference_recipes.json
+-- dietary_terms.json
```

Each layer imports only from layers below it (UI -> services -> domain; infra is used by UI and services; domain imports nothing else from the project).

## Frameworks and Libraries

- **Streamlit** for the user interface, as a multipage app registered with `st.navigation`:
  - onboarding and settings pages for dietary preferences (checklist plus optional free text)
  - receipt upload (`st.file_uploader`, up to three images)
  - editable ingredient list (`st.data_editor`), where users rename, remove, or add ingredients and pantry staples; unclear rows are highlighted with the raw receipt text and a plain-language explanation; receipt quantities appear in a read-only "On receipt" column
  - preference form: number of dishes (1, 2, 3), maximum total time (15, 30, 45, 60 minutes), calorie range per dish (Below 400, 400-below 600, 600-800 kcal, No preference), cuisine
  - recipe cards and plan totals, including Plan B explanations and labels
  - follow-up action buttons with structured inputs, and a chat box (`st.chat_input`)
  - Planner page with a combined shopping list and "already have" marks
- **Anthropic Python SDK** (`anthropic`) to call Claude, using **vision** (receipts) and **structured JSON output** (ingredient list, recipe plans, follow-up responses). Tool use is not required.
- **Pydantic v2** for every structured object passed between the model, the services, and the UI. Main models (full list in `aidlc-docs/inception/application-design/component-methods.md`):
  - `ReceiptItem` (raw text, ingredient name, on-receipt quantity and unit, status, plain-language `explanation`)
  - `IngredientRow` and `ConfirmedIngredientList`
  - `DietaryPreferences`
  - `RecipePreferences` (`dish_count`, `max_total_minutes`, `calorie_range`, `cuisine`)
  - `RecipeIngredient`, `Dish`, and `MealPlan` (total time, total calories, time assumptions, Plan B flags and changes)
  - `ShoppingListItem` and `ShoppingList`
  - `FollowUpAction` and `ChatTurn`
  - Result objects such as `ExtractionResult`, `PipelineResult`, `FollowUpResult`, and `LLMResponse`
- **Pillow** to validate, resize, and strip metadata from uploaded images before they are sent to the model
- **python-dotenv** to load the API key from a local `.env` file

## LLM Provider and Workflow

- **Provider:** Anthropic Claude API (hosted).
- **Planned model:** **Claude Sonnet 5.5** (`claude-sonnet-5-5`) for all model calls. The model name is read from the `RECEIPTCHEF_MODEL` environment variable, so the team can use `claude-haiku-5-5` during development to reduce cost.
- **Single client module:** all model calls go through `src/receiptchef/infra/llm_client.py`, the only module that imports `anthropic`. It implements an `LLMClient` protocol, so services depend on the protocol and tests can inject a fake. If API credits become a problem, only this file changes. A local Ollama vision model through Ollama's OpenAI-compatible endpoint is the backup.

The product works in model stages run by the app, with a required user confirmation step between receipt interpretation and recipe generation.

### Onboarding and dietary preferences (no model call)

- On first launch the app explains its purpose and asks about allergies and dietary restrictions: a checklist (peanuts, tree nuts, dairy, eggs, gluten, shellfish, vegetarian, vegan, halal, no pork, no beef) plus optional free text (max 200 characters). Users can save, choose "I have none", or skip.
- Preferences are applied to every recipe generation, Plan B, Planner request, and follow-up. The app never relaxes them to meet a time, calorie, or dish-count preference.

### Stage 1: Receipt interpretation (one vision call per image)

- **Input:** one to three receipt photos. The app makes one vision call per image, in order, and merges the results into one list.
- **Output:** a JSON list of `ReceiptItem` objects per image, with:
  - `raw_text`, the text as printed, for example `"GRN ONION BCH"`
  - `ingredient_name`, for example `"green onions"`
  - `quantity` and `unit`, when they can be read; shown read-only as "On receipt" and **never treated as the amount available**
  - `status`: `"food"`, `"unclear"`, or `"non_food"`
  - `explanation`: a short plain-language note for unclear or unfamiliar products, for example "a bunch of green onions / scallions"
- `non_food` items (bags, detergent, tax lines) are dropped. `unclear` items are highlighted for review. Repeated items across or within receipts are flagged for the user to merge or keep, not merged silently.
- If one image is unreadable or is not a grocery receipt, the app identifies that image and asks for a clearer copy of it only; items from the other images are still shown.
- The output is validated with Pydantic. If validation fails, the app retries once for that image, then shows an error.

### User confirmation (no model call)

- The user edits the ingredient list: correcting names, removing items they no longer have, and adding ingredients or pantry staples such as oil, salt, rice, or soy sauce. Unresolved unclear or duplicate items must be confirmed, edited, or removed before the list can be confirmed.
- There is no "amount available" input; recipes state the amount they need and the user decides whether they have enough.
- **Only the confirmed list is sent to Stage 2.**

### Stage 2: Recipe pipeline (app-coordinated)

- **Input:** the confirmed ingredients (or, in Planner, the user's description), `RecipePreferences`, and `DietaryPreferences`:
  - number of dishes: 1, 2, or 3 (no servings or number-of-people input)
  - maximum total time: 15, 30, 45, or 60 minutes, for preparing and cooking the **complete set** of dishes
  - calorie range **per dish**, using all quantities listed in that recipe: Below 400, 400-below 600, 600-800 kcal, or No preference
  - cuisine: Chinese, Italian, Mexican, or No preference
- **Output:** a `MealPlan` with exactly the requested number of dishes, or a conflict explanation plus Plan B. Each dish has a name, cuisine, ingredient quantities, steps, approximate calories for the dish, and ingredients grouped as confirmed, assumed pantry, or additional. The plan has the total time (with assumptions such as preparing one dish while another cooks) and total calories, which always equal the sum of the dish estimates.
- **No preference:** the model chooses among Chinese, Italian, and Mexican based on the ingredients and other preferences, and states the chosen cuisine on each dish.
- **Pipeline** (fixed order, run by `RecipePipeline` in `services/recipe_pipeline.py`; the model does not choose which steps run):
  1. **Reference lookup** - the app finds matching reference dishes and includes them in the prompt.
  2. **Generate** - the model returns the plan with per-ingredient calorie estimates and allergen tags.
  3. **Calculate** - Python computes each dish's calories and the plan total. Model-reported totals are never used.
  4. **Check** - Python checks total time, each dish's calorie range, cuisine, dish count, dietary preferences, and ingredients not on the confirmed list (these must be listed as additional or assumed pantry items).
  5. **Revise** - if checks fail, the app sends the specific problems back and asks for a revision: **at most 2 revisions (3 generation calls per request)**. If the plan still fails, the app explains the conflict and offers Plan B.

| Pipeline function | What it does | Implementation |
| --- | --- | --- |
| `find_references` (`domain/reference_search.py`) | Returns reference dishes for a cuisine that use some of the given ingredients, so suggestions stay close to real Chinese, Italian, and Mexican cooking | Local `data/reference_recipes.json` (about 30 dishes per cuisine: name, typical ingredients, typical total time) |
| `apply_totals` (`domain/calories.py`) | Takes the model's per-ingredient calorie estimates for the quantities used (including oil and sauces) and returns per-dish and plan totals. The model supplies the estimates; the arithmetic is done in Python so the totals are consistent. No servings division. | Pure Python function |
| `check_plan` (`domain/checks.py`) | Checks a plan against the preferences, dietary preferences, and the confirmed list. Returns problems: over max total time, a dish outside the calorie range, wrong cuisine, wrong dish count, dietary conflicts, and ingredients not on the confirmed list that are not listed as additional or assumed pantry items. | Pure Python function |
| `find_conflicts` (`domain/dietary.py`) | Detects dietary conflicts from a team-written term list (for example peanuts -> peanut oil, peanut butter, satay) plus the model's allergen tags; a conflict from either source fails the check | Pure Python function using `data/dietary_terms.json` |

Calorie estimates come from the model, with no external nutrition database or API in the MVP. They are always labeled **"approximate"** in the UI and shown as whole numbers. The selected range guides suggestions, but the app does not claim actual calories fall within it.

### Plan B

- When the selected conditions cannot all be met, the app first explains which requirements conflict, for example "Three dishes can't fit in 15 minutes with these ingredients".
- Plan B clearly lists every change from the original request (for example "2 dishes instead of 3"). The change list is computed by the Python checker, not written by the model.
- Plan B is labeled either "Uses your confirmed ingredients" or "Requires additional purchases", and is never presented as meeting the original conditions.
- Dietary restrictions still apply and are never listed as a change. If the final attempt still has a dietary or unconfirmed-ingredient problem, no plan is shown and the app explains that it could not find a safe plan.
- Plan B uses the result of the last allowed generation call, so it never adds a fourth call.
- If the confirmed ingredients are insufficient for a reasonable recipe, the app explains what is missing and offers to create a shopping list in Planner.

### Follow-up actions and chat

- Below the current plan (Generator and Planner), the app offers action buttons: Replace an ingredient; Suggest a different dish; Change the number of dishes; Adjust time or calorie preference; Explain a cooking step; Suggest other meals with these ingredients; Identify additional ingredients to buy.
- Buttons that need details use structured inputs, such as dropdowns for the dish, ingredient, or step, and a short text field (max 60 characters) for a replacement ingredient.
- A chat box (max 500 characters) answers open questions about the current plan. The model replies with an answer, a plan change, or a polite refusal (off-topic requests, or attempts to override rules or drop a saved allergy).
- Every follow-up uses the current ingredient list, current plan, and saved dietary preferences. Any change to the recipes goes back through the calculate, check, and revise steps, and updates quantities, dish calories, total calories, total time, additional ingredients, and the shopping list.

### Planner and shopping list

- The user describes what they want to eat in free text (max 500 characters) and chooses the same preferences. The model proposes dishes directly, and the user refines them through follow-up.
- Planner recipes go through the same pipeline and show per-dish calories, total calories, and total time.
- The combined shopping list totals repeated ingredients across dishes in recipe units, plus an approximate store-purchase unit suggested by the model (for example "chicken thigh: 600 g - about 1 family pack").
- If a confirmed Generator ingredient list exists in the session, matching items are automatically marked "already have"; the user can undo any mark or mark more items.
- Users can move from a Generator plan to Planner without a new model call. The shopping list is always rebuilt from the current recipes and quantities.

### Session data

- No database, no user accounts, and no ongoing pantry tracking.
- The ingredient list, receipt images (in memory only), plans, and conversation live in Streamlit `st.session_state` (through a typed `SessionStore`) for the current session only.
- **Exception:** dietary preferences are saved to a local JSON file at `.receiptchef/preferences.json` in the project folder (`.receiptchef/` is listed in `.gitignore`). They are kept until the user clicks "Clear preferences".

## Cloud and Deployment

- **The product runs locally**; there is no cloud deployment or hosting. Each teammate runs it on their laptop or in a GitHub Codespace with `uv run streamlit run src/receiptchef/app.py` and opens `http://localhost:8501`.
- The only external service is the **Anthropic API**, called over HTTPS.
- Receipt photos and results are not stored after the session ends. The only file the app writes is the dietary preferences file.
- No Docker or cloud services for the MVP.

## Testing

- **pytest** with **pytest-cov**: 80% line coverage minimum for `src/receiptchef/`, excluding the Streamlit UI code (`app.py` and `ui/`)
- **ruff** for linting and formatting
- **mypy** for type checking
- **Hypothesis** for property-based testing of pure functions and serialization round-trips (for example preferences JSON round-trip, total calories always equal the sum of dish calories, combined shopping-list totals equal the sum across dishes). Shared domain generators live in `tests/strategies.py`, and seeds are logged so failures can be reproduced.
- **No real API calls in unit tests.** Services receive an `LLMClient`; tests inject `FakeLLMClient` (`tests/fakes/fake_llm_client.py`), which returns recorded responses and counts calls, so tests are free, fast, and repeatable.
- **Pure functions** (calorie totals, plan checks, dietary conflicts, ingredient merging, shopping-list building, reference search, image preparation, Pydantic validation) are unit tested directly.
- Tests are organized by unit of work (`tests/u1_foundation/` to `tests/u5_planner/`), with journey checkpoints in `tests/integration/`.
- **Receipt fixtures:**
  - 5-10 real receipts covering Wegmans, Tops, Aldi, Walmart / Target, and at least one Asian grocery store, with card numbers and loyalty IDs blurred, stored in `tests/fixtures/receipts/`
  - each receipt paired with a JSON file of the expected ingredients
  - a manually run integration test marked `@pytest.mark.live` calls the real API and **reports** the percentage of expected ingredients found; there is no pass/fail threshold for the MVP
- **Calorie spot check:** `scripts/check_calories.py` compares about 10 calorie estimates against nutrition labels or reference values recorded in `tests/fixtures/calorie_reference.json`, to catch obvious errors. It is run manually, not in CI.

## Do NOT Use

| Prohibited | Reason | Use Instead |
| --- | --- | --- |
| LangChain, LlamaIndex, CrewAI | Extra layers we don't need for one vision call per receipt and an app-coordinated pipeline; harder to debug and test | Anthropic SDK directly |
| pytesseract, EasyOCR, other OCR libraries | Extra system dependencies, and they return raw text that still needs interpreting | Claude vision with Pydantic validation |
| External nutrition APIs (USDA FoodData Central, Nutritionix, Edamam) | Out of scope for the MVP; adds API keys and matching logic | Model calorie estimates labeled "approximate", totals computed in `domain/calories.py` |
| SQLite, PostgreSQL, Firebase, any database | No pantry tracking or accounts in the MVP | `st.session_state`; local JSON file for dietary preferences only |
| Image generation APIs (DALL·E, Stable Diffusion) | Food photos are out of scope | Text recipe cards |
| Flask, Django, React | Too much setup for a local MVP | Streamlit |
| `eval()`, `exec()`, `pickle` | Security: could run code that comes from model output | `json` parsing plus Pydantic validation |
| Hard-coded API keys, committed `.env` or `secrets.toml` | Secrets would be exposed in the repository | `.env` file listed in `.gitignore`, loaded with python-dotenv |
| pip, poetry, conda | Project uses uv only | uv |
| black, flake8, isort | Replaced by one tool | ruff |

## Security Basics

- `ANTHROPIC_API_KEY` is stored only in a local `.env` file. `.env` is in `.gitignore`, and a `.env.example` with an empty value is committed instead. If the key is missing, the app shows a setup message instead of crashing.
- **Uploads:** one to three images per request, JPEG, PNG, or WebP only, at most 10 MB each. Images are resized to a maximum of 1568 px on the long edge before sending.
- **Privacy:**
  - EXIF metadata (including GPS location) is stripped before an image leaves the machine.
  - The system prompt tells the model to ignore payment details, card numbers, and loyalty IDs and never return them.
  - Images and results are not written to disk. The only file written is the git-ignored dietary preferences file.
- **Prompt injection:** receipt text, user-typed ingredient names, Planner descriptions, follow-up inputs, chat messages, and dietary free text are treated as data, never as instructions; prompts place them in clearly delimited data sections. Limits: ingredient names 60 characters, the list 60 items, dietary free text 200 characters, Planner descriptions and chat messages 500 characters. Every model output is validated with Pydantic, and every plan is checked in Python before it is shown.
- **Pipeline safety:** pipeline functions only read the bundled `data/reference_recipes.json` and `data/dietary_terms.json` or do arithmetic and matching. Nothing runs shell commands, opens URLs, or executes model output.
- **Limits:** at most 3 generation calls (2 revisions) and 4,000 output tokens per call, to control cost and prevent runaway loops.
- **API failures:** the client retries once automatically on a network error, rate limit, or timeout; if that fails, the app shows a friendly error with a "Try again" button and keeps the user's images, ingredient list, preferences, plan, and conversation.
- **Logging:** log stage names, step names, timings, revision counts, and errors. Never log the API key, images, full prompts, or dietary preferences.
- **Food safety:** recipe cards and Plan B note that calories are approximate, that the app cannot guarantee recipes are allergen-free, and that it gives no medical advice.

## Example Code Patterns

### A typical endpoint (Streamlit page handler for the recipe step)

```python
import streamlit as st

from receiptchef.domain.models import CalorieRange, Cuisine, RecipePreferences
from receiptchef.services.recipe_pipeline import PlanRequest, RecipePipeline
from receiptchef.ui.session import SessionStore
from receiptchef.ui.widgets import notices, plan_view


def render_preferences_and_results(store: SessionStore, pipeline: RecipePipeline) -> None:
    st.header("2. Choose your preferences")

    confirmed = store.confirmed_list()
    if confirmed is None:
        st.info("Upload a receipt and confirm your ingredient list first.")
        st.stop()

    dish_count = st.radio("Number of dishes", [1, 2, 3], horizontal=True)
    max_minutes = st.radio(
        "Maximum total time for all dishes (minutes)", [15, 30, 45, 60], horizontal=True
    )
    calorie_range = st.radio(
        "Calories per dish (all listed quantities)", [c.value for c in CalorieRange]
    )
    cuisine = st.radio("Cuisine", [c.value for c in Cuisine], horizontal=True)

    if st.button("Suggest recipes"):
        request = PlanRequest(
            source="generator",
            confirmed=confirmed,
            description=None,
            preferences=RecipePreferences(
                dish_count=dish_count,
                max_total_minutes=max_minutes,
                calorie_range=CalorieRange(calorie_range),
                cuisine=Cuisine(cuisine),
            ),
            dietary=store.dietary(),
            current_plan=None,
            change=None,
        )
        with st.spinner("Finding recipes that fit your groceries..."):
            result = pipeline.run(request)
        store.push_result(result)

    result = store.current_result()
    if result is None:
        return
    if result.status == "unavailable":
        st.error("We couldn't reach the recipe service. Your ingredients and choices are saved.")
        st.button("Try again")
        return
    plan_view.render(result)  # dish cards, totals, Plan B explanation and labels
    notices.allergen_notice()
```

### A typical function (pipeline step)

```python
from receiptchef.domain.models import CheckProblem, Dish, MealPlan

MAX_KCAL_PER_INGREDIENT = 5000


def apply_totals(plan: MealPlan) -> tuple[MealPlan, list[CheckProblem]]:
    """Fill in each dish's calories and the plan total from the model's per-ingredient estimates.

    Each ingredient line is the quantity the recipe uses, including oil and sauces, with the
    model's estimated kcal for that quantity. Totals are computed here, never taken from the
    model. Impossible estimates are reported as problems so the pipeline asks for a revision.
    """
    problems: list[CheckProblem] = []
    dishes: list[Dish] = []
    for index, dish in enumerate(plan.dishes):
        for ingredient in dish.ingredients:
            if not 0 <= ingredient.estimated_kcal <= MAX_KCAL_PER_INGREDIENT:
                problems.append(
                    CheckProblem(kind="invalid_estimate", dish_index=index, detail=ingredient.name)
                )
        dish_kcal = sum(ingredient.estimated_kcal for ingredient in dish.ingredients)
        dishes.append(dish.model_copy(update={"kcal": round(dish_kcal)}))

    total = sum(dish.kcal or 0 for dish in dishes)
    return plan.model_copy(update={"dishes": dishes, "total_kcal": total}), problems
```

### A typical test

```python
from hypothesis import given

from receiptchef.domain.calories import apply_totals
from receiptchef.domain.checks import check_plan
from receiptchef.domain.models import CalorieRange, Cuisine, RecipePreferences
from receiptchef.services.recipe_pipeline import RecipePipeline
from tests.strategies import meal_plans


def test_dish_and_plan_totals(sample_plan_two_dishes) -> None:
    plan, problems = apply_totals(sample_plan_two_dishes)
    assert problems == []
    assert [dish.kcal for dish in plan.dishes] == [450, 520]
    assert plan.total_kcal == 970


@given(meal_plans())
def test_total_always_equals_sum_of_dishes(plan) -> None:
    totalled, _ = apply_totals(plan)
    assert totalled.total_kcal == sum(dish.kcal for dish in totalled.dishes)


def test_plan_over_time_limit_is_reported(sample_plan_two_dishes, confirmed_list, no_dietary, dietary_terms) -> None:
    prefs = RecipePreferences(
        dish_count=2,
        max_total_minutes=15,
        calorie_range=CalorieRange.NO_PREFERENCE,
        cuisine=Cuisine.CHINESE,
    )
    plan = sample_plan_two_dishes.model_copy(update={"total_minutes": 45})
    problems = check_plan(plan, prefs, no_dietary, confirmed_list, dietary_terms)
    assert any(problem.kind == "time" for problem in problems)


def test_pipeline_uses_at_most_three_generation_calls(fake_llm_client, generator_request, reference_data, dietary_terms) -> None:
    """The pipeline runs end to end against recorded LLM responses, with no network call."""
    pipeline = RecipePipeline(fake_llm_client, reference_data, dietary_terms)
    result = pipeline.run(generator_request)
    assert result.generation_calls <= 3
    assert len(fake_llm_client.calls) <= 3
```
