# Mode: run — drive the whole atlas to completion

The autopilot. Reads the ledger, picks the next target, dispatches it to a fresh subagent, banks the
result, repeats. The human types one command and answers two gates.

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

## Wave sizes

Between three and five subagents per wave. Beyond that, review quality drops: you are the only thing
checking their work, and reviewing eight reports properly costs more context than reading five.

Between waves, bank the results and update `_progress.md` before dispatching the next. A run that
crashes mid-wave loses that wave. A run that crashes between waves loses nothing.

## Dispatching a target

One subagent, one target. The prompt carries no project knowledge, because `_scout.md` holds it and
the subagent reads it first. Keep the prompt to the target and the boundary:

```
Use the ariadne skill, mode `map`, target `<unit>`.

Read <atlas>/_scout.md first for the truth-source table and the branch decision, then
<atlas>/_progress.md for what already exists.

Write ONLY inside <atlas>/services/<unit>/. Do not touch the root index.md, do not touch
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
the atlas is that its citations hold. A failed spot-check means that target goes back out, not that
you fix it yourself.

## The two human gates

Stop and wait. Do not infer approval from silence.

**Gate 1, after `scout`:** present the plan — units found, branch per unit, truth sources, proposed
wave order, target count. The human corrects what only they know: which service matters, which is
dead, which branch really ships, what the vocabulary means.

**Gate 2, after the first `map`:** present the first unit's docs as a *template*, not as content.
Ask the reviewer what is missing that every future doc will also miss, and what is noise that is
about to be repeated sixty times. Apply the answer to the mode reference before wave two.

## Resuming

A run is resumable because the ledger holds the state. On any later invocation, `run` reads
`_progress.md`, finds the first unstarted target in the earliest incomplete wave, and continues.
There is no separate resume command, and no run-state file beyond the ledger.

If the ledger and the filesystem disagree — a folder exists but the ledger calls it unstarted — the
filesystem wins for existence and the ledger wins for completeness. Treat the folder as a partial
target: dispatch it with instructions to extend, not to rewrite.

## Unattended running

For a run that should keep going without anyone present, drive it on an interval and let each tick
take one wave. The tick prompt is the same every time, since the ledger holds the position:

```
/ariadne run
```

Bound it: stop the loop when the ledger has no unstarted target, and stop it at a gate rather than
guessing what the human would have said. A loop that answers its own gates produces sixty documents
built on an unreviewed template.
