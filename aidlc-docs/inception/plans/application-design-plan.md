# ReceiptChef - Application Design Plan

**Stage**: INCEPTION - Application Design
**Inputs**: `requirements.md`, `stories.md`, `personas.md`, `execution-plan.md` (all approved)
**Outputs**: `aidlc-docs/inception/application-design/` - `components.md`, `component-methods.md`, `services.md`, `component-dependency.md`, `application-design.md`

Please answer the questions in Section 3 by filling in the letter after each `[Answer]:` tag. If you choose Other, describe your choice on the line after the tag. Detailed business rules (exact check logic, calorie rules, prompt wording) are decided later in Functional Design; these questions are about structure.

---

## 1. Draft Component List (for Reference)

This is my starting point. Your answers may change how these are grouped or split.

| Component | Purpose | Main stories |
|---|---|---|
| UI pages (Streamlit) | Onboarding, Generator, Planner, settings, shared follow-up panel | All |
| Session state store | Holds images, ingredient list, plans, conversation for the session | UPL, REV, FUP, PLN, ERR |
| Preferences store | Reads, writes, clears the local preferences JSON file | ONB |
| LLM client | The only module that calls the Anthropic API; retry, token limit, model name | All model-using stories, ERR-01 |
| Image preparation | Validate, resize, strip EXIF | UPL-01, TRU-02 |
| Receipt extractor | Vision call(s), schema validation, merge across receipts, duplicate flags | UPL-02, UPL-03, REV-01 |
| Reference recipe lookup | Search `data/reference_recipes.json` | GEN-02 |
| Calorie calculator | Per-dish and plan totals in Python | GEN-02, RES-02 |
| Constraint checker | Time, calories, cuisine, dish count, dietary, unconfirmed ingredients | GEN-02, PLB-03 |
| Recipe pipeline service | Lookup -> generate -> calculate -> check -> revise (max 2) -> Plan B | GEN-02, PLB-01 to 03, FUP-09, PLN-01 |
| Follow-up service | Action buttons and chat, routed to explain-only or plan-change | FUP-01 to 09 |
| Shopping list builder | Combine quantities, store units, "already have" marking | PLN-02 to 04 |
| Domain models (Pydantic) | All structured objects | All |

---

## 2. Execution Checklist

- [x] **Step 1 - Components**
  - [x] 1.1 Finalize component list and package layout (Q1)
  - [x] 1.2 Define UI structure (Q2) and session state approach (Q3)
  - [x] 1.3 Write responsibilities and interfaces for each component
- [x] **Step 2 - Methods**
  - [x] 2.1 Write method signatures with input and output types for each component
  - [x] 2.2 Define the LLM client interface and test fake (Q12)
  - [x] 2.3 Define the error types or result objects (Q11)
- [x] **Step 3 - Services**
  - [x] 3.1 Define the recipe pipeline service and how Generator, Planner, and follow-up reuse it (Q5)
  - [x] 3.2 Define extraction orchestration (Q6)
  - [x] 3.3 Define follow-up and chat routing (Q7)
  - [x] 3.4 Define where dietary checking and store units come from (Q8, Q9)
  - [x] 3.5 Define prompt organization (Q4)
- [x] **Step 4 - Dependencies**
  - [x] 4.1 Dependency matrix (no cycles; UI depends on services, services on domain and LLM client)
  - [x] 4.2 Data flow diagrams for Journeys A-D
  - [x] 4.3 Preferences file location (Q10)
- [x] **Step 5 - Consolidate and validate**
  - [x] 5.1 Write `application-design.md` combining all artifacts
  - [x] 5.2 Check every story maps to at least one component
  - [x] 5.3 Validate Mermaid and ASCII diagrams

### Mandatory Artifacts
- [x] Generate components.md with component definitions and high-level responsibilities
- [x] Generate component-methods.md with method signatures (business rules detailed later in Functional Design)
- [x] Generate services.md with service definitions and orchestration patterns
- [x] Generate component-dependency.md with dependency relationships and communication patterns
- [x] Validate design completeness and consistency

---

## 3. Questions

## Question 1
**Package layout.** How should code inside `src/receiptchef/` be organized?

A) Flat: one module per component (for example `llm_client.py`, `extraction.py`, `pipeline.py`, `checks.py`, `shopping.py`, `preferences.py`, `models.py`, `app.py`). Simple to navigate for a small team.

B) Layered subpackages: `ui/`, `services/`, `domain/` (models and pure functions), `infra/` (LLM client, image preparation, preferences file)

C) Feature subpackages: `generator/`, `planner/`, `shared/` (pipeline, checks, LLM client, models)

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2
**UI structure.** How should the Streamlit UI be organized?

A) Streamlit multipage app: separate pages for Generator, Planner, and Settings, with onboarding shown on first launch

B) One `app.py` with tabs for Generator and Planner, and settings in the sidebar

C) One `app.py` with sidebar navigation between Generator, Planner, and Settings

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
**Session state.** How should the UI read and write session data?

A) A typed session-state wrapper (one component with methods such as `get_confirmed_list()` and `set_current_plan()`), so pages never use raw `st.session_state` keys

B) Pages use `st.session_state` directly with shared key constants

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
**Prompt organization.** Where should system prompts and prompt templates live?

A) A dedicated `prompts.py` module with one function per prompt (for example `build_generation_prompt(...)`)

B) Plain-text template files in a `prompts/` directory, loaded at runtime

C) Inline in each service

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
**Pipeline reuse.** Generator, Plan B, Planner, and plan-changing follow-ups all need generate -> calculate -> check -> revise. How should this be shared?

A) One `RecipePipeline` service with a request object describing the source (receipt list, Planner description, or follow-up change) and the current plan; all callers use it

B) Separate services for Generator, Planner, and follow-up that share the calculator and checker functions but each run their own loop

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
**Receipt extraction calls.** How should up to three images be sent to the model?

A) One vision call per image, run one after another, then merged by the app (makes it easy to report which image failed, US-UPL-03)

B) One vision call per image, run at the same time (faster), then merged by the app

C) One vision call containing all images, with the model told to label which image each item came from

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
**Follow-up and chat routing.** How should the app decide whether a request only explains or changes the plan?

A) Action buttons are fixed: each button is known to be explain-only or plan-changing. For chat, the model returns a structured response of type "answer" or "plan change"; plan changes then go through the pipeline checks.

B) Same as A, but chat can only answer questions; to change the plan the user must use a button

C) Every follow-up (button or chat) always returns a full revised plan, even for explanations

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
**Dietary check source.** How should the constraint checker detect dietary conflicts (for example peanuts in US-PLB-03)?

A) Both: a bundled term list per restriction (for example `data/dietary_terms.json`: peanuts -> peanut, groundnut, peanut oil, satay ...) checked in Python, **plus** the model must tag each ingredient with any allergens or restriction conflicts it knows of. A conflict from either source fails the check.

B) Bundled term list only (deterministic, easy to test, may miss unusual names)

C) Model tags only (flexible, but not deterministic)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
**Store-purchase units for the shopping list** (US-PLN-02). Where do they come from?

A) The model includes a suggested store unit for each ingredient when it generates the recipe; the shopping-list builder combines quantities in Python and uses the model's unit suggestion

B) A separate model call when the shopping list is built, given the combined quantities

C) A small bundled lookup table (for example `data/store_units.json`) for common items, with the model's suggestion as a fallback

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10
**Preferences file location** (git-ignored, NFR-2.7).

A) In the user's home directory: `~/.receiptchef/preferences.json` (outside the repository, so it can't be committed by mistake)

B) In the project folder: `.receiptchef/preferences.json`, listed in `.gitignore`

C) Configurable through an environment variable, defaulting to option A

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 11
**Error handling style between components.**

A) Typed exceptions (for example `ExtractionError`, `LLMUnavailableError`, `PlanConflict`), caught in the UI layer, which shows the friendly message

B) Result objects returned from services (for example `PipelineResult` with status `ok`, `plan_b`, `no_safe_plan`, or `error`); exceptions only for unexpected failures

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12
**LLM client interface and test fake** (NFR-1.4, NFR-5.3).

A) Define an `LLMClient` protocol (interface) with methods such as `extract_receipt(...)` and `generate_json(...)`; the real Anthropic client and a `FakeLLMClient` with recorded responses both implement it; services receive the client as a constructor argument

B) One concrete Anthropic client module; tests replace it with monkeypatching

C) Other (please describe after [Answer]: tag below)

[Answer]: A
