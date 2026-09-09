# Command: system — the whole repo → `system.md`

One file at the root of the manual, above every unit. It answers what a unit doc cannot: what the
stack is, what shape the code is in, what this project's layer names actually mean, and which rules
hold in every unit. Written once, early, and read by every later session.

`map` writes `services/<unit>/architecture.md` — one unit's startup and wiring. This is the other
thing: the project's, and the conventions that outlive any one unit.

Without it every unit doc re-explains the same layering in slightly different words, and sixty
subagents each invent their own name for the same thing.

## When it runs

After bootstrap, before the first `map`. `_scout.md` already holds the units, the boundaries, the
manifests and the vocabulary, so this is cheap; and running it first means wave-two subagents are
handed the conventions instead of guessing at them one repo read at a time.

Re-run it only when a convention changes, not per wave.

## Read order

1. **`_scout.md`.** Units, edges, truth sources, vocabulary. Do not re-derive any of it.
2. **Manifests and lockfiles**, every unit. Stack and versions, pinned at a `file:line`.
3. **Tooling config.** Linter, formatter, type checker, CI workflow, pre-commit, editorconfig. This
   is the only place a convention is enforced rather than merely believed.
4. **Two units' entry points**, the most and least typical. Layer names are read out of directory
   structure and import direction, not out of a framework's documentation.
5. **One cross-cutting file per concern**: error mapping, config loading, auth, logging, migrations.
6. **Tests of one unit.** Test layout is a convention, and usually an unwritten one.

Stop there. Depth is `map`'s job.

## The file

````markdown
---
unit: <project>
branch: <branch>
commit: <sha>
written: <YYYY-MM-DD>
status: ✅ | 🟡 | 📋
---

# <project> — system

## Stack
| Concern | Choice | Version | Pinned at |
|---------|--------|---------|-----------|
Runtime, framework, database, cache, queue, build, test, deploy. One row each. A concern the
project does not have gets the row anyway, with `none — <what it does instead>`.

## Shape
Units and the edges between them, one component diagram, every edge cited. Say what talks to what
and by what mechanism. Do not repeat what each unit does — that is the root `index.md` table.

## Layers
| Layer | Lives in | May call | Must not call | Evidence |
|-------|----------|----------|---------------|----------|
Use this project's own words for the layers, taken from the `_scout.md` vocabulary — not the
framework's textbook names. The `Must not call` column is the useful one: it is the rule an agent
is about to break.

## Conventions
| Rule | Example | Enforced by |
|------|---------|-------------|
Naming, file layout, error handling, logging, config access, test location and naming, migrations,
commits. `Enforced by` is a linter rule at `file:line`, a CI step, or `nothing`.

## Cross-cutting
One short section each, only for what exists: configuration, errors, auth, logging, migrations,
i18n. What the mechanism is, where it is implemented, what a unit must do to take part.

## Divergences
Where the code breaks its own rules, cited. Which side is the mistake, or `unknown`.

## Open questions
````

## Rules

- **A convention needs three examples and one search for a counter-example.** Two files agreeing is
  a coincidence; a rule with a known exception is a rule, and an unchecked rule is a guess that
  sixty subagents are about to copy.
- **`Enforced by: nothing` is a finding, not a blank.** It tells the next agent the rule holds by
  habit and will not stop them breaking it.
- **Name layers after the repo, never after the framework.** If the code says `handler`, the doc
  says `handler`, even when the textbook word is `controller`.
- **No unit specifics.** Anything true of one unit only belongs in that unit's folder. When it turns
  out two units disagree, that is a `Divergences` row, not a paragraph here.
- **Unit docs link here and never restate.** `map` cites `../../system.md` for a convention; a unit
  doc that re-explains the layering is the duplication this file exists to kill.
