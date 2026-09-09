---
name: wtfm
description: Write The Fine Manual. Read an unfamiliar codebase in stages and write documentation another agent can navigate — chunked, indexed, every claim carrying a file:line citation — plus diagrams, plain-language explainers for humans, and an AGENTS.md that makes future agents read it first. Runs itself from a goal and a ledger, fanning work out across subagents, and resumes where it stopped when a session dies. Use when landing in a new or undocumented project, documenting a repo or service, tracing a cross-service flow, explaining how a system works, drawing architecture or sequence diagrams, setting up a docs vault, resuming a half-finished documentation run, or checking whether existing docs still match the code. Trigger words: onboard, document this project, write docs for agents, code map, architecture diagram, explain this system, docs vault, docs-first, RTFM.
---

# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM. This skill writes it: many small files, each answering one
question, reachable through a router index, every claim carrying a `file:line` citation.

The reader is an agent that arrives next week with no context and a budget of three file reads.
Optimise for it. Humans are served by `explain`, a different artifact with different rules.

## First move

Find the row that matches what was asked. Do that, and nothing else.

| They asked for | Do | Read |
|---|---|---|
| Anything broad: document this, onboard me, write docs | `run` | keep reading this file |
| One repo, service or package documented | `map <unit>` | [map.md](references/map.md) |
| One request traced across units | `flow <name>` | [flow.md](references/flow.md) |
| How something works, explained to a person | `explain <topic>` | [explain.md](references/explain.md) |
| Whether the docs still match the code | `verify` | [verify.md](references/verify.md) |
| To change the objective or scope of a run | edit `_goal.md` | [run.md](references/run.md) |

Every one of these needs `_scout.md` to exist. If it does not, run the scout stage first, whatever
was asked for. It is one session and everything else is guesswork without it.

**Read only the reference for what you are doing.** Loading all of them wastes the context the
actual work needs.

## Running `run`

`run` is the whole job. It performs the stages below in order, and the human answers two gates.
Do not do the documenting yourself here: your context is the only thing that spans every wave, and
spending it reading a repo kills the run halfway. **You dispatch, review and bank.**

1. **Scout.** One session, read-only, writes `_scout.md` and nothing else.
   → [scout.md](references/scout.md)
2. **Gate 1.** Present the plan. Stop and wait for a human. They correct what only they know: which
   service is dead, which branch really ships, what the acronym means.
3. **Bootstrap.** Create the skeleton, seed `_goal.md` and `_progress.md`, wire in `AGENTS.md`.
   → [bootstrap.md](references/bootstrap.md)
4. **First unit.** `map` exactly one, alone. → [map.md](references/map.md)
5. **Gate 2.** Present it as a *template*, not as content. Ask what is missing that every future doc
   will also miss, and what is noise about to be repeated sixty times. Apply the answer before wave
   three, because this is the last cheap moment to fix it.
6. **Remaining units**, three to five per wave, in parallel.
7. **Flows**, in parallel, once every unit a flow crosses is mapped.
8. **Explainers**, then **verify** as a single final pass.

Stop when `_goal.md` says done. Report what was written and what is still open.

### Dispatching

One subagent per target. The prompt carries no project knowledge, because `_scout.md` holds it:

```
Use the wtfm skill, mode `map`, target `<unit>`.

Read <manual>/_scout.md first for the truth-source table and the branch decision, then
<manual>/_progress.md for what already exists.

Write ONLY inside <manual>/services/<unit>/. Do not touch the root index.md, do not touch
_progress.md, do not touch another unit's folder.

Return: files written, status tag counts, open questions, and the branch and commit you read.
```

Four rules stop a fan-out corrupting the manual:

1. **A subagent writes only inside its own target's folder.** Never the root index, never the ledger.
2. **A target is claimed `🔄` in the ledger before dispatch.** A claim written afterwards does not
   survive the crash it exists for.
3. **Bank each report the moment it lands**, not at the end of the wave. Whatever is unbanked when
   the session dies is lost.
4. **Spot-check one citation per report** before banking. A subagent that fabricated one fabricated
   others, and citations holding is the whole point.

Banking a report means: add the unit's row to the root `index.md`, flip its ledger cells, merge its
open questions into the numbered table, rewrite `## Now`. Then commit the manual if it is in git.
One commit per wave is a restore point.

### The three files

| File | Holds | Owner | Changes |
|---|---|---|---|
| `_goal.md` | Objective, scope, depth, definition of done | the human | rarely |
| `_scout.md` | Units, branches, truth sources, vocabulary | scout | once |
| `_progress.md` | Where the work got to. The ledger | the run | every target |

The goal stays out of the ledger so the human can steer between waves without interrupting a
session. Folded in, it would be overwritten by every status update.

### When a session dies

Context fills, credit runs out, a laptop closes. Resuming is the same command: `run` reads
`_goal.md` and `_progress.md`, finds the earliest incomplete wave, continues. No resume verb and no
run-state file, because a second state file is a second thing that can disagree with reality.

**A `🔄` row is a crash survivor, not work in progress.** Nothing is still running. Resolve each
against the filesystem before dispatching anything new:

| On disk | Do |
|---|---|
| Folder missing | Dispatch fresh |
| Folder partial | Dispatch to extend, not rewrite |
| Folder looks complete | Spot-check one citation, then bank |

Never promote a `🔄` row to done because its folder exists. It was never reviewed.

## Rules every stage obeys

**Cite or omit.** Every technical claim names `path/file.ext:LINE`, repo-relative. A claim you could
not trace to a line goes in `## Open questions`, not in the doc. This is what makes the manual worth
more than the next model's guess: a citation is confirmed in one read, where an uncited paragraph
costs exactly what it cost before the doc existed.

**Document the authored file, not what it generates.** Generated output restates the same fact one
step later and goes stale silently. `_scout.md` lists which is which for this project.

**Tag every section** `✅ implemented` (cited), `🟡 partial` (name what is missing), or `📋 spec-only`
(no code yet). A doc with no tags claims everything works, which is a claim nobody verified.
Describing a stub as though it ran is the most common documentation failure in a live codebase.

**Prove which branch is true before reading a line.** The default branch is often not where work
lands. Compare candidates by counting the files that carry meaning — schema directory, interface
definitions, route table — and record branch and commit in every doc's frontmatter. Units may be
true on different branches; say so per unit.

```bash
git -C <repo> fetch --all
git -C <repo> branch -r --sort=-committerdate | head
```

**Spec contradicting code is a finding.** Record both and name the contradiction. Never silently
pick whichever makes a tidier doc. That sentence is usually the most valuable one in the manual.

**One fact, one place.** A table lives in one file; everything else links to it. Links run one
direction, so a reader always knows which way to walk:

```
features  →  services  →  flows  →  code
```

**One file, one question**, target 400 lines. **`index.md` is a router, never content**: a table of
need → path. **The root index stays under 200 lines**, because every agent pays it every session.

**Frontmatter on every doc**, so `verify` can tell stale from wrong:

```yaml
---
unit: <repo or package name>
branch: <branch>
commit: <sha>
written: <YYYY-MM-DD>
status: ✅ | 🟡 | 📋
---
```

**The ledger is the handoff.** Read `_progress.md` first, write it last, every session without
exception. One session documents one target; "document the backend" is a table of sessions. An
unwritten ledger row costs the next session a full re-discovery pass.

**Documentation is crystallised thinking, not a place to think.** Work the code out first, then
write. A doc that reasons on the page is a draft that escaped.

## Every stage, same shape

1. Read `_scout.md` for truth sources and the branch decision, `_progress.md` for what exists.
2. Check out the true branch. Record branch and commit.
3. Read written specs first where they exist, since they name intent the code may have missed.
   Then read the code.
4. Write the doc. Update the router index and the ledger.
5. Report files written, status tag counts, open questions. Claim a target is documented only for
   the code actually read.

Diagram rules are shared by `map`, `flow` and `explain`: [diagrams.md](references/diagrams.md).
