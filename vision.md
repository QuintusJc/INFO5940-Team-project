# ReceiptChef - Product Vision

## 1. What We Are Building and for Whom

ReceiptChef helps international students figure out what to cook with groceries from local supermarkets. Some ingredients they used at home may be difficult to find, while unfamiliar products can make everyday cooking harder. Online recipes may also require ingredients they do not have or more time than they want to spend.

We want to help students explore more meal options, make better use of groceries they have already bought, and avoid unnecessary purchases. Users can choose meals based on their available cooking time, calorie preferences, and preferred cuisine.

ReceiptChef will offer two main features in the same app:

- **Generator:** Upload grocery receipts to find out what to cook with available ingredients.
- **Planner:** Describe what you want to eat and get a shopping list for the suggested recipes.

The product will use the Anthropic Claude API with `claude-sonnet-5-5` to interpret receipts, generate recipes, estimate calories, and answer follow-up questions. The MVP will run as a local web application.

## 2. MVP Features in Scope

### 2.1 Onboarding and Dietary Preferences

When users first open ReceiptChef, the app will briefly explain its purpose and ask about food allergies or dietary restrictions. Users can provide their preferences, say they have none, or skip the question.

The app will save these preferences and use them in later recipe suggestions and follow-up conversations. Users will be able to review and change them.

These preferences will apply to both Generator and Planner, including alternative suggestions. The app will not automatically ignore an allergy or dietary restriction to meet a time, calorie, or dish-count preference.

### 2.2 Generator: Grocery Receipts to Recipes

Users can upload up to three English-language grocery receipt images in one request. The model will identify food items and turn receipt entries into understandable ingredient names.

The app will show a combined ingredient list before generating recipes. Users can:

- Correct ingredient names.
- Remove non-food items or ingredients they no longer have.
- Add ingredients and pantry staples they already have.
- Review repeated or unclear entries.

Unclear items will be presented for confirmation rather than silently treated as correct. If an image cannot be read, the app will ask for a clearer upload.

Recipes will be based on the confirmed ingredient list. The app will not assume that everything on a receipt is still available or that the purchased quantity is the amount remaining.

### 2.3 Recipe Preferences

Users will choose the following before generating recipes:

| Setting | Options |
|---|---|
| Number of dishes | One, two, or three |
| Maximum total preparation and cooking time | 15, 30, 45, or 60 minutes |
| Estimated calorie range for each dish | Below 400, 400–below 600, or 600–800 kcal, or No preference |
| Cuisine | Chinese, Italian, Mexican, or No preference |

Users will choose how many different dishes they want, not the number of people or servings. Each recipe will still include suggested ingredient quantities so users know how much to use.

The time limit applies to preparing and cooking the complete set of dishes. It does not give each dish a separate allowance of the selected time. The estimate will account for the steps needed to make the full set and state any important assumptions, such as preparing some items while another dish cooks.

The calorie range applies to each dish using all quantities listed in that recipe. It is not a per-person or per-serving target.

No preference for cuisine allows the model to choose among Chinese, Italian, and Mexican styles based on the available ingredients and other preferences.

### 2.4 Recipe Results

When the ingredients and preferences allow it, ReceiptChef will generate the requested number of dishes.

Each dish will include:

- A dish name.
- Ingredients and suggested quantities.
- Clear cooking instructions.
- Estimated calories for the dish using the listed quantities.
- Any additional ingredients needed.

The complete plan will also display:

- **Estimated total preparation and cooking time for all dishes.**
- **Estimated total calories for all dishes combined.**

The total calorie estimate will equal the sum of the individual dish estimates. It will describe the complete quantities in the proposed recipes, not assume that one person will eat everything.

Calories will be estimated by AI based on the recipe quantities, including cooking oil and sauces. The app will label these figures as approximate and avoid decimal-level precision. The selected calorie range will guide recipe suggestions, but the app will not claim that actual calories are guaranteed to fall within that range.

Recipes will prioritize confirmed ingredients. Any unconfirmed pantry items or additional ingredients will be listed clearly rather than assumed to be available.

### 2.5 Alternative Suggestions: Plan B

If the app cannot meet all selected conditions, it will explain which requirements conflict before suggesting a Plan B.

Plan B will show what can reasonably be made with the confirmed ingredients and identify any changes to the original request. For example, it may suggest fewer dishes within the selected time or explain that the requested dishes would take longer.

The app will distinguish between:

- A plan that uses confirmed ingredients.
- A plan that requires additional purchases.

It will not present an alternative as meeting the original conditions when it does not. Allergies and dietary restrictions will continue to apply.

If the confirmed ingredients are insufficient for a reasonable recipe, the app will explain what is missing and offer to help create a shopping list. Users can accept the alternative, change their preferences, or ask a follow-up question.

### 2.6 Follow-Up Questions and Changes

Users can continue the conversation about the current plan. They can ask to:

- Replace an ingredient.
- Suggest a different dish.
- Change the number of dishes.
- Adjust the time or calorie preference.
- Explain a cooking step.
- Suggest other meals using the available ingredients.
- Identify additional ingredients to buy.

Follow-up requests will use the current ingredient list and saved dietary preferences.

When a request changes the recipes, the app will update the ingredient quantities, individual dish calorie estimates, total calories, total cooking time, and additional ingredients as needed.

### 2.7 Planner: Meal Ideas to a Shopping List

Planner lets users start by describing what they want to eat instead of uploading a receipt.

The model will suggest or clarify the dishes and provide recipes with ingredient quantities. Users can apply the supported dish-count, cuisine, time, and calorie preferences to the plan.

Once users are happy with the proposed dishes, the app will produce a combined shopping list. Repeated ingredients will be combined so users can see the total amount needed across the recipes.

Users can identify ingredients they already have and remove them from the shopping list. If a confirmed ingredient list is available from Generator, Planner can use it to help identify what still needs to be purchased.

Planner will show individual dish calorie estimates, the estimated total calories, and the total preparation and cooking time for the plan.

Users can also move from Generator to Planner when they decide to buy ingredients needed for a suggested recipe. Shopping lists will be based on the current recipes and quantities rather than generated independently of the meal plan.

## 3. Features Explicitly Out of Scope

The MVP will not include:

- Food-photo recognition.
- Non-English receipt support.
- More than three receipt images in one request.
- More than three dishes in one plan.
- Inputs for the number of people or servings.
- Nutrition database integration or precise nutrition analysis.
- Personalized daily calorie prescriptions or long-term diet plans.
- Automatic grocery ordering or supermarket price comparisons.
- Exact grocery cost calculations or guaranteed savings.
- Automatic tracking of pantry stock and remaining quantities.
- Medical dietary advice or guarantees that recipes are allergen-free.
- Cloud deployment as a course-project requirement.

## 4. Open Questions

The main product flow is defined above. We will resolve the following details through team discussion and the AI-DLC Inception process:

- Which local supermarket receipt formats should we prioritize for testing?
- How should the app present unfamiliar abbreviations or products that need user confirmation?
- When a recipe depends on the amount of an ingredient available, how should the app ask the user to clarify it?
- How long should dietary preferences be stored, and how can users clear them?
- How should receipt images be handled after the ingredients have been extracted?
