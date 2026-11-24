# Technical Environment: ReceiptChef

ReceiptChef is a local web app. A user uploads a photo of a grocery receipt, and an LLM agent reads the receipt, adds the items to the user's pantry, works out which ingredients should be used first, and suggests a recipe (Chinese, Italian, Mexican, or "no preference", which picks one of the three at random) that uses the most urgent ingredients.

## Language and Package Manager

- **Python 3.12+**
- **uv** for all package management (no pip, poetry, or conda)
- `pyproject.toml` for all project and tool configuration
- `uv.lock` committed to Git
- Application code lives in `src/receiptchef/`, tests in `tests/`, and all AI-DLC artifacts stay in `aidlc-docs/`

## Frameworks and Libraries

- **Streamlit** for the user interface: image upload (`st.file_uploader`), optional phone camera capture (`st.camera_input`), pantry view, and recipe display
- **Anthropic Python SDK** (`anthropic`) to call Claude, using its built-in **vision** (reading the receipt image) and **tool use** (the agent loop)
- **Pydantic v2** for every structured object passed between the LLM, the tools, and the UI (receipt items, pantry items, recipes)
- **SQLite** through Python's built-in `sqlite3` module for the local pantry database
- **Pillow** to validate, resize, and strip metadata from uploaded images before they are sent to the model
- **python-dotenv** to load the API key from a local `.env` file

## LLM Provider and Agent Design

- **Provider:** Anthropic Claude API (hosted). Planned model: `claude-sonnet-5-5`. It reads receipt photos and supports tool use. The model name is set with the `RECEIPTCHEF_MODEL` environment variable, so the team can switch to `claude-haiku-5-5` to cut cost during development.
- **Fallback:** if API credits run out, the same code path can point at a local vision model in **Ollama** (for example `qwen2.5vl`) through Ollama's OpenAI-compatible endpoint. All LLM calls go through one `llm_client.py` module so only that file changes.
- **Agent pattern:** a single tool-using agent. Claude decides which tools to call and in what order, uses the results, and returns a final recipe. The loop stops after a maximum of **8 tool calls** per request.
- **Agent tools (all implemented locally in Python):**

| Tool | What it does | Data source |
| --- | --- | --- |
| `parse_receipt` | Turns the receipt image into a list of food items (name, quantity, unit). Non-food items such as bags or detergent are dropped. | Claude vision, validated by Pydantic |
| `add_to_pantry` | Saves parsed items with today's purchase date | SQLite `pantry` table |
| `get_pantry` | Returns everything currently in the pantry with purchase dates | SQLite |
| `lookup_shelf_life` | Returns the typical fridge or pantry shelf life in days for an ingredient | Local `data/shelf_life.json` (about 150 common ingredients) |
| `rank_by_urgency` | Sorts pantry items by days left (shelf life minus days since purchase), most urgent first | Pure Python function |
| `search_recipes` | Finds recipes in the chosen cuisine that use the given ingredients, so suggestions are grounded in real dishes | Local `data/recipes.json` (about 30 recipes per cuisine) |
| `mark_used` | Removes or reduces ingredients after the user cooks a recipe | SQLite |

- **Cuisine choice:** Chinese, Italian, or Mexican. "No preference" is resolved in Python with `random.choice` before the agent runs, so the result is reproducible in tests with a fixed seed.
- **Optional dish image:** Claude does not generate images. Showing a picture of the finished dish is out of scope for the MVP. If time allows, the app will show a stock image stored with the recipe in `recipes.json` instead of generating one.

## Cloud and Deployment

- **The product runs locally**; there is no cloud deployment or hosting. Each teammate runs it on their laptop or in a GitHub Codespace with `uv run streamlit run src/receiptchef/app.py` and opens it at `http://localhost:8501`.
- The only external service is the **Anthropic API**, called over HTTPS. All data (pantry database, uploaded images, recipe and shelf-life files) stays on the local machine.
- No Docker, cloud database, or user accounts for the MVP. There is one local user per installation.

## Testing

- **pytest** with **pytest-cov**: 80% line coverage minimum for `src/receiptchef/`, excluding the Streamlit UI file
- **ruff** for linting and formatting
- **mypy** for type checking the tools and models
- **No real API calls in unit tests.** The Anthropic client is replaced with a fake that returns recorded responses, so tests are free, fast, and deterministic.
- **Receipt fixtures:** 5–10 real receipt photos (with personal details blurred) in `tests/fixtures/receipts/`, each paired with a JSON file listing the expected items. They are used for one optional, manually run integration test marked `@pytest.mark.live` that calls the real API.
- Pure functions (`rank_by_urgency`, shelf-life lookup, pantry database operations) are tested directly with an in-memory SQLite database.

## Do NOT Use

| Prohibited | Reason | Use Instead |
| --- | --- | --- |
| LangChain, LlamaIndex, CrewAI | Extra abstraction layers we don't need for one agent with a few tools; harder to debug and test | Anthropic SDK tool use directly |
| pytesseract, EasyOCR, other OCR libraries | Extra system dependencies, and they return raw text we would still have to parse | Claude vision with Pydantic validation |
| Flask, Django, React | Too much setup for a local MVP | Streamlit |
| PostgreSQL, MongoDB, Firebase | Needs a server or cloud account; the app runs locally | SQLite (`sqlite3`) |
| `eval()`, `exec()`, `pickle` | Security: could run code that comes from model output | `json` parsing plus Pydantic validation |
| Hard-coded API keys, `st.secrets` committed to Git | Secrets would be exposed in the public repo | `.env` file listed in `.gitignore`, loaded with python-dotenv |
| pip, poetry, conda | Project uses uv only | uv |
| black, flake8, isort | Replaced by one tool | ruff |

## Security Basics

- `ANTHROPIC_API_KEY` lives only in a local `.env` file. `.env` is in `.gitignore`, and a `.env.example` with an empty value is committed instead.
- **Uploads:** only JPEG, PNG, and WebP, at most 10 MB. Images are resized to a maximum of 1568 px on the long edge before sending.
- **Privacy:** EXIF metadata (including GPS location) is stripped with Pillow before an image leaves the machine. Receipt images are not saved to disk after parsing; only the extracted food items are stored.
- **Prompt injection:** receipt text is treated as data, never as instructions. The system prompt tells the model to ignore any instructions found in the image, and every tool input from the model is validated with Pydantic before it runs.
- **Tool safety:** tools can only read or write the local pantry and read the two bundled JSON files. No tool runs shell commands, opens URLs, or writes outside `data/`.
- **Limits:** a maximum of 8 tool calls per request and 2,000 output tokens per response, to control cost and prevent runaway loops.
- **Logging:** log tool names, timings, and errors. Never log the API key or full images. Prompt logging is off by default.
- **Food safety:** recipes show a short note that shelf-life estimates are general guidance and that users should check food before eating it. The app does not make allergy claims.

## Example Code Patterns

### A typical endpoint (Streamlit page handler)

```python
import streamlit as st

from receiptchef.agent import run_recipe_agent
from receiptchef.images import prepare_image, ImageValidationError
from receiptchef.models import CuisineChoice, RecipeResult

st.title("ReceiptChef")

uploaded = st.file_uploader("Upload a receipt photo", type=["jpg", "jpeg", "png", "webp"])
cuisine = st.radio("Cuisine", [c.value for c in CuisineChoice], horizontal=True)

if uploaded and st.button("Suggest a recipe"):
    try:
        image_bytes = prepare_image(uploaded.getvalue())  # validate, resize, strip EXIF
    except ImageValidationError as e:
        st.error(str(e))
        st.stop()

    with st.spinner("Reading your receipt and checking your pantry..."):
        result: RecipeResult = run_recipe_agent(image_bytes, CuisineChoice(cuisine))

    st.subheader("Use these first")
    for item in result.urgent_items:
        st.write(f"- {item.name}: about {item.days_left} day(s) left")

    st.subheader(result.recipe.title)
    st.markdown(result.recipe.instructions_markdown)
```

### A typical function (agent tool)

```python
from datetime import date

from receiptchef.models import PantryItem, UrgentItem
from receiptchef.shelf_life import lookup_shelf_life


def rank_by_urgency(items: list[PantryItem], today: date) -> list[UrgentItem]:
    """Return pantry items sorted by days left before they spoil, most urgent first.

    Items with an unknown shelf life are placed last rather than guessed.
    """
    ranked: list[UrgentItem] = []
    for item in items:
        shelf_days = lookup_shelf_life(item.name)  # None if not in shelf_life.json
        if shelf_days is None:
            days_left = None
        else:
            days_left = shelf_days - (today - item.purchased_on).days
        ranked.append(UrgentItem(name=item.name, quantity=item.quantity, days_left=days_left))

    return sorted(ranked, key=lambda u: (u.days_left is None, u.days_left or 0))
```

### A typical test

```python
from datetime import date

import pytest

from receiptchef.models import PantryItem
from receiptchef.tools import rank_by_urgency


@pytest.fixture
def pantry() -> list[PantryItem]:
    return [
        PantryItem(name="rice", quantity=1, unit="bag", purchased_on=date(2026, 11, 1)),
        PantryItem(name="spinach", quantity=1, unit="bunch", purchased_on=date(2026, 11, 20)),
        PantryItem(name="dragonfruit jam", quantity=1, unit="jar", purchased_on=date(2026, 11, 20)),
    ]


def test_most_perishable_item_comes_first(pantry: list[PantryItem]) -> None:
    ranked = rank_by_urgency(pantry, today=date(2026, 11, 23))
    assert ranked[0].name == "spinach"


def test_unknown_shelf_life_is_ranked_last(pantry: list[PantryItem]) -> None:
    ranked = rank_by_urgency(pantry, today=date(2026, 11, 23))
    assert ranked[-1].name == "dragonfruit jam"
    assert ranked[-1].days_left is None


def test_agent_uses_fake_llm(fake_llm_client, sample_receipt_bytes) -> None:
    """The agent loop runs end to end against recorded LLM responses, with no network call."""
    from receiptchef.agent import run_recipe_agent
    from receiptchef.models import CuisineChoice

    result = run_recipe_agent(sample_receipt_bytes, CuisineChoice.ITALIAN, client=fake_llm_client)
    assert result.recipe.cuisine == "Italian"
    assert fake_llm_client.tool_calls_made <= 8
```