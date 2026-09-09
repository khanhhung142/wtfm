# run — the loop's detail

SKILL.md holds the stage order, the dispatch prompt, the four fan-out rules and the resume table.
This file holds what did not fit there: the goal template, wave sizing, and what to say at each gate.

## `_goal.md`

Written at gate 1, owned by the human, read at the start of every wave. It is what makes the loop
terminable. Without a written definition of done, a documentation run has no natural end and keeps
finding things to document until the budget is gone.

````markdown
# Goal

## Decisions
Settled at gate 1, on <date>. Change a row and the next wave follows it.

| # | Decision | Answer |
|---|----------|--------|
| 1 | Dead units | |
| 2 | True branch, per unit | |
| 3 | Vocabulary conflicts resolved | |
| 4 | Manual location | |
| 5 | Depth per unit | |
| 6 | Audience | |
| 7 | Agent file patched | |

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

**Gate 1, after scout.** Seven decisions, and they are the whole gate. Ask all seven at once, in
one message, each with your recommendation already filled in so the human corrects rather than
composes. A gate that asks open questions gets a shrug; a gate that proposes answers gets edits.

| # | Decision | Propose | They alone know |
|---|----------|---------|-----------------|
| 1 | Which units are dead | Anything with no recent commits and no inbound imports | Whether it still runs in production |
| 2 | Which branch is true, per unit | The one with more of the files that matter, with the counts shown | Which one actually deploys |
| 3 | Vocabulary conflicts | Every case of one name covering two concepts, or two names covering one | Which meaning owns the word |
| 4 | Where the manual lives | In-repo for one repo; a sibling folder plus symlinks for several | Whether the docs may be committed |
| 5 | Depth per unit | Full for units on the request path, sketch for the rest | Which units they are about to work in |
| 6 | Who reads it | Agents only unless they say otherwise | Whether anyone is being onboarded |
| 7 | Which agent file to patch | The one that exists, extended not replaced | Whether it is shared with a team |

Present the evidence for each, not just the proposal. "develop has 21 schema files, main has 1" lets
them correct you in four words. "I will read develop" does not.

Write every answer into `_goal.md` before bootstrapping, including the ones they did not contest.
That file is the decision record: a later session must be able to see that reading `develop` was a
decision somebody made, not a habit that crept in.

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
