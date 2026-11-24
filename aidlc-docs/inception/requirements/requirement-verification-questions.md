# ReceiptChef - Requirements Verification Questions

Please answer each question by filling in the letter choice after the `[Answer]:` tag. If none of the options match, choose the last option (Other) and describe your preference after the tag. Where a question says "multiple allowed", list several letters (for example `A, C`).

These questions were generated after reading `vision.md`, `technical-environment.md`, and `README.md`. They are grouped into four parts:

- **Part 1** - Conflicts between the input documents (must be resolved before requirements can be written)
- **Part 2** - Open questions listed in `vision.md` Section 4
- **Part 3** - Other functional and non-functional gaps
- **Part 4** - AI-DLC extension opt-ins

---

## Part 1: Conflicts Between Input Documents

`vision.md` and `README.md` agree with each other, but `technical-environment.md` describes an earlier version of the product in several places. Each question below quotes both sides.

## Question 1
**Receipts per request.** `vision.md` 2.2 says users can upload **up to three** receipt images in one request. `technical-environment.md` says the user uploads **one photo** and that uploads are "one image at a time", with Stage 1 being "one vision call". Which should the MVP support?

A) Up to three receipt images per request, combined into one ingredient list (vision.md / README.md)

B) Exactly one receipt image per request (technical-environment.md)

C) Up to three images, but each uploaded and extracted one at a time, then merged into one list

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
**Servings vs. number of dishes.** `vision.md` 2.3 says users choose the **number of dishes (1-3)** and explicitly puts "inputs for the number of people or servings" out of scope. `technical-environment.md` uses **servings 1-6** and "calories per serving". Which model is correct?

A) Number of dishes (1, 2, or 3); no servings input; calories are per dish for the listed quantities (vision.md / README.md)

B) Servings (1-6); the agent returns up to three recipes; calories per serving (technical-environment.md)

C) Both: number of dishes plus a servings input

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
**Time preference.** `vision.md` 2.3 offers **15, 30, 45, or 60 minutes** as the maximum total time for the **complete set of dishes**. `technical-environment.md` uses a **10-120 minute slider** and an estimated total time **per recipe**. Which should the MVP use?

A) Fixed options 15 / 30 / 45 / 60 minutes, applied to the whole set of dishes (vision.md / README.md)

B) A 10-120 minute slider, applied per recipe (technical-environment.md)

C) Fixed options 15 / 30 / 45 / 60 minutes, but shown both per dish and as a total for the set

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
**Calorie preference.** `vision.md` 2.3 offers per-dish ranges **Below 400, 400-below 600, 600-800 kcal, or No preference**. `technical-environment.md` uses a **min/max slider (200-1200) per serving**. Which should the MVP use?

A) Fixed per-dish ranges: Below 400 / 400-below 600 / 600-800 / No preference (vision.md / README.md)

B) Min/max slider per serving (technical-environment.md)

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
**Recipe generation orchestration.** `README.md` says the initial plan uses "model calls coordinated by the app ... **rather than autonomous tool selection**". `technical-environment.md` describes a **tool-using agent** where "Claude decides which tools to call and in what order" (`search_reference_recipes`, `calculate_calories`, `check_recipe_constraints`, max 10 tool calls). Which approach should Construction follow?

A) Tool-using agent: Claude chooses when to call the three tools, capped at 10 tool calls (technical-environment.md)

B) App-coordinated pipeline: the app calls the model, then always runs the calorie and constraint checks itself in a fixed order, and asks the model to revise if checks fail (README.md)

C) Hybrid: the model may call `search_reference_recipes` as a tool, but the app always runs `calculate_calories` and `check_recipe_constraints` on every draft before showing it

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 6
**Updating the technical environment document.** `technical-environment.md` does not cover Planner, onboarding / dietary preferences, follow-up conversation, Plan B, or combined totals. After the conflicts above are resolved, what should happen to that document?

A) The team (Zhiqu) updates `technical-environment.md` to match the approved requirements before Construction

B) Leave `technical-environment.md` as-is; the approved `requirements.md` takes precedence wherever they differ

C) The AI proposes the edits to `technical-environment.md` for team review

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Part 2: Open Questions From vision.md Section 4

## Question 7
**Receipt formats to prioritize for testing.** Which local supermarket receipt formats should the test fixtures prioritize? (multiple allowed)

A) Wegmans

B) Tops

C) Aldi

D) Walmart / Target

E) Asian grocery stores (any local store the team shops at)

F) Other (please describe after [Answer]: tag below)

[Answer]: ABCDE

## Question 8
**Presenting unclear items.** How should the app present abbreviations or unfamiliar products that need confirmation (for example `GRN ONION BCH`)?

A) In the same editable ingredient table, with unclear rows highlighted and showing the raw receipt text next to the model's best-guess name

B) In a separate "Please review" section above the main list; items move into the main list only after the user confirms, edits, or removes them

C) As option A, plus a short plain-language explanation of unfamiliar products (for example "a bunch of green onions / scallions")

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 9
**Ingredient quantity clarification.** The app must not assume the purchased quantity is what remains. When a recipe depends on how much of an ingredient is available, how should the app handle it?

A) Pre-fill the quantity from the receipt (when readable) as an editable "amount available" column; the user may change it or leave it blank

B) Leave quantities blank by default; recipes state the amount they need and the user decides whether they have enough

C) Ask the user only for the specific ingredients a proposed recipe depends on, at the time the recipe is proposed

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 10
**Dietary preference storage.** `vision.md` says preferences are saved and reused; `technical-environment.md` says no database and session-only `st.session_state`. How long should dietary preferences be stored?

A) Current browser session only; the user is asked again each time the app is opened

B) Saved in a local JSON file on the user's machine (not a database), kept until the user clears them with a "Clear preferences" button

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 11
**Receipt image handling after extraction.** What should happen to uploaded receipt images after ingredients are extracted?

A) Discard from memory immediately after extraction succeeds; never written to disk

B) Keep in memory for the current session only (so the user can re-run extraction), never written to disk, removed when the session ends

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Part 3: Functional and Non-Functional Gaps

## Question 12
**Dietary preference input format during onboarding.** How should users enter allergies and dietary restrictions?

A) Free text only

B) A checklist of common items (for example peanuts, tree nuts, dairy, eggs, gluten, shellfish, vegetarian, vegan, halal, no pork, no beef) plus an optional free-text field

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 13
**Planner priority for the December 16 deadline.** `README.md` says to prioritize Generator, then build Planner. How should Planner be treated in the requirements?

A) Must-have: Generator and Planner both required for the MVP

B) Should-have: Generator is must-have; Planner is required but may be simplified if time runs short

C) Could-have: Planner is a stretch goal

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 14
**Follow-up conversation interface.** How should users make follow-up requests about the current plan (vision.md 2.6)?

A) A free-text chat box under the recipes

B) Predefined action buttons (for example "Replace an ingredient", "Different dish", "Explain a step") plus a free-text chat box

C) Predefined action buttons only

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 15
**Planner input and clarification.** When a user describes what they want to eat in Planner, how should vague requests (for example "something warm and spicy") be handled?

A) The model proposes dishes directly, applying the selected preferences; the user refines through follow-up

B) The model may ask up to one round of clarifying questions before proposing dishes

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 16
**Shopping list units.** How should the combined shopping list express quantities?

A) Recipe units, combined across dishes (for example "green onions: 6 stalks", "chicken thigh: 600 g")

B) Recipe units plus a suggested store-purchase unit (for example "chicken thigh: 600 g - about 1 family pack")

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 17
**Using the Generator list in Planner.** When a confirmed ingredient list from Generator exists, how should Planner use it?

A) Automatically mark matching shopping-list items as "already have"; the user can undo

B) Ask the user whether to use the confirmed list before applying it

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 18
**UI and recipe language.** What language should the UI and generated recipes use?

A) English only

B) English UI, with an option for the user to request recipes in another language through follow-up

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 19
**Response time expectation.** What is an acceptable wait for recipe generation (with a progress spinner)?

A) Up to about 30 seconds

B) Up to about 60 seconds

C) No explicit target for the MVP; best effort

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 20
**API failure handling.** When the Anthropic API call fails (network error, rate limit, timeout), what should the app do?

A) Show a friendly error with a "Try again" button; keep the user's ingredient list and preferences

B) Retry automatically once, then show the friendly error with a "Try again" button; keep the user's data

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 21
**Receipt extraction quality target.** What success criterion should the live receipt test (`@pytest.mark.live`) use?

A) At least 80% of expected food ingredients found across the fixture receipts

B) At least 90% of expected food ingredients found

C) Report the percentage only; no pass/fail threshold for the MVP

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 22
**Milestone stopping point.** `README.md` says "For this milestone, we will complete Inception, approve the units of work, and stop before Construction." Should this workflow stop there?

A) Yes - stop after Units Generation is approved; Construction starts in a later session

B) No - continue into Construction in this workflow

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part 4: AI-DLC Extension Opt-Ins

## Question 23: Security Extensions
Should security extension rules be enforced for this project?

A) Yes - enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No - skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after [Answer]: tag below)

[Answer]: B

*Note: the Security Basics section of `technical-environment.md` (API key handling, EXIF stripping, prompt-injection limits, tool safety) will be captured as requirements regardless of this answer.*

## Question 24: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices** for building resilient systems, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability - covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early - it is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft of your resiliency posture** to build on and validate - not a finished, production-certified result.

A) Yes - apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No - skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 25: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes - enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial - enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No - skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after [Answer]: tag below)

[Answer]: B
