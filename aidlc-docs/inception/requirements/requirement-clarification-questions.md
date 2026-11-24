# ReceiptChef - Requirements Clarification Questions

Thank you for answering all 25 questions. Most answers are clear and consistent. I found **one contradiction** and **three ambiguities** that would change the design, so I need to resolve them before writing `requirements.md`.

Please answer by filling in the letter choice after each `[Answer]:` tag. If none of the options match, choose the last option (Other) and describe your preference.

---

## Contradiction 1: Buttons-only follow-up vs. open-ended follow-up questions

You chose **predefined action buttons only** for follow-up requests (Q14: C). However:

- `vision.md` 2.5 says users can "accept the alternative, change their preferences, or **ask a follow-up question**".
- `README.md` says the model will "**answer follow-up questions**" and that "Users can **ask follow-up questions** to adjust the plan".
- You also chose (Q15: A) that in Planner, "the user **refines through follow-up**", so all Planner refinement would also go through buttons.

Several of the actions in `vision.md` 2.6 also need user input to work. For example, "Replace an ingredient" needs to know which ingredient and with what, "Suggest a different dish" needs to know which dish, and "Explain a cooking step" needs to know which step.

### Clarification Question 1
How should follow-up actions collect the details they need?

A) Buttons only, each opening a small structured form: dropdowns to choose the dish, ingredient, or step, plus a short optional text field (60 characters at most) for things like a replacement ingredient. There is no open chat box. "Ask a follow-up question" in vision.md / README.md means these structured actions.

B) Buttons only, with dropdowns and no typing at all. For example, the model proposes replacement options and the user picks one.

C) Revisit Q14: predefined action buttons **plus** a free-text chat box (Q14 option B), so users can also ask open-ended questions

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Ambiguity 1: Reference recipes in the app-coordinated pipeline

You chose an **app-coordinated pipeline** (Q5: B) instead of a tool-using agent. Option B described the app always running the calorie and constraint checks. It did not say what happens to `search_reference_recipes` and `data/reference_recipes.json` (about 30 dishes per cuisine), which `technical-environment.md` uses to keep suggestions close to real Chinese, Italian, and Mexican cooking.

### Clarification Question 2
Should the app-coordinated pipeline still use the bundled reference recipes?

A) Yes. Before each recipe-generation call, the app looks up matching reference dishes for the chosen cuisine and confirmed ingredients and includes them in the prompt as context.

B) No. Drop `data/reference_recipes.json` from the MVP; the model generates recipes from its own knowledge.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 2: Revision limit when checks fail

With the app-coordinated pipeline (Q5: B), the app asks the model to revise when the calorie or constraint checks fail. `technical-environment.md` capped the agent at **10 tool calls**, but that cap does not apply to a fixed pipeline. A new limit is needed to control cost and response time.

### Clarification Question 3
How many times may the app ask the model to revise a plan that fails the checks before it stops and explains the conflict and offers Plan B (vision.md 2.5)?

A) Up to 2 revisions (at most 3 generation calls per request)

B) Up to 3 revisions (at most 4 generation calls per request)

C) 1 revision (at most 2 generation calls per request)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 3: Receipt quantities when quantities are blank by default

You chose to **leave quantities blank by default**, with recipes stating the amount they need (Q9: B). `technical-environment.md` Stage 1 extracts `quantity` and `unit` from the receipt when they are readable. It is unclear whether that information should still be extracted and shown.

### Clarification Question 4
What should the app do with quantities printed on the receipt?

A) Extract them and show them as read-only "on receipt" information next to each ingredient, but never treat them as the amount available

B) Do not extract or show receipt quantities at all; the ingredient list contains names only

C) Other (please describe after [Answer]: tag below)

[Answer]: A
