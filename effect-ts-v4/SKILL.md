---
name: effect-ts
description: Expert guidance for building with Effect-TS v4. Use this skill whenever the user is working with Effect, writing TypeScript with functional effects, or using any @effect/* packages. Trigger on imports from 'effect', mentions of Effect.gen, Layer, Schema, Stream, fiber concurrency, typed errors, or dependency injection with layers — even if the user doesn't say "Effect" explicitly. Also use this when the user asks about error handling, dependency injection, or async workflows in TypeScript and their project uses Effect.
---

# Effect-TS v4

## Prerequisites: Local Effect v4 Repository

Before starting any Effect v4 work, verify the local repository exists:

```bash
ls ~/.effect-smol/packages/effect/src
```

If missing, clone it:

```bash
git clone https://github.com/Effect-TS/effect-smol ~/.effect-smol
```

This is the Effect v4 development repository (`effect-smol`). Since v4 is actively evolving and documentation may lag, the source is the authoritative reference — exact function signatures, required parameters, type constraints, and available overloads are all here. Migration guides also live in this repo: `MIGRATION.md` (v3→v4) and `packages/effect/SCHEMA.md` (Schema changes).

## Overview

This skill guides you in writing correct, idiomatic Effect-TS v4 code. Effect v4 consolidates the ecosystem into a single `effect` package with unified versioning. The core programming model — Effect, Layer, Schema, Stream — is unchanged from v3, but the package structure has been reorganized and expanded.

## v4 Package Structure

Effect v4 ships as a single versioned package. All ecosystem packages share the same version number, eliminating version mismatch issues:

```bash
npm install effect@4
```

**Stable core** — safe to use in production:
```typescript
import { Effect, Layer, Schema, Stream, Fiber, Queue, Ref, Scope } from "effect"
import { Context, Data, Config, Match, Option, Either } from "effect"
import { Array, Record, Cause, Exit, Runtime } from "effect"
```

**Unstable modules** (`effect/unstable/*`) — functional but may have breaking changes in minor releases. Covers HTTP, SQL, RPC, AI, CLI, workflows, and clustering:
```typescript
import { ... } from "effect/unstable/http"
import { ... } from "effect/unstable/sql"
import { ... } from "effect/unstable/rpc"
import { ... } from "effect/unstable/ai"
```

**Platform-specific packages** remain separate:
```typescript
import { NodeRuntime } from "@effect/platform-node"
// @effect/sql-pg, @effect/sql-sqlite, etc.
```

See `references/packages.md` for the full module breakdown and when to use unstable vs stable APIs.

## Core Patterns

### Errors

Model failures as tagged unions with `Data.TaggedError`. This makes errors typed, exhaustive, and easy to handle:

```typescript
class NotFound extends Data.TaggedError("NotFound")<{ id: string }> {}
class Unauthorized extends Data.TaggedError("Unauthorized")<{}> {}

const getUser = (id: string): Effect.Effect<User, NotFound | Unauthorized> =>
  Effect.gen(function* () {
    const row = yield* db.findUser(id)
    if (!row) return yield* new NotFound({ id })
    return row
  })

// Catch a specific error branch
const result = yield* getUser(id).pipe(
  Effect.catchTag("NotFound", () => Effect.succeed(guestUser)),
)
```

Effect failures are **not** thrown exceptions. Never wrap `yield*` in try-catch — it won't catch anything. Use `return yield*` to make early failure explicit and readable.

### Services and Layers

Define services with `Context.Tag`, implement with layers, inject at the program edge:

```typescript
// Service interface
class UserRepo extends Context.Tag("UserRepo")<
  UserRepo,
  { findById: (id: string) => Effect.Effect<User, NotFound> }
>() {}

// Implementation
const UserRepoLive = Layer.succeed(UserRepo, {
  findById: (id) => Effect.tryPromise({
    try: () => db.query(`SELECT * FROM users WHERE id = $1`, [id]),
    catch: (e) => new DatabaseError({ cause: e }),
  }),
})

// Usage
const program = Effect.gen(function* () {
  const repo = yield* UserRepo
  return yield* repo.findById("123")
})

// Wire up at the edge
Effect.runPromise(program.pipe(Effect.provide(UserRepoLive)))
```

Compose layers for larger programs: `Layer.provide`, `Layer.merge`, `Layer.provideMerge`.

### Composing Effects

```typescript
// Sequential chaining
const result = yield* fetchUser(id).pipe(
  Effect.flatMap((user) => fetchProfile(user.profileId)),
  Effect.tap((profile) => Effect.log(`Loaded ${profile.id}`)),
  Effect.map((profile) => profile.displayName),
)

// Concurrent execution
const [users, products] = yield* Effect.all(
  [fetchUsers(), fetchProducts()],
  { concurrency: "unbounded" },
)
```

### Schema

Schema is stable in the core `effect` package. Use it for validation, serialization, and type derivation:

```typescript
import { Schema } from "effect"

const User = Schema.Struct({
  id: Schema.String,
  email: Schema.String.pipe(Schema.pattern(/^.+@.+$/)),
  age: Schema.Number.pipe(Schema.int(), Schema.positive()),
})

type User = Schema.Schema.Type<typeof User>

// Decode with typed errors
const decode = Schema.decodeUnknown(User)
const user = yield* decode(rawInput)
```

### Config

Read environment variables without side effects:

```typescript
const AppConfig = Config.all({
  port: Config.integer("PORT").pipe(Config.withDefault(3000)),
  dbUrl: Config.string("DATABASE_URL"),
})

const program = Effect.gen(function* () {
  const { port, dbUrl } = yield* AppConfig
})
```

## Research Strategy

When answering Effect questions, consult sources in this order:

1. **Local Effect v4 source** (`~/.effect-smol/packages/effect/src/`) — authoritative for exact types, function signatures, and available overloads. Since v4 is actively evolving, this beats any external docs.
2. **Reference files in this skill** — package structure, module catalog, migration notes (`references/packages.md`)
3. **Existing project code** — check for established patterns before introducing new ones

**Finding things in the source:**

```
# Find a module's source file
Glob: ~/.effect-smol/packages/effect/src/<Module>.ts

# Search for a function or type across all sources
Grep: pattern="export.*functionName" path="~/.effect-smol/packages/effect/src" glob="*.ts"

# Find which file exports a given symbol
Grep: pattern="export.*MyType" path="~/.effect-smol/packages/effect/src" glob="*.ts" output_mode="files_with_matches"
```

For unstable modules, look under `~/.effect-smol/packages/effect/src/unstable/`.
For migration guidance, read `~/.effect-smol/MIGRATION.md` or `~/.effect-smol/packages/effect/SCHEMA.md`.

## Code Style Enforcement

The Effect docs define official code style guidelines. When this skill is active, treat the following as **hard rules**, not suggestions:

- **Use `runMain` for entrypoints**
  - When generating code that runs an Effect program as the main application on Node, always use `NodeRuntime.runMain(program)` from `@effect/platform-node`.
  - Do not show `Effect.runPromise(program)` as the primary way to run a long-lived app or server in `main` on Node. If the user does this, explicitly call it out and show a `NodeRuntime.runMain` version.
  - If the code clearly targets Bun or Browser, prefer the corresponding `*Runtime.runMain` helper when demonstrating the main entrypoint.

- **Avoid tacit / point-free usage**
  - Do not generate tacit style such as `Effect.map(fn)` / `Effect.flatMap(fn)` (where the function is the first argument) or `flow` from `effect/Function`.
  - Prefer explicit lambdas instead, for example `Effect.map((x) => fn(x))`, to preserve type inference and clearer stack traces.
  - When user code uses tacit style or `flow`, proactively mention that Effect discourages this pattern and provide an explicit-style rewrite.

Always ensure generated examples and suggested rewrites follow these rules. When following the guideline would materially change behavior, explain the trade-offs and show a guideline-compliant alternative.

## What to Avoid

| Pattern | Problem | Instead |
|---------|---------|---------|
| `try { yield* x } catch` | Effect failures aren't thrown | `Effect.catchTag` / `Effect.catchAll` |
| `as never` / `as any` | Masks real type errors | Fix the type; use `Effect.absurd` if truly impossible |
| Mixing `async/await` with `Effect.gen` | Creates hidden promises outside Effect's scheduler | Wrap with `Effect.tryPromise` |
| Old `@effect/platform` imports | APIs may have moved into `effect/unstable/*` in v4 | Check `references/packages.md` |
