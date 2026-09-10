# run — the loop's detail

SKILL.md holds the stage order, the dispatch prompt, the four fan-out rules and the resume table.
This file holds the goal template, wave sizing, and what to say at each gate.

## `_run/goal.md`

Written at gate 1, owned by the human, read at the start of every wave. It is what makes the loop
terminable: without a written definition of done, a documentation run keeps finding things to document
until the budget is gone.

````markdown
# Goal

## Decisions
Settled at gate 1, on <date>. Change a row and the next wave follows it.

| # | Decision | Answer |
|---|----------|--------|
| 1 | Dead units | |
| 2 | Stable branch per unit, and who switches to it | |
| 3 | Vocabulary conflicts resolved | |
| 4 | Manual location, and whether commits are authorised | |
| 5 | Depth per unit | |
| 6 | Audience | |
| 7 | Agent file patched; citation lint installed or declined | |
| 8 | May local traps be written as code comments | |

## Objective
One or two lines. What this manual is for, and who reads it.

## In scope
Units, flows, topics. Copy from the scout report and cut.

## Out of scope
Named explicitly, with the reason: deprecated, third-party, generated, about to be deleted. A unit
merely left off the in-scope list gets documented by a later wave reading the omission as an oversight.

## Depth
`sketch` — index only, for units nobody will touch this quarter.
`standard` — index, plus a second file only where a recurring question needs one.
`deep` — plus flows, and an explainer if a human reads this.
Set per unit. Most projects have three units that matter and nine that do not.

## Done when
- [ ] `system.md` names the stack, layers, conventions and authored sources, each cited
- [ ] `glossary.md` covers every term whose meaning is not obvious from the word, with provenance
- [ ] Every in-scope unit has an index; extra files exist only where a question needed one
- [ ] Every in-scope flow traces end to end with no unverified hop
- [ ] Every confirmed decision with reliable rationale has a `decisions/` file
- [ ] **No doc contains a claim that an authored file already makes**
- [ ] **No doc contains a sentence that could go false without a file being renamed**
- [ ] Graduation done: authored sources and vocabulary copied out of `_run/`; nothing routes there
- [ ] `AGENTS.md` points at the manual and states the authority model
- [ ] `verify` reports zero Wrong findings

## Stop
What ends the run rather than continuing: a budget, a date, a wave count.
````

The two bolded conditions are the ones that decide whether this manual is still worth reading next
year. The others decide whether it was finished.

This file is the steering wheel. The human edits it between waves to drop a unit, downgrade one to
`sketch`, or stop before flows, and the next wave picks the change up without a session being
interrupted or the project re-explained.

## Wave sizes

Three to five subagents per wave. Past that, review quality drops: you are the only thing checking
their work, and reviewing eight reports properly costs more context than reading five.

`system` runs alone and first: one session for the whole repo, and every subagent after it is handed
the conventions instead of inventing its own.

The first `map` runs alone next. It is the template, and every flaw in it is about to be repeated.

`decide` runs alone and last, because the open-questions table it harvests is not full until the rest
of the run has filled it.

**Graduation runs after `decide` and before `verify`**, and it is not optional. The `_run/` files are
scaffolding: useful while building, misleading once left standing. Copy the authored-source map into
`system.md` and surviving vocabulary into `glossary.md`, then confirm nothing outside `_run/` links
into it.

## What to say at the gates

Do not infer approval from silence. Stop and wait.

**Gate 1, after scout.** Eight decisions, asked at once, in one message, each with your recommendation
already filled in. A gate that asks open questions gets a shrug; a gate that proposes answers gets
edits.

| # | Decision | Propose | They alone know |
|---|----------|---------|-----------------|
| 1 | Which units are dead | No recent commits, no inbound imports | Whether it still runs in production |
| 2 | Stable branch per unit, and who switches | The one with more of the files that matter, counts shown, no checkout run | Which one actually deploys, and what is uncommitted |
| 3 | Vocabulary conflicts | Every one name / two concepts, and two names / one concept | Which meaning owns the word |
| 4 | Manual location, and may you `git commit` | In-repo for one repo; sibling plus symlinks for several | Whether docs may be committed at all |
| 5 | Depth per unit | Full on the request path, sketch elsewhere | Which units they are about to work in |
| 6 | Who reads it | Agents only unless they say otherwise; skip `explain/` if so | Whether anyone is being onboarded |
| 7 | Agent file, and the citation lint | The file that exists, extended not replaced; lint offered, never installed unasked | Whether it is shared with a team, and what may touch their build |
| 8 | May a local trap be a code comment | Yes — comments only, one separate diff | Whether source edits from a docs run are acceptable |

Present the evidence, not just the proposal. "develop has 21 schema files, main has 1" lets them
correct you in four words. "I will read develop" does not.

**Decision 8 is the one that decides whether this manual is still true in a year.** A comment beside
the ORM hook is reviewed with the hook and moves with it; the same sentence in a doc is reviewed by
nobody. Say that, and say plainly it means editing their source — comments only, in a diff they can
drop whole. Refused, traps go in the unit docs and the run continues.

**Decision 2 is the only one where getting on with it costs them something.** Phrase it as a branch
question plus a switching offer — *"develop looks stable: 21 schema files against main's 1. Switch to
it yourself, or say the word and I will run `git switch develop`?"* — and until they pick, read
whatever is checked out. Never run the checkout on the strength of your own recommendation.

Write every answer into `_run/goal.md` before bootstrapping, including the uncontested ones. A later
session must be able to see that reading `develop` was a decision somebody made, not a habit that
crept in.

**Gate 2, after the first unit.** Present the docs as a template, not as content. Ask three things:

```
Review this as a template, not for correctness:
1. What is missing that every future doc will also miss?
2. What is noise that is about to be repeated across N more units?
3. Which lines here will be false in six months?
```

Question 3 is the important one and this is the last cheap moment to ask it. Anything they name gets
cut, or moved into the code as a comment, before wave three — not verified later. A run that ships
sixty docs and a verification schedule has failed; the goal is sixty docs that do not need one.

Apply the answers to the mode reference, then continue.

## Unattended running

Drive it on an interval, one wave per tick. The tick prompt is constant, since the ledger holds the
position:

```
/wtfm run
```

Bound it: stop when the ledger has no unstarted target, and stop *at* a gate rather than guessing what
the human would have said.

## Reporting

After each wave: targets banked, source files commented, new open questions by number, what goes out
next. On a resume, also say how many `🔄` rows you found and what you did with each — a run that
silently re-dispatches four targets looks identical to one that silently skipped them.
