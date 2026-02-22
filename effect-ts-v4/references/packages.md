# Effect v4 Package Reference

## Repository

Effect v4 lives in the `effect-smol` repository during its beta phase:
```bash
git clone https://github.com/Effect-TS/effect-smol ~/.effect-smol
```

Source files: `~/.effect-smol/packages/effect/src/`
Migration guide (v3→v4): `~/.effect-smol/MIGRATION.md`
Schema migration: `~/.effect-smol/packages/effect/SCHEMA.md`

## Installation

```bash
npm install effect@4.0.0-beta.0
# All @effect/* packages are versioned to match — no more version mismatch
```

## Stable Core (`effect`)

Everything at the top-level `effect` import is production-stable and won't break in minor releases.

| Module | What's in it |
|--------|-------------|
| `Effect` | Core effect type, all combinators (`map`, `flatMap`, `gen`, `all`, etc.) |
| `Layer` | Dependency injection — compose and provide services |
| `Schema` | Runtime validation, serialization, type derivation |
| `Stream` | Lazy effectful streams with backpressure |
| `Fiber` | Fiber lifecycle (fork, join, interrupt) |
| `Queue` | Concurrent queues (bounded, unbounded, dropping, sliding) |
| `Ref` | Mutable references |
| `Scope` | Resource lifecycle management |
| `Context` | Service container and `Tag` definitions |
| `Data` | `TaggedError`, `TaggedClass`, structural equality |
| `Config` | Environment variable loading and validation |
| `Match` | Exhaustive pattern matching |
| `Option` | Nullable values without null |
| `Either` | Synchronous success/failure |
| `Cause` | Structured error cause trees |
| `Exit` | Result of a completed fiber |
| `Runtime` | Running effects and custom runtimes |
| `Array`, `Record` | Immutable collection utilities |

## Unstable Modules (`effect/unstable/*`)

Part of the `effect` package but may have breaking changes in minor releases. They are intended for use — just expect adjustments as they mature. Once stable, they graduate to a top-level export.

| Import path | Domain |
|-------------|--------|
| `effect/unstable/http` | HTTP client and server |
| `effect/unstable/sql` | SQL/database client abstractions |
| `effect/unstable/rpc` | RPC framework |
| `effect/unstable/ai` | AI model integrations |
| `effect/unstable/cli` | CLI argument parsing and apps |
| `effect/unstable/workflow` | Durable workflows |
| `effect/unstable/cluster` | Distributed clustering |

17 unstable modules ship in v4 beta total — search `~/.effect-smol/packages/effect/src/unstable/` for the full list.

## Separate Packages in effect-smol

These packages live in the `effect-smol` monorepo and are versioned alongside `effect`:

| Package | Use when |
|---------|----------|
| `@effect/platform-node` | Node.js runtime (server apps) |
| `@effect/platform-bun` | Bun runtime |
| `@effect/platform-browser` | Browser environment |
| `@effect/sql-*` | Database drivers (pg, sqlite, mysql, mssql) |
| `@effect/ai-*` | AI provider integrations |
| `@effect/opentelemetry` | Tracing and observability |
| `@effect/vitest` | Testing utilities |

Source for these lives at `~/.effect-smol/packages/<package-name>/src/`.

## What Moved from v3

| v3 package | v4 location |
|-----------|-------------|
| `@effect/platform` (HTTP, FileSystem) | `effect/unstable/http` + `@effect/platform-node` |
| `@effect/rpc` | `effect/unstable/rpc` |
| `@effect/cluster` | `effect/unstable/cluster` |
| `@effect/schema` | `effect` (Schema is now in core) |

If you encounter a v3 import that doesn't resolve in v4, grep the effect-smol source to find where it moved:
```
Grep: pattern="export.*SymbolName" path="~/.effect-smol/packages/effect/src" glob="*.ts"
```
