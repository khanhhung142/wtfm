# Command: map — one unit → `services/<unit>/`

The workhorse. One unit, one session, one folder. This is the doc an agent opens when it has been
told to change something inside this unit and knows nothing about it.

**You will read far more than you write.** That ratio is the mode working. Read the logic to find the
two invariants that span it, then write the two invariants and cite the logic.

## Read order

Do not reorder. Each step tells you what to look for in the next, and reading logic before schema
means reading logic without knowing what the nouns are.

0. **`system.md`.** The layers and conventions this repo already follows. Read it before the code so
   you name things the way the rest of the manual does, and record a divergence when this unit
   disagrees rather than inventing a second vocabulary.
1. **Manifest and build files.** Language, framework, codegen commands, what is generated.
2. **Authored sources for this unit** (from the scout report): interfaces, schemas, approved specs.
3. **Persistence definitions.** Tables, collections, indexes, migrations.
4. **Wiring.** The container, context or module listing this unit's dependencies. One file usually
   enumerates everything the unit can reach, and it is the fastest map of its blast radius.
5. **Config.** The checked-in template, plus how it reaches the code.
6. **Business logic**, one folder at a time. The bulk of the reading, and almost none of the writing.
7. **Cross-cutting machinery.** Middleware, interceptors, ORM hooks, error mapping, auth. These
   change every request and every query invisibly, and they are where the invariants live.
8. **Tests.** Confirm what you inferred, and mine them for edge cases the code does not state.

## Files to write

**`index.md` alone is the correct output for most units.** A unit does not earn a second file by
existing; it earns one when a distinct question keeps being asked and the index cannot hold the
answer. Every extra file is another thing to keep true for a year.

| File | Add only when |
|------|---------------|
| `index.md` | Always. Its Invariants section holds the cross-file rules |
| `invariants.md` | That section outgrew the index — usually because decision 8 refused code comments, so local traps landed here too |
| `architecture.md` | Startup or wiring spans files and is not obvious from the entry point |

There is no `api.md`, `data.md`, `events.md` or `config.md`. Each was a pointer plus a list of traps,
and both halves now have better homes: the pointer belongs in `index.md` under Authored sources, and a
trap belongs beside the code it is about. A surface that genuinely needs a locator gets one table in
`index.md`, not a file of its own.

Say in your report which files you did not write and why: *"no locator table: the surface is one
OpenAPI document"* is a finding, not a gap.

## `index.md`

````markdown
---
kind: index
unit: <name>
read: <branch> @ <sha>
written: <YYYY-MM-DD>
---

# <unit>

**Owns:** one line. **Does not own:** one line — this is the more useful half.

## Authored sources
| Question | File | Generated from it, do not cite |
|---|---|---|
| What tables exist | `db/schema.prisma` | `src/generated/` |
| What endpoints exist | `api/openapi.yaml` | `src/client/` |

Where a question has no single authored file, the locator table below answers it instead.

## Entry points
| Trigger | Door in |
|---|---|
| HTTP | `internal/http/router.go#Register` |
| Worker | `cmd/worker/main.go#main` |

## Dependencies
In and out, by mechanism, cited. What breaks this unit, and what this unit breaks.

## Invariants
Cross-file rules, each with `Guarded by:`. Local ones are comments in the code — link, do not repeat.

## Read next
| To do | Read |
|---|---|
| Change an endpoint | `api/openapi.yaml`, then the handler it names |
| Understand this unit's place | [../../system.md](../../system.md) |
| Know why it is shaped this way | [../../decisions/07-…](../../decisions/07-….md) |

## Not built at <sha>
`📋` rows. Empty is a fine section — say `nothing found`.

## Open questions
````

Do not copy endpoint, table or config counts. A command derives those; give the command once.

## The locator table

Only when a surface is scattered across many files **and** no registry already indexes it. Thirty
controllers with no route table earn one; an OpenAPI document does not.

| Use case | Endpoint | Entry | Touches | Calls out |
|----------|----------|-------|---------|-----------|
| Register user | `POST /users` | `internal/user/register.go#Register` | `users`, `profiles` (one tx) | `id-service.Next` |

Every column is an address: the name a human searches for, the route, the door, the tables, the
outbound call. **There is deliberately no column for what the code does.** A one-line behaviour
paraphrase is the most tempting thing to put here and the first thing to go stale; `Touches` and
`Calls out` already let a reader pick their row out of forty, and they only change when the design
does. A handler that is empty or throws not-implemented gets `📋 not built at <sha>`.

The reader's next move is always to open `Entry`. Write the row that gets them there, not the row
that tries to save them the trip.

## Where each trap goes

A trap is the most valuable thing you will find and the fastest-rotting thing you could write. Where
it goes decides whether it is still true next year:

| The trap is | Write it as | Why |
|---|---|---|
| About one line or function | A comment at that line | Moves with the code, reviewed in the same PR |
| Enforceable | A note in your report that a test is missing | A failing test beats any sentence |
| Spanning several files | `index.md` invariants, or `invariants.md` | No single file can own it |

Only the third row is the manual's. The first two a doc structurally cannot do: be reviewed alongside
the code it describes, or fail when it becomes false.

Granted decision 8: add the comment, one or two lines, keep every such edit in one separate diff the
human can review or drop whole, and list the files in your report. Never restructure code, never fix
the trap, never touch anything but a comment.

### Name what guards each invariant

```markdown
- **Every read here is tenant-scoped by an ORM hook**, not by the query.
  `internal/db/hooks.go#BeforeQuery` — a raw query bypasses it silently.
  Guarded by: nothing.
- **`orders` and `order_items` must be written in one transaction.**
  `internal/order/create.go#Create`
  Guarded by: `internal/order/create_test.go#TestCreateRollsBackItems`
```

`Guarded by: <test>` is evidence a test appears to exercise the claim — not a guarantee; it may be
weak, skipped or absent from CI. `Guarded by: nothing` is the useful half: it marks a claim that can
quietly become false with nothing objecting, and it is usually a missing test somebody should write,
which is a better outcome than a better doc. Grep the test names before writing `nothing` — tests are
where the previous person recorded the same trap.

## Rules

- **An invariant whose reason is invisible in the code is a `decisions/` entry, not a paragraph
  here.** Cite it by number. The unit doc says what the rule is; the decision says what it cost and
  what would reverse it, and only one of those survives the next refactor.
- Never paste generated code. Cite the definition it came from.
- A unit with no persistence says so and points at the unit that persists for it, rather than
  inventing tables from field names it passes through.
- When the unit is large, split by domain across two sessions and say in `index.md` what the second
  session still owes. Half a unit documented well beats a whole one skimmed.
- **Report every source file you commented**, separately from the docs, so that diff can be reviewed
  or reverted on its own.
- Read [diagrams.md](diagrams.md) before drawing anything.
