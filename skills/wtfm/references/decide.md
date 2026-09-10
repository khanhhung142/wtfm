# Command: decide — record a decision → `decisions/<n>-<slug>.md`

The only doc in the manual that code cannot replace. Everything else in here restates something a
file already says, more cheaply and in one place; this one holds what no file says and no file ever
will — what was considered instead, and why it lost.

A codebase shows the option that won. It never shows the three that were rejected, so every agent
that arrives later re-proposes one of them. That is the cost this file exists to stop.

## Why this one does not rot

Every other doc claims something about the code *now*, which is a claim the next commit can falsify.
A decision claims something about a *moment*, and a moment is finished. Code moving on does not make
a decision wrong — it makes it **superseded**, which is a new file, not an edit.

So the freshness rules elsewhere in this manual do not apply here. A five-year-old ADR is not stale.
An ADR edited to match new code is destroyed.

## The failure mode

**A fabricated rationale is unfalsifiable.** Every other claim in this manual is checkable: a reader
opens the cited line and the claim holds or does not. Nobody can check "we chose Postgres because we
expected write contention". If it was invented, it is invented forever, and the next agent will
defend a constraint that never existed.

Fabrication is therefore the one failure this mode must not commit. `Options: unknown — nobody
recorded them` is a correct, useful ADR. An ADR with plausible reasoning you reconstructed from the
shape of the code is worse than no ADR at all.

## Where rationale actually survives

Harvest before you ask. Most of it is recoverable, and each source is cheap:

| Source | Command or place | Yields |
|---|---|---|
| Revert and re-revert commits | `git log --oneline --grep='revert' -i` | An option that was *tried* and lost |
| Commit message bodies | `git log --format='%H%n%b' -- <path>` | The only prose most repos have |
| Merge commits and PR links | `git log --merges --format='%s %b'` | Where discussion happened, if it still resolves |
| Dead code and feature flags | search for disabled branches, `if false`, unused adapters | An abandoned approach still in the tree |
| Comments containing *because*, *don't*, *HACK*, *workaround*, *do not* | `grep -rniE '(because\|workaround\|hack\|do not \|don.t )' <unit>` | A constraint someone hit at 3am |
| Tests named for a bug or an issue number | test file names and `t.Run` strings | A failure the design now prevents |
| The manual's own open questions | `_progress.md` | Questions a human already answered in chat |
| Config toggles that exist for one caller | config template versus code | A negotiated exception |

What none of that yields, you **ask**. A decision is the one artifact where the human is a truth
source, and asking three questions is cheaper than fabricating one answer.

## Read order

1. `_progress.md` open questions. Several are already decisions waiting to be written down.
2. The harvest table above, for the unit or area in question.
3. The unit doc, for the shape the decision produced.
4. Then ask the human what is still missing — options, and the constraint that ruled them out.

## The file

````markdown
---
decision: <short imperative title>
date: <YYYY-MM-DD, or `unknown` with what bounds it>
status: accepted | superseded by <file> | reversed | unknown
confidence: cited | reported | inferred
units: <units this constrains>
---

# <n>. <Title>

## Context
What was true when this was decided: the constraint, the load, the deadline, the team. Cite it where
it is still visible in code or history; say `unknown` where it is not.

## Options
| Option | Cost | Why not |
|--------|------|---------|
Mark the chosen row. Two or more rows, or this is not a decision — it is a description, and it
belongs in the unit doc instead. `unknown — nobody recorded them` is a legitimate single row.

## Decision
One sentence, present tense. What the project does.

## Consequences
What this makes cheap. What it makes expensive. What it forbids outright — that last line is the one
an agent is about to violate.

## What would reverse this
The condition under which this stops being right: a scale number, a dependency dying, a requirement
arriving. This is the most useful section in the file, because it turns a historical note into a
trigger somebody can check.

## Evidence
Where each part came from: a commit sha, a PR, `asked <name>, <date>`, or `inferred, unconfirmed`.
````

## `confidence`, and why it is a separate field from `status`

| Value | Means | Allowed to do |
|---|---|---|
| `cited` | Rationale found in commits, comments or code | Stand as written |
| `reported` | A human said so, and is named with a date | Stand as written |
| `inferred` | You worked it out from the shape of the code | **Must be confirmed before an agent may act on it** |

An `inferred` ADR is a hypothesis wearing a decision's clothes. Keep it — it is still the best
starting point anyone has — but never let it lose the label. Promote it only when a human confirms,
and record who and when.

## Rules

- **Never invent an option.** The rejected alternatives are the entire value of the file and the one
  thing you cannot derive. Guess them and the file becomes a liability that reads like an asset.
- **Supersede, never edit.** New decision, new numbered file, old one flips to
  `superseded by <file>`. The record of having changed your mind is worth more than either decision.
- **Numbers are permanent.** Unit docs cite `decisions/07`. Renumbering breaks every one of them.
- **One decision per file.** A file covering three is cited for all three and superseded for none.
- **No status tags, no `file:line` requirement in the body.** This doc is not describing code. The
  `Evidence` section carries the coordinates instead.
- **Harvest the ledger.** Every answered open question that shaped code is an ADR nobody wrote. A run
  that ends with an empty `decisions/` folder documented a codebase and threw away the reasoning.
