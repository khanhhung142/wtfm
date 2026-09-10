---
name: wtfm
description: "Write The Fine Manual. Write the half of the documentation that code cannot write about itself, and refuse the half that goes stale: addresses instead of paraphrases, cross-file invariants instead of behaviour, a domain glossary, and numbered decision records. Local traps are written as code comments so they cannot drift. Runs itself from a goal and a ledger, fans work out across subagents, and resumes where it stopped when a session dies. Use when landing in a new or undocumented project, documenting a repo or service, tracing a cross-service flow, explaining how a system works, drawing architecture or sequence diagrams, setting up a docs vault, recording an architecture decision, building a domain glossary, resuming a half-finished documentation run, fixing docs that keep going out of date, or checking whether existing docs still match the code. Trigger words: onboard, document this project, write docs for agents, docs are outdated, stale docs, two sources of truth, code map, architecture diagram, explain this system, tech stack, coding conventions, docs vault, docs-first, ADR, architecture decision record, why did we build it this way, glossary, ubiquitous language, RTFM."
---

# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM. This skill writes the half of the documentation that code
cannot write about itself, and refuses to write the other half.

The reader is an agent that arrives next week with no context and a budget of three file reads. It
does not need the code explained to it. It needs to know where to look, what it must not break, what
the words mean here, and what was already tried and rejected. Humans get `explain`, a different
artifact with different rules.

## The one rule

**A manual that cannot contradict the code never has to be reconciled with it.**

Two sources of truth is not a problem to solve with review discipline. It is content to not write.
Each ban below deletes a sentence that could go false on its own:

| Never write | Because | Instead |
|---|---|---|
| A table a script could regenerate | Wrong by Thursday, believed anyway | Cite the authored file |
| A payload, a field list, a schema | The schema restated one step later | Cite the schema |
| A paraphrase of what a function does | Rots on the next refactor | Cite the function |
| A step-by-step of code you just read | That is the code, transcribed | Cite the entry point |
| A count of endpoints, tables or keys | A command derives it | Give the command |
| A trap that lives in one file | A comment beside it cannot drift; a doc can | Write the comment, link it |
| Anything about what is deployed | You read a revision, not a server | Name the revision you read |

**The test for every line you keep: could this become false without any file being renamed or
moved?** If yes it is a behaviour claim — cut it, or push it into the code as a comment or a test.

What survives is durable by construction: addresses, cross-file invariants, vocabulary, decisions.
Those change when the *design* changes, and a design change is a finding worth having. Read deeply
and write little: if a finished doc looks thin, that is this rule working, not a gap.

**Authority follows the question.** Code at the read revision is evidence of implementation; an
approved spec states intent; an ADR preserves rationale at a moment; the glossary records domain
language; an index only routes. Where implementation prose and code disagree, the prose is wrong.
Where an approved spec and code disagree, that is drift: record both, change neither, never average
them.

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
| To change the objective or scope of a run | edit `_run/goal.md` | [run.md](references/run.md) |

Every row needs `_run/scout.md` to exist. If it does not, run the scout stage first, whatever was
asked for: one session, and everything else is guesswork without it.

**Read only the reference for what you are doing.** Loading all of them wastes the context the work
needs.

## Running `run`

`run` is the whole job. Do not do the documenting yourself: your context is the only thing that
spans every wave, and spending it reading a repo kills the run halfway. **You dispatch, review, bank.**

1. **Scout.** One session, read-only, writes `_run/scout.md` and nothing else.
   → [scout.md](references/scout.md)
2. **Gate 1.** Stop and wait for a human. Eight decisions in one message, each with your
   recommendation and its evidence, so they correct rather than compose. Answers go into
   `_run/goal.md`. → [run.md](references/run.md)
3. **Bootstrap.** Skeleton, `_run/goal.md`, `_run/progress.md`, the `AGENTS.md` patch.
   → [bootstrap.md](references/bootstrap.md)
4. **System.** One session: stack, layers, conventions, authored sources, `glossary.md`, and
   `specs/index.md` for explicitly approved intent. Everything after it links here instead of
   restating. → [system.md](references/system.md)
5. **First unit.** `map` exactly one, alone. → [map.md](references/map.md)
6. **Gate 2.** Present it as a *template*, not as content. → [run.md](references/run.md)
7. **Remaining units**, three to five per wave, in parallel.
8. **Flows**, in parallel, once every unit a flow crosses is mapped.
9. **Explainers**, in parallel, only if a human reads this manual.
10. **Decisions, if evidenced.** One session over answered open questions and git archaeology. Write
    only what history or a human supports. → [decide.md](references/decide.md)
11. **Graduate.** Copy the authored-source map into `system.md` and surviving vocabulary into
    `glossary.md`, then stop pointing at `_run/`. It is a finished log, not a description of the
    codebase, and a reader sent there next year gets a wave number presented as current state.
12. **Verify** once, as the closing check.

An empty `decisions/` folder is valid when no rationale survived. Fabricating a reason is worse than
recording that it is unknown. Stop when `_run/goal.md` says done; report what was written and what
is still open.

### Dispatching

One subagent per target. The prompt carries no project knowledge, because the scout report holds it:

```
Use the wtfm skill, mode `map`, target `<unit>`.

Read <manual>/_run/scout.md for the authored-source map and the branch decision, then
<manual>/system.md for the layers and conventions this repo already follows, then
<manual>/_run/progress.md for what exists. Link to system.md for a convention; never restate it.

Write ONLY inside <manual>/services/<unit>/. Do not touch the root index.md, _run/, or another
unit's folder. Code comments for local traps: <granted, one separate diff | refused>.

Return: files written, source files you commented, open questions, and the revision you read.
```

Four rules stop a fan-out corrupting the manual:

1. **A subagent writes only inside its own target's folder.** Never the root index, never `_run/`.
2. **A target is claimed `🔄` before dispatch.** A claim written afterwards does not survive the
   crash it exists for.
3. **Bank each report the moment it lands.** Whatever is unbanked when the session dies is lost.
4. **Spot-check one navigation claim per report** before banking. One fabricated path means inspect
   the rest.

Banking: add the unit's row to the root `index.md`, flip its ledger cells, merge its open questions
into the numbered table, rewrite `## Now`. Commit per wave only if the human authorised commits.

**No subagents available?** Same targets, one per session. The ledger is the handoff either way; a
wave of one is slower, not different.

### The three run files

They live in `_run/` so the root of the manual holds only durable artifacts. That is structure doing
the work a rule would otherwise have to: nothing in `_run/` can be mistaken for a description of the
codebase, because of where it sits.

| File | Holds | Owner | Changes |
|---|---|---|---|
| `_run/goal.md` | Objective, scope, depth, definition of done | the human | rarely |
| `_run/scout.md` | Units, branches, authored sources, vocabulary | scout | once |
| `_run/progress.md` | Where the work got to. The ledger | the run | every target |

The goal stays out of the ledger so the human can steer between waves without interrupting a
session. Folded in, it would be overwritten by every status update.

### When a session dies

Resuming is the same command: `run` reads the goal and the ledger, finds the earliest incomplete
wave, continues. No resume verb and no run-state file, because a second state file is a second thing
that can disagree with reality.

**A `🔄` row is a crash survivor, not work in progress.** Nothing is still running.

| On disk | Do |
|---|---|
| Folder missing | Dispatch fresh |
| Folder partial | Dispatch to extend, not rewrite |
| Folder looks complete | Spot-check one navigation claim, then bank |

Never promote a `🔄` row to done because its folder exists. It was never reviewed.

## Rules every stage obeys

**Cite the least precision that gets the reader there.** `internal/http/`,
`internal/http/order.go`, or `internal/http/order.go#Create`. Use the nearest named function, type,
table, route constant or config key. A line number may follow as a disposable jump hint, never as
the identity of a citation. A claim you could not trace goes in `## Open questions`.

**A fact about one file belongs in that file.** A comment on the ORM hook moves with the hook, is
reviewed in the same pull request by the same person, and cannot drift from it. A doc can do none of
those three. Ranked by what it costs to keep true: a test that fails when the claim breaks, a comment
beside the code, a lint rule, then prose in the manual. Prose is the last resort. Writing comments
edits source, so it needs decision 8 at gate 1 — refused, the trap goes in `invariants.md` and the
run accepts that it will rot there.

**Document the authored file, not what it generates.** Generated output restates the same fact one
step later and goes stale silently. The scout report lists which is which during the run; `system.md`
carries that list afterwards.

**Mark what does not exist; assume the rest does.** The manual indexes what is there, so `✅` on
everything is noise and `🟡 partial` is a judgement that expires. One marker: `📋 not built at <sha>`
on a row or section whose code is a stub or absent. Dated, it is a fact about a revision instead of a
claim about now — and when somebody builds the thing, the doc is out of date rather than wrong.
Describing a stub as though it ran is the most common documentation failure in a live codebase.

**Never mutate the human's git.** Read-only only: `log`, `branch`, `show`, `diff`, `status`,
`ls-tree`. No `fetch`, `checkout`, `switch`, `pull`, `stash`, `merge`, `commit` unless they approved
that exact command. A wrong doc costs one re-read; a clobbered working tree costs their afternoon.

**Ask which branch is stable. Never switch to it yourself.** The default branch is often not where
work lands, so gather evidence read-only, then let the human decide. Until they answer, document
whatever is checked out and say so. Units may be stable on different branches.

```bash
git -C <repo> branch -a --sort=-committerdate | head          # no fetch
git -C <repo> ls-tree -r --name-only <candidate> -- <schema dir> | wc -l
```

**One fact, one place.** A table lives in one file; everything else links to it. Links run one
direction, so a reader always knows which way to walk:

```
system  →  services  →  flows  →  code
```

**One file, one question**, target 400 lines. **`index.md` is a router, never content.** **The root
index stays under 200 lines**, because every agent pays it every session.

**Frontmatter on every doc**, so `verify` can tell stale from wrong. `kind` implies authority; the
mapping is stated once in the root index.

```yaml
---
kind: index | map | flow | glossary
unit: <repo or package name>
read: <branch> @ <sha>
written: <YYYY-MM-DD>
---
```

Two kinds do not carry `read:`, because neither describes a revision: `decision` (a moment) and
`explainer` (a mechanism). Stamping either invites re-verification it does not need.

**The ledger is the handoff.** Read `_run/progress.md` first, write it last, every session. One
session documents one target.

**Documentation is crystallised thinking, not a place to think.** Work the code out first, then
write. A doc that reasons on the page is a draft that escaped.

## Every stage, same shape

1. Read `_run/scout.md` for authored sources and the branch decision, `_run/progress.md` for what
   exists.
2. Work on the branch named in `_run/goal.md`. Do not check it out. Record `read: <branch> @ <sha>`.
3. Choose read order by task:
   - Current behaviour, debugging, explanation: index → code and tests → spec to compare intent.
   - Implementing a requirement: approved spec → index → current code and tests.
   - Architecture change: relevant ADRs → current code → affected specs.
   Existing prose that is neither an approved spec nor an ADR is a claim to verify, not authority.
4. Write the doc, applying the one rule to every line. Update the router index and the ledger.
5. Report files written, source files commented, open questions. Claim a target is documented only
   for the code actually read.

Diagram rules are shared by `map`, `flow` and `explain`: [diagrams.md](references/diagrams.md).
