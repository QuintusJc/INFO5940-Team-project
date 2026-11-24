# ReceiptChef - Personas

**Source**: `story-generation-plan.md` Section 1A (Q1: B). These personas are fictional people that stand for the main user's different situations. All three are international students using the same app; no persona is an administrator or developer.

---

## P1 - Mei, the Recent Arrival

| Attribute | Details |
|---|---|
| **Background** | First-semester master's student who moved to the US from China six weeks ago. Shops at Wegmans and an Asian grocery store near campus. |
| **Cooking skill** | Comfortable with home-style Chinese cooking; unsure how to use many US products. |
| **Dietary context** | No allergies or restrictions (chooses "I have none" at onboarding). |
| **Goals** | Use the groceries already bought before they go bad; learn what unfamiliar products are; cook something familiar or try something new without extra shopping. |
| **Frustrations** | Receipt lines like `GRN ONION BCH` or `KS BNLS SKNLS CKN THGH` are hard to decode; online recipes call for ingredients they do not have; recipes take longer than expected. |
| **Typical usage** | Uploads two or three receipts after a weekly shop, corrects the ingredient list, asks for two dishes in 30 minutes, and asks the app to explain steps. |
| **What success looks like** | "I understood every item on my receipt and cooked two dishes with what I already had." |

---

## P2 - Omar, the Student With a Peanut Allergy

| Attribute | Details |
|---|---|
| **Background** | Second-year undergraduate from Jordan. Shops at Aldi and Tops for low prices. |
| **Cooking skill** | Beginner; follows recipes step by step. |
| **Dietary context** | **Severe peanut allergy** and eats **halal** (no pork, no alcohol in cooking). Selects "peanuts" and "halal" in the onboarding checklist. |
| **Goals** | Get quick meals that never include peanuts or pork; trust that alternative suggestions still respect the allergy; avoid reading every label twice. |
| **Frustrations** | Many recipe sites suggest peanut sauces or peanut oil, especially for Asian-style dishes; "helpful" apps sometimes drop restrictions to meet other goals; has little time between classes. |
| **Typical usage** | Uploads one receipt, often chooses a 15-minute limit and three dishes, which frequently cannot be met; relies on Plan B and the chat box. |
| **What success looks like** | "When the app could not do what I asked, it told me why and gave me a safe alternative. My allergy was never the thing that changed." |

---

## P3 - Lucía, the Busy Planner

| Attribute | Details |
|---|---|
| **Background** | PhD student from Mexico with a teaching assistantship. Shops once a week at Walmart or Target. |
| **Cooking skill** | Intermediate; likes Mexican and Italian food. |
| **Dietary context** | Vegetarian. |
| **Goals** | Decide meals before shopping; get one combined shopping list with sensible amounts; avoid buying things already at home. |
| **Frustrations** | Meal-planning apps produce separate lists per recipe with repeated items; amounts like "600 g" are hard to match to store packages; often forgets what is already in the fridge. |
| **Typical usage** | Types a description in Planner ("something warm and filling, not too spicy"), asks for two or three dishes, changes the number of dishes, then uses the shopping list in the store. Sometimes starts in Generator and moves to Planner when ingredients are missing. |
| **What success looks like** | "I got one shopping list with combined amounts, and the app already crossed off what I bought last week." |

---

## Persona-to-Epic Map

| Epic | P1 Mei | P2 Omar | P3 Lucía |
|---|---|---|---|
| ONB - Onboarding and dietary preferences | Primary | Primary | Secondary |
| UPL - Receipt upload and extraction | Primary | Primary | Secondary |
| REV - Ingredient review and confirmation | Primary | Secondary | Secondary |
| GEN - Preferences and recipe generation | Primary | Primary | Primary |
| RES - Recipe results | Primary | Primary | Primary |
| PLB - Plan B | Secondary | Primary | Secondary |
| FUP - Follow-up actions and chat | Primary | Primary | Primary |
| PLN - Planner and shopping list | Secondary | Secondary | Primary |
| ERR - API failure handling | All | All | All |
| TRU - Trust and setup | All | Primary | All |

Story-level persona mapping is in `stories.md` (each story names its persona) and in the persona map at the end of `stories.md`.
