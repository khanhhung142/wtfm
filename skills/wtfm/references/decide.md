# Command: decide — record a decision → `decisions/<n>-<slug>.md`

A codebase shows the option that won. It never shows the three that were rejected, so every agent that
arrives later re-proposes one of them. That is the cost this file exists to stop.

## Why this one never rots

Every other doc claims something about the code *now*, which the next commit can falsify. A decision
claims something about a *moment*, and a moment is finished. Code moving on does not make a decision
wrong — it makes it **superseded**, which is a new file, not an edit.

So the freshness rules elsewhere do not apply here. A five-year-old ADR is not stale. An ADR edited to
match new code is destroyed. This is the highest-value writing in the manual per hour spent, and the
only part that is permanent by construction.

## The failure mode

**A fabricated rationale is unfalsifiable.** Every other claim here is checkable: open the cited line
and it holds or it does not. Nobody can check "we chose Postgres because we expected write contention".
Invented once, it is invented forever, and the next agent will defend a constraint that never existed.

`Options: unknown — nobody recorded them` is a correct, useful ADR. An ADR with plausible reasoning
reconstructed from the shape of the code is worse than no ADR at all.

## Where rationale actually survives

Harvest before you ask. Most of it is recoverable, and each source is cheap:

| Source | Command or place | Yields |
|---|---|---|
| Revert and re-revert commits | `git log --oneline --grep=revert -i` | An option that was tried and lost |
| Commit message bodies | `git log --format='%H%n%b' -- <path>` | The only prose most repos have |
| Merge commits and PR links | `git log --merges --format='%s %b'` | Where the discussion happened |
| Dead code and feature flags | disabled branches, `if false`, unused adapters | An abandoned approach still in the tree |
| Comments saying *because*, *HACK*, *do not* | `grep -rniE '(because\|workaround\|hack\|do not )' <unit>` | A constraint someone hit at 3am |
| Tests named for a bug or issue | test file names and `t.Run` strings | A failure the design now prevents |
| The run's own open questions | `_run/progress.md` | Questions a human already answered in chat |
| Config toggles that exist for one caller | template versus code | A negotiated exception |

What none of that yields, you **ask**. A decision is the one artifact where the human is a truth
source, and asking three questions is cheaper than fabricating one answer.

## Read order

1. `_run/progress.md` open questions. Several are already decisions waiting to be written down.
2. The harvest table above, for the unit in question.
3. The unit doc, for the shape the decision produced.
4. Then ask the human what is missing — the options, and the constraint that ruled them out.

## The file

````markdown
---
kind: decision
decision: <short imperative title>
date: <YYYY-MM-DD, or `unknown` with what bounds it>
status: accepted | superseded by <file> | reversed | unknown
confidence: cited | reported
units: <units this constrains>
---

# <n>. <Title>

## Context
What was true when this was decided: the constraint, the load, the deadline, the team. Cite it where it
is still visible in code or history; say `unknown` where it is not.

## Options
| Option | Cost | Why not |
|--------|------|---------|

Mark the chosen row. Two or more rows, or this is not a decision — it is a description, and it belongs
in the unit doc. `unknown — nobody recorded them` is a legitimate single row.

## Decision
One sentence, present tense. What the project does.

## Consequences
What this makes cheap. What it makes expensive. What it forbids outright — that last line is the one an
agent is about to violate.

## What would reverse this
The condition under which this stops being right: a scale number, a dependency dying, a requirement
arriving. **The most useful section in the file**, because it turns a historical note into a trigger
somebody can check.

## Evidence
Where each part came from: a commit sha, a PR, or `asked <name>, <date>`.
````

No `read:` revision and no markers. This document is not describing code at a revision; `Evidence`
carries the coordinates instead.

## `confidence`, and why it is separate from `status`

| Value | Means | Allowed to do |
|---|---|---|
| `cited` | Rationale found in commits, comments or code | Stand as written |
| `reported` | A human said so, and is named with a date | Stand as written |

There is no third value. A rationale reconstructed from code shape is a hypothesis: keep it as a
numbered open question until a human confirms it, then record who and when.

## Rules

- **Never invent an option.** The rejected alternatives are the entire value of the file and the one
  thing you cannot derive. Guess them and the file becomes a liability that reads like an asset.
- **Supersede, never edit.** New decision, new numbered file, the old one flips to `superseded by
  <file>`. The record of having changed your mind is worth more than either decision alone.
- **Numbers are permanent.** Unit docs cite `decisions/07`. Renumbering breaks every one of them.
- **One decision per file.** A file covering three gets cited for all three and superseded for none.
- **Harvest the ledger.** An answered open question that shaped code may be an ADR when the answer
  includes the rationale. An empty `decisions/` folder is correct when no reliable rationale survives.
