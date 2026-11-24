# ReceiptChef - Story Plan Clarification Questions

Thank you for answering all 12 questions in `story-generation-plan.md`. Ten answers are clear. I found **one missing description** and **one conflict between answers** that would change how the stories are written.

Please answer by filling in the letter after each `[Answer]:` tag. If you choose Other, write your description on the line after the tag.

**Answers recorded so far**: Q1 B (three personas), Q2 A (feature epics plus journeys), Q3 B (25-35 stories), Q4 A (As a / I want / so that), Q5 A (Given / When / Then), Q6 B (checkable behavior plus a manual quality checklist), Q7 A (one story per follow-up button plus chat and plan-update stories), Q8 A (separate "Trust and setup" epic), Q9 C (errors inside each story; API failure gets its own story), Q10 A, B, C, D (four journeys; returning-user journey not included), Q11 A (FR / NFR IDs on each story plus a coverage table), Q12 D (no description).

---

## Missing Description: Question 12

Question 12 is answered **D) Other**, but no description was written after the `[Answer]:` tag, so I don't know which label you want.

### Clarification Question 1
Which additional label should each story carry?

A) No extra label; all stories are Must (Q12 option A)

B) A "Demo" label on stories used in the four journeys you chose (Q10: A, B, C, D), and "Supporting" on the rest (Q12 option B)

C) A build-order label: "Generator phase" or "Planner phase" (Q12 option C)

D) Both labels: Demo / Supporting **and** Generator phase / Planner phase (combines B and C)

E) Other (please describe after [Answer]: tag below)

[Answer]: D

---

## Conflict: Story Count

You chose **25-35 stories** (Q3: B). Two other answers add stories:

- Q7 A gives follow-up **9 stories**: 7 action buttons, the chat box, and plan updates after a change.
- Q8 A adds a separate "Trust and setup" epic. It would have about **5-6 stories**: API key setup message, EXIF stripping, payment and loyalty ID filtering, input length limits, logging rules, and the allergy / medical notice.

With the other epics (onboarding, upload, review, preferences and pipeline, results, Plan B, Planner, API failure), my estimate is **38-42 stories**. That is above the 25-35 range.

### Clarification Question 2
How should the story count be handled?

A) Allow about 38-42 stories. Keep Q7 A and Q8 A as chosen; each story still covers one user goal that can be built and tested on its own.

B) Stay within 25-35. Keep one story per follow-up button (Q7 A), but merge the Trust and setup items into 2-3 stories: (1) setup and API key message, (2) privacy handling of images and receipt data, (3) safety notices and input limits.

C) Stay within 25-35. Keep the Trust and setup epic (Q8 A), but group the follow-up buttons into fewer stories, like Q7 option B: buttons that change the plan, buttons that only explain, and the chat box.

D) Other (please describe after [Answer]: tag below)

[Answer]: B
