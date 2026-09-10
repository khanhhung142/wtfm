---
name: wtfm
description: "Write The Fine Manual. Read an unfamiliar codebase in stages and write documentation another agent can navigate — chunked, indexed, every claim carrying a file:line citation — plus diagrams, plain-language explainers for humans, and an AGENTS.md that makes future agents read it first. Runs itself from a goal and a ledger, fanning work out across subagents, and resumes where it stopped when a session dies. Use when landing in a new or undocumented project, documenting a repo or service, tracing a cross-service flow, explaining how a system works, drawing architecture or sequence diagrams, setting up a docs vault, recording an architecture decision, building a domain glossary, resuming a half-finished documentation run, or checking whether existing docs still match the code. Trigger words: onboard, document this project, write docs for agents, code map, architecture diagram, explain this system, tech stack, coding conventions, layer architecture, docs vault, docs-first, ADR, architecture decision record, why did we build it this way, glossary, domain vocabulary, ubiquitous language, RTFM."
---

# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM. This skill writes it: many small files, each answering one
question, reachable through a router index, every claim carrying a `file:line` citation.

The reader is an agent that arrives next week with no context and a budget of three file reads.
Optimise for it. Humans are served by `explain`, a different artifact with different rules.

**The code is the truth. This manual is the index to it, and the record of why it is that way.**
It exists to get the next agent to the right forty lines in one read instead of forty, and to tell
it the things no file can say — what was considered instead, and what the project's words mean.
A manual that competes with the code for authority is worse than no manual, because now there are
two answers and the reader cannot tell which one shipped.

## First move

Find the row that matches what was asked. Do that, and nothing else.

| They asked for | Do | Read |
|---|---|---|
| Anything broad: document this, onboard me, write docs | `run` | keep reading this file |
| The stack, layers or conventions of the whole repo | `system` | [system.md](references/system.md) |
| One repo, service or package documented | `map <unit>` | [map.md](references/map.md) |
| One request traced across units | `flow <name>` | [flow.md](references/flow.md) |
| How something works, explained to a person | `explain <topic>` | [explain.md](references/explain.md) |
| Why the code is like this, recorded | `decide <topic>` | [decide.md](references/decide.md) |
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
2. **Gate 1.** Stop and wait for a human. Put seven decisions in front of them in one message, each
   with your recommendation and the evidence for it, so they correct rather than compose: dead
   units, stable branch per unit and who switches to it, vocabulary conflicts, where the manual lives, depth per unit, who
   reads it, which agent file to patch. Their answers go into `_goal.md`, which is the decision
   record. → [run.md](references/run.md)
3. **Bootstrap.** Create the skeleton, seed `_goal.md` and `_progress.md`, wire in `AGENTS.md`.
   → [bootstrap.md](references/bootstrap.md)
4. **System.** One session: stack, shape, layers, conventions for the whole repo, plus
   `glossary.md` — what this project's words mean. Everything after it links to both instead of
   restating them. → [system.md](references/system.md)
5. **First unit.** `map` exactly one, alone. → [map.md](references/map.md)
6. **Gate 2.** Present it as a *template*, not as content. Ask what is missing that every future doc
   will also miss, and what is noise about to be repeated sixty times. Apply the answer before wave
   three, because this is the last cheap moment to fix it.
7. **Remaining units**, three to five per wave, in parallel.
8. **Flows**, in parallel, once every unit a flow crosses is mapped.
9. **Explainers**, in parallel.
10. **Decisions.** One session over the answered open questions and the git archaeology, then ask
    the human what is still missing. → [decide.md](references/decide.md)
11. **Verify** as a single final pass.

A run that ends with an empty `decisions/` folder documented the code and threw away the reasoning,
which was the half the code could not state for itself. Stage 10 is not optional padding.

Stop when `_goal.md` says done. Report what was written and what is still open.

### Dispatching

One subagent per target. The prompt carries no project knowledge, because `_scout.md` holds it:

```
Use the wtfm skill, mode `map`, target `<unit>`.

Read <manual>/_scout.md first for the truth-source table and the branch decision,
<manual>/system.md for the layers and conventions this repo already follows, then
<manual>/_progress.md for what already exists. Link to system.md for a convention; never restate it.

Write ONLY inside <manual>/services/<unit>/. Do not touch the root index.md, do not touch
_progress.md, do not touch another unit's folder.

Return: files written, status tag counts, open questions, and the branch and commit you read.
```

**No subagents in this agent?** Run the same targets one at a time, one per session, and stop when
context gets tight. The ledger is what makes that work: it is the handoff either way, and a wave of
one is only slower, not different. Everything below still applies.

Four rules stop a fan-out corrupting the manual:

1. **A subagent writes only inside its own target's folder.** Never the root index, never the ledger.
2. **A target is claimed `🔄` in the ledger before dispatch.** A claim written afterwards does not
   survive the crash it exists for.
3. **Bank each report the moment it lands**, not at the end of the wave. Whatever is unbanked when
   the session dies is lost.
4. **Spot-check one citation per report** before banking. A subagent that fabricated one fabricated
   others, and citations holding is the whole point.

Banking a report means: add the unit's row to the root `index.md`, flip its ledger cells, merge its
open questions into the numbered table, rewrite `## Now`. Then commit the manual if it is in git *and* the human authorised commits.
One commit per wave is a restore point — only if the human authorised commits at gate 1.

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

**Code is the truth. The manual is the index and the why.** Two rules follow from that, and they are
the ones that keep this manual from becoming the thing it was written to replace:

- *Where a doc and the code disagree, the code is right and the doc is a bug.* Never reword the code
  to match the doc, never split the difference, never pick whichever reads better. Fix the doc, or
  file it as a `verify` finding when you are not in a session that may write there.
- *Do not restate what a file already states.* A table you could regenerate with a script is a table
  that will be wrong by Thursday and believed anyway. Point at the authored file and spend the words
  on what reading it will not tell you: which of those rows is a stub, which one is load-bearing,
  which one is a trap.

What survives that test is the manual's actual job: navigation (where is this), constraints (what
must not be broken), vocabulary (what the words mean here), and decisions (why, and what lost).

**Write the slow half.** Every claim has a decay rate, and you choose which ones to make. A doc built
from fast-decaying claims needs verifying forever; one built from slow-decaying claims barely needs
verifying at all. Same unit, same session, different half-life — the difference is what you chose to
write down.

| Claim | Rots when | What it costs you |
|---|---|---|
| A line number | anything above it is edited | a check every commit |
| A behaviour paraphrase | the logic changes | a check every feature |
| A surface list | a route or table is added | a check every feature |
| A symbol or path | a rename or a refactor | a check a quarter |
| An invariant or a trap | the *design* changes | almost never — and then it is a finding worth having |
| A word's meaning | the domain is renamed | almost never |
| A decision | never; it gets superseded | never |

Prefer the bottom of that table. Where the top is genuinely what the reader needs — the surface
really is scattered, the entry point really is a line — write it, but know you have just bought
maintenance, and buy the least precision that still gets the reader there. `internal/http/` is a
better citation than `internal/http/order.go:23 Create` when the point is where handlers live.

**A fact about one file belongs in that file.** A comment on the ORM hook moves with the hook, is
reviewed in the same pull request by the same person, and cannot drift from it — the manual can do
none of those three. So when a trap is local, write the comment and have the doc link to it. Keep
the manual for what no single file can own: what spans files, what the words mean, what was
rejected. That is not a smaller manual by accident. It is a manual with nothing in it that rots
faster than the reason it was written.

**Cite or omit, and cite an anchor with the line.** Every technical claim names
`path/file.ext:LINE Symbol`, repo-relative — `internal/user/register.go:34 Register`. A claim you
could not trace goes in `## Open questions`, not in the doc. This is what makes the manual worth more
than the next model's guess: a citation is confirmed in one read, where an uncited paragraph costs
exactly what it cost before the doc existed.

The anchor is what stops the manual rotting one line at a time. A bare line number is wrong the
moment somebody inserts twelve lines above it — every citation in the file breaks at once, and none
of them look broken. A symbol survives that, breaks only on a rename, and lets `verify` repair the
number with one `grep`. Use the nearest named thing: a function, type, table, route constant, config
key. Where nothing is named — a migration body, a YAML block — anchor on the literal that identifies
it, and say so.

**Document the authored file, not what it generates.** Generated output restates the same fact one
step later and goes stale silently. `_scout.md` lists which is which for this project.

**Tag every section** `✅ implemented` (cited), `🟡 partial` (name what is missing), or `📋 spec-only`
(no code yet). A doc with no tags claims everything works, which is a claim nobody verified.
Describing a stub as though it ran is the most common documentation failure in a live codebase.

**Never mutate the human's git.** Read-only commands only: `git log`, `git branch`, `git show`,
`git diff`, `git status`, `git ls-tree`. No `fetch`, `checkout`, `switch`, `pull`, `stash`, `merge`,
`commit` — nothing that writes — unless the human said yes to that exact command first. They may
have uncommitted work in the tree, a dev server running against it, or a checkout somebody else
depends on. A wrong doc costs one re-read; a clobbered working tree costs their afternoon.

**Ask which branch is stable. Never switch to it yourself.** The default branch is often not where
work lands, so gather the evidence read-only — count the files that carry meaning (schema
directory, interface definitions, route table) per candidate — then put the counts to the human and
ask which branch is stable, per unit. Offer both ways out in the same message: they switch, or they
authorise you to switch for them. Until they answer, document whatever branch is already checked
out and say in the doc that this is what happened. Units may be stable on different branches; say so
per unit. Record branch and commit in every doc's frontmatter.

```bash
git -C <repo> branch -a --sort=-committerdate | head          # no fetch
git -C <repo> ls-tree -r --name-only <candidate> -- <schema dir> | wc -l
```

**Spec contradicting code is a finding.** Record both and name the contradiction. Never silently
pick whichever makes a tidier doc. That sentence is usually the most valuable one in the manual.

**One fact, one place.** A table lives in one file; everything else links to it. Links run one
direction, so a reader always knows which way to walk:

```
system  →  services  →  flows  →  code
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
2. Work on the branch the human named in `_goal.md`. Do not check it out yourself — if the tree is
   on another branch, stop and ask. Record branch and commit.
3. Read written specs first where they exist, since they name intent the code may have missed.
   Then read the code.
4. Write the doc. Update the router index and the ledger.
5. Report files written, status tag counts, open questions. Claim a target is documented only for
   the code actually read.

Diagram rules are shared by `map`, `flow` and `explain`: [diagrams.md](references/diagrams.md).
