# Database

How is data represented?

Related: [SYNC.md](SYNC.md) (sync metadata and outbox) · [ARCHITECTURE.md](ARCHITECTURE.md#data-flow-rules) (data flow).

## Local Database

Use SQLite as the primary persistent database on the iPhone. SQLite is the source of truth for persistent application data.

SQLite should contain the structured application data required for the app to operate offline.

The database should represent the application's domain rather than storing the entire application state as arbitrary JSON.

Core domains should include:

```text
Identity
Nutrition
Training
Recovery
AI
```

## Database Engineering Rules

UI components must not execute raw SQL directly. Use:

```text
UI
 ↓
hooks / application logic
 ↓
database service/repository (lib/database)
 ↓
SQLite
```

- Database writes involving multiple related records must use transactions.
- Schema changes must be implemented through migrations.
- Never modify an existing migration that may already have been applied. Create a new migration instead.
- Foreign keys and relationships should be explicit.
- Persistent entities must use stable IDs suitable for synchronization. Even if the first version only has one user, use stable IDs and a schema that can support accounts and synchronization later.
- Synchronizable records follow the sync metadata conventions in [SYNC.md](SYNC.md#sync-metadata). The tables below list domain fields; synchronizable tables also carry that metadata.
- Tables and columns use snake_case; table names are plural.

## Data Integrity

Whenever an action (including an AI action) modifies persistent data:

- validate the requested data;
- use database transactions where multiple related records must change together;
- preserve existing IDs where an item is being corrected rather than recreated;
- avoid duplicate records;
- keep estimated and verified states explicit;
- maintain synchronization metadata.

For example, barcode verification should update the existing meal item rather than creating a second copy of the food.

---

## Identity

```text
users
─────
id
email
created_at
updated_at
```

```text
user_goals
──────────
id
user_id
calorie_target
protein_target
carb_target
fat_target
weight_target
...
```

---

## Nutrition

### Foods

A `food` represents a generic or canonical food.

```text
foods
─────
id
name
brand
source
created_at
updated_at
```

Example:

```text
food_id = 123
name = "Chicken breast"
brand = null
```

### Products

A `product` represents a concrete branded product that can be identified by barcode.

```text
products
────────
id
food_id
brand
name
barcode
serving_size
serving_unit
created_at
updated_at
```

Example:

```text
brand = Continente
name = Atum ao natural
barcode = 560...
```

A generic food and a concrete product should remain conceptually separate.

### Nutrients

Do not create a huge fixed table with one column for every possible nutrient.

Use a nutrient entity:

```text
nutrients
─────────
id
name
unit
```

and a relationship:

```text
food_nutrients
──────────────
food_id
nutrient_id
amount
basis
```

This allows the system to support both macronutrients and micronutrients without continually changing the schema.

### Meals

```text
meals
─────
id
user_id
date
meal_type
created_at
updated_at
```

A meal contains multiple food items:

```text
meal_items
──────────
id
meal_id
food_id
product_id
quantity
unit
estimated
verified
created_at
updated_at
```

Example:

```text
Lunch
│
├── Pasta       200g
├── Chicken     150g
└── Olive oil     3g
```

A meal item may initially reference a generic food and later be associated with a concrete product after barcode scanning.

The meal itself should not need to be recreated.

### Barcode Verification Flow

The barcode is part of the food-data accuracy workflow.

The intended flow is:

```text
Scan barcode
      ↓
Identify product
      ↓
Retrieve nutritional information
      ↓
Associate product with food item
      ↓
Keep the existing quantity
      ↓
Update meal item
      ↓
Mark item as verified
```

Example:

```text
John:
"100g pasta, 150g tuna and cocktail sauce."
```

John creates estimated items:

```text
Pasta             ~100g
Tuna              ~150g
Cocktail sauce     ~30g
```

The user can later scan the tuna barcode.

The application replaces the generic/estimated food reference with the identified product while preserving the consumed quantity.

The item then becomes verified.

---

## Training

### Exercises

```text
exercises
─────────
id
name
category
equipment
```

### Muscle Groups

```text
muscle_groups
─────────────
id
name
```

### Exercise / Muscle Relationships

```text
exercise_muscles
────────────────
exercise_id
muscle_id
role
```

For example:

```text
Bench Press
│
├── Chest      primary
├── Triceps    secondary
└── Shoulders  secondary
```

This relationship is important for recovery calculations and for allowing John to understand which muscles are affected by a workout.

### Workouts

```text
workouts
────────
id
user_id
name
date
duration
status
created_at
updated_at
```

```text
workout_exercises
─────────────────
id
workout_id
exercise_id
position
```

```text
sets
────
id
workout_exercise_id
set_number
reps
weight
rpe
rir
rest
completed
```

This should support a Strong-style workout experience.

---

## Recovery

Do not store only a value such as:

```text
Chest = 82%
```

Instead, store the training events and stimuli that allow the recovery engine to calculate the current state.

```text
recovery_events
───────────────
id
user_id
muscle_group_id
workout_id
stimulus
volume
intensity
created_at
```

The recovery engine can then calculate an estimated current state based on:

- time since the last stimulus;
- training volume;
- intensity;
- recent workouts;
- affected muscle groups;
- other relevant available context.

Example:

```text
Chest
last stimulus: 18h ago
volume: high
intensity: high

→ estimated recovery: 63%
```

This makes the recovery model replaceable and extensible without losing the underlying training data.

---

## AI

John's conversation history can be stored separately:

```text
conversations
─────────────
id
user_id
created_at
updated_at
```

```text
messages
────────
id
conversation_id
role
content
created_at
```

Conversation history is for context and user visibility.

It must **not** be treated as the source of truth for application data.

If John says:

> "I registered 200g of pasta."

the application should create a structured `meal_item`.

It should not rely on searching old messages later to determine what the user ate.

---

## Sync

The `sync_queue` table and the sync metadata columns are defined in [SYNC.md](SYNC.md).
