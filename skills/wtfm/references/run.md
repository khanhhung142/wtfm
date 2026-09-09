# run — the loop's detail

SKILL.md holds the stage order, the dispatch prompt, the four fan-out rules and the resume table.
This file holds what did not fit there: the goal template, wave sizing, and what to say at each gate.

## `_goal.md`

Written at gate 1, owned by the human, read at the start of every wave. It is what makes the loop
terminable. Without a written definition of done, a documentation run has no natural end and keeps
finding things to document until the budget is gone.

````markdown
# Goal

## Objective
One or two lines. What this manual is for, and who reads it.

## In scope
Units, flows and topics. Copy from `_scout.md` and cut.

## Out of scope
Named explicitly, with the reason: deprecated, third-party, generated, about to be deleted. A unit
merely left off the in-scope list gets documented by a later wave that reads the omission as an
oversight.

## Depth
`sketch` — index and use-case table only, for units nobody will touch this quarter.
`standard` — the full file set.
`deep` — plus flows and an explainer.
Set per unit where they differ. Most projects have three units that matter and nine that do not.

## Done when
Checkable conditions, not adjectives.
- [ ] Every in-scope unit has an index and a use-case table
- [ ] Every in-scope flow traces end to end with no unverified hop
- [ ] `verify` reports zero Wrong findings
- [ ] `AGENTS.md` points at the manual

## Stop
What ends the run rather than continuing to the next wave: a budget, a date, a wave count.
````

This file is the steering wheel. The human edits it between waves to drop a unit, downgrade one to
`sketch`, or stop before flows, and the next wave picks the change up without a session being
interrupted or the project being re-explained.

## Wave sizes

Three to five subagents per wave. Past that, review quality drops: you are the only thing checking
their work, and reviewing eight reports properly costs more context than reading five.

The first `map` is deliberately alone. It is the template, and every flaw in it is about to be
repeated across the rest.

## What to say at the gates

Do not infer approval from silence. Stop and wait.

**Gate 1, after scout.** Present: units found, branch per unit with the evidence, truth sources,
proposed wave order, target count. Ask for the four things code cannot answer.

```
Before I start: which of these units are dead or about to be deleted, which branch really ships
for each, are any of these two names the same concept, and what should the manual be for?
```

Turn the answers into `_goal.md` before bootstrapping.

**Gate 2, after the first unit.** Present the docs as a template, not as content.

```
Review these as a template rather than for correctness: what is missing that every future doc will
also miss, and what is noise that is about to be repeated across N more units?
```

Apply the answer to the mode reference, then continue. A gate answered by guessing produces sixty
documents built on an unreviewed template.

## Unattended running

To keep going without anyone present, drive it on an interval and let each tick take one wave. The
tick prompt is constant, since the ledger holds the position:

```
/wtfm run
```

Bound it. Stop when the ledger has no unstarted target, and stop *at* a gate rather than guessing
what the human would have said.

## Reporting

After each wave, one short report: targets banked, status tag counts, new open questions by number,
what goes out next. On a resume, also say how many `🔄` rows you found and what you did with each. A
run that silently re-dispatches four targets looks identical to one that silently skipped them.
