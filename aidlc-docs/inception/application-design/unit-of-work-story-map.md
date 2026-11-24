# ReceiptChef - Unit of Work Story Map

Each of the 33 stories is assigned to exactly **one primary unit** (where its acceptance criteria are implemented and tested). **Supporting units** provide parts the story needs or extend it.

## 1. Story Assignments

| Story | Title | Primary unit | Supporting units | Labels |
|---|---|---|---|---|
| US-ONB-01 | First-launch onboarding | U1 | - | Demo, Generator phase |
| US-ONB-02 | Saved preferences applied everywhere | U1 | U3 (dietary check in pipeline), U4, U5 | Demo, Generator phase |
| US-ONB-03 | Review, edit, and clear preferences | U1 | U3 (regenerate offer) | Supporting, Generator phase |
| US-TRU-01 | Setup and API key message | U1 | - | Supporting, Generator phase |
| US-ERR-01 | Recover from API failures | U1 | U2, U3, U4, U5 (show "Try again") | Supporting, Generator phase |
| US-UPL-01 | Upload one to three receipts | U2 | - | Demo, Generator phase |
| US-UPL-02 | Extract food items into one list | U2 | U1 (LLM client) | Demo, Generator phase |
| US-UPL-03 | Clearer photo when an image can't be read | U2 | - | Supporting, Generator phase |
| US-REV-01 | Understand unclear and unfamiliar items | U2 | - | Demo, Generator phase |
| US-REV-02 | Edit the ingredient list | U2 | - | Demo, Generator phase |
| US-REV-03 | Confirm the list before recipes | U2 | U3 (uses confirmed list) | Demo, Generator phase |
| US-TRU-02 | Privacy of images and receipt data | U2 | U1 (logging, preferences location) | Supporting, Generator phase |
| US-GEN-01 | Choose recipe preferences | U3 | - | Demo, Generator phase |
| US-GEN-02 | Generate a plan that is checked and revised | U3 | U1 (LLM client) | Demo, Generator phase |
| US-RES-01 | View each dish | U3 | - | Demo, Generator phase |
| US-RES-02 | View plan totals | U3 | - | Demo, Generator phase |
| US-PLB-01 | Conflict explanation and labeled Plan B | U3 | - | Demo, Generator phase |
| US-PLB-02 | Not enough ingredients | U3 | U5 (shopping list handoff) | Demo, Generator phase |
| US-PLB-03 | Plan B keeps my peanut allergy | U3 | U4 (Scenario 5: follow-up and chat) | Demo, Generator phase |
| US-TRU-03 | Safety notices and input limits | U3 | U1 (notices widget, model limits), U2, U4 | Demo, Generator phase |
| US-FUP-01 | Replace an ingredient | U4 | U3 | Demo, Generator phase |
| US-FUP-02 | Suggest a different dish | U4 | U3 | Supporting, Generator phase |
| US-FUP-03 | Change the number of dishes | U4 | U3 | Demo, Generator phase |
| US-FUP-04 | Adjust time or calorie preference | U4 | U3 | Supporting, Generator phase |
| US-FUP-05 | Explain a cooking step | U4 | - | Demo, Generator phase |
| US-FUP-06 | Suggest other meals with these ingredients | U4 | U3 | Supporting, Generator phase |
| US-FUP-07 | Identify additional ingredients to buy | U4 | U5 (shopping list handoff) | Demo, Generator phase |
| US-FUP-08 | Ask a question in the chat box | U4 | - | Demo, Generator phase |
| US-FUP-09 | Plan updates after a change | U4 | U3, U5 (Scenario 2: shopping list follows) | Demo, Generator phase |
| US-PLN-01 | Describe what I want and get dishes | U5 | U3 | Demo, Planner phase |
| US-PLN-02 | Combined shopping list | U5 | - | Demo, Planner phase |
| US-PLN-03 | Mark what I already have | U5 | U2 (confirmed list) | Demo, Planner phase |
| US-PLN-04 | Move from Generator to Planner | U5 | U3, U4 | Demo, Planner phase |

**Cross-unit acceptance notes**:
- US-PLB-03 Scenario 5 (follow-up and chat keep the allergy) is verified at the **U4 checkpoint**; Scenarios 1-4 and 6 at the U3 checkpoint.
- US-FUP-09 Scenario 2 (shopping list follows changes) is verified at the **U5 checkpoint**.
- US-ERR-01 is implemented in U1 (client retry, result status, SessionStore) and re-checked at each later checkpoint for that unit's model calls.

## 2. Totals per Unit

| Unit | Stories | Demo | Supporting |
|---|---|---|---|
| U1 Foundation | 5 | 2 | 3 |
| U2 Receipt Intake | 7 | 5 | 2 |
| U3 Recipe Engine | 8 | 8 | 0 |
| U4 Follow-Up | 9 | 6 | 3 |
| U5 Planner | 4 | 4 | 0 |
| **Total** | **33** | **25** | **8** |

All 33 stories are assigned. Story phase labels are consistent: U1-U4 stories are Generator phase; U5 stories are Planner phase.

## 3. Journeys to Units

| Journey | Steps | Units needed |
|---|---|---|
| A - Generator happy path (Mei) | 1 onboarding; 2-5 upload, review, confirm; 6-7 preferences and results; 8-9 follow-up | U1; U2; U3; U4 |
| B - Plan B with a peanut allergy (Omar) | 1 onboarding; 2 upload and confirm; 3-5 generate and Plan B; 6 chat refusal; 7 notice | U1; U2; U3; U4; U3 |
| C - Generator to Planner (Mei) | 1 insufficient ingredients; 2 what else to buy; 3-4 handoff and shopping list | U3; U4; U5 |
| D - Planner from scratch (Lucía) | 1-2 propose dishes; 3 change dish count; 4-5 shopping list and marks | U5 (+U3); U4; U5 |

## 4. Component Coverage Check

Every component in `components.md` belongs to exactly one owner unit (shared files list contributing units in `unit-of-work.md` Section 3):

| Unit | Components owned |
|---|---|
| U1 | C-01, C-02, C-05, C-07, C-13, C-14, C-21, C-22, C-24, C-25, C-26, T-01, T-02, `notices.py` (C-06) |
| U2 | C-03, C-08, C-15, C-23, `ingredient_table.py` (C-06) |
| U3 | C-09, C-12, C-16, C-17, C-18, C-19, `preference_form.py` and `plan_view.py` (C-06) |
| U4 | C-10, `followup_panel.py` (C-06) |
| U5 | C-04, C-11, C-20, `shopping_list_view.py` (C-06) |

C-06 (widgets) is split by file across units. All 26 components and 2 test-support components are covered.
