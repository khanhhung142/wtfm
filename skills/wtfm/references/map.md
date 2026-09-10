# Command: map — one unit → `services/<unit>/`

The workhorse. One unit, one session, one folder. This is the doc an agent opens when it has been
told to change something inside this unit and knows nothing about it.

## Read order

Do not reorder. Each step tells you what to look for in the next, and reading logic before schema
means reading logic without knowing what the nouns are.

0. **`system.md`.** The layers and conventions this repo already follows. Read it before the code so
   you name things the way the rest of the manual does, and record a divergence when this unit
   disagrees rather than inventing a second vocabulary.
1. **Manifest and build files.** Language, framework versions, codegen commands, what is generated.
2. **Authored sources for this unit** (from `_scout.md`): interfaces, schemas and approved specs.
   Keep their authority separate: code shows implementation, while an approved spec states intent.
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

Start with the smallest useful set. Every unit gets `index.md`; add `pitfalls.md` only when there
are verified cross-file invariants, implicit behaviour or dangerous exceptions. Create a topic file
only when the index would otherwise need to answer a distinct recurring question. A backend service
does not earn seven files merely by existing.

| File | Contents |
|------|----------|
| `index.md` | Router: ownership, entry points, boundaries, relevant specs/ADRs/flows, and where to read next |
| `pitfalls.md` | Only verified traps and invariants that no single source file can own |
| `architecture.md` | Only when startup or wiring spans files and is not obvious from the entry point |
| `data.md` | Only for cross-table invariants, tenancy, soft delete or transaction boundaries |
| `api.md` | Only when the surface is scattered or has exceptions not visible at registration |
| `events.md` | Only when async boundaries, retry or ordering need a map |
| `config.md` | Only keys with non-obvious operational failure modes |

`index.md` records what the unit owns, branch and commit read, entry points and dependencies in and
out. Do not copy endpoint, table or config counts that a command can derive.

## The derivable test

Before writing any table, ask: **could a script regenerate this from an authored file?** If yes, do
not write it. It will be wrong by Thursday, it will be believed anyway, and the file it copied was
already correct, already versioned, and already where the reader was going to end up.

That gives every surface doc — `data.md`, `api.md`, `config.md`, `events.md` — the same three-part
shape, and the middle part is conditional:

1. **The pointer.** One cited line naming where the authoritative definition lives. Always written.
2. **A locator table, only when the surface is scattered.** If the definition sits in one authored
   file — an OpenAPI document, `schema.prisma`, a `.proto`, a migrations directory — there is
   nothing to index; the file *is* the index and you point at it. If instead the surface is spread
   across thirty controllers or a hundred `HandleFunc` calls, then no single file answers "what
   endpoints exist", and building that answer is the most valuable thing this doc does.
3. **The traps.** What the authoritative file does not say. Always written, and always the reason
   the doc earns its place.

**A locator table carries addresses, never payloads.** `POST /orders →
internal/http/order.go#Create, auth: member, 🟡` is a locator: it survives a field being added to the request body, and it
takes a reader somewhere. Reproducing that request body is a payload: it is the schema restated one
step later, it goes stale the first time somebody adds a field, and it saves the reader a read they
should be doing anyway.

The traps are things like these, and none of them are visible in the schema or the route table:

- An ORM hook or middleware that silently scopes every query by tenant.
- Two tables that must be written in one transaction, and what breaks when they are not.
- The three routes out of sixty that skip the auth middleware, and why.
- A column that is nullable in the schema and required by every code path that reads it.
- A config key whose wrong value fails at 3am rather than at boot.
- A handler that is a stub, in a file full of handlers that are not.

If a surface doc has a pointer and no traps, say so — `no exceptions found, checked <what>` — and
keep it short. A one-screen doc that is entirely true beats a forty-row table nobody re-checked.

### Name what guards each trap

A trap is a claim about behaviour, and behaviour changes. End each one with what would notice:

```markdown
- **`POST /orders/:id/refund` skips the auth middleware.** Registered on the bare mux at
  `internal/http/router.go#Register`, deliberate — see [decisions/04](../../decisions/04-...md).
  Guarded by: `internal/http/refund_test.go#TestRefundRejectsUnsignedBody`
- **Every handler here is tenant-scoped by an ORM hook**, not by its own query.
  Guarded by: nothing — a raw query bypasses it silently.
```

`Guarded by: <test>` is evidence that a test appears to exercise the claim. It is not a guarantee:
the test may be weak, skipped or absent from CI. Record the CI job too when verified.

`Guarded by: nothing` is the useful half. It marks a claim that can quietly become false with nothing
anywhere objecting — and those are exactly the claims a `verify` session should spend its reading on.
It is also, quite often, a missing test somebody should write, which is a better outcome than a
better doc.

Search for the guard before you write `nothing`. A trap you cannot find a test for is worth one
`grep` through the test names, because tests are where the previous person recorded the same trap.

## The use-case table

Use this table only when behaviour is scattered and agents repeatedly need a locator. Do not create
it when an authored route, command or RPC registry already provides the same index.

| Use case | Endpoint | Entry | Does | Touches | Calls out | Status |
|----------|----------|-------|------|---------|-----------|--------|
| Register user | `POST /users` | `internal/user/register.go#Register` | validate, dedupe, write user + profile in one transaction, emit `user.created` | `users`, `profiles` | `id-service.Next` | ✅ |

**Does** is one line, and its only job is to let a reader pick this row out of forty. It names the
shape — what gets touched, and what ordering matters — and then stops. It is not the doc comment,
which describes intent rather than behaviour; not a paraphrase of the function name, which adds
nothing; and not a transcription of the branch structure, which changes every commit and would make
this table a paraphrase of code you are now on the hook to maintain.

The reader's next move is always to open `Entry`. Write the row that gets them there, not the row
that tries to save them the trip.

A handler that is empty, returns nil, or throws not-implemented is `📋`, and the row says so. Half
of a mid-flight codebase is stubs, and treating a stub as working is the failure this table exists
to prevent.

## Rules

- Cross-cutting machinery gets documented once in `architecture.md`, and `data.md` links to it. A
  reader who misses an ORM hook that scopes every query by tenant writes a query that returns
  another customer's rows.
- Never paste generated code. Cite the definition it came from.
- **A constraint whose reason is invisible in the code is a `decisions/` entry, not a paragraph
  here.** Cite it by number. The unit doc says what the rule is; the decision says what it cost and
  what would reverse it, and only one of those two survives the next refactor.
- A unit with no persistence says so and points at the unit that persists for it, rather than
  inventing tables from the field names it passes through.
- When the unit is large, split by domain across two sessions and say in `index.md` what the second
  session still owes. Half a unit documented well beats a whole one skimmed.
- Read [references/diagrams.md](diagrams.md) before drawing the component diagram.
