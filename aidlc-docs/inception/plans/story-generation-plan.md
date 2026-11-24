# ReceiptChef - Story Generation Plan

**Stage**: INCEPTION - User Stories (Part 1 approved 2026-11-24T07:25:29Z; Part 2 generation complete)
**Input**: `aidlc-docs/inception/requirements/requirements.md` (approved 2026-11-24, including assumptions A-1 to A-6)
**Outputs**: `aidlc-docs/inception/user-stories/personas.md`, `aidlc-docs/inception/user-stories/stories.md`

Please answer the questions in Section 3 by filling in the letter after each `[Answer]:` tag. When all are answered, tell me and I will review them, then ask you to approve this plan before generating stories.

---

## 1. Story Breakdown Approaches (Reference for Question 2)

| Approach | How stories are organized | Benefits | Trade-offs |
|---|---|---|---|
| **User Journey-Based** | Follow the user's path: open app -> onboarding -> upload -> review -> preferences -> results -> follow-up -> Planner | Easy to read as a demo script; shows handoffs | Shared behavior (pipeline, follow-up) appears in several journeys |
| **Feature-Based** | One group per feature: Onboarding, Generator, Preferences, Results, Plan B, Follow-up, Planner | Maps directly to FR-1 to FR-10; easy traceability | Less sense of end-to-end flow |
| **Persona-Based** | Grouped by persona | Highlights different needs | ReceiptChef has one main user type, so groups would overlap heavily |
| **Domain-Based** | Grouped by domain: ingredients, recipes, preferences, shopping | Aligns with future data models | Abstract for non-technical reviewers |
| **Epic-Based** | Epics with sub-stories | Clear hierarchy, good for splitting work later | Needs a rule for what becomes an epic |

**Recommended hybrid**: Epics by feature (matching FR-1 to FR-10), with a short end-to-end journey section that references story IDs. This keeps traceability and also gives the team a demo script.

---

## 1A. Approved Story Approach (From Answers)

Sources: Section 3 answers and `story-plan-clarification-questions.md` (CQ1: D, CQ2: B).

| Topic | Decision |
|---|---|
| Personas (Q1 B) | Three personas: (1) recent arrival unfamiliar with US product names; (2) student with an allergy or dietary restriction; (3) busy student who plans and shops ahead (Planner-focused) |
| Organization (Q2 A) | Epics by feature (FR-1 to FR-10) plus an end-to-end journeys section |
| Granularity (Q3 B, CQ2 B) | 25-35 stories, each one user goal that can be built and tested on its own |
| Template (Q4 A) | "As a [persona], I want [goal], so that [benefit]" with ID, title, persona, priority, and labels |
| Acceptance criteria (Q5 A) | Given / When / Then scenarios |
| AI outputs (Q6 B) | Criteria cover only checkable app behavior; a separate manual quality checklist in stories.md is used with the fixture receipts |
| Follow-up (Q7 A) | 9 stories: one per action button (7), the chat box, and plan updates after a change |
| Trust and setup (Q8 A, CQ2 B) | Separate epic with 3 stories: (1) setup and API key message; (2) privacy handling of images and receipt data (EXIF, payment / loyalty IDs, no disk writes, logging rules); (3) safety notices and input limits |
| Errors (Q9 C) | Error scenarios inside each related story; API failure handling (FR-10) is its own story |
| Journeys (Q10 ABCD) | Generator happy path; Generator with Plan B; Generator to Planner; Planner from scratch. No returning-user journey (preference stories still exist) |
| Traceability (Q11 A) | Each story lists FR / NFR IDs; coverage table at the end |
| Labels (Q12 D, CQ1 D) | Every story carries **Demo** or **Supporting** (Demo = used in at least one of the four journeys) **and** **Generator phase** or **Planner phase** (build order) |

**Label rules**:
- Onboarding, Trust and setup, API failure, and follow-up stories are **Generator phase**, because they are built first and reused by Planner.
- Planner stories, including the Generator-to-Planner handoff and "already have" marking, are **Planner phase**.

**Target story budget** (about 33): Onboarding 3, Receipt upload 3, Ingredient review 3, Preferences and generation 2, Results 2, Plan B 3, Follow-up 9, Planner 4, API failure 1, Trust and setup 3.

**Requested addition (2026-11-24 change request)**: a dedicated Plan B story for the dietary-restriction persona with a **peanut allergy**.
- **Story**: As a student with a peanut allergy, I want Plan B to keep my allergy in place when my original request cannot be met, so that I can trust any alternative the app suggests.
- **Acceptance criteria will cover**:
  - Plan B dishes, ingredient lists, additional ingredients to buy, and assumed pantry items contain no peanuts or peanut products (for example peanut oil, peanut butter, satay or kung pao sauce with peanuts).
  - When the only way to meet the original request would need peanuts (for example a confirmed ingredient is peanuts, or the typical dish uses them), the conflict explanation says the allergy is kept and the request changed instead; the allergy is never listed as a changed condition.
  - Peanut items on the confirmed ingredient list are not used in Plan B.
  - The constraint check flags any peanut ingredient, and the plan is revised (up to 2 revisions) before it is shown; if it still fails, no peanut-containing plan is displayed.
  - Follow-up changes to Plan B (button or chat) keep the allergy in place, including a chat request such as "just add peanuts anyway" (the app declines and explains).
  - The allergen notice (no allergen-free guarantee) is shown with Plan B.
- **Persona**: Persona 2 (student with an allergy or dietary restriction).
- **Labels**: **Demo** (journey B, "Generator with Plan B", will be written for Persona 2 with a peanut allergy) and **Generator phase**.
- **Traceability**: FR-1.7, FR-1.8, FR-1.9, FR-5.5, FR-5.6, FR-5.7, FR-7.5, FR-8.4, FR-8.7, NFR-4.4.

---

## 2. Execution Checklist (Part 2: Generation)

- [x] **Step 1 - Personas**
  - [x] 1.1 Define personas according to Question 1
  - [x] 1.2 For each persona: background, goals, frustrations, cooking skill, dietary context, typical usage
  - [x] 1.3 Save `aidlc-docs/inception/user-stories/personas.md`
- [x] **Step 2 - Story structure**
  - [x] 2.1 Set up the epic / grouping structure according to Question 2
  - [x] 2.2 Apply the story template according to Question 4 and the ID scheme (for example `US-GEN-03`)
- [x] **Step 3 - Write stories by epic** (granularity per Question 3)
  - [x] 3.1 Onboarding and dietary preferences (FR-1)
  - [x] 3.2 Receipt upload and extraction (FR-2)
  - [x] 3.3 Ingredient review and confirmation (FR-3)
  - [x] 3.4 Recipe preferences and generation (FR-4, FR-5)
  - [x] 3.5 Recipe results (FR-6)
  - [x] 3.6 Plan B (FR-7), including the peanut-allergy Plan B story (see Section 1A)
  - [x] 3.7 Follow-up actions and chat (FR-8; split per Question 7)
  - [x] 3.8 Planner and shopping list (FR-9)
  - [x] 3.9 Error handling (FR-10; placement per Question 6)
  - [x] 3.10 Setup, privacy, and safety notices (NFR-2, NFR-4; per Question 8)
- [x] **Step 4 - Acceptance criteria**
  - [x] 4.1 Write acceptance criteria for every story in the format from Question 5
  - [x] 4.2 Express AI-output criteria as checkable app behavior per Question 6
  - [x] 4.3 Add edge-case and error criteria per Question 9
- [x] **Step 5 - Journeys and mapping**
  - [x] 5.1 Write end-to-end journeys per Question 10 (journey B uses Persona 2 with a peanut allergy)
  - [x] 5.2 Map personas to stories
  - [x] 5.3 Add FR / NFR traceability to each story per Question 11
- [x] **Step 6 - Quality check**
  - [x] 6.1 Check every story against INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable)
  - [x] 6.2 Confirm every FR in `requirements.md` is covered by at least one story
  - [x] 6.3 Save `aidlc-docs/inception/user-stories/stories.md`

### Mandatory Artifacts
- [x] Generate stories.md with user stories following INVEST criteria
- [x] Generate personas.md with user archetypes and characteristics
- [x] Ensure stories are Independent, Negotiable, Valuable, Estimable, Small, Testable
- [x] Include acceptance criteria for each story
- [x] Map personas to relevant user stories

---

## 3. Questions

## Question 1
**Personas.** Which personas should the stories use?

A) One primary persona: an international student cooking with US supermarket groceries

B) Three personas that vary the main user's situation: (1) a recent arrival unfamiliar with US product names, (2) a student with an allergy or dietary restriction (for example, halal or peanut allergy), (3) a busy student who plans meals and shops ahead (Planner-focused)

C) Two personas: (1) the Generator user (has groceries, wants to cook) and (2) the Planner user (knows what to eat, needs a shopping list)

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2
**Breakdown approach.** How should stories be organized? (See Section 1.)

A) Hybrid: epics by feature (FR-1 to FR-10) plus a short end-to-end journey section (recommended)

B) Feature-based only

C) User journey-based only

D) Epic-based by domain (ingredients, recipes, preferences, shopping)

E) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
**Story granularity.** How detailed should stories be?

A) Coarse: about 10-12 stories, roughly one per FR group

B) Medium: about 25-35 stories, each covering one user goal that can be built and tested on its own (for example, "upload up to three receipts", "review unclear items")

C) Fine: about 45-60 stories, close to one per FR line

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4
**Story format.** Which template should each story use?

A) "As a [persona], I want [goal], so that [benefit]" with ID, title, persona, and priority

B) Option A plus a short "Notes" field for UI hints and open points

C) Job story format: "When [situation], I want to [motivation], so I can [outcome]"

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
**Acceptance criteria format.**

A) Given / When / Then scenarios

B) Checklist of testable statements

C) Given / When / Then for main and error scenarios, plus a short checklist for fixed rules (labels, limits, wording)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
**Acceptance criteria for AI outputs.** Model output varies between runs. How should criteria for AI-generated content (extraction, recipes, explanations) be written?

A) Only checkable app behavior: structure, required fields, labels, limits, and pipeline checks (for example, "every dish shows approximate whole-number calories", "total equals the sum of dishes"); quality is judged separately

B) Option A, plus a manual review rubric for quality (for example, "instructions are clear to a beginner", "ingredient names match the receipt"), used with the fixture receipts

C) Write criteria about output quality directly in each story

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7
**Follow-up stories.** How should FR-8 be split?

A) One story per action button (7) plus one story for the chat box and one for "plan updates after a change"

B) Three stories: action buttons that change the plan, action buttons that only explain, and the chat box

C) One story for all follow-up behavior

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
**Non-functional stories.** Should setup, privacy, and safety items (missing API key message, EXIF stripping, no allergy guarantee notice, input length limits) be stories?

A) Yes, as a separate "Trust and setup" epic with its own stories

B) No separate stories; include them as acceptance criteria in the related feature stories

C) Only user-visible items (API key setup message, safety notices) become stories; internal items (EXIF stripping, logging rules) stay as acceptance criteria

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
**Error and edge cases.** Where should error scenarios (unreadable image, API failure, too many images, missing ingredients) go?

A) As acceptance criteria inside the related story

B) As separate error stories

C) Inside the related story, except API failure handling (FR-10), which gets its own story because it applies everywhere

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 10
**End-to-end journeys.** Which journeys should be written out (multiple allowed)?

A) Generator happy path: first launch -> onboarding -> 3 receipts -> review -> 2 dishes -> results

B) Generator with Plan B: conditions conflict -> explanation -> Plan B accepted

C) Generator to Planner: missing ingredients -> shopping list with "already have" marks

D) Planner from scratch: description -> dishes -> follow-up change -> shopping list

E) Returning user: saved preferences -> edit -> clear preferences

F) Other (please describe after [Answer]: tag below)

[Answer]: ABCD

## Question 11
**Traceability.** How should stories link to requirements?

A) Each story lists the FR / NFR IDs it covers, plus a coverage table at the end of stories.md

B) Coverage table only

C) No explicit links

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 12
**Story priority labels.** All FRs are Must for the MVP (Q13). Should stories carry an additional label?

A) No extra label; all stories are Must

B) Mark which stories are needed for the December 16 demo journeys versus supporting stories

C) Mark build order: Generator stories first, then Planner (matches README)

D) Other (please describe after [Answer]: tag below)

[Answer]: BC
