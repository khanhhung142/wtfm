# Stage: bootstrap — create the manual and wire it in

Runs once, after the scout gate. Creates the skeleton, connects it to the repos, and edits the agent
instructions so every future agent reads the manual before it reads code.

## Where the manual lives

**Single repo:** `docs/manual/` inside it, committed with the code. Docs and code move together, and
one pull request can change both. Prefer this whenever there is one repo.

**Several repos:** a sibling directory, `<workspace>/<project>-manual/`, its own git repo. Docs about
six services belong to none of them, and a doc that must be committed six times gets committed once
and rots in five.

Each repo then gets a symlink to its own slice, so an agent working in one repo finds its docs at a
predictable path:

```bash
ln -s ../<project>-manual/services/<repo> <repo>/docs
echo 'docs' >> <repo>/.git/info/exclude
```

`.git/info/exclude` is local and untracked. Editing the repo's `.gitignore` instead leaves a permanent
modification in every working tree, shows up in every `git status`, and eventually gets committed by
accident. Even this is a write inside their `.git`, so run it only once decision 4 has put the manual
in that repo.

## Skeleton

```
<manual>/
├── index.md            # router: need → path. Under 200 lines, forever
├── system.md           # stack, layers, conventions, authored sources
├── glossary.md         # what this project's words mean
├── specs/              # links to approved intent; never inferred by wtfm
├── services/<unit>/     # one folder per unit: index.md, rarely a second file
├── flows/              # cross-unit traces
├── explain/            # human-facing explainers, only if a human reads this
├── decisions/          # numbered ADRs. Never edited, only superseded
├── _run/               # this run's log. Not part of the manual
│   ├── goal.md         #   objective, scope, done. The human owns it
│   ├── scout.md        #   written once by scout
│   └── progress.md     #   the ledger
└── check-citations.sh  # optional, only if they said yes. See verify.md
```

**Why `_run/` is a folder.** Everything above it describes the codebase and is worth reading next
year. Everything inside it describes one documentation run and is misleading the day that run ends.
Two directories cannot be confused the way two filename prefixes can, and no rule has to be remembered
for it to hold.

`services/<unit>` folder names equal the unit's directory name exactly. A doc folder that renames its
unit costs every future agent one lookup, forever.

## `index.md`

Read on every session by every agent, so it stays a table.

````markdown
# <project> manual

**Agents: read this file first, then read the code it points you at.** Written by the `wtfm` skill.
Open only what the task needs.

This manual holds only what code cannot state about itself: where things are, what must not break
across files, what the words mean here, and which options were rejected. **Anything derivable from an
authored file was deliberately left out** — payloads, field lists, endpoint counts. Go read that file;
`system.md` says which files those are.

**Authority follows the question.**

| Read this | For | Never for |
|---|---|---|
| Code at the revision in a doc's frontmatter | What is implemented | What is deployed |
| `specs/` | Intended behaviour | What currently runs |
| `decisions/` | Why, and what was rejected | Current design |
| `glossary.md` | What a word means here | |
| any `index.md` | Where to look | Behaviour |

Where implementation prose and code disagree, fix the prose. Where an approved spec and code
disagree, record drift: change neither, average never.

| Need | Path |
|------|------|
| Stack, layers, conventions, which files are authoritative | [system.md](system.md) |
| What a word means in this project | [glossary.md](glossary.md) |
| What the system is intended to do | `specs/` |
| One unit's internals | `services/<unit>/index.md` |
| A request end to end | `flows/<name>.md` |
| Plain-language walkthrough | `explain/<topic>.md` |
| Why the code is like this, and what lost | `decisions/` |

## Units

| Unit | Does | Docs |
|------|------|------|

## Markers
`📋 not built at <sha>` marks code that is a stub or absent at that revision. Everything else is
indexed because it exists. There is no "verified" marker: the frontmatter revision is the claim.

## Rules for agents
1. Match authority to the question, per the table above.
2. Cite navigation by path and stable anchor — `internal/user/register.go#Register`. Line numbers are
   disposable jump hints.
3. Generated output is not documentation input. `system.md` lists which sources are authored.
4. Never copy a table between docs; link it. Never restate what an authored file says; point at it.
5. Before proposing an architectural change, check `decisions/` for whether it was already tried.
6. Write new docs with the `wtfm` skill, not by hand.

`_run/` is the log of the run that wrote this manual. It describes that run, not this codebase — read
it only to resume or audit the run.
````

## `_run/progress.md`

````markdown
# Progress ledger

**Read first, update last.** No session holds the whole project. This file is the handoff.

`—` not started · `🔄` claimed · `✅` done · `⛔` blocked, say by what.
Done means written, links resolved, sampled claims checked — not "the file exists".

**A `🔄` you are reading is a crash survivor.** Nothing is running. Check the folder on disk and
either dispatch fresh, extend, or spot-check and bank.

## Now
> Wave: <n> — <what it covers>
> In flight: <claimed but not banked, or none>
> Just landed: <one line>
> Next: <one line>
> Blocked on: <open question numbers, or none>

## Root docs
| Doc | Status | Notes |
|-----|--------|-------|
| system.md | — | |
| glossary.md | — | |
| specs/ inventory | — | approved intent only |
| graduation | — | authored sources + vocabulary copied out of `_run/` |

## Units
| Unit | Wave | Status | Docs written | Source files commented | Read | Notes |
|------|------|--------|--------------|------------------------|------|-------|

One status cell per unit: a unit is one session's target. `Docs written` lists what it actually
earned — `index` for most. A column per possible topic file would turn the optional file list in
[map.md](map.md) into a checklist, and a checklist gets filled in.

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

Numbering open questions matters more than it looks. A numbered question is citable from a doc, so the
doc records the gap in one token instead of restating the problem in a paragraph.

## The agent instructions patch

Add to `AGENTS.md`, or `CLAUDE.md` where the project uses one. **Extend the existing file, never
replace it.** Keep it short: it is loaded on every turn of every session forever.

````markdown
## The manual

`<manual>/` is an index to this codebase, not a copy of it. It holds what code cannot state about
itself — where things are, what must not break across files, what the words mean here, which options
were rejected. Anything derivable from an authored file was deliberately left out, so when you want a
payload, a field list or an endpoint list, open the authored file `<manual>/system.md` names.

Authority follows the question: code at a doc's recorded revision shows implementation; `specs/`
states intent; `decisions/` preserves rationale; `glossary.md` records domain language; an index only
routes. Where implementation prose and code disagree, fix the prose. Where an approved spec and code
disagree, record drift — never average them, never silently change one to match the other.

Before coding, debugging, or answering an architecture question:

1. Read `<manual>/index.md` — it is a router, so this is cheap.
2. Follow it to the one or two docs the task needs. Do not load the whole manual.
3. **Open the files it cites.** A doc's job is finished when you know which lines to read.

| Need | Path |
|------|------|
| One unit's internals | `<manual>/services/<unit>/index.md` |
| A request end to end | `<manual>/flows/<name>.md` |
| Intended behaviour | `<manual>/specs/` |
| What a word means here | `<manual>/glossary.md` |
| Why the code is like this, and what was rejected | `<manual>/decisions/` |

Frontmatter carries the revision a doc was written against. When a claim matters:

```bash
git log --oneline <that-sha>..HEAD -- <unit path> | wc -l
```

`0` — it describes the recorded revision. A handful — use it as a map, inspect changed areas. Many, or
a rename in between — treat it as a navigation hypothesis only. Old docs often still point at the
right code; age never grants behavioural authority.

`📋 not built at <sha>` means that code was a stub or absent at that revision. Check before assuming
it is still missing.

Docs are written by the `wtfm` skill — write new ones with it rather than by hand.
````

## Rules

- Bootstrap writes structure, never content. Empty tables and headings are correct output. A skeleton
  with invented rows is worse than an empty one, because the invention gets believed.
- **The agent patch never routes to `_run/`.** Pointed at from `AGENTS.md`, a ledger is read as
  current state forever. Durable content graduates into `system.md` and `glossary.md`.
- `specs/` holds only documents explicitly approved as intent, or links to them. Never promote a
  README or an explainer to spec by inference.
- Seed `_run/progress.md` unit rows from the scout report, all `—`. That list is what `run` consumes.
- Seed `_run/goal.md` from the eight gate-1 decisions, including uncontested ones. A decision nobody
  wrote down becomes a habit nobody can question.
- **Offer the link lint, do not install it.** `check-citations.sh` (see [verify.md](verify.md)) catches
  missing `path#Symbol` addresses, not semantic drift. It is also a file in somebody's repository and
  possibly a step in their CI, so it is theirs to accept. Ask once, at gate 1.
