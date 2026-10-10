# Product

What are we building?

Related: [AI.md](AI.md) (how John works internally) · [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) (how it looks) · [DATABASE.md](DATABASE.md) (how it is stored).

## Overview

We are building an AI-powered personal fitness mobile app using Expo and React Native. The initial target is an iOS-first application.

The app combines three core experiences:

1. **Nutrition tracking**, inspired by MyFitnessPal.
2. **Workout tracking**, inspired by Strong.
3. **John**, a personal AI fitness assistant that works across the nutrition and training systems.

The goal is not to replace traditional tracking interfaces with AI.

The traditional interfaces should remain fast, familiar, predictable, and complete.

AI should act as an intelligent layer on top of the app, helping the user:

- register food and workouts using natural language or voice;
- create workouts before training;
- adapt workouts during training;
- answer questions about nutrition and training;
- help the user reach daily calorie and macro targets;
- use training history and estimated muscle recovery to make better decisions;
- reduce the amount of manual data entry required.

The app should feel like:

**MyFitnessPal + Strong + a personal AI fitness assistant.**

This is intended to become a real production-quality application while remaining understandable during development.

## Main Screens

The application should initially be structured around a small number of core screens.

```text
Home
Nutrition
Training
John
Profile / Goals
```

Recovery does not necessarily need to be a separate top-level screen.

Recovery information should be surfaced where it is useful, particularly in:

- Home;
- Training;
- John.

## Home Screen

The main screen should provide a concise overview of the user's current day.

It should include information such as:

- calories consumed / target;
- protein consumed / target;
- other relevant macros;
- meals completed;
- today's workout;
- workout progress;
- muscle recovery;
- relevant daily activity;
- access to John.

John should be accessible directly from the Home screen.

---

## 1. Nutrition

The nutrition experience should closely follow the interaction model of MyFitnessPal.

Users should be able to:

- browse their daily food diary;
- add breakfast, lunch, dinner and snacks;
- search for foods;
- manually add foods;
- define quantities;
- view calories;
- view macronutrients;
- view micronutrients;
- create recipes/meals;
- review previous meals;
- scan product barcodes;
- adjust quantities quickly.

The nutrition tracker is a core product feature and should work independently of John.

Users must always be able to manually register and edit their food.

### Food Verification

Each food item displayed in a meal should provide quick access to:

- **barcode verification**
- **quantity/weight adjustment**

The food row should visually communicate whether the food has been verified against a real product.

#### Verified food

Use a subtle green circular indicator containing the barcode icon.

This means the food is associated with a verified product/barcode and its nutritional information comes from the identified product.

#### John-estimated food

Use the barcode icon with a subtle visual indicator representing John.

This means John added or estimated the food, but the food has not yet been associated with a specific product.

The user can later scan the barcode and replace the generic/estimated food with the real product.

Example:

```text
John:

Pasta       ~100g
Tuna        ~150g
Cocktail sauce ~30g
```

The user can then scan the tuna barcode and convert it into the exact product while keeping the quantity.

The goal is:

**AI provides speed. Barcode/product data provides accuracy.**

The data-level flow is in [DATABASE.md — Barcode Verification Flow](DATABASE.md#barcode-verification-flow). Visual and accessibility requirements are in [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md#domain-specific-ui).

---

## 2. Training

The training experience should closely follow the interaction model of Strong.

Users should be able to:

- create workouts;
- select exercises;
- organise exercises;
- define sets;
- define repetitions;
- define weight;
- record completed sets;
- track workout history;
- track progression;
- track volume;
- track PRs;
- perform strength training;
- perform calisthenics;
- create custom exercises.

The normal workout experience should not depend on AI.

A user should be able to open a workout and simply:

```text
Start Workout
→ perform exercise
→ record set
→ continue
```

The training interface should remain fast and practical during an actual gym session.

---

## 3. Recovery

The app should maintain an estimated recovery state for different muscle groups.

Examples:

```text
Chest       85%
Back        92%
Shoulders   61%
Quads       34%
Hamstrings  28%
Biceps      90%
Triceps     72%
```

Recovery is an estimated value, not a medical or physiological measurement.

It should be derived from relevant training history and other available context.

Recovery information is used primarily to help John make better training recommendations.

For example:

> "What should I train today?"

John can consider:

- previous workouts;
- recently trained muscle groups;
- training volume;
- exercise history;
- recovery estimates;
- user goals;
- available time;
- available equipment.

How recovery is stored and calculated: [DATABASE.md — Recovery](DATABASE.md#recovery).

---

## 4. John — AI Fitness Assistant

John is the AI layer of the application.

John should be able to understand natural language and interact with the user's nutrition and training data.

John should not replace the existing nutrition or workout interfaces.

Instead, John should be able to **read from and write to the existing systems**.

### Nutrition

> "John, I ate 200g of pasta, 150g of chicken and some olive oil."

John should:

1. understand the food;
2. estimate missing quantities where necessary;
3. determine the appropriate meal;
4. create the meal items;
5. mark uncertain/estimated foods appropriately.

### Nutrition planning

> "I still need 60g of protein and have 500 calories left."

John should analyse the current day and suggest realistic options.

### Daily planning

> "I'm having fish pasta at my mother-in-law's tonight. How can I finish the day without exceeding my calories?"

John should consider the user's current nutritional state and recommend how to distribute the remaining calories and macros.

### John and Training

John can create workouts when explicitly requested.

Example:

> "John, create today's workout. I have 50 minutes and want to focus on chest."

John should create the workout in the normal Training system.

When the user arrives at the gym, they should simply open the generated workout and press **Start Workout**.

The workout should then behave like a normal Strong-style workout.

### John During a Workout

John should also be available during an active workout.

The user can ask for changes without restarting the workout.

Example:

> "John, my shoulder hurts but I still want to train chest."

John should consider:

- what has already been completed;
- what remains in the workout;
- the target muscle groups;
- the user's request;
- the affected area.

John can then modify the remaining exercises.

For example, it could:

- remove exercises that may aggravate the affected area;
- replace complex movements with simpler alternatives;
- reduce load;
- reduce volume;
- choose more controlled movements;
- preserve the intended training focus where reasonably possible.

The important behaviour is:

**Only modify the remaining workout.**

Completed sets and exercises should not be unnecessarily changed.

For pain/injury-related situations, John should avoid diagnosing the user and should use appropriate caution. The app is a fitness tool, not a medical system.

---

## Apple Health / HealthKit

Where appropriate, the application may integrate with Apple Health / HealthKit.

Potential data includes:

- steps;
- workouts;
- sleep;
- weight;
- active energy;
- other relevant fitness data.

Do not duplicate large amounts of HealthKit data unnecessarily.

Only persist information that is actually required by the application.

Health data can provide additional context to:

- Home;
- Recovery;
- John;
- training recommendations;
- nutrition recommendations.

---

## Important Constraints

The application is **not** a language-learning app.

Do not introduce concepts from the previous teaching project such as:

- lessons;
- XP;
- language selection;
- vocabulary;
- Clerk;
- Stream;
- video teachers;
- lesson completion.

The core product is:

```text
Nutrition
    +
Training
    +
Recovery
    +
John AI
```

The traditional tracking systems should remain useful without AI.

AI should make the application **faster, more contextual and more adaptive**, rather than becoming a mandatory interface for basic functionality.

The app must support offline usage for core functionality.

SQLite is the local source of truth.

PostgreSQL is the synchronized cloud representation.

Synchronization must happen in the background and must not prevent the user from using the app.

---

## Final Product Principle

The application should feel like:

> **A complete nutrition and workout tracker that happens to have an intelligent personal coach built into it.**

The user should be able to manually track everything exactly as they would in a traditional fitness application.

But whenever they want help, they can simply talk to John:

> "What should I eat?"

> "Log this."

> "What should I train today?"

> "Make me a workout."

> "I only have 30 minutes."

> "My shoulder is bothering me. Adapt the workout."

> "I still need 60g of protein."

John should understand the user's current state, interact with the application's data, and help them make decisions without taking control away from the user.

The application should remain useful with no internet connection, while seamlessly synchronizing data and using cloud capabilities whenever they provide additional value.
