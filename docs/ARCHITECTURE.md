# Architecture

How does the system work?

Related: [DATABASE.md](DATABASE.md) · [SYNC.md](SYNC.md) · [AI.md](AI.md). What is actually installed today is listed in [CLAUDE.md — Current Project State](../CLAUDE.md#current-project-state).

## Tech Stack

Target stack:

- Expo
- React Native
- TypeScript
- Expo Router
- NativeWind / Tailwind CSS
- Zustand
- SQLite for local application data
- iOS native modules where required for Apple platform capabilities
- Apple Speech / Foundation Models where appropriate for local AI functionality
- PostgreSQL on the backend for synchronized cloud data
- Backend/API layer for synchronization and secure server-side operations

Do not introduce new major libraries unless there is a strong reason. Dependency policy: [CLAUDE.md — Dependency Rules](../CLAUDE.md#dependency-rules).

## Local-First Data Architecture

The application should use a **local-first architecture**.

The primary local data store should be SQLite.

The iPhone is the local source of truth for normal app interactions. The cloud is a synchronized replica used for backup, account recovery, multi-device synchronization, and future online functionality.

The app must remain usable when there is no internet connection.

The architecture should therefore follow:

```text
                     APP
                      │
                      ▼
               ┌─────────────┐
               │   SQLite    │
               │  on iPhone  │
               └──────┬──────┘
                      │
              ┌───────┴────────┐
              │                │
           Offline           Online
              │                │
              ▼                ▼
        App continues      Sync Engine
        to work normally       │
                               ▼
                         PostgreSQL
```

The application should not require a network request for normal operations such as:

- adding food;
- editing food quantities;
- recording sets;
- creating or modifying workouts;
- viewing history;
- viewing recovery;
- querying locally available user data.

When internet connectivity is available, local changes should synchronize with the backend. Details: [SYNC.md](SYNC.md).

## Layers

Keep presentation, application logic and persistence separate.

```text
UI                         app/, components/
 ↓
Hooks / Application Logic  hooks/
 ↓
Domain Services            lib/
 ↓
SQLite / APIs / AI
 ↓
Sync queue → Sync Engine → Backend API → PostgreSQL
```

For deterministic business logic:

- calculations should be implemented by the application;
- validation should be implemented by the application;
- database writes should be implemented by the application;
- state transitions should be implemented by the application.

AI should interpret user intent and select appropriate tools/actions.

Do not put deterministic business rules inside prompts when they can be implemented reliably in code.

## Folder Structure

Use the existing project architecture unless there is a strong reason to change it. As features are added, grow towards a structure similar to:

```text
app/
  (tabs)/
    index.tsx
    nutrition.tsx
    training.tsx
    john.tsx
    profile.tsx

components/
  nutrition/
  training/
  john/
  common/

constants/

data/

hooks/

lib/
  database/
  ai/
  health/
  barcode/
  sync/

store/

types/

assets/
```

Create folders when the first file that needs them is created, not in advance.

### `app/`

Use this primarily for routes and screens.

Screens should primarily compose components and call hooks/stores/services, but should not contain:

- large reusable UI blocks;
- complex business logic;
- database implementation details;
- large AI orchestration logic.

### `components/`

Create reusable components only when necessary.

Create a component when:

- it is reused in multiple places;
- it makes a screen substantially easier to read;
- it represents a clear UI concept;
- it encapsulates meaningful behaviour.

Do not create tiny one-off components too early.

When unsure, ask:

> Should this UI be extracted into a reusable component, or should it stay inside the current screen for now?

Prefer a simple local implementation until reuse or complexity justifies extraction.

### `hooks/`

Use hooks for reusable React behaviour and UI/application integration.

Do not move arbitrary logic into hooks simply to make files smaller.

### `lib/`

Use `lib/` for infrastructure and domain services such as:

- database;
- AI;
- barcode;
- HealthKit;
- synchronization;
- external APIs.

Keep infrastructure concerns out of presentation components.

- The `lib/database` layer should encapsulate SQLite access.
- The `lib/sync` layer should handle the local outbox, connectivity and synchronization with the backend.
- The `lib/ai` layer should encapsulate local/cloud model interaction and John tools.

### `store/`

Use Zustand for appropriate global/session/UI state.

Do not use the store as a replacement for the database.

## State Management

Use Zustand for global client state where appropriate.

Use local React state for temporary UI state.

Persistent application data such as:

- nutrition;
- meals;
- foods;
- products;
- workouts;
- sets;
- recovery;
- history;

should belong to the SQLite data layer rather than being treated as Zustand state.

Zustand should represent application/session/UI state, not become the primary database.

Examples of appropriate Zustand state:

```text
currentWorkout
isJohnOpen
selectedMeal
activeWorkout
UI state
user preferences
```

SQLite remains the source of truth for persistent domain data.

## Data Flow Rules

Persistent domain data flows from SQLite to the UI.

Do not maintain a second authoritative copy of persistent data in Zustand. Zustand may reference persistent entities by ID but should not become their source of truth.

After a successful local database mutation:

1. SQLite reflects the new state immediately.
2. The UI reflects the local state.
3. A sync operation is queued when relevant.
4. Network synchronization happens asynchronously.

The frontend must never wait for PostgreSQL synchronization before confirming a normal local operation.

## Failure Philosophy and Error Handling

Errors should be handled deliberately.

Local success and cloud success are different states:

- A successful SQLite write counts as a successful local user action.
- A later synchronization failure must not undo the user's local action.
- Never show a successful AI/database action unless the underlying application operation actually succeeded.
- Never silently swallow errors.
- Do not use broad try/catch blocks that hide programming errors.

For user-facing errors:

- explain what happened;
- provide a useful next action where possible;
- do not expose raw stack traces or implementation details.

Sync errors: [SYNC.md — Sync Errors](SYNC.md#sync-errors). AI errors: [AI.md — AI Errors](AI.md#ai-errors).

## Performance

The app should feel immediate for normal interactions.

Do not make a user wait for:

- a cloud request to record a food;
- a cloud request to record a workout set;
- a cloud request to open local history;
- a cloud request to display recovery based on locally available data.

Use SQLite/local computation first wherever practical.

Cloud operations should happen asynchronously when they are not required for the immediate interaction.

## Security and Privacy

Treat nutrition, training, health and AI conversation data as private user data.

Never expose secret API keys in the mobile application.

Use the backend/API layer for:

- cloud AI calls requiring secrets;
- privileged database operations;
- authentication-related server operations;
- other sensitive credentials.

Local AI and local database operations may run directly on-device where appropriate.

Prefer local processing for sensitive operations where practical.

Do not send user data to a cloud service unnecessarily.

When cloud AI is used, only provide the context required for the specific task.

## Domain Types

Important domain types should include concepts such as:

```text
Food
FoodProduct
Nutrient
Meal
MealItem

Exercise
MuscleGroup
Workout
WorkoutExercise
WorkoutSet

RecoveryState

AIAction
AIMessage

SyncOperation
SyncStatus
```

## Simplicity

Avoid overengineering.

The architecture should support the product's actual requirements without prematurely implementing:

- complex distributed systems;
- unnecessary microservices;
- CRDT-based synchronization;
- excessive abstraction;
- large state-management frameworks.

The local-first architecture is intentional, but it should remain simple.

Start with:

```text
Expo
 ↓
SQLite
 ↓
Local application logic
```

Then add:

```text
Sync Engine
 ↓
Backend API
 ↓
PostgreSQL
```

when cloud synchronization is implemented.
