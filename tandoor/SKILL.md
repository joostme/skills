---
name: tandoor
description: Access a local Tandoor Recipes instance using a configured base URL and API key.
---

## What I do

- Access a self-hosted Tandoor instance with static environment variable names so requests are consistent across sessions.
- Work with recipes, meal plans, meal types, shopping lists, foods, keywords, and units through Tandoor's HTTP API.
- Use the endpoint and payload guidance in this skill directly without relying on any external spec file.

## Required environment variables

- `TANDOOR_BASE_URL`: Full base URL for the local Tandoor instance, including any subpath. Examples: `http://localhost:8080`, `https://home.example.com/tandoor`.
- `TANDOOR_API_KEY`: Tandoor API token used for authenticated API access.

## Authentication rules

- Do not proceed with API calls unless both `TANDOOR_BASE_URL` and `TANDOOR_API_KEY` are available.
- Normalize the base URL by removing a trailing slash before appending API paths.
- Send the API key in the `Authorization` header.
- Use `Authorization: Bearer <TANDOOR_API_KEY>` unless the local instance is known to require a different Authorization header format.
- Send `Content-Type: application/json` for JSON writes.

## Base conventions

- Treat `TANDOOR_BASE_URL + /api/...` as the API root.
- List endpoints generally support `page` and `page_size`.
- Detail endpoints generally use `GET`, `PUT`, `PATCH`, and `DELETE` on `/{id}/`.
- Prefer `PATCH` for partial updates.
- Date-only filters usually use `YYYY-MM-DD`.
- Datetime fields usually use ISO 8601.

## Core endpoints

- Recipes
  - `GET /api/recipe/`: list recipes
  - `POST /api/recipe/`: create recipe
  - `GET /api/recipe/{id}/`: fetch recipe
  - `PATCH /api/recipe/{id}/`: update recipe fields
  - `DELETE /api/recipe/{id}/`: delete recipe
- Meal plans
  - `GET /api/meal-plan/`: list meal plans
  - `POST /api/meal-plan/`: create meal plan
  - `GET /api/meal-plan/{id}/`: fetch meal plan
  - `PATCH /api/meal-plan/{id}/`: update meal plan
  - `DELETE /api/meal-plan/{id}/`: delete meal plan
- Meal types
  - `GET /api/meal-type/`: list meal types
  - `POST /api/meal-type/`: create meal type
  - `GET /api/meal-type/{id}/`: fetch meal type
  - `PATCH /api/meal-type/{id}/`: update meal type
  - `DELETE /api/meal-type/{id}/`: delete meal type
- Shopping
  - `GET /api/shopping-list/`: list shopping lists
  - `POST /api/shopping-list/`: create shopping list
  - `GET /api/shopping-list/{id}/`: fetch shopping list
  - `PATCH /api/shopping-list/{id}/`: update shopping list
  - `DELETE /api/shopping-list/{id}/`: delete shopping list
  - `GET /api/shopping-list-entry/`: list shopping entries
  - `POST /api/shopping-list-entry/`: create shopping entry
  - `GET /api/shopping-list-entry/{id}/`: fetch shopping entry
  - `PATCH /api/shopping-list-entry/{id}/`: update shopping entry
  - `DELETE /api/shopping-list-entry/{id}/`: delete shopping entry
- Supporting entities
  - `GET /api/food/`, `POST /api/food/`, `GET /api/food/{id}/`, `PATCH /api/food/{id}/`, `DELETE /api/food/{id}/`
  - `GET /api/keyword/`, `POST /api/keyword/`, `GET /api/keyword/{id}/`, `PATCH /api/keyword/{id}/`, `DELETE /api/keyword/{id}/`
  - `GET /api/unit/`, `POST /api/unit/`, `GET /api/unit/{id}/`, `PATCH /api/unit/{id}/`, `DELETE /api/unit/{id}/`

## Common query parameters

- Recipes: `page`, `page_size`, `keywords`, `keywords_and`, `keywords_or`, `foods`, `foods_and`, `foods_or`, `books`, `internal`, `include_children`, `createdon`, `createdon_gte`, `createdon_lte`, `cookedon_gte`, `cookedon_lte`
- Meal plans: `from_date`, `to_date`, `meal_type`, `page`, `page_size`
- Meal types: `page`, `page_size`
- Foods: `query`, `limit`, `random`, `root`, `root_tree`, `tree`, `updated_at`, `page`, `page_size`
- Keywords: `query`, `limit`, `random`, `root`, `root_tree`, `tree`, `updated_at`, `page`, `page_size`
- Units: `query`, `limit`, `random`, `updated_at`, `page`, `page_size`
- Shopping entries: `mealplan`, `updated_after`, `page`, `page_size`

## Common request bodies

- Create recipe with `POST /api/recipe/`

```json
{
  "name": "Weeknight Pasta",
  "description": "Fast tomato pasta.",
  "servings": 2,
  "working_time": 15,
  "waiting_time": 0,
  "source_url": "https://example.com/pasta",
  "keywords": [{ "id": 12 }],
  "steps": [
    {
      "instruction": "Boil pasta and reserve some water.",
      "ingredients": []
    },
    {
      "instruction": "Cook garlic in oil and add tomatoes.",
      "ingredients": [
        { "food": { "id": 10 }, "unit": { "id": 3 }, "amount": 250 },
        { "food": { "id": 22 }, "unit": { "id": 5 }, "amount": 2 }
      ]
    }
  ],
  "internal": true,
  "show_ingredient_overview": true,
  "private": false
}
```

- Practical recipe payload guidance
  - Always include `name` and `steps`
  - Use `keywords` as nested objects when tagging recipes
  - Each step should include `instruction`
  - Ingredient entries are nested under each step and typically include food, unit, and amount
  - Optional top-level recipe fields include `description`, `servings`, `servings_text`, `working_time`, `waiting_time`, `source_url`, `internal`, `private`, and `show_ingredient_overview`

- Create meal plan with `POST /api/meal-plan/`

```json
{
  "recipe": 123,
  "title": "Pasta night",
  "servings": 2,
  "from_date": "2026-03-12",
  "to_date": "2026-03-12",
  "meal_type": 1,
  "note": "Use pantry tomatoes first",
  "addshopping": true
}
```

- Practical meal plan payload guidance
  - Always include `from_date`, `meal_type`, and `servings`
  - Use either a linked recipe or a free-text `title`
  - `addshopping` can be used to create related shopping items

- Create meal type with `POST /api/meal-type/`

```json
{
  "name": "Dinner",
  "order": 3,
  "default": true,
  "color": "#cc6b2c"
}
```

- Create shopping list with `POST /api/shopping-list/`

```json
{
  "name": "Weekend shopping",
  "description": "Market run",
  "color": "#4c7a5a"
}
```

- Create shopping entry with `POST /api/shopping-list-entry/`

```json
{
  "shopping_lists": [1],
  "food": 10,
  "unit": 3,
  "amount": 2,
  "checked": false
}
```

- Create keyword with `POST /api/keyword/`

```json
{
  "name": "Vegetarian",
  "description": "No meat"
}
```

- Create food with `POST /api/food/`

```json
{
  "name": "Tomato",
  "plural_name": "Tomatoes",
  "description": "Fresh tomato",
  "ignore_shopping": false
}
```

- Create unit with `POST /api/unit/`

```json
{
  "name": "g",
  "description": "grams"
}
```

## Operating rules

- If a user asks for meal planning, first resolve meal type IDs from `GET /api/meal-type/` before creating meal plans.
- If a user asks for recipe creation with tags, ingredient names, or units that may not exist, look up existing keywords, foods, and units first.
- When creating shopping entries, prefer linking to existing shopping lists instead of inventing new list IDs.
- For updates, fetch the current object first when ID-specific fields or nested relationships are unclear.
- Be conservative with deletes and confirm the exact target object when ambiguity exists.

## When to use me

Use this skill when you need to inspect or automate a self-hosted Tandoor Recipes instance through its API and you have the base URL and API key available.
