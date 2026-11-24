# ReceiptChef - Requirements

**Status**: Draft for team review
**Date**: 2026-11-24
**Sources**: `vision.md`, `technical-environment.md`, `README.md`, `requirement-verification-questions.md` (Q1-Q25), `requirement-clarification-questions.md` (CQ1-CQ4)

---

## 1. Intent Analysis Summary

| Item | Assessment |
|---|---|
| **User request** | "Using AI-DLC, I want to start a new project. Please read vision.md, technical-environment.md, and README.md, then begin the AI-DLC workflow." |
| **Request type** | New Project (greenfield) |
| **Scope estimate** | System-wide: a new local Streamlit web app with onboarding, Generator, Planner, follow-up conversation, and Claude API integration |
| **Complexity estimate** | Moderate-to-Complex: multimodal LLM calls, a constraint-checking and revision pipeline, two linked features, five-person team |
| **Requirements depth** | Comprehensive (multiple stakeholders, conflicting source documents, traceability needed) |

### 1.1 Source Precedence

Where `technical-environment.md` conflicts with `vision.md` / `README.md`, the team chose the vision / README behavior (Q1-Q5). **This document takes precedence** over all three sources. Proposed edits to bring `technical-environment.md` in line are in `technical-environment-proposed-changes.md` for team review (Q6: C).

### 1.2 Key Decisions From Questions

| Ref | Decision |
|---|---|
| Q1 | Up to 3 receipt images per request, combined into one ingredient list |
| Q2 | User chooses number of dishes (1-3); no servings input; calories per dish |
| Q3 | Max total time 15 / 30 / 45 / 60 min, applied to the whole set of dishes |
| Q4 | Per-dish calorie range: Below 400 / 400-below 600 / 600-800 kcal / No preference |
| Q5 | App-coordinated pipeline (no autonomous tool selection) |
| Q6 | AI proposes `technical-environment.md` edits for team review |
| Q7 | Test fixtures from Wegmans, Tops, Aldi, Walmart / Target, and Asian grocery stores |
| Q8 | Unclear items highlighted in the same table with raw text, best guess, and a plain-language explanation |
| Q9 | Quantities left blank by default; recipes state the amount needed |
| Q10 | Dietary preferences saved in a local JSON file until cleared |
| Q11 | Receipt images kept in memory for the session only; never written to disk |
| Q12 | Dietary input: checklist of common items plus optional free text |
| Q13 | Generator and Planner are both Must-have |
| Q14 / CQ1 | Follow-up: predefined action buttons **plus** a free-text chat box |
| Q15 | Planner proposes dishes directly; user refines through follow-up |
| Q16 | Shopping list: combined recipe units plus a suggested store-purchase unit |
| Q17 | Planner auto-marks items found in the Generator confirmed list as "already have" (undoable) |
| Q18 | English only |
| Q19 | No explicit response-time target; best effort with progress indicator |
| Q20 | One automatic retry on API failure, then friendly error with "Try again" |
| Q21 | Live receipt test reports extraction percentage only (no pass/fail threshold) |
| Q22 | This workflow stops after Units Generation is approved |
| Q23-Q25 | Security Baseline: No. Resiliency Baseline: No. Property-Based Testing: Partial |
| CQ2 | Bundled reference recipes are included in each recipe-generation prompt |
| CQ3 | Up to 2 revisions (at most 3 generation calls per request) |
| CQ4 | Receipt quantities extracted and shown read-only as "on receipt"; never treated as amount available |

---

## 2. Users and Context

- **Primary user**: An international student cooking with groceries from local US supermarkets, who may not recognize abbreviated receipt entries or local products.
- **Usage context**: Runs locally on a teammate's laptop or GitHub Codespace (`http://localhost:8501`). Single user per running instance; no accounts.
- **Supported cuisines**: Chinese, Italian, Mexican.

---

## 3. Functional Requirements

Priority uses MoSCoW. Every requirement below is **Must** unless marked otherwise.

### FR-1 Onboarding and Dietary Preferences

| ID | Requirement |
|---|---|
| FR-1.1 | On first launch (no saved preferences file), the app shows a short explanation of what ReceiptChef does, then asks about allergies and dietary restrictions. |
| FR-1.2 | Preferences are entered through a checklist of common items: peanuts, tree nuts, dairy, eggs, gluten, shellfish, vegetarian, vegan, halal, no pork, no beef. An optional free-text field allows other restrictions (max 200 characters). |
| FR-1.3 | The user can submit preferences, choose "I have none", or skip. Skipping records that onboarding was seen so it is not shown again on every launch. |
| FR-1.4 | Preferences are saved to a local JSON file on the user's machine (not a database) and reused across app restarts. |
| FR-1.5 | The user can review and edit preferences at any time from a settings area. |
| FR-1.6 | A "Clear preferences" button deletes the saved preferences file. The next launch behaves as a first launch. |
| FR-1.7 | Saved preferences are applied to every recipe generation, Plan B, Planner request, and follow-up response in both Generator and Planner. |
| FR-1.8 | The app never relaxes an allergy or dietary restriction to meet a time, calorie, or dish-count preference. |
| FR-1.9 | The app shows a notice that it cannot guarantee recipes are allergen-free and does not give medical advice. |

### FR-2 Generator: Receipt Upload and Extraction

| ID | Requirement |
|---|---|
| FR-2.1 | The user can upload 1 to 3 receipt images in one request. Accepted formats: JPEG, PNG, WebP. Max 10 MB per image. More than 3 images is rejected with a clear message. |
| FR-2.2 | Before any image is sent to the model, the app validates it, strips EXIF metadata (including GPS), and resizes it to at most 1568 px on the long edge. |
| FR-2.3 | The model extracts, for each receipt line: raw text as printed, a best-guess ingredient name, quantity and unit when readable, and a status of `food`, `unclear`, or `non_food`. |
| FR-2.4 | For `unclear` items and unfamiliar products, the model also returns a short plain-language explanation (for example, "GRN ONION BCH: a bunch of green onions / scallions"). |
| FR-2.5 | `non_food` lines (bags, detergent, tax, totals) are excluded from the ingredient list. |
| FR-2.6 | Items extracted from all uploaded receipts are combined into one ingredient list. Repeated items across or within receipts are flagged for the user to review (merge or keep separate) rather than silently merged. |
| FR-2.7 | If an image is unreadable or is not a grocery receipt, the app identifies which image failed and asks for a clearer upload. Items from the other readable images are still shown. |
| FR-2.8 | Extraction output is validated against a schema. If validation fails, the app retries once, then shows an error. |
| FR-2.9 | The model is instructed to ignore payment details, card numbers, and loyalty IDs and never return them. |
| FR-2.10 | Uploaded images are kept in memory for the current session only (so extraction can be re-run) and are never written to disk. They are discarded when the session ends. |

### FR-3 Generator: Ingredient Review and Confirmation

| ID | Requirement |
|---|---|
| FR-3.1 | The combined ingredient list is shown in one editable table before any recipe is generated. |
| FR-3.2 | Unclear rows are visually highlighted and show the raw receipt text, the best-guess name, and the plain-language explanation side by side. |
| FR-3.3 | The user can rename, remove, and add ingredients, including pantry staples (oil, salt, rice, soy sauce, etc.). |
| FR-3.4 | Quantities read from the receipt are shown in a read-only "on receipt" column. They are informational only and are never treated as the amount available. |
| FR-3.5 | There is no "amount available" input. Recipes state the amount they need; the user decides whether they have enough. |
| FR-3.6 | Ingredient names are limited to 60 characters and the list to 60 items. |
| FR-3.7 | The user must explicitly confirm the list. Only the confirmed list is used for recipe generation. Unresolved unclear items must be confirmed, edited, or removed before confirmation. |

### FR-4 Recipe Preferences

| ID | Requirement |
|---|---|
| FR-4.1 | Before generating, the user chooses: number of dishes (1, 2, 3); maximum total time (15, 30, 45, 60 minutes); per-dish calorie range (Below 400, 400-below 600, 600-800 kcal, No preference); cuisine (Chinese, Italian, Mexican, No preference). |
| FR-4.2 | There is no servings or number-of-people input. |
| FR-4.3 | The time limit applies to preparing and cooking the complete set of dishes, not to each dish separately. |
| FR-4.4 | The calorie range applies to each dish using all quantities listed in that recipe. It is not per person or per serving. |
| FR-4.5 | "No preference" cuisine lets the model choose among Chinese, Italian, and Mexican, and each dish states its chosen cuisine. |

### FR-5 Recipe Generation Pipeline (App-Coordinated)

| ID | Requirement |
|---|---|
| FR-5.1 | Recipe generation is a fixed pipeline controlled by the app, not an autonomous tool-using agent. The model does not choose which steps run. |
| FR-5.2 | **Step 1 - Reference lookup**: before each generation call, the app looks up matching dishes in the bundled `data/reference_recipes.json` (about 30 dishes per cuisine: name, typical ingredients, typical total time) for the selected cuisine (or all three for No preference) and the confirmed ingredients, and includes them in the prompt as context. |
| FR-5.3 | **Step 2 - Generate**: the model returns a structured plan with the requested number of dishes, including per-ingredient calorie estimates for the listed quantities (including oil and sauces). |
| FR-5.4 | **Step 3 - Calculate**: the app computes each dish's calorie total and the plan's combined total in Python from the model's per-ingredient estimates. |
| FR-5.5 | **Step 4 - Check**: the app checks the plan against: total time vs. limit, each dish's calories vs. range, cuisine, dish count, dietary preferences, and whether each ingredient is on the confirmed list (if not, it must be listed as an additional ingredient or an assumed pantry item). |
| FR-5.6 | **Step 5 - Revise**: if any check fails, the app sends the specific problems back to the model and asks for a revision. At most **2 revisions** (at most **3 generation calls** per request). |
| FR-5.7 | If the plan still fails after 2 revisions, the app stops and presents the conflict explanation and Plan B (FR-7). |
| FR-5.8 | All model output is parsed as JSON and validated against a schema before use. No model output is ever executed. |
| FR-5.9 | Recipe-generation output is limited to 4,000 output tokens per model call. |

### FR-6 Recipe Results

| ID | Requirement |
|---|---|
| FR-6.1 | Each dish shows: name, cuisine, ingredients with suggested quantities, clear step-by-step instructions, approximate calories for the dish, and any additional ingredients needed. |
| FR-6.2 | The plan shows estimated total preparation and cooking time for all dishes, with any important assumptions (for example, preparing one item while another cooks). |
| FR-6.3 | The plan shows estimated total calories for all dishes combined, equal to the sum of the individual dish estimates. |
| FR-6.4 | All calorie figures are labeled "approximate" and shown as whole numbers (no decimals). The app does not claim actual calories fall within the selected range. |
| FR-6.5 | Recipes prioritize confirmed ingredients. Unconfirmed pantry items and additional ingredients are listed separately and clearly, never assumed available. |
| FR-6.6 | From any result, the user can send the plan to Planner to build a shopping list for missing ingredients (see FR-9.7). |

### FR-7 Plan B (Conditions Cannot All Be Met)

| ID | Requirement |
|---|---|
| FR-7.1 | When the selected conditions cannot all be met, the app first explains which requirements conflict (for example, "three dishes cannot fit in 15 minutes with these ingredients"). |
| FR-7.2 | The app then offers a Plan B and clearly identifies every change from the original request (for example, fewer dishes, longer time). |
| FR-7.3 | Plan B is labeled as either "uses confirmed ingredients" or "requires additional purchases". |
| FR-7.4 | Plan B is never presented as meeting the original conditions when it does not. |
| FR-7.5 | Allergies and dietary restrictions continue to apply to Plan B. |
| FR-7.6 | If the confirmed ingredients are insufficient for a reasonable recipe, the app explains what is missing and offers to create a shopping list (handoff to Planner). |
| FR-7.7 | The user can accept Plan B, change preferences and regenerate, or make a follow-up request. |

### FR-8 Follow-Up Requests

| ID | Requirement |
|---|---|
| FR-8.1 | Below the current plan (Generator and Planner), the app offers predefined action buttons: Replace an ingredient; Suggest a different dish; Change the number of dishes; Adjust time or calorie preference; Explain a cooking step; Suggest other meals with these ingredients; Identify additional ingredients to buy. |
| FR-8.2 | Actions that need details collect them with structured inputs (for example, a dropdown for which dish, ingredient, or step, and a short text field up to 60 characters for a replacement ingredient). |
| FR-8.3 | A free-text chat box lets the user ask open-ended questions about the current plan. Messages are limited to 500 characters (assumption, see Section 7). |
| FR-8.4 | Every follow-up uses the current confirmed ingredient list (or Planner's "already have" list), the current plan, and the saved dietary preferences. |
| FR-8.5 | Follow-ups that change the recipes go through the same calculate / check / revise pipeline (FR-5.4 to FR-5.7), and the app updates ingredient quantities, per-dish calories, total calories, total time, additional ingredients, and (in Planner) the shopping list. |
| FR-8.6 | Follow-ups that only explain (for example, "Explain a step") return text and do not change the plan. |
| FR-8.7 | Chat messages and typed text are treated as user requests about cooking, never as instructions that override the system prompt, dietary preferences, or app rules. Off-topic requests receive a short polite redirect. |
| FR-8.8 | Conversation history is kept for the current session only. |

### FR-9 Planner: Meal Ideas to Shopping List

| ID | Requirement |
|---|---|
| FR-9.1 | The user can start Planner by describing what they want to eat in free text (max 500 characters), without uploading a receipt. |
| FR-9.2 | The user applies the same preferences as Generator (dish count, cuisine, time, calorie range). |
| FR-9.3 | The model proposes dishes directly, applying the preferences and dietary restrictions, without a clarifying-question round. The user refines through follow-up (FR-8). |
| FR-9.4 | Planner recipes go through the same pipeline (FR-5), and show per-dish calories, total calories, and total time. |
| FR-9.5 | Once the user accepts the dishes, the app produces a combined shopping list. Repeated ingredients across dishes are combined into a total amount in recipe units. |
| FR-9.6 | Each shopping-list line shows the combined recipe quantity plus a suggested store-purchase unit (for example, "chicken thigh: 600 g - about 1 family pack"). |
| FR-9.7 | If a confirmed Generator ingredient list exists in the session, matching shopping-list items are automatically marked "already have". The user can undo any mark. |
| FR-9.8 | The user can mark or unmark any item as "already have"; marked items are removed from the to-buy list. |
| FR-9.9 | The shopping list is always derived from the current recipes and quantities, and is recalculated whenever the recipes change. |
| FR-9.10 | The user can move from a Generator plan to Planner to buy ingredients for a suggested recipe; the Generator dishes and confirmed list carry over. |

### FR-10 Error Handling

| ID | Requirement |
|---|---|
| FR-10.1 | On an Anthropic API failure (network error, rate limit, timeout), the app retries automatically once. If the retry fails, it shows a friendly error with a "Try again" button. |
| FR-10.2 | On any error, the user's ingredient list, preferences, current plan, and conversation are kept. |
| FR-10.3 | A progress indicator is shown during every model call. |
| FR-10.4 | If the API key is missing, the app shows a setup message pointing to `.env.example` instead of crashing. |

---

## 4. Non-Functional Requirements

### NFR-1 Technology Stack

| ID | Requirement |
|---|---|
| NFR-1.1 | Python 3.12+, managed with **uv** only; configuration in `pyproject.toml`; `uv.lock` committed. |
| NFR-1.2 | Streamlit UI; Anthropic Python SDK (direct, no LangChain / LlamaIndex / CrewAI); Pydantic v2 for every structured object; Pillow for image handling; python-dotenv for configuration. |
| NFR-1.3 | Model `claude-sonnet-5-5`, read from the `RECEIPTCHEF_MODEL` environment variable (teams may use `claude-haiku-5-5` in development). |
| NFR-1.4 | All model calls go through a single client module (`src/receiptchef/llm_client.py`), so the provider can be swapped (backup: local Ollama vision model). |
| NFR-1.5 | Layout: application code in `src/receiptchef/`, tests in `tests/`, bundled data in `data/`. No application code in `aidlc-docs/`. |
| NFR-1.6 | Prohibited: OCR libraries, external nutrition APIs, any database (SQLite, PostgreSQL, Firebase), image-generation APIs, Flask / Django / React, `eval()` / `exec()` / `pickle`, pip / poetry / conda, black / flake8 / isort. |
| NFR-1.7 | The only persistent file the app writes is the dietary preferences JSON file (FR-1.4). This is an approved exception to the earlier "session-only" rule and is not a database. |

### NFR-2 Privacy and Security Basics

The Security Baseline extension is disabled (Q23), but these project-specific basics are required:

| ID | Requirement |
|---|---|
| NFR-2.1 | `ANTHROPIC_API_KEY` is stored only in a local `.env` file listed in `.gitignore`; `.env.example` with an empty value is committed. No hard-coded keys. |
| NFR-2.2 | EXIF metadata is stripped before any image leaves the machine. Images and results are never written to disk. |
| NFR-2.3 | Payment details, card numbers, and loyalty IDs are never returned or displayed. |
| NFR-2.4 | Receipt text, ingredient names, Planner descriptions, follow-up inputs, and chat messages are treated as data, not instructions (prompt-injection mitigation). Input length limits: ingredient names 60 chars, list 60 items, dietary free text 200 chars, Planner description and chat 500 chars. |
| NFR-2.5 | All model output is validated with Pydantic before use; nothing from model output is executed. |
| NFR-2.6 | Logging records stage names, step names, timings, revision counts, and errors only. Never log the API key, images, full prompts, or dietary preferences. |
| NFR-2.7 | The preferences JSON file is stored in a user-local, git-ignored location. |

### NFR-3 Performance and Cost

| ID | Requirement |
|---|---|
| NFR-3.1 | No explicit response-time target for the MVP; best effort with a visible progress indicator (Q19). |
| NFR-3.2 | Cost controls: at most 3 generation calls per recipe request (FR-5.6), at most 4,000 output tokens per call, one automatic retry on API failure (FR-10.1), one retry on extraction validation failure (FR-2.8). |

### NFR-4 Usability and Content

| ID | Requirement |
|---|---|
| NFR-4.1 | English-only UI and recipes. |
| NFR-4.2 | Plain-language explanations suitable for international students unfamiliar with US product names. |
| NFR-4.3 | Calories always labeled "approximate", whole numbers only. |
| NFR-4.4 | Recipe cards and Plan B include a note that the app makes no allergy, medical, or precise-nutrition claims. |

### NFR-5 Testing and Quality

| ID | Requirement |
|---|---|
| NFR-5.1 | pytest with pytest-cov: at least 80% line coverage for `src/receiptchef/`, excluding the Streamlit UI file. |
| NFR-5.2 | ruff for linting and formatting; mypy for type checking. |
| NFR-5.3 | No real API calls in unit tests: `llm_client` is replaced by a fake client returning recorded responses. |
| NFR-5.4 | Pure functions (calorie totals, constraint checks, shopping-list combination, reference lookup, image preparation, schema validation, preferences file read / write) are unit tested directly. |
| NFR-5.5 | Receipt fixtures: 5-10 real receipts covering Wegmans, Tops, Aldi, Walmart / Target, and at least one Asian grocery store, with card numbers and loyalty IDs blurred, stored in `tests/fixtures/receipts/`, each paired with an expected-ingredients JSON file. |
| NFR-5.6 | A manually run `@pytest.mark.live` test calls the real API and **reports** the percentage of expected ingredients found. No pass/fail threshold for the MVP (Q21). |
| NFR-5.7 | Calorie spot check: `scripts/check_calories.py` compares about 10 estimates against `tests/fixtures/calorie_reference.json`; run manually. |
| NFR-5.8 | Test scenarios must cover: receipt interpretation, recipe clarity, dietary-preference handling, shopping-list consistency, and behavior when conditions cannot be met (Plan B). |

### NFR-6 Property-Based Testing (Extension: Partial)

| ID | Requirement |
|---|---|
| NFR-6.1 | **Hypothesis** is the PBT framework (PBT-09), added as a dev dependency and run through pytest. |
| NFR-6.2 | PBT applies to pure functions and serialization round-trips. Blocking rules: PBT-02 (round-trip), PBT-03 (invariants), PBT-07 (domain generators), PBT-08 (shrinking and seed reproducibility), PBT-09 (framework). Other PBT rules are advisory. |
| NFR-6.3 | Candidate properties to be confirmed in Functional Design: preferences JSON write / read round-trip; Pydantic model serialize / parse round-trip; total calories always equals the sum of dish calories; combined shopping-list total for each ingredient equals the sum across recipes; marking "already have" never adds items to the to-buy list; constraint check flags every non-confirmed ingredient not listed as additional or pantry. |

### NFR-7 Deployment

| ID | Requirement |
|---|---|
| NFR-7.1 | Runs locally only: `uv run streamlit run src/receiptchef/app.py` at `http://localhost:8501`. No cloud deployment, Docker, or hosting. |
| NFR-7.2 | The only external service is the Anthropic API over HTTPS. |

---

## 5. Out of Scope (MVP)

- Food-photo recognition; image generation
- Non-English receipts or UI
- More than 3 receipt images per request; more than 3 dishes per plan
- Servings or number-of-people inputs
- Nutrition database integration or precise nutrition analysis
- Personalized daily calorie targets or long-term diet plans
- Grocery ordering, price comparison, cost calculation, or guaranteed savings
- Automatic pantry stock or remaining-quantity tracking
- Medical dietary advice or allergen-free guarantees
- User accounts, databases, cloud deployment

---

## 6. Constraints and Milestones

- **Final deadline**: December 16, 2026. Generator and Planner are both Must-have (Q13); Generator is built first.
- **This milestone**: complete Inception and approve units of work, then stop before Construction (Q22).
- **Extensions**: Security Baseline disabled; Resiliency Baseline disabled; Property-Based Testing Partial.

---

## 7. Assumptions (Please Confirm or Change During Review)

| ID | Assumption | Reason |
|---|---|---|
| A-1 | Chat messages and Planner descriptions are limited to 500 characters; dietary free text to 200. | CQ1 added an open chat box; a length limit is needed for prompt-injection and cost control. |
| A-2 | Duplicate items are flagged for review (not merged automatically). | vision.md 2.2: "Review repeated or unclear entries". |
| A-3 | Unresolved unclear items block confirmation until the user confirms, edits, or removes them. | vision.md 2.2: unclear items must not be silently treated as correct. |
| A-4 | If one of several images is unreadable, items from the readable images are still shown and only the failed image needs re-upload. | Keeps user progress with multi-image upload. |
| A-5 | Skipping onboarding is remembered (in the preferences file) so it is not shown on every launch. | Avoids repeated prompts; user can still open settings. |
| A-6 | Suggested store-purchase units are model estimates and labeled as approximate. | No price or product database (out of scope). |

---

## 8. Traceability to Vision

| vision.md section | Requirements |
|---|---|
| 2.1 Onboarding and Dietary Preferences | FR-1 |
| 2.2 Generator: Receipts | FR-2, FR-3 |
| 2.3 Recipe Preferences | FR-4 |
| 2.4 Recipe Results | FR-5, FR-6 |
| 2.5 Plan B | FR-7 |
| 2.6 Follow-Up | FR-8 |
| 2.7 Planner | FR-9 |
| 3 Out of Scope | Section 5 |
| 4 Open Questions | Q7 -> NFR-5.5; Q8 -> FR-3.2; Q9 / CQ4 -> FR-3.4, FR-3.5; Q10 -> FR-1.4, FR-1.6; Q11 -> FR-2.10 |
