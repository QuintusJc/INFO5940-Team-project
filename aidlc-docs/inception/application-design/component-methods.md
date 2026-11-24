# ReceiptChef - Component Methods

Signatures are Python 3.12 with type hints. They define inputs, outputs, and purpose only. **Business rules (exact checks, thresholds, merge logic, prompt wording) are defined later in Functional Design.** Names may be refined in Functional Design if needed, as long as the layer rules in `component-dependency.md` hold.

---

## 1. Domain Models (`domain/models.py`, C-13)

Key models only; fields are indicative.

```python
class Cuisine(StrEnum): CHINESE, ITALIAN, MEXICAN, NO_PREFERENCE
class CalorieRange(StrEnum): BELOW_400, FROM_400_TO_600, FROM_600_TO_800, NO_PREFERENCE
class ItemStatus(StrEnum): FOOD, UNCLEAR, NON_FOOD
class DietaryOption(StrEnum): PEANUTS, TREE_NUTS, DAIRY, EGGS, GLUTEN, SHELLFISH, VEGETARIAN, VEGAN, HALAL, NO_PORK, NO_BEEF

class DietaryPreferences(BaseModel):
    options: set[DietaryOption]
    other_text: str | None          # max 200 chars
    onboarding_state: Literal["saved", "none", "skipped"]

class ReceiptItem(BaseModel):
    image_index: int                # 0-2
    raw_text: str
    ingredient_name: str            # max 60 chars
    quantity: float | None
    unit: str | None
    status: ItemStatus
    explanation: str | None

class IngredientRow(BaseModel):     # one row of the review table
    row_id: str
    name: str                       # max 60 chars
    on_receipt: str | None          # read-only display, e.g. "2 lb"
    raw_text: str | None
    explanation: str | None
    needs_review: bool              # unclear or duplicate, unresolved
    duplicate_group: str | None
    source: Literal["receipt", "user"]

class ConfirmedIngredientList(BaseModel):
    items: list[str]                # max 60 items

class RecipePreferences(BaseModel):
    dish_count: Literal[1, 2, 3]
    max_total_minutes: Literal[15, 30, 45, 60]
    calorie_range: CalorieRange
    cuisine: Cuisine

class RecipeIngredient(BaseModel):
    name: str
    quantity: str                   # e.g. "300 g"
    estimated_kcal: int
    source: Literal["confirmed", "assumed_pantry", "additional"]
    allergen_tags: list[DietaryOption]   # model-declared (Q8: A)
    store_unit_hint: str | None          # model-suggested (Q9: A)

class Dish(BaseModel):
    name: str
    cuisine: Cuisine
    ingredients: list[RecipeIngredient]
    steps: list[str]
    kcal: int | None                # filled by calories.py, never by the model

class MealPlan(BaseModel):
    dishes: list[Dish]              # 1-3
    total_minutes: int
    time_assumptions: list[str]
    total_kcal: int | None          # filled by calories.py
    is_plan_b: bool
    changes_from_request: list[str] # filled by checks.py for Plan B
    purchase_label: Literal["confirmed_only", "requires_purchases"]

class CheckProblem(BaseModel):
    kind: Literal["time", "calories", "cuisine", "dish_count", "dietary", "unconfirmed_ingredient", "invalid_estimate"]
    dish_index: int | None
    detail: str

class ShoppingListItem(BaseModel):
    name: str
    combined_quantity: str          # e.g. "5 cloves"
    store_unit_hint: str | None     # labeled approximate
    already_have: bool
    auto_marked: bool               # came from Generator confirmed list

class ShoppingList(BaseModel):
    to_buy: list[ShoppingListItem]
    already_have: list[ShoppingListItem]

class FollowUpAction(BaseModel):
    kind: Literal["replace_ingredient", "different_dish", "change_dish_count", "adjust_preferences",
                  "explain_step", "other_meals", "identify_additional"]
    dish_index: int | None
    ingredient_name: str | None
    step_index: int | None
    replacement: str | None         # max 60 chars
    new_dish_count: int | None
    drop_dish_index: int | None
    new_preferences: RecipePreferences | None

class ChatTurn(BaseModel):
    role: Literal["user", "assistant"]
    text: str                       # user text max 500 chars
```

## 2. Result Objects (`domain/results.py`, C-14, Q11: B)

```python
class LLMResponse(BaseModel, Generic[M]):
    status: Literal["ok", "unavailable", "invalid_output"]
    value: M | None
    error_kind: str | None          # e.g. "timeout", "rate_limit"; never contains prompt text

class PrepareResult(BaseModel):
    status: Literal["ok", "rejected"]
    image: PreparedImage | None     # in-memory bytes + media type
    reason: str | None              # "too_large", "unsupported_type", "unreadable"

class ImageExtraction(BaseModel):
    image_index: int
    status: Literal["ok", "unreadable", "not_a_receipt", "invalid_output", "unavailable"]
    items: list[ReceiptItem]

class ExtractionResult(BaseModel):
    per_image: list[ImageExtraction]
    rows: list[IngredientRow]       # merged, non_food removed, duplicates flagged

class PipelineResult(BaseModel):
    status: Literal["ok", "plan_b", "insufficient_ingredients", "no_safe_plan", "unavailable"]
    plan: MealPlan | None
    conflict_explanation: str | None
    missing_ingredients: list[str]
    generation_calls: int           # 1-3
    problems_last_attempt: list[CheckProblem]

class FollowUpResult(BaseModel):
    kind: Literal["explanation", "plan_updated", "additional_list", "declined", "unavailable"]
    text: str | None
    pipeline: PipelineResult | None

class PreferencesLoadResult(BaseModel):
    status: Literal["missing", "ok", "corrupt"]
    preferences: DietaryPreferences | None
```

---

## 3. UI Layer

### C-01 App shell (`app.py`)
| Method | Purpose |
|---|---|
| `main() -> None` | Load settings; if API key missing, render setup message and stop; load preferences; route to onboarding if missing / corrupt; register pages with `st.navigation`; run selected page |
| `build_services(settings: Settings) -> Services` | Create `AnthropicLLMClient`, data, and services once per session (cached) |

### C-02 to C-05 Pages
| Method | Purpose |
|---|---|
| `onboarding.render(store: SessionStore, prefs_store: PreferencesStore) -> None` | Intro and dietary form |
| `generator.render(store: SessionStore, services: Services) -> None` | Generator flow |
| `planner.render(store: SessionStore, services: Services) -> None` | Planner flow |
| `settings.render(store: SessionStore, prefs_store: PreferencesStore) -> None` | Edit / clear preferences |

### C-06 Widgets
| Method | Purpose |
|---|---|
| `ingredient_table.render(rows: list[IngredientRow]) -> list[IngredientRow]` | Editable table; returns edited rows |
| `preference_form.render(current: RecipePreferences \| None) -> RecipePreferences \| None` | Returns choices when submitted |
| `plan_view.render(result: PipelineResult) -> None` | Dish cards, totals, Plan B explanation and labels, notices |
| `followup_panel.render(plan: MealPlan, conversation: list[ChatTurn]) -> FollowUpAction \| str \| None` | Returns a button action or chat text |
| `shopping_list_view.render(sl: ShoppingList) -> set[str]` | Returns names the user marked as "already have" |
| `notices.allergen_notice() -> None`, `notices.calorie_notice() -> None` | Standard safety text |

### C-07 SessionStore (`ui/session.py`)
| Method | Purpose |
|---|---|
| `images() -> list[PreparedImage]` / `set_images(images)` / `replace_image(index, image)` | In-memory receipt images |
| `extraction() -> ExtractionResult \| None` / `set_extraction(result)` | Latest extraction |
| `rows() -> list[IngredientRow]` / `set_rows(rows)` | Review table state |
| `confirmed_list() -> ConfirmedIngredientList \| None` / `set_confirmed_list(lst)` | Confirmed list |
| `recipe_preferences() -> RecipePreferences \| None` / `set_recipe_preferences(p)` | Last chosen preferences |
| `current_result() -> PipelineResult \| None` / `push_result(r)` / `previous_result() -> PipelineResult \| None` | Current plan and history (US-FUP-06) |
| `conversation() -> list[ChatTurn]` / `append_turn(turn)` | Chat history (session only) |
| `planner_state() -> PlannerState` / `set_planner_state(s)` | Planner description, plan, marks |
| `dietary() -> DietaryPreferences \| None` / `set_dietary(p)` | Cached copy of saved preferences |

---

## 4. Service Layer

### C-08 ReceiptExtractionService
| Method | Purpose |
|---|---|
| `__init__(llm: LLMClient)` | Inject client |
| `extract(images: list[PreparedImage]) -> ExtractionResult` | One call per image in order; validation retry once per image; merge |
| `extract_one(image: PreparedImage, image_index: int) -> ImageExtraction` | Used for re-uploading one image (US-UPL-03) |
| `merge_into(existing: list[IngredientRow], new: ImageExtraction) -> list[IngredientRow]` | Add replacement-image items without losing edits |

### C-09 RecipePipeline
| Method | Purpose |
|---|---|
| `__init__(llm: LLMClient, reference: ReferenceData, dietary_terms: DietaryTerms)` | Inject dependencies |
| `run(request: PlanRequest) -> PipelineResult` | Full loop: lookup, generate, calculate, check, revise (max 2), outcome |

```python
class PlanRequest(BaseModel):
    source: Literal["generator", "planner", "followup"]
    confirmed: ConfirmedIngredientList | None   # None for Planner from scratch
    description: str | None                     # Planner text, max 500 chars
    preferences: RecipePreferences
    dietary: DietaryPreferences
    current_plan: MealPlan | None               # for follow-up changes
    change: str | FollowUpAction | None         # chat change text or button action
```

### C-10 FollowUpService
| Method | Purpose |
|---|---|
| `__init__(llm: LLMClient, pipeline: RecipePipeline)` | Inject dependencies |
| `handle_action(action: FollowUpAction, context: FollowUpContext) -> FollowUpResult` | Explain-only actions call the model for text; plan-changing actions call `pipeline.run`; `identify_additional` computed locally |
| `handle_chat(text: str, context: FollowUpContext) -> FollowUpResult` | Chat call returns `answer` or `plan_change`; plan change goes to `pipeline.run` |

`FollowUpContext` holds the current plan, confirmed list or Planner "already have" list, recipe preferences, dietary preferences, and the conversation.

### C-11 PlannerService
| Method | Purpose |
|---|---|
| `__init__(pipeline: RecipePipeline)` | Inject pipeline |
| `propose(description: str, preferences: RecipePreferences, dietary: DietaryPreferences) -> PipelineResult` | Planner from scratch |
| `from_generator(result: PipelineResult, confirmed: ConfirmedIngredientList) -> PlannerState` | Handoff without a model call |
| `shopping_list(plan: MealPlan, confirmed: ConfirmedIngredientList \| None, manual_marks: set[str], unmarked: set[str]) -> ShoppingList` | Delegates to `domain.shopping` |

### C-12 Prompts (`services/prompts.py`)
| Function | Purpose |
|---|---|
| `extraction_prompt() -> PromptParts` | System and instruction text for one receipt image |
| `generation_prompt(request: PlanRequest, references: list[ReferenceDish]) -> PromptParts` | First generation call |
| `revision_prompt(request: PlanRequest, previous: MealPlan, problems: list[CheckProblem]) -> PromptParts` | Revision call |
| `explain_prompt(plan: MealPlan, action: FollowUpAction, dietary: DietaryPreferences) -> PromptParts` | Explain a step |
| `chat_prompt(context: FollowUpContext, text: str) -> PromptParts` | Chat routing call |

`PromptParts` = system text + list of messages, with user and receipt text inside delimited data blocks.

---

## 5. Domain Functions

| Module | Function | Purpose |
|---|---|---|
| `ingredients.py` | `merge_extractions(per_image: list[ImageExtraction]) -> list[IngredientRow]` | Drop non_food, combine, flag duplicates |
| | `resolve_duplicate(rows, group_id: str, action: Literal["merge", "keep_both"]) -> list[IngredientRow]` | Apply user choice |
| | `pending_review_count(rows: list[IngredientRow]) -> int` | Items blocking confirmation |
| | `to_confirmed(rows: list[IngredientRow]) -> ConfirmedIngredientList` | Build confirmed list (fails if review pending or empty) |
| `calories.py` | `apply_totals(plan: MealPlan) -> tuple[MealPlan, list[CheckProblem]]` | Fill dish `kcal` and plan `total_kcal`; flag impossible estimates |
| | `display_kcal(value: int) -> str` | "about 513 kcal (approximate)" |
| `checks.py` | `check_plan(plan, preferences, dietary, confirmed, terms) -> list[CheckProblem]` | All checks |
| | `changes_from_request(problems: list[CheckProblem]) -> list[str]` | Plan B change list (never includes dietary) |
| | `purchase_label(plan: MealPlan) -> Literal["confirmed_only", "requires_purchases"]` | Plan B label |
| `dietary.py` | `find_conflicts(ingredients: list[RecipeIngredient], dietary: DietaryPreferences, terms: DietaryTerms) -> list[CheckProblem]` | Term list + model tags |
| `reference_search.py` | `find_references(cuisine: Cuisine, ingredients: list[str], data: ReferenceData, limit: int) -> list[ReferenceDish]` | Ranked matches |
| `shopping.py` | `build_shopping_list(plan, confirmed, manual_marks, unmarked) -> ShoppingList` | Combine quantities, attach hints, marks |
| | `combine_quantities(quantities: list[str]) -> str` | Combine like units |
| | `additional_items(plan: MealPlan) -> dict[int, list[RecipeIngredient]]` | For US-FUP-07 |

---

## 6. Infrastructure

### C-21 Config
| Method | Purpose |
|---|---|
| `load_settings() -> Settings` | `api_key: str \| None`, `model: str` (default `claude-sonnet-5-5`), `preferences_path: Path` (`.receiptchef/preferences.json`) |

### C-22 LLM client (`infra/llm_client.py`)
```python
class LLMClient(Protocol):
    def vision_json(self, prompt: PromptParts, image: PreparedImage, schema: type[M]) -> LLMResponse[M]: ...
    def text_json(self, prompt: PromptParts, schema: type[M]) -> LLMResponse[M]: ...

class AnthropicLLMClient:          # implements LLMClient
    def __init__(self, api_key: str, model: str, max_output_tokens: int = 4000) -> None: ...
```
One automatic retry on network error, rate limit, or timeout (FR-10.1). JSON parsing and Pydantic validation happen here; an invalid response returns `status="invalid_output"` (services decide whether to retry).

### C-23 Image preparation
| Method | Purpose |
|---|---|
| `prepare_image(data: bytes, filename: str) -> PrepareResult` | Check type (JPEG / PNG / WebP) and size (10 MB), resize to 1568 px, strip EXIF |

### C-24 Preferences store
| Method | Purpose |
|---|---|
| `__init__(path: Path)` | File location |
| `load() -> PreferencesLoadResult` | missing / ok / corrupt |
| `save(prefs: DietaryPreferences) -> None` | Create folder if needed; write JSON |
| `clear() -> None` | Delete file |

### C-25 Data loader
| Method | Purpose |
|---|---|
| `load_reference_recipes(path: Path) -> ReferenceData` | Validated reference dishes |
| `load_dietary_terms(path: Path) -> DietaryTerms` | Validated term list |

### C-26 Logging
| Method | Purpose |
|---|---|
| `configure_logging() -> None` | Set format and level |
| `log_event(stage: str, step: str, duration_ms: int, revisions: int \| None = None, error_kind: str \| None = None) -> None` | Only allowed fields can be logged |

---

## 7. Test Support

| Component | Methods |
|---|---|
| `FakeLLMClient` | `__init__(responses: list[object])`; implements `vision_json` and `text_json`; `calls: list[str]` for assertions such as "at most 3 generation calls" |
| `tests/strategies.py` | Hypothesis strategies: `dietary_preferences()`, `recipe_ingredients()`, `dishes()`, `meal_plans()`, `ingredient_rows()`, `quantities()` (PBT-07) |
