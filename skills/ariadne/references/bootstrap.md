# Mode: bootstrap — create the atlas and wire it in

Runs once, after the scout gate. Creates the skeleton, connects it to the repos, and edits the
agent instructions so that every future agent reads the atlas before it reads code.

## Where the atlas lives

Two layouts. `scout` proposes one; this mode builds it.

**Single repo:** `docs/atlas/` inside the repo, committed with the code. Docs and code move
together, and a pull request can change both. Prefer this whenever there is one repo.

**Several repos in a workspace:** a sibling directory, `<workspace>/<project>-atlas/`, its own git
repo. Docs about six services belong to none of them, and a doc that must be committed six times is
a doc that will be committed once and rot in five.

Each repo then gets a symlink to its own slice, not to the whole atlas, so an agent working in one
repo sees that repo's docs at a predictable path:

```bash
ln -s ../<project>-atlas/services/<repo> <repo>/docs
```

Keep the link out of that repo's history without touching a tracked file:

```bash
echo 'docs' >> <repo>/.git/info/exclude
```

`.git/info/exclude` is local and untracked. Editing the repo's `.gitignore` instead leaves a
permanent modification in the working tree of every repo, which shows up in every `git status` and
eventually gets committed by accident.

## Skeleton

```
<atlas>/
├── index.md          # router: need → path. Under 200 lines, forever
├── _scout.md         # written by scout; the truth-source table lives here
├── _progress.md      # the ledger
├── services/<unit>/  # one folder per unit, each with its own index.md
├── flows/            # cross-unit traces
├── explain/          # human-facing explainers
└── decisions/        # one file per decision that shaped the code
```

`services/<unit>` folder names equal the unit's directory name exactly. A doc folder that renames
its unit costs every future agent one lookup, forever.

## `index.md`

The router. It is read on every session by every agent, so it stays a table.

````markdown
# <project> atlas

**Agents: read this file first.** Written by the `ariadne` skill. Open only what the task needs.

| Need | Path |
|------|------|
| What is already documented, what is next | [_progress.md](_progress.md) |
| Units, branches, truth sources, vocabulary | [_scout.md](_scout.md) |
| One unit's internals | `services/<unit>/index.md` |
| A request end to end | `flows/<name>.md` |
| Plain-language walkthrough | `explain/<topic>.md` |
| Why the code is like this | `decisions/` |

## Units

| Unit | Does | Docs | Status |
|------|------|------|--------|

## Status tags
`✅ implemented` · `🟡 partial` · `📋 spec-only`. Untagged means unfinished doc.

## Rules for agents
1. Cite `file:line` for every technical claim.
2. Truth sources are in [_scout.md](_scout.md). Generated output is not documentation input.
3. Never copy a table between docs. Link it.
````

## `_progress.md`

````markdown
# Progress ledger

**Read first, update last.** No session holds the whole project. This file is the handoff.

`—` not started · `🔄` in progress, say what is left · `✅` done · `⛔` blocked, say by what.
Done means written **and** citations verified, not "the file exists".

## Now
> Just landed: <one line>
> Next: <one line>
> Blocked on: <open question numbers, or none>

## Units
| Unit | index | service | data | api | events | config | Branch @ commit | Notes |
|------|-------|---------|------|-----|--------|--------|-----------------|-------|

## Flows
| Flow | Status | Units it crosses | Notes |
|------|--------|------------------|-------|

## Explainers
| Topic | Status | For whom |
|-------|--------|----------|

## Open questions
| # | Question | Raised by | Blocks | Answer |
|---|----------|-----------|--------|--------|

## Session log
| Date | Target | Result |
|------|--------|--------|
````

Numbering open questions matters more than it looks. A numbered question is citable from a doc, so
the doc records the gap in one token instead of restating the whole problem in a paragraph.

## The agent instructions patch

Add to `AGENTS.md`, or `CLAUDE.md` where the project already uses one. Keep it short, since it is
loaded on every turn of every session forever. Extend the existing file rather than replacing it.

````markdown
## Docs first

Before coding, debugging, or answering an architecture question:

1. Read `<atlas>/index.md`.
2. Read `<atlas>/_progress.md` when you need current state or open questions.
3. Open only what the task needs. Do not load the whole atlas into context.

| Need | Path |
|------|------|
| One unit's internals | `<atlas>/services/<unit>/index.md` |
| A request end to end | `<atlas>/flows/<name>.md` |
| Why the code is like this | `<atlas>/decisions/` |

Truth sources are listed in `<atlas>/_scout.md`. Cite `file:line` for technical claims. Docs are
written by the `ariadne` skill; write new ones with it rather than by hand.
````

## Rules

- Bootstrap writes structure, never content. Empty tables and headings are correct output here.
  A skeleton with invented rows is worse than an empty one, because the invention gets believed.
- Seed `_progress.md` unit rows from `_scout.md`, all `—`. That list is what `run` consumes.
- If an `AGENTS.md` or `CLAUDE.md` already exists, insert the docs-first section and leave the rest
  untouched. Say in the report which file was edited and what was added.
