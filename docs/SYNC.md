# Synchronization

How does local ↔ cloud synchronization work?

Related: [ARCHITECTURE.md — Local-First Data Architecture](ARCHITECTURE.md#local-first-data-architecture) · [DATABASE.md](DATABASE.md).

Status: not implemented yet. Build the local app first (Expo → SQLite → local application logic), then add Sync Engine → Backend API → PostgreSQL when cloud synchronization is implemented.

## Overview

The application should support full local interaction and synchronize changes when connectivity is available.

```text
iPhone
  │
  ▼
SQLite
  │
  ▼
Sync Engine
  │
  ▼
Backend API
  │
  ▼
PostgreSQL
```

The application should not block the user while synchronization is happening.

Do not make the application depend on the backend for every read/write operation.

The `lib/sync` layer handles the local outbox, connectivity and synchronization with the backend.

## Sync Metadata

Each synchronizable entity should have metadata such as:

```text
id
created_at
updated_at
deleted_at
sync_status
```

Example:

```text
meal_items

id
meal_id
food_id
quantity
...
created_at
updated_at
deleted_at
sync_status
```

Possible sync states include:

```text
pending
syncing
synced
failed
```

## Sync Queue / Outbox

Use an outbox-style queue to track local changes that need to reach the backend.

```text
sync_queue
──────────
id
entity
entity_id
operation
payload
created_at
attempts
```

When the user performs an action offline:

```text
User
 ↓
SQLite
 ↓
sync_status = pending
 ↓
sync_queue
```

When internet connectivity returns:

```text
sync_queue
 ↓
Sync Engine
 ↓
Backend API
 ↓
PostgreSQL
 ↓
success
 ↓
sync_status = synced
```

The sync engine should retry failed operations when appropriate.

## Sync Rules

- Write the domain change and its `sync_queue` entry in the same SQLite transaction, so a local change is never saved without being queued (or queued without being saved).
- Sync operations must be safe to retry: repeated attempts must not create duplicate records on the backend. Use the stable client-generated IDs.
- The frontend never waits for PostgreSQL before confirming a normal local operation.

## Cloud Database

Use PostgreSQL on the backend as the synchronized server-side representation of the user's data.

The cloud database should support:

- account recovery;
- backup;
- synchronization;
- multiple devices;
- future web applications;
- server-side operations where required.

The mobile application should not connect directly to PostgreSQL.

Use a backend/API layer between the mobile application and PostgreSQL. The backend also handles operations requiring secrets or privileged access (see [ARCHITECTURE.md — Security and Privacy](ARCHITECTURE.md#security-and-privacy)).

## Sync Conflicts

The first version can assume a single primary device per user.

Do not overengineer conflict resolution with CRDTs or complex distributed systems unless the product requirements actually require them.

For future multi-device synchronization, use consistent metadata such as:

```text
updated_at
```

and establish a clear conflict-resolution strategy.

The synchronization architecture should remain extensible without adding unnecessary complexity to the first version.

## Sync Errors

- Keep the local change.
- Mark it as failed/pending.
- Retry appropriately.
- Do not lose the user's data simply because the network failed.

A sync failure never undoes the user's local action (see [ARCHITECTURE.md — Failure Philosophy](ARCHITECTURE.md#failure-philosophy-and-error-handling)).

## Validating Sync and Database Changes

For database or synchronization changes, also validate:

- migrations;
- offline behaviour;
- sync retry behaviour;
- duplicate prevention;
- data consistency after reconnecting.
