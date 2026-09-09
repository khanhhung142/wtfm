# Mode: map — one unit → `services/<unit>/`

The workhorse. One unit, one session, one folder. This is the doc an agent opens when it has been
told to change something inside this unit and knows nothing about it.

## Read order

Do not reorder. Each step tells you what to look for in the next, and reading logic before schema
means reading logic without knowing what the nouns are.

1. **Manifest and build files.** Language, framework versions, codegen commands, what is generated.
2. **Truth sources for this unit** (from `_scout.md`): interface or schema definitions. This is the
   unit's contract with everyone else, and the contract is the spine of the doc.
3. **Persistence definitions.** Tables, collections, indexes, migrations.
4. **Wiring.** The container, context or module that lists the unit's dependencies: databases,
   caches, clients of other units, queues. One file usually enumerates everything this unit can
   reach, and it is the fastest map of its blast radius.
5. **Config.** The checked-in template, plus how it reaches the code.
6. **Business logic**, one folder at a time. Behaviour lives here and this is the bulk of the work.
7. **Cross-cutting machinery.** Middleware, interceptors, ORM hooks, error mapping, auth. These
   change every request and every query invisibly.
8. **Tests.** Confirm the behaviour you inferred, and mine them for edge cases the code does not
   state.

## Files to write

Adapt the set to the unit. A backend service earns all of these; a UI unit swaps `data`/`api` for
routes and state; a library gets `index` plus one file per exported area. Write the file when the
unit has that surface, and skip it when it does not.

| File | Contents |
|------|----------|
| `index.md` | Router for this unit, plus the quick-facts table. Written first, updated last |
| `service.md` | Purpose, what it owns, stack and versions, what is generated versus hand-written |
| `architecture.md` | Startup sequence, dependencies, middleware chain, stores. One component diagram |
| `layers.md` | The request path, and the use-case table below. The unit's code index |
| `data.md` | One row per table or collection: purpose, key fields, indexes, relations, soft-delete and tenancy behaviour |
| `api.md` | Every endpoint or RPC: name, request → response, auth required, entry `file:line`, status tag |
| `events.md` | Queue in and out, scheduled jobs, or "none, synchronous only" when that is true and verified |
| `config.md` | Key ↔ environment variable ↔ struct field ↔ what breaks when it is wrong |

`index.md` quick facts: what the unit owns, branch and commit read, entry point, count of endpoints
and tables, dependencies in and out, status tag counts.

## The use-case table

The most-read table in the atlas. One row per unit of behaviour.

| Use case | Endpoint | Entry | Does | Touches | Calls out | Status |
|----------|----------|-------|------|---------|-----------|--------|
| Register user | `POST /users` | `internal/user/register.go:34` | validate → dedupe on email → transaction: insert user + profile → emit `user.created` | `users`, `profiles` | `id-service.Next` | ✅ |

**Does** is the actual branch structure in one or two lines: validation, then transaction, then side
effect. It is not the doc comment, which describes intent rather than behaviour, and not a
paraphrase of the function name, which adds nothing.

A handler that is empty, returns nil, or throws not-implemented is `📋`, and the row says so. Half
of a mid-flight codebase is stubs, and treating a stub as working is the failure this table exists
to prevent.

## Rules

- Cross-cutting machinery gets documented once in `architecture.md`, and `data.md` links to it. A
  reader who misses an ORM hook that scopes every query by tenant writes a query that returns
  another customer's rows.
- Never paste generated code. Cite the definition it came from.
- A unit with no persistence says so and points at the unit that persists for it, rather than
  inventing tables from the field names it passes through.
- When the unit is large, split by domain across two sessions and say in `index.md` what the second
  session still owes. Half a unit documented well beats a whole one skimmed.
- Read [references/diagrams.md](diagrams.md) before drawing the component diagram.
