# INFO5940-Team-project
# ReceiptChef

A receipt-to-recipe app that helps international students find meals to cook with groceries from local supermarkets.

## Team

| Name | NetID | Responsibilities |
|---|---|---|
| ShuYi Wang | sw2536 | AI-DLC workflow and process records |
| Yueming Wu | yrw2 | AI-DLC workflow and process records |
| Zhiqu Ran | zr226 | Technical environment document |
| Hefeng Zhang | hz822 | README and vision document |
| Jiahui Chen | jc3954 | README and vision document |

- **Coding agent:** Claude Code
- **Development methodology:** AI-DLC v1.0.1
- **Product API:** Anthropic Claude API
- **Planned model:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)
- **Deployment:** Local web application

## Problem and Rationale

As international students, we sometimes find it hard to decide what to cook with groceries from local supermarkets. Some ingredients we used at home may be difficult to find, while unfamiliar products can leave us unsure about what to buy or how to prepare them. Online recipes can help, but they may require more ingredients or cooking time than we have. FridgeChef will help students turn groceries they have already purchased into practical meal options. Users can choose meals based on their available cooking time and preferred calorie range. We hope this will make everyday cooking easier, introduce more food choices, and help students use what they have before spending money on additional groceries.

An LLM is useful for this task because the app needs to interpret grocery receipt entries and suggest recipes for different combinations of ingredients. Receipt entries may use abbreviations or product names that are not immediately recognizable as ingredients. The model can help turn these entries into an understandable list, then combine the confirmed ingredients with the user’s preferences to generate cooking instructions. Users will review the list before requesting recipes, since receipt interpretation can contain mistakes and an item they bought may no longer be available.

## Planned LLM Integration

We plan to use the Anthropic Claude API with `claude-sonnet-5-5` for receipt interpretation and recipe generation. Users will upload a photo of a grocery receipt, and the model will identify food items and organize them into an editable ingredient list. Users can correct names, remove items, and add ingredients or pantry staples they already have. Only this confirmed list will be used to generate recipes. Unclear entries will be presented for review, and an unreadable receipt will prompt the user to upload a clearer photo. Our planned workflow uses model calls for receipt interpretation and recipe generation, with user confirmation between the two stages. For the MVP, we plan to use AI calorie estimates rather than connect an external nutrition database.

After reviewing their ingredients, users will choose the number of servings, maximum total preparation and cooking time, preferred calorie range per serving, and cuisine. Our planned cuisine options are Chinese, Italian, Mexican, and No preference. No preference allows the model to choose among the three supported cuisines based on the available ingredients. The app will suggest up to three recipes, each with ingredient quantities, steps, estimated total time, estimated calories per serving, and any additional ingredients needed. Recipes will prioritize confirmed ingredients and clearly identify assumed pantry items. Calorie estimates will be based on the suggested quantities, including oil and sauces, and the number of servings. They will be labeled as approximate and used as a general reference rather than a guarantee that a meal meets an exact calorie limit.

## Scope and Feasibility

By the December 16 final project deadline, we plan to complete a local web app covering receipt upload, ingredient review, preference selection, and recipe generation. We will initially support one clear image of an English-language grocery receipt at a time and test receipts collected from local supermarkets. We will review whether the app identifies ingredients correctly, gives understandable cooking instructions, and responds appropriately to the selected preferences. We also plan to check a small set of calorie estimates against nutrition labels or reference data to identify obvious errors. The MVP will not include food-photo recognition, automatic grocery ordering, supermarket price comparisons, ongoing pantry tracking, long-term meal plans, or precise nutrition analysis. Limiting the input format and supported cuisines will help us deliver a complete, usable workflow within the course timeline.

## Collaboration Plan

We will meet every Tuesday and Thursday, either in person or on Zoom, and dedicate at least one full-team meeting to discussing the problem, target users, and project scope. Between meetings, we will use Google Docs comments and GitHub to share updates and give feedback. Drafts of the README, vision document, and technical environment document will first be shared in Google Docs for everyone to review. The members responsible for each document will revise it based on the feedback and upload the reviewed version to GitHub, where everyone can check the latest changes. ShuYi and Yueming will coordinate one shared AI-DLC workflow, Zhiqu will lead the technical environment document, and Hefeng and Jiahui will lead the README and vision document. All five members will participate in answering the agent’s questions, reviewing its proposals, and approving or requesting changes. Once Inception begins, changes to AI-DLC-generated artifacts will be requested through the shared workflow so the documents and process records stay consistent. We will check and commit `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md` with our work. For this milestone, we will complete Inception, approve the units of work, and stop before Construction.
