# INFO5940-Team-project
## ReceiptChef

An app that helps international students turn grocery receipts into recipes and meal ideas into shopping lists.

## Team

| Name | NetID | Responsibilities |
|---|---|---|
| ShuYi Wang | sw2536 | AI-DLC workflow and process records |
| Zhiqu Ran | zr226 | Technical environment document |
| Jiahui Chen | jc3954 | README and vision document |
| Hefeng Zhang | hz822 | README and vision document |
| Yueming Wu | yrw2 | AI-DLC workflow and process records |


- **Coding agent:** Claude Code
- **Development methodology:** AI-DLC v1.0.1
- **Product API:** Anthropic Claude API
- **Planned model:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)
- **Deployment:** Local web application

## Problem and Rationale for Using LLMs

As international students, we sometimes find it hard to decide what to cook with groceries from local supermarkets. Some ingredients we used at home may be difficult to find, while unfamiliar products can leave us unsure about what to make. Online recipes may also require ingredients we do not have or more time than we want to spend. ReceiptChef will help students explore more meals, use groceries they have already bought, and avoid unnecessary purchases. Users can start with grocery receipts to find out what they can cook, or describe what they want to eat and get a shopping list.

An LLM is useful because the app needs to interpret receipt entries and suggest recipes for different combinations of ingredients and preferences. It can help turn shortened product names into understandable ingredients, explain cooking steps, and suggest changes when users ask follow-up questions. It can also use recipe quantities to prepare a shopping list. Users will review extracted ingredients and suggested plans so they can correct mistakes before continuing.

## Planned LLM Integration

We plan to use the Anthropic Claude API with `claude-sonnet-5-5` for receipt interpretation, recipe generation, calorie estimates, and follow-up questions. During onboarding, users can provide food allergies or dietary restrictions, which the app will save and use in later suggestions. Users can review and change these preferences. In Generator, users can upload one to three English-language grocery receipt images. The app will combine the extracted food items into an editable ingredient list so users can correct names, remove unavailable items, and add ingredients they already have. Recipes will use this confirmed list rather than assume that everything on a receipt is still available. Our initial plan uses model calls coordinated by the app, with user review between receipt interpretation and recipe generation, rather than autonomous tool selection.

Users will choose one to three dishes, a total preparation and cooking time limit of 15, 30, 45, or 60 minutes, a calorie range for each dish, and a cuisine. The number of dishes is independent of the number of receipts uploaded. Calorie options will be below 400, 400–below 600, or 600–800 kcal, or No preference. Cuisine options will be Chinese, Italian, Mexican, and No preference, which lets the model choose among the three supported styles. Each recipe will include quantities, instructions, estimated calories, and any additional ingredients needed. The app will also show the estimated total preparation and cooking time and total calories for all dishes combined. Users will not enter a number of people or servings; calorie estimates will refer to the quantities listed in each recipe. If the conditions cannot all be met, the app will explain why and offer a Plan B with the changes clearly identified, while keeping dietary restrictions in place. Users can ask follow-up questions to adjust the plan. In Planner, users can describe what they want to eat, review the suggested recipes, and get a combined shopping list that accounts for ingredients they confirm they already have. Calories will be estimated by AI, labeled as approximate, and will not rely on an external nutrition database.

## Project Scope and Feasibility

By the December 16 final project deadline, we plan to complete a local web app with both Generator and Planner. The MVP will support up to three English-language receipt images per request and one to three dishes per plan. We will prioritize the receipt-to-recipe flow, then build Planner using the same recipe quantities and ingredient information. Follow-up conversations will focus on adjusting the current recipes, explaining steps, and identifying additional ingredients. We will test receipt interpretation, recipe clarity, dietary preference handling, shopping-list consistency, and responses when selected conditions cannot be met. We will also check representative calorie estimates against nutrition labels or reference data for obvious errors. The MVP will not include food-photo recognition, automatic grocery ordering, price comparisons, automatic pantry tracking, long-term meal plans, or precise nutrition analysis. The duration and method of preference storage will be finalized during Inception and kept consistent with the vision and technical environment documents.

## Collaboration Plan

We will meet every Tuesday and Thursday, either in person or on Zoom, and dedicate at least one full-team meeting to discussing the problem, target users, and project scope. Between meetings, we will use Google Docs comments and GitHub to share updates and give feedback. Drafts of the README, vision document, and technical environment document will first be shared in Google Docs for everyone to review. The members responsible for each document will revise it based on the feedback and upload the reviewed version to GitHub, where everyone can check the latest changes. ShuYi and Yueming will coordinate one shared AI-DLC workflow, Zhiqu will lead the technical environment document, and Hefeng and Jiahui will lead the README and vision document. All five members will participate in answering the agent’s questions, reviewing its proposals, and approving or requesting changes. Once Inception begins, changes to AI-DLC-generated artifacts will be requested through the shared workflow so the documents and process records stay consistent. We will check and commit `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md` with our work. For this milestone, we will complete Inception, approve the units of work, and stop before Construction.
