# ReceiptChef - Unit of Work Plan

**Stage**: INCEPTION - Units Generation (Part 1 approved 2026-11-24; Part 2 generation complete)
**Inputs**: `requirements.md`, `stories.md` (33 stories), `application-design.md` (26 components), `execution-plan.md` (all approved)
**Outputs**: `aidlc-docs/inception/application-design/unit-of-work.md`, `unit-of-work-dependency.md`, `unit-of-work-story-map.md`

ReceiptChef is **one deployable application** (a single local Streamlit app). Units of work here are **modules** inside that app, used to plan, divide, and order Construction work. They are not separate services.

Please answer the questions in Section 3 by filling in the letter after each `[Answer]:` tag. If you choose Other, describe your choice on the line after the tag.

---

## 1. Candidate Decompositions (Reference for Question 1)

### Option A - Five units

| Unit | Contents (components) | Stories |
|---|---|---|
| U1 Foundation | Domain models and results (C-13, C-14), config (C-21), LLM client and `FakeLLMClient` (C-22, T-01), logging (C-26), data loader (C-25), preferences store (C-24), SessionStore (C-07), app shell (C-01), onboarding and settings pages (C-02, C-05), notices widget, Hypothesis strategies (T-02) | ONB-01, 02, 03; TRU-01; ERR-01 |
| U2 Receipt Intake | Image preparation (C-23), extraction service (C-08), ingredient merge (C-15), ingredient table widget, Generator page upload and review sections | UPL-01, 02, 03; REV-01, 02, 03; TRU-02 |
| U3 Recipe Engine | Reference search (C-19), calories (C-16), checks (C-17), dietary rules (C-18), prompts (C-12), RecipePipeline (C-09), preference form and plan view widgets, Generator page results section, `data/reference_recipes.json`, `data/dietary_terms.json` | GEN-01, 02; RES-01, 02; PLB-01, 02, 03; TRU-03 |
| U4 Follow-Up | FollowUpService (C-10), follow-up panel widget | FUP-01 to 09 |
| U5 Planner | PlannerService (C-11), shopping list builder (C-20), Planner page (C-04), shopping list widget | PLN-01 to 04 |

### Option B - Four units
Same as Option A, but U4 Follow-Up is merged into U3 Recipe Engine (17 stories in one unit).

### Option C - Two units (matches story phase labels)
- **Generator unit**: everything in U1-U4 (29 stories)
- **Planner unit**: U5 (4 stories)

| Option | Benefits | Trade-offs |
|---|---|---|
| A | Small units; clear owners; U2 and U3 can be built in parallel after U1 | More hand-offs and per-unit design documents |
| B | Fewer documents; pipeline and follow-up designed together | U3 becomes large |
| C | Matches labels exactly; least overhead | Generator unit is very large and hard to divide among five people |

---

## 2. Execution Checklist (Part 2: Generation)

- [x] **Step 1 - Define units**
  - [x] 1.1 Apply the decomposition from Question 1
  - [x] 1.2 For each unit: purpose, components, responsibilities, interfaces it provides and uses, definition of done
  - [x] 1.3 Document code organization strategy (greenfield): directory layout from `components.md`, which files each unit owns, and shared files
- [x] **Step 2 - Dependencies**
  - [x] 2.1 Unit dependency matrix
  - [x] 2.2 Build sequence and parallel work per Question 2
  - [x] 2.3 Shared contracts and their ownership per Question 4
  - [x] 2.4 Integration checkpoints per Question 6
- [x] **Step 3 - Story map**
  - [x] 3.1 Assign all 33 stories to exactly one primary unit
  - [x] 3.2 Note supporting units for cross-unit stories
  - [x] 3.3 Map journeys A-D to the units they need
- [x] **Step 4 - Team and data**
  - [x] 4.1 Ownership per Question 3
  - [x] 4.2 Bundled data and fixture tasks per Questions 5 and 7
- [x] **Step 5 - Validate**
  - [x] 5.1 Every component in `components.md` belongs to exactly one unit
  - [x] 5.2 Every story is assigned; no dependency cycles between units
  - [x] 5.3 Validate diagrams

### Mandatory Artifacts
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work.md` with unit definitions and responsibilities
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md` with dependency matrix
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md` mapping stories to units
- [x] **Greenfield only**: Document code organization strategy in `unit-of-work.md`
- [x] Validate unit boundaries and dependencies
- [x] Ensure all stories are assigned to units

**Categories evaluated without a question**: *Technical considerations / deployment* - all units ship in one local Streamlit app with the same runtime, so there are no differing scalability or deployment needs. *Code organization* - directory layout was decided in Application Design (Q1: B); Step 1.3 documents file ownership only.

---

## 3. Questions

## Question 1
**Decomposition.** Which decomposition should be used? (See Section 1.)

A) Option A - five units: Foundation, Receipt Intake, Recipe Engine, Follow-Up, Planner

B) Option B - four units: Foundation, Receipt Intake, Recipe Engine with Follow-Up, Planner

C) Option C - two units: Generator, Planner

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
**Build sequence.** In what order should units be built in Construction?

A) Foundation first; then Receipt Intake and Recipe Engine **in parallel** (Recipe Engine uses a sample confirmed list until Receipt Intake is ready); then Follow-Up; then Planner

B) Strictly one after another: Foundation -> Receipt Intake -> Recipe Engine -> Follow-Up -> Planner

C) Foundation first; then all remaining units in parallel

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
**Ownership.** How should units be assigned to team members?

A) Record an owner (or pair) for each unit in `unit-of-work.md`; the team tells me the names in the answer below

B) Leave owners blank; the team assigns them at the next meeting and updates the document

C) Everyone works on every unit together in order (no per-unit owners)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
**Shared contracts.** Domain models (`models.py`, `results.py`) and the `LLMClient` protocol are used by every unit. How should they be managed?

A) Foundation defines them first; later units may add new models, but changes to existing fields need a short note in the unit's Functional Design and agreement from the affected unit owners

B) Each unit defines the models it needs as it goes

C) Freeze them after Foundation; no changes without reopening Foundation

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
**Bundled data content.** Who creates `data/reference_recipes.json` (about 30 dishes per cuisine) and `data/dietary_terms.json` (terms per restriction, for example peanuts -> peanut oil, satay)?

A) The AI drafts both files during the Recipe Engine unit's Code Generation; the team reviews them, with special attention to the dietary terms

B) The team writes both files by hand before Recipe Engine Construction starts

C) The AI drafts `reference_recipes.json`; the team writes `dietary_terms.json` by hand because it is safety-related

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6
**Integration checkpoints.** How should units be checked together?

A) After each unit, run the journey steps that unit enables (for example, after Recipe Engine, Journeys A and B up to Plan B), using the `FakeLLMClient`, and one manual run with the real API

B) Integrate only after all units are complete, then run Journeys A-D

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
**Receipt fixtures.** The 5-10 real receipts (Wegmans, Tops, Aldi, Walmart / Target, Asian grocery store) with card numbers and loyalty IDs blurred are needed for Receipt Intake testing. When and by whom?

A) Team members collect and blur them before Receipt Intake Construction begins; each person contributes 1-2 receipts and writes the expected-ingredients JSON for their own receipts

B) Collected during Receipt Intake Construction by the unit owner

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## 4. Follow-Up Questions (Round 1)

Answers recorded: Q1 A (five units), Q2 A (Foundation, then Receipt Intake and Recipe Engine in parallel, then Follow-Up, then Planner), Q3 A, Q4 A, Q5 C, Q6 A, Q7 A. Two answers need more detail.

## Follow-Up Question 1 (from Question 3)
You chose **A) Record an owner (or pair) for each unit**, but no names were written. Please fill in the table below, then put `Done` after the `[Answer]:` tag. A person may own more than one unit. If you would rather assign owners later, choose B.

| Unit | Owner(s) |
|---|---|
| U1 Foundation | |
| U2 Receipt Intake | |
| U3 Recipe Engine | |
| U4 Follow-Up | |
| U5 Planner | |

A) Done - table filled in above

B) Change to Question 3 option B: leave owners blank; the team assigns them at the next meeting and updates `unit-of-work.md`

C) Other (please describe after [Answer]: tag below)

[Answer]: B (recorded from the team's chat reply on 2026-11-24): "Owners should be agreed by the whole team, not decided by one person at 3 AM. We will assign owners at our Thursday meeting, before Construction starts, and ask the agent to record them through the workflow then."

## Follow-Up Question 2 (from Question 5)
You chose **C**: the AI drafts `reference_recipes.json` and the team writes `dietary_terms.json` by hand. The Recipe Engine's dietary check depends on that file, so it needs a deadline. When should the team deliver it?

A) After U3 Recipe Engine Functional Design defines the file format, and before U3 Code Generation starts. Code Generation waits for the file.

B) During U3 Code Generation. The AI uses a small sample file (only for tests, clearly marked) until the team's file replaces it, and U3 is not marked complete until the team's file is in place.

C) Before Construction starts (the AI provides the expected format now in `unit-of-work.md`)

D) Other (please describe after [Answer]: tag below)

[Answer]: A (recorded from the team's chat reply on 2026-11-24): "The dietary check is safety-related, so we don't want code built against a placeholder file that might never get fully replaced. Waiting for Functional Design to define the format means we write the terms once, in the right shape. The team will write the file between Recipe Engine Functional Design and Code Generation, and code generation for the dietary check waits until it is reviewed."

---

## 5. Decisions Summary (for Plan Approval)

| Topic | Decision |
|---|---|
| Decomposition (Q1 A) | Five units in one app: U1 Foundation, U2 Receipt Intake, U3 Recipe Engine, U4 Follow-Up, U5 Planner (contents as in Section 1, Option A) |
| Build sequence (Q2 A) | U1 first; then U2 and U3 in parallel (U3 uses a sample confirmed list until U2 is ready); then U4; then U5 |
| Ownership (Q3 A -> Follow-Up 1 B) | Owners left blank in `unit-of-work.md`; the whole team assigns them at the Thursday meeting before Construction and records them through the workflow |
| Shared contracts (Q4 A) | U1 defines `models.py`, `results.py`, and the `LLMClient` protocol; later units may add models; changing existing fields needs a note in that unit's Functional Design and agreement from affected unit owners |
| Bundled data (Q5 C, Follow-Up 2 A) | AI drafts `data/reference_recipes.json` in U3 Code Generation for team review. The team hand-writes `data/dietary_terms.json` after U3 Functional Design defines its format; **no placeholder file is used**; Code Generation for the dietary check waits until the team's file is written and reviewed |
| Integration (Q6 A) | After each unit, run the journey steps it enables with `FakeLLMClient`, plus one manual run with the real API |
| Receipt fixtures (Q7 A) | Each team member collects and blurs 1-2 receipts and writes the expected-ingredients JSON for them, before U2 Construction begins |
