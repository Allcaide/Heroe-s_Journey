# AI — John

How does John behave?

Related: [PRODUCT.md — John](PRODUCT.md#4-john--ai-fitness-assistant) (product scenarios) · [DATABASE.md — AI](DATABASE.md#ai) (conversation storage).

## Role of John

John is the AI layer of the application. John should understand natural language and interact with the user's nutrition and training data.

John should not replace the existing nutrition or workout interfaces. Instead, John should **read from and write to the existing systems** — the same Nutrition and Training systems the user uses manually.

## Core Principle: AI Interprets, the Application Calculates

Do not ask the AI to calculate things that the application can calculate reliably.

For example:

John interprets:

```text
"150g chicken"
```

The application retrieves the nutritional data and calculates:

```text
150g × nutritional values
```

Similarly, John can identify that the user performed:

```text
Bench press
80kg
8 reps
```

but the application stores and calculates:

- volume;
- progression;
- PRs;
- historical statistics.

**AI interprets. The application calculates and stores.**

The LLM should interpret intent, not become the source of truth for deterministic data.

## AI Tools

John should interact with the application through explicit tools/actions rather than directly manipulating UI state.

Potential tools include:

```text
getTodayNutrition()
searchFood()
getFoodDetails()
addMealItem()
updateMealItem()
removeMealItem()

getTodayWorkout()
getRecentWorkouts()
getMuscleRecovery()

createWorkout()
updateWorkout()
addExercise()
addSet()
updateSet()

getUserGoals()
```

The AI should interpret the user's natural language and call the appropriate application actions.

The application remains responsible for:

- calculations;
- validation;
- database writes;
- nutrition calculations;
- workout state;
- business rules.

The `lib/ai` layer encapsulates local/cloud model interaction and John tools.

## AI Tool Safety

AI actions that modify user data should use explicit application tools.

```text
User request
    ↓
John interprets intent
    ↓
Tool call
    ↓
Application validates input
    ↓
SQLite transaction
    ↓
UI updates
```

Avoid allowing the model to directly manipulate arbitrary database records.

Prefer narrowly scoped tools such as:

```text
addMealItem()
updateMealItem()
createWorkout()
addExercise()
addSet()
updateSet()
```

This makes AI actions easier to validate, test and audit.

Data modified by AI follows the [Data Integrity rules](DATABASE.md#data-integrity): validate, use transactions, preserve IDs on correction, avoid duplicates, keep estimated/verified explicit, maintain sync metadata.

Foods John adds without an identified product must be stored as estimated (`estimated`), so the user can verify them later by barcode.

## Workout Adaptation Rules

When John adapts an active workout:

- **Only modify the remaining workout.** Completed sets and exercises should not be unnecessarily changed.
- Consider what has already been completed, what remains, the target muscle groups, the user's request and the affected area.
- For pain/injury-related situations, avoid diagnosing the user and use appropriate caution. The app is a fitness tool, not a medical system.

## Behaviour Guidelines

John should not only respond to explicit commands.

When appropriate, John should proactively identify useful opportunities in the user's current context.

Examples:

- If the user has very little protein left for the day, John can suggest practical ways to reach the target.
- If the user is planning dinner and has limited calories remaining, John can suggest how to structure the meal.
- If the user's workout plan conflicts with recent muscle recovery, John can flag it and suggest alternatives.
- If a food entered by John is only an estimate, the app should make it easy to verify it with a barcode later.
- If a workout has been generated but the user has limited time, John should be able to adapt the remaining session.
- If the user's current state makes a requested plan inefficient, John should explain the issue and suggest a better option rather than blindly following the request.

However, proactive suggestions should not become intrusive.

Do not interrupt the user unnecessarily.

John should distinguish between:

- an action the user explicitly requested;
- a useful suggestion;
- an important warning.

Explicit actions should be executed.

Suggestions should normally be presented as suggestions unless the user asks John to apply them.

John should understand the user's current state and help them make decisions without taking control away from the user.

## Conversation History

Conversation history is for context and user visibility. It must **not** be treated as the source of truth for application data.

If John says "I registered 200g of pasta.", the application must have created a structured `meal_item`. It should not rely on searching old messages later to determine what the user ate. Schema: [DATABASE.md — AI](DATABASE.md#ai).

## On-Device vs Cloud AI

The application should prefer **on-device AI where practical**, particularly for privacy-sensitive user data. The initial target is an iOS-first application.

Potential local capabilities include:

- speech recognition;
- local language model inference;
- structured output;
- tool calling;
- local database access.

Use iOS native modules where required for Apple platform capabilities, and Apple Speech / Foundation Models where appropriate for local AI functionality.

The architecture should allow a cloud AI fallback for tasks that cannot reasonably be handled on-device.

The application should not depend entirely on cloud AI for basic food logging, workout logging, or querying local user data.

AI should also follow the local-first philosophy.

For simple, fast, privacy-sensitive tasks, prefer local processing where practical:

```text
Speech
 ↓
Local model
 ↓
Structured action
 ↓
SQLite
```

For more demanding reasoning tasks, cloud AI may be used when connectivity is available:

```text
User
 ↓
Cloud AI
 ↓
Complex reasoning
 ↓
Response / structured actions
```

The application should be able to decide whether a task should be handled locally or through a cloud model.

Examples of tasks that may be suitable for local processing:

- speech recognition;
- simple food logging;
- simple workout logging;
- querying local data;
- basic structured commands.

Examples of tasks that may benefit from cloud processing:

- complex reasoning;
- long-term analysis;
- more demanding nutritional/training planning;
- tasks that exceed the capabilities of the available on-device model.

The user should not need to understand this distinction.

## Privacy

- Cloud AI calls requiring secrets go through the backend/API layer. Never expose secret API keys in the mobile application.
- Do not send user data to a cloud service unnecessarily.
- When cloud AI is used, only provide the context required for the specific task.

## AI Errors

- Do not silently claim an action was completed if it was not.
- Distinguish between an AI estimate and verified application data.
- Allow the user to retry or perform the action manually.
