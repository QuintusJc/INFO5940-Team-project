# Technical Environment: ReceiptChef

ReceiptChef is a local web app that helps international students turn groceries they have already bought into practical meals. The user uploads one photo of an English-language grocery receipt. The LLM turns the receipt entries into an editable ingredient list, the user confirms or corrects it, and then the user chooses servings, maximum total time, a calorie range per serving, and a cuisine (Chinese, Italian, Mexican, or No preference). A recipe agent then suggests up to three recipes, each with ingredient quantities, steps, estimated total time, approximate calories per serving, and any extra ingredients needed.

## Language and Package Manager

- **Python 3.12+**
- **uv** for all package management (no pip, poetry, or conda)
- `pyproject.toml` for all project and tool configuration
- `uv.lock` committed to Git
- Application code lives in `src/receiptchef/`, tests in `tests/`, bundled data in `data/`. AI-DLC artifacts stay in `aidlc-docs/`, and no application code goes there.

## Frameworks and Libraries

- **Streamlit** for the user interface:
  - receipt upload (`st.file_uploader`)
  - editable ingredient list (`st.data_editor`), where users rename, remove, or add ingredients and pantry staples
  - preference form (servings, max time, calorie range, cuisine)
  - recipe cards
- **Anthropic Python SDK** (`anthropic`) to call Claude, using **vision** (reading the receipt photo), **structured JSON output** (the ingredient list), and **tool use** (the recipe agent)
- **Pydantic v2** for every structured object passed between the model, the tools, and the UI (`ReceiptItem`, `ConfirmedIngredient`, `RecipePreferences`, `Recipe`, `RecipeSet`)
- **Pillow** to validate, resize, and strip metadata from uploaded images before they are sent to the model
- **python-dotenv** to load the API key from a local `.env` file

## LLM Provider and Workflow

- **Provider:** Anthropic Claude API (hosted).
- **Planned model:** **Claude Sonnet 5.5** (`claude-sonnet-5-5`) for both stages. The model name is read from the `RECEIPTCHEF_MODEL` environment variable, so the team can use `claude-haiku-5-5` during development to reduce cost.
- **Single client module:** all model calls go through `src/receiptchef/llm_client.py`. If API credits become a problem, only this file changes. A local Ollama vision model through Ollama's OpenAI-compatible endpoint is the backup.

The product works in two model stages, with a required user confirmation step between them.

### Stage 1: Receipt interpretation (one vision call)

- **Input:** one receipt photo.
- **Output:** a JSON list of `ReceiptItem` objects with:
  - `raw_text`, the text as printed, for example `"GRN ONION BCH"`
  - `ingredient_name`, for example `"green onions"`
  - `quantity` and `unit`, when they can be read
  - `status`: `"food"`, `"unclear"`, or `"non_food"`
- `non_food` items (bags, detergent, tax lines) are dropped. `unclear` items are highlighted so the user can review them.
- If the model reports that the receipt is unreadable or not a grocery receipt, the app asks the user to upload a clearer photo. No ingredient list is shown in that case.
- The output is validated with Pydantic. If validation fails, the app retries once, then shows an error.

### User confirmation (no model call)

- The user edits the ingredient list: correcting names, removing items they no longer have, and adding ingredients or pantry staples such as oil, salt, rice, or soy sauce.
- **Only the confirmed list is sent to Stage 2.**

### Stage 2: Recipe agent (tool-using agent)

- **Input:** the confirmed ingredients plus `RecipePreferences`:
  - servings: 1–6
  - max total time: 10–120 minutes
  - calorie range per serving: min and max kcal
  - cuisine
- **Output:** a `RecipeSet` with up to three `Recipe` objects. Each recipe has a title, cuisine, ingredient quantities, steps, estimated total time, approximate calories per serving, `extra_ingredients_needed`, and `assumed_pantry_items`.
- **No preference:** the agent chooses among Chinese, Italian, and Mexican based on which fits the confirmed ingredients best, and states the chosen cuisine on each recipe.
- **Agent loop:** Claude decides which tools to call and in what order, uses the results, and revises recipes that break the user's constraints before returning them. The loop stops after **10 tool calls**. If no valid recipe is found, the app explains which constraint could not be met (for example "no recipe under 15 minutes with these ingredients").

| Tool | What it does | Implementation |
| --- | --- | --- |
| `search_reference_recipes` | Returns reference dishes for a cuisine that use some of the given ingredients, so suggestions stay close to real Chinese, Italian, and Mexican cooking | Local `data/reference_recipes.json` (about 30 dishes per cuisine: name, typical ingredients, typical total time) |
| `calculate_calories` | Takes the model's per-ingredient calorie estimates for the quantities used (including oil and sauces) and the number of servings, and returns the total and per-serving calories. The model supplies the estimates; the arithmetic is done in Python so the totals are consistent. | Pure Python function |
| `check_recipe_constraints` | Checks a draft recipe against the preferences and the confirmed list. Returns problems: over max time, calories outside the range, wrong cuisine, and which ingredients are not on the confirmed list (these must be listed as extra or assumed pantry items). | Pure Python function |

Calorie estimates come from the model, with no external nutrition database or API in the MVP. They are always labeled **"approximate"** in the UI.

### Session data

- No database, no user accounts, and no ongoing pantry tracking.
- The ingredient list, preferences, and recipes live in Streamlit `st.session_state` for the current session only.

## Cloud and Deployment

- **The product runs locally**; there is no cloud deployment or hosting. Each teammate runs it on their laptop or in a GitHub Codespace with `uv run streamlit run src/receiptchef/app.py` and opens `http://localhost:8501`.
- The only external service is the **Anthropic API**, called over HTTPS.
- Receipt photos and results are not stored after the session ends.
- No Docker or cloud services for the MVP.

## Testing

- **pytest** with **pytest-cov**: 80% line coverage minimum for `src/receiptchef/`, excluding the Streamlit UI file
- **ruff** for linting and formatting
- **mypy** for type checking models and tools
- **No real API calls in unit tests.** `llm_client` is replaced with a fake client that returns recorded responses, so tests are free, fast, and repeatable.
- **Pure functions** (`calculate_calories`, `check_recipe_constraints`, image preparation, Pydantic validation) are unit tested directly.
- **Receipt fixtures:**
  - 5–10 real receipts from local supermarkets, with card numbers and loyalty IDs blurred, stored in `tests/fixtures/receipts/`
  - each receipt paired with a JSON file of the expected ingredients
  - a manually run integration test marked `@pytest.mark.live` calls the real API and reports how many expected ingredients were found
- **Calorie spot check:** `scripts/check_calories.py` compares about 10 calorie estimates against nutrition labels or reference values recorded in `tests/fixtures/calorie_reference.json`, to catch obvious errors. It is run manually, not in CI.

## Do NOT Use

| Prohibited | Reason | Use Instead |
| --- | --- | --- |
| LangChain, LlamaIndex, CrewAI | Extra layers we don't need for one vision call and one agent with three tools; harder to debug and test | Anthropic SDK directly |
| pytesseract, EasyOCR, other OCR libraries | Extra system dependencies, and they return raw text that still needs interpreting | Claude vision with Pydantic validation |
| External nutrition APIs (USDA FoodData Central, Nutritionix, Edamam) | Out of scope for the MVP; adds API keys and matching logic | Model calorie estimates labeled "approximate", totals computed by `calculate_calories` |
| SQLite, PostgreSQL, Firebase, any database | No pantry tracking or accounts in the MVP | `st.session_state` |
| Image generation APIs (DALL·E, Stable Diffusion) | Food photos are out of scope | Text recipe cards |
| Flask, Django, React | Too much setup for a local MVP | Streamlit |
| `eval()`, `exec()`, `pickle` | Security: could run code that comes from model output | `json` parsing plus Pydantic validation |
| Hard-coded API keys, committed `.env` or `secrets.toml` | Secrets would be exposed in the repository | `.env` file listed in `.gitignore`, loaded with python-dotenv |
| pip, poetry, conda | Project uses uv only | uv |
| black, flake8, isort | Replaced by one tool | ruff |

## Security Basics

- `ANTHROPIC_API_KEY` is stored only in a local `.env` file. `.env` is in `.gitignore`, and a `.env.example` with an empty value is committed instead.
- **Uploads:** one image at a time, JPEG, PNG, or WebP only, at most 10 MB. Images are resized to a maximum of 1568 px on the long edge before sending.
- **Privacy:**
  - EXIF metadata (including GPS location) is stripped before an image leaves the machine.
  - The system prompt tells the model to ignore payment details, card numbers, and loyalty IDs and never return them.
  - Images and results are not written to disk.
- **Prompt injection:** receipt text and user-typed ingredient names are treated as data, never as instructions. Ingredient names are limited to 60 characters and the list to 60 items. Every tool input from the model is validated with Pydantic before the tool runs.
- **Tool safety:** tools only read the bundled `data/reference_recipes.json` or do arithmetic. No tool runs shell commands, opens URLs, or writes files.
- **Limits:** a maximum of 10 tool calls and 4,000 output tokens per recipe request, to control cost and prevent runaway loops.
- **Logging:** log stage names, tool names, timings, and errors. Never log the API key, images, or full prompts.
- **Food safety:** recipe cards note that calories are approximate. The app makes no allergy or medical claims.

## Example Code Patterns

### A typical endpoint (Streamlit page handler for the recipe step)

```python
import streamlit as st

from receiptchef.agent import run_recipe_agent, RecipeAgentError
from receiptchef.models import Cuisine, RecipePreferences

st.header("2. Choose your preferences")

confirmed = st.session_state.get("confirmed_ingredients")
if not confirmed:
    st.info("Upload a receipt and confirm your ingredient list first.")
    st.stop()

servings = st.number_input("Servings", min_value=1, max_value=6, value=2)
max_minutes = st.slider("Max total time (minutes)", 10, 120, 30, step=5)
kcal_min, kcal_max = st.slider("Calories per serving", 200, 1200, (400, 700), step=50)
cuisine = st.radio("Cuisine", [c.value for c in Cuisine], horizontal=True)

if st.button("Suggest recipes"):
    prefs = RecipePreferences(
        servings=servings,
        max_total_minutes=max_minutes,
        kcal_per_serving_min=kcal_min,
        kcal_per_serving_max=kcal_max,
        cuisine=Cuisine(cuisine),
    )
    try:
        with st.spinner("Finding recipes that fit your groceries..."):
            result = run_recipe_agent(confirmed, prefs)
    except RecipeAgentError as e:
        st.error(f"Couldn't find a recipe: {e.reason}")
        st.stop()

    for recipe in result.recipes:
        with st.container(border=True):
            st.subheader(f"{recipe.title} ({recipe.cuisine})")
            st.caption(
                f"About {recipe.total_minutes} min · ~{recipe.kcal_per_serving} kcal per serving (approximate)"
            )
            st.markdown(recipe.to_markdown())
            if recipe.extra_ingredients_needed:
                st.warning("You'll also need: " + ", ".join(recipe.extra_ingredients_needed))
```

### A typical function (agent tool)

```python
from receiptchef.models import CalorieLine, CalorieResult


def calculate_calories(lines: list[CalorieLine], servings: int) -> CalorieResult:
    """Total the model's per-ingredient calorie estimates and divide by servings.

    Each line is one ingredient at the quantity the recipe uses, including oil and sauces,
    with the model's estimated kcal for that quantity. Rejects impossible values instead of
    silently accepting them, so the agent has to re-estimate.
    """
    if servings < 1:
        raise ValueError("servings must be at least 1")

    problems = [line.ingredient for line in lines if line.estimated_kcal < 0 or line.estimated_kcal > 5000]
    if problems:
        return CalorieResult(ok=False, total_kcal=0, kcal_per_serving=0, problems=problems)

    total = sum(line.estimated_kcal for line in lines)
    return CalorieResult(
        ok=True,
        total_kcal=round(total),
        kcal_per_serving=round(total / servings),
        problems=[],
    )
```

### A typical test

```python
import pytest

from receiptchef.models import CalorieLine, Cuisine, RecipePreferences
from receiptchef.tools import calculate_calories, check_recipe_constraints


def test_calories_divided_by_servings() -> None:
    lines = [
        CalorieLine(ingredient="chicken thigh, 400 g", estimated_kcal=840),
        CalorieLine(ingredient="vegetable oil, 1 tbsp", estimated_kcal=120),
        CalorieLine(ingredient="rice, 300 g cooked", estimated_kcal=390),
    ]
    result = calculate_calories(lines, servings=3)
    assert result.ok
    assert result.total_kcal == 1350
    assert result.kcal_per_serving == 450


def test_impossible_calorie_estimate_is_flagged() -> None:
    lines = [CalorieLine(ingredient="soy sauce, 1 tbsp", estimated_kcal=9000)]
    result = calculate_calories(lines, servings=1)
    assert not result.ok
    assert result.problems == ["soy sauce, 1 tbsp"]


def test_recipe_over_time_limit_is_reported(sample_recipe, confirmed_ingredients) -> None:
    prefs = RecipePreferences(
        servings=2, max_total_minutes=20, kcal_per_serving_min=300,
        kcal_per_serving_max=800, cuisine=Cuisine.CHINESE,
    )
    sample_recipe.total_minutes = 45
    problems = check_recipe_constraints(sample_recipe, prefs, confirmed_ingredients)
    assert any("time" in p for p in problems)


def test_agent_returns_at_most_three_recipes(fake_llm_client, confirmed_ingredients, default_prefs) -> None:
    """The agent loop runs end to end against recorded LLM responses, with no network call."""
    from receiptchef.agent import run_recipe_agent

    result = run_recipe_agent(confirmed_ingredients, default_prefs, client=fake_llm_client)
    assert 1 <= len(result.recipes) <= 3
    assert fake_llm_client.tool_calls_made <= 10
```