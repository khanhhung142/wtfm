# Command: system — the whole repo → `system.md` + `glossary.md` + `specs/index.md`

Three root artifacts, written in one session because they come from the same broad reading. They
answer what a unit doc cannot: the stack, the project's own layer names, the shared rules, which files
are authoritative, the domain vocabulary, and where approved intent lives.

Without it every unit doc re-explains the same layering in slightly different words, and sixty
subagents each invent their own name for the same thing.

## When it runs

After bootstrap, before the first `map`. The scout report already holds the units, boundaries,
manifests and vocabulary, so this is cheap — and running it first means wave-two subagents are handed
the conventions instead of guessing at them one repo read at a time.

Re-run only when a convention changes, never per wave.

## Read order

1. **`_run/scout.md`.** Units, edges, authored sources, vocabulary. Do not re-derive any of it.
2. **Manifests and lockfiles.** Which choices were made, and where each is pinned.
3. **Tooling config.** Linter, formatter, type checker, CI workflow, pre-commit, editorconfig. The
   only place a convention is enforced rather than merely believed.
4. **Two units' entry points**, the most and least typical. Layer names come out of directory
   structure and import direction, not out of a framework's documentation.
5. **One cross-cutting file per concern**: error mapping, config loading, auth, logging, migrations.
6. **Tests of one unit.** Test layout is a convention, and usually an unwritten one.

Stop there. Depth is `map`'s job.

## `system.md`

````markdown
---
kind: map
unit: <project>
read: <branch> @ <sha>
written: <YYYY-MM-DD>
---

# <project> — system

## Stack
| Concern | Choice | Pinned in |
|---------|--------|-----------|
| Runtime | Node | `package.json`, `.nvmrc` |
| Database | Postgres | `docker-compose.yml`, `db/schema.prisma` |
| Queue | none — Postgres row polling, `internal/jobs/` | |

Runtime, framework, database, cache, queue, build, test, deploy. **No version column** — the lockfile
has it and is never wrong. `Pinned in` is the address the reader goes to for the number. A concern the
project does not have gets the row anyway, with `none — <what it does instead>`; that row prevents an
agent adding Redis to a project that deliberately has none.

## Shape
Units and their edges, one component diagram, every edge cited. What talks to what, by what mechanism.
Not what each unit does — that is the root `index.md` table.

## Layers
| Layer | Lives in | May call | Must not call | Enforced by |
|-------|----------|----------|---------------|-------------|

This project's own words, from the scout vocabulary — not the framework's textbook names.
`Must not call` is the useful column: it is the rule an agent is about to break.

## Conventions
| Rule | Example | Enforced by |
|------|---------|-------------|

Naming, file layout, error handling, logging, config access, test location, migrations, commits.
`Enforced by` is a linter rule cited by path and key, a CI step, or `nothing`.

## Authored sources
| Question | Authoritative file | Authority | Generated from it, do not cite |
|---|---|---|---|

Copied out of the scout report at graduation, because this is the table a reader needs next year and
`_run/` is a log they should not be sent to. The manual's most durable table: it says which files to
trust and which are output, and it changes only when the tooling changes.

## Cross-cutting
One short section each, only for what exists: configuration, errors, auth, logging, migrations, i18n.
What the mechanism is, where it is implemented, what a unit must do to take part.

## Divergences
Where the code breaks its own rules, cited. Which side is the mistake, or `unknown`.

## Open questions
````

## `glossary.md`

The other thing code cannot say, and the file with the longest half-life in the manual. A reader can
see the codebase has a type called `Order`. What no file states is that an `Order` here is unpaid
until a `Payment` attaches, that what the warehouse calls an order is a `Shipment`, and that
`order_id` on the legacy table means something else entirely. Each of those is a bug an agent is about
to write.

````markdown
---
kind: glossary
unit: <project>
read: <branch> @ <sha>
written: <YYYY-MM-DD>
---

# <project> — glossary

| Term | Means here | Does **not** mean | Provenance | Words the project avoids |
|------|-----------|-------------------|------------|--------------------------|
| Order | A submitted basket; unpaid until a Payment attaches | The warehouse's meaning — that is a Shipment | approved domain spec | purchase, cart |

## Collisions
One word, two concepts. Name both, name which owns the word, cite the loser so the next agent
recognises it on sight. The section that prevents cross-unit bugs.

## Aliases
One concept, several words, usually at a seam between two teams. Give the canonical word and say what
to read the others as.
````

**Only terms whose meaning is not obvious from the word.** A glossary that defines `User` teaches
nothing and buries the three rows that matter. `Does not mean` is where the value is: it is the
misreading somebody has already made.

## `specs/index.md`

A router to explicitly approved intent, never a rewrite of it. `kind: index`; the linked specs carry
intent authority, not the router.

| Need | Approved spec | Approval evidence | Scope |
|------|---------------|-------------------|-------|

If approval cannot be established, the document goes in open questions instead. Existing prose,
READMEs and diagrams do not become specs because they look authoritative.

## Rules

- **A convention needs three examples and one search for a counter-example.** Two files agreeing is a
  coincidence; a rule with a known exception is still a rule; an unchecked rule is a guess sixty
  subagents are about to copy.
- **`Enforced by: nothing` is a finding, not a blank.** It tells the next agent the rule holds by
  habit and will not stop them breaking it.
- **Name layers after the repo, never after the framework.** If the code says `handler`, the doc says
  `handler`, even when the textbook word is `controller`.
- **No unit specifics.** Anything true of one unit belongs in that unit's folder. Two units
  disagreeing is a `Divergences` row, not a paragraph here.
- **Every glossary row names its provenance.** Prefer an approved domain spec or a named owner with a
  date. Code shows where a word is used but often cannot define its business meaning: mark those
  `observed`, and leave inferred meanings in open questions until a human confirms them.
- **This file outlives the run, so it must not depend on it.** Never write "see `_run/scout.md`" in a
  doc that ships.
