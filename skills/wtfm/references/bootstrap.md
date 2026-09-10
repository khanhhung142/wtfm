# Stage: bootstrap — create the manual and wire it in

Runs once, after the scout gate. Creates the skeleton, connects it to the repos, and edits the
agent instructions so that every future agent reads the manual before it reads code.

## Where the manual lives

Two layouts. `scout` proposes one; this mode builds it.

**Single repo:** `docs/manual/` inside the repo, committed with the code. Docs and code move
together, and a pull request can change both. Prefer this whenever there is one repo.

**Several repos in a workspace:** a sibling directory, `<workspace>/<project>-manual/`, its own git
repo. Docs about six services belong to none of them, and a doc that must be committed six times is
a doc that will be committed once and rot in five.

Each repo then gets a symlink to its own slice, not to the whole manual, so an agent working in one
repo sees that repo's docs at a predictable path:

```bash
ln -s ../<project>-manual/services/<repo> <repo>/docs
```

Keep the link out of that repo's history without touching a tracked file:

```bash
echo 'docs' >> <repo>/.git/info/exclude
```

`.git/info/exclude` is local and untracked. Editing the repo's `.gitignore` instead leaves a
permanent modification in the working tree of every repo, which shows up in every `git status` and
eventually gets committed by accident. Even this one is a write inside their `.git`, so run it
only once decision 4 at gate 1 has put the manual in that repo.

## Skeleton

```
<manual>/
├── index.md          # router: need → path. Under 200 lines, forever
├── system.md         # stack, shape, layers, conventions. The whole repo, above any unit
├── glossary.md       # what this project's words mean. What code cannot say, part one
├── _goal.md          # objective, scope, definition of done. The human owns this
├── _scout.md         # written by scout; the truth-source table lives here
├── _progress.md      # the ledger
├── services/<unit>/  # one folder per unit, each with its own index.md
├── flows/            # cross-unit traces
├── explain/          # human-facing explainers
├── decisions/        # numbered ADRs. What code cannot say, part two
└── check-citations.sh  # optional, and only if they said yes. See verify.md
```

`services/<unit>` folder names equal the unit's directory name exactly. A doc folder that renames
its unit costs every future agent one lookup, forever.

## `index.md`

The router. It is read on every session by every agent, so it stays a table.

````markdown
# <project> manual

**Agents: read this file first, then read the code it points you at.** Written by the `wtfm` skill.
Open only what the task needs.

**The code is the truth. This manual is the index to it, and the record of why it is that way.** It
gets you to the right file in one read instead of twenty. It does not replace opening that file, and
where the two disagree, the code is right and the doc is a bug — fix the doc, never the reading.

| Need | Path |
|------|------|
| What this manual is for, and when it is done | [_goal.md](_goal.md) |
| What is already documented, what is next | [_progress.md](_progress.md) |
| Units, branches, truth sources, vocabulary | [_scout.md](_scout.md) |
| Stack, layers, conventions of this repo | [system.md](system.md) |
| What a word means in this project | [glossary.md](glossary.md) |
| One unit's internals | `services/<unit>/index.md` |
| A request end to end | `flows/<name>.md` |
| Plain-language walkthrough | `explain/<topic>.md` |
| Why the code is like this, and what lost | `decisions/` |

## Units

| Unit | Does | Docs | Status |
|------|------|------|--------|

## Status tags
`✅ implemented` · `🟡 partial` · `📋 spec-only`. Untagged means unfinished doc.

## Rules for agents
1. Code wins. A doc that contradicts the code is a bug in the doc — fix it or file it, never the
   other way round.
2. Cite `file:line Symbol` for every technical claim — the line to jump to, the symbol to find it
   again after the line moves.
3. Truth sources are in [_scout.md](_scout.md). Generated output is not documentation input.
4. Never copy a table between docs. Link it. Never restate what an authored file already says —
   point at it.
5. Conventions and layer names live in [system.md](system.md), the project's words in
   [glossary.md](glossary.md). Follow them, or record a divergence.
````

## `_progress.md`

````markdown
# Progress ledger

**Read first, update last.** No session holds the whole project. This file is the handoff.

`—` not started · `🔄` claimed · `✅` done · `⛔` blocked, say by what.
Done means written **and** citations verified, not "the file exists".

**A `🔄` you are reading is a crash survivor.** Nothing is running. Check the folder on disk and
either dispatch it fresh, extend it, or spot-check and bank it.

## Now
> Wave: <n> — <what it covers>
> In flight: <targets claimed but not banked, or none>
> Just landed: <one line>
> Next: <one line>
> Blocked on: <open question numbers, or none>

## Root docs
| Doc | Status | Notes |
|-----|--------|-------|
| system.md | — | |
| glossary.md | — | |

## Units
| Unit | Wave | index | service | data | api | events | config | Branch @ commit | Notes |
|------|------|-------|---------|------|-----|--------|--------|-----------------|-------|

## Flows
| Flow | Status | Units it crosses | Notes |
|------|--------|------------------|-------|

## Explainers
| Topic | Status | For whom |
|-------|--------|----------|

## Decisions
| # | Decision | Confidence | Status | Notes |
|---|----------|------------|--------|-------|

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
## The manual

`<manual>/` is an index to this codebase, not a copy of it. **The code is the truth.** The manual
exists to get you to the right file in one read instead of twenty, and to tell you the two things no
file can: what the words mean here, and why the code is like this.

**Where a doc and the code disagree, the code is right.** Do not average them, do not prefer the one
that reads better, and never change code to match a doc. Fix the doc, or record it as a finding.

Before coding, debugging, or answering an architecture question:

1. Read `<manual>/index.md` — it is a router, so this is cheap.
2. Follow it to the one or two docs the task needs. Do not load the whole manual into context.
3. **Open the files it cites.** A doc's job is finished when you know which lines to read.
4. Read `<manual>/_progress.md` when you need current state or open questions.

| Need | Path |
|------|------|
| One unit's internals | `<manual>/services/<unit>/index.md` |
| A request end to end | `<manual>/flows/<name>.md` |
| What a word means here | `<manual>/glossary.md` |
| Why the code is like this, and what was rejected | `<manual>/decisions/` |

A doc's frontmatter carries the commit it was written against. When a claim matters, check how far
the code has moved since:

```bash
git log --oneline <doc-commit>..HEAD -- <unit path> | wc -l
```

`0` — citations resolve, act on them. A handful — paths hold, line numbers are suspect, so jump by
the symbol in the citation rather than the number. Many — read the doc as a map of where to look,
not as a statement of fact, and confirm in the file before you quote it. A doc is wrong about
details long before it is wrong about where things live, which is why an old one is still worth
opening.

Truth sources are listed in `<manual>/_scout.md`. Cite `file:line Symbol` for technical claims. Docs are
written by the `wtfm` skill; write new ones with it rather than by hand — and before you propose an
architectural change, check `decisions/` for whether it was already tried.
````

## Rules

- Bootstrap writes structure, never content. Empty tables and headings are correct output here.
  A skeleton with invented rows is worse than an empty one, because the invention gets believed.
- Seed `_progress.md` unit rows from `_scout.md`, all `—`. That list is what `run` consumes.
- Seed `_goal.md` from the seven gate-1 decisions, including the ones the human did not contest.
  It is the decision record, and a decision nobody wrote down becomes a habit nobody can question.
- If an `AGENTS.md` or `CLAUDE.md` already exists, insert the docs-first section and leave the rest
  untouched. Say in the report which file was edited and what was added.
- **Offer the citation lint, do not install it.** `check-citations.sh` — the script in
  [verify.md](verify.md) — turns every stale citation into a build failure the day it happens, which
  is most of what `verify` would otherwise be for. It is also a file in somebody's repository and
  possibly a step in their CI, so it is theirs to accept. Ask once, at gate 1 alongside the commit
  question, and say plainly what it would add and where it would run.
