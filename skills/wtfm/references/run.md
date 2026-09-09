# Mode: run — drive the whole manual to completion

The autopilot. Reads the goal and the ledger, picks the next target, dispatches it to a fresh
subagent, banks the result, repeats until the goal is met. The human types one command and answers
two gates.

Do not do the documenting yourself in this mode. Your context is the only thing that survives all
the waves; spending it reading a repo means the run dies halfway. **You dispatch, review and bank.**

## The dependency graph

Waves run in order. Inside a wave, targets are independent and go out in parallel.

```
scout  ──▶  [human gate]  ──▶  bootstrap  ──▶  map unit#1  ──▶  [human gate]
                                                                     │
                              ┌──────────────────────────────────────┘
                              ▼
                     map units #2..N   (parallel)
                              │
                              ▼
                     flows              (parallel; needs every unit it crosses mapped)
                              │
                              ▼
                     explain topics     (parallel)
                              │
                              ▼
                     verify             (single pass, last)
```

`map unit#1` is deliberately alone. The first unit is the template. Every flaw in it is about to be
repeated across the rest, and it is the last cheap moment to fix one.

## `_goal.md`

Written at the scout gate, owned by the human, read at the start of every wave. It is what makes the
loop terminable: without a written definition of done, a documentation run has no natural end and
will keep finding things to document until the budget is gone.

````markdown
# Goal

## Objective
One or two lines. What this manual is for, and who reads it.

## In scope
Units, flows and topics to document. Copy from `_scout.md` and cut.

## Out of scope
Named explicitly. A unit left off the in-scope list gets documented by a future wave that assumes
the omission was an oversight, so say which ones are deliberate and why: deprecated, third-party,
generated, about to be deleted.

## Done when
Checkable conditions, not adjectives.
- [ ] Every in-scope unit has an index and a use-case table
- [ ] Every in-scope flow traces end to end with no unverified hop
- [ ] `verify` reports zero Wrong findings
- [ ] `AGENTS.md` points at the manual

## Depth
`sketch` — index and use-case table only, for units nobody will touch this quarter.
`standard` — the full file set.
`deep` — plus flows and an explainer.
Per unit where they differ, since most projects have three units that matter and nine that do not.

## Stop
Conditions that end the run rather than continuing to the next wave: budget, a date, or a wave count.
````

The human edits this file between waves. That is the steering wheel: drop a unit, downgrade a unit
to `sketch`, stop before flows. The run picks up the change on its next wave without anyone having
to interrupt a session or re-explain the project.

## Wave sizes

Between three and five subagents per wave. Beyond that, review quality drops: you are the only thing
checking their work, and reviewing eight reports properly costs more context than reading five.

Bank each report the moment it lands rather than at the end of the wave, and claim each target in
the ledger before dispatching it. Those two habits are what bound the damage when a session dies:
the loss is whatever was in flight, not the whole wave.

Commit the manual at the end of each wave when it is in a git repo. One commit per wave is a restore
point, and it makes the run auditable after the fact.

## Dispatching a target

One subagent, one target. The prompt carries no project knowledge, because `_scout.md` holds it and
the subagent reads it first. Keep the prompt to the target and the boundary:

```
Use the wtfm skill, mode `map`, target `<unit>`.

Read <manual>/_scout.md first for the truth-source table and the branch decision, then
<manual>/_progress.md for what already exists.

Write ONLY inside <manual>/services/<unit>/. Do not touch the root index.md, do not touch
_progress.md, do not touch another unit's folder.

Return: files written, status tag counts (✅/🟡/📋), open questions, and the branch and
commit you read. Nothing else.
```

Substitute `map`/`flow`/`explain` and the folder boundary per mode. The boundary paragraph is not
boilerplate: two subagents editing one file lose each other's work, and the ledger is the file they
would all want to edit.

## Banking a wave

For each report, in the parent:

1. Add the unit's row to the root `index.md` registry, one line, linking to the unit index.
2. Update the unit's status cells in `_progress.md`, and add one session-log row per target.
3. Merge cross-cutting open questions into the ledger's open-question table. Number them, so later
   docs can cite `#12` instead of restating the problem.
4. Rewrite `## Now` to two lines: what just landed, what goes out next.

**Spot-check one citation per report** before banking it. Open the file, confirm the line says what
the doc claims. A subagent that fabricates one citation fabricated others, and the whole point of
the manual is that its citations hold. A failed spot-check means that target goes back out, not that
you fix it yourself.

## The two human gates

Stop and wait. Do not infer approval from silence.

**Gate 1, after `scout`:** present the plan — units found, branch per unit, truth sources, proposed
wave order, target count. The human corrects what only they know: which service matters, which is
dead, which branch really ships, what the vocabulary means.

**Gate 2, after the first `map`:** present the first unit's docs as a *template*, not as content.
Ask the reviewer what is missing that every future doc will also miss, and what is noise that is
about to be repeated sixty times. Apply the answer to the mode reference before wave two.

## Resuming after a dead session

Sessions end mid-run: context fills, credit runs out, a laptop closes. Nothing special happens when
they do, which is the point. The next invocation reads the same three files and carries on.

**Resuming is the same command.** `wtfm run` reads `_goal.md` and `_progress.md`, finds the earliest
incomplete wave, and continues. No resume verb, no run-state file beyond the ledger. A second state
file is just a second thing that can disagree with reality.

**Every `🔄` row is a crash survivor**, never work in progress: nothing is still running. Resolve
each against the filesystem before dispatching anything new.

| On disk | Meaning | Do |
|---|---|---|
| Folder missing | Died before writing | Dispatch fresh |
| Folder partial | Died mid-write | Dispatch to extend, not rewrite |
| Folder looks complete | Died after writing, before banking | Spot-check one citation, then bank |

Never promote a `🔄` row to done because its folder exists. It was never reviewed, and an unreviewed
target that looks finished is precisely what the status tags exist to catch.

Report on resume what you found: how many rows were `🔄`, and what you did with each. A run that
silently re-dispatches four targets looks identical to a run that silently skipped them.

## Unattended running

For a run that should keep going without anyone present, drive it on an interval and let each tick
take one wave. The tick prompt is the same every time, since the ledger holds the position:

```
/wtfm run
```

Bound it: stop the loop when the ledger has no unstarted target, and stop it at a gate rather than
guessing what the human would have said. A loop that answers its own gates produces sixty documents
built on an unreviewed template.
