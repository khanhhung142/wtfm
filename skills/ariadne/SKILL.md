---
name: ariadne
description: Read an unfamiliar codebase in stages and lay down an atlas — chunked, indexed, citation-backed documentation another agent can navigate, plus diagrams and a plain-language explainer for humans, plus an AGENTS.md that makes future agents read it first. Runs itself: one command fans the work out across subagents and drives it to completion from a ledger. Use when landing in a new or undocumented project, documenting a repo or service, tracing a cross-service flow, explaining how a system works, drawing architecture or sequence diagrams, setting up a docs vault, or checking whether existing docs still match the code. Trigger words: onboard, document this project, write docs for agents, code map, architecture diagram, explain this system, docs vault, docs-first.
---

# ariadne

Theseus did not memorise the labyrinth. He laid a thread.

This skill turns a codebase nobody has documented into an **atlas**: many small files, each
answering one question, reachable through a router index, every claim carrying a line citation. It
is written for an agent that arrives later with no context and a budget of three file reads.
Optimise for that reader. Humans are served by `explain`, a different artifact with different rules.

**Documentation is crystallised thinking, not a place to think.** Work the code out first, then
write. A doc that reasons on the page is a draft that escaped.

## Modes

`$1` is the mode.

| Mode | Does | Output | Reference |
|------|------|--------|-----------|
| `run` | **Drives every other mode to completion.** Default | the whole atlas | [references/run.md](references/run.md) |
| `scout` | Recon. Learn the project, propose a plan | `_scout.md` only | [references/scout.md](references/scout.md) |
| `bootstrap` | Create the atlas, wire it into the repos | skeleton + `AGENTS.md` patch | [references/bootstrap.md](references/bootstrap.md) |
| `map <unit>` | Document one repo, service or package | `services/<unit>/` | [references/map.md](references/map.md) |
| `flow <name>` | Trace one request end to end | `flows/<name>.md` | [references/flow.md](references/flow.md) |
| `explain <topic>` | Teach a human how something works | `explain/<topic>.md` | [references/explain.md](references/explain.md) |
| `verify` | Re-resolve citations, report drift | ledger update | [references/verify.md](references/verify.md) |

Diagram rules are shared: [references/diagrams.md](references/diagrams.md), read by `map`, `flow`
and `explain`.

**Read only the reference for the mode you run.** No mode given: run `run`.

## The loop

The reason a docs project stalls is that a human has to think of the next prompt thirty times. They
do not. `run` reads the ledger, picks the next target, dispatches it, banks the result and repeats.

Each target gets a **fresh subagent**, because context exhaustion, not difficulty, is what makes the
second unit in a session shallower than the first. Fanning out is not an optimisation here; it is
the only way the tenth unit gets the same quality as the first.

Three rules keep a fan-out from corrupting the atlas:

1. **A subagent writes only inside its own target's folder.** Never the root index, never another
   unit's files.
2. **A subagent never writes the ledger.** It returns a report. The parent, which is the only writer,
   banks it. Parallel writers to one ledger lose rows.
3. **The parent updates the root index and the ledger after each wave**, then starts the next.

Humans gate twice: on the scout plan, and on the first completed unit, which is where a template
flaw is cheap to fix and after which it is repeated sixty times. Everything else runs unattended.

## The rules every mode obeys

### 1. Cite or omit

Every technical claim names `path/file.ext:LINE`, repo-relative. A claim you could not trace to a
line does not go in the doc. It goes in that doc's `## Open questions`.

This rule is what makes the atlas worth more than the next model's guess. An agent that reads a
citation can confirm it in one read. An uncited paragraph is worth nothing, because re-deriving the
fact costs exactly what it cost before the doc existed.

### 2. Truth sources, and generated output

Some files are authored; others are produced from them. Document and cite the authored file.
Generated output restates the same fact one step later, and goes stale without saying so.

`scout` writes the project's truth-source table into `_scout.md`, and every later mode reads it.
The shape, though the actual entries are per-project:

| Question | Truth source | Not this |
|---|---|---|
| What is the API? | hand-written schema or interface definition | generated clients, generated docs |
| What is stored? | migrations or schema definitions | ORM output, a dump of the live database |
| What is configured? | the config template in the repo | a running container's environment |

### 3. Status tags

Real codebases are half-built. Every section carries one:

- `✅ implemented` — the code exists and is cited.
- `🟡 partial` — some of it exists; name exactly what is missing.
- `📋 spec-only` — described somewhere, no code yet.

A doc with no status tags claims everything works, which is a claim nobody verified. Describing a
stub as though it ran is the most common documentation failure in a live codebase. A function that
returns nil or throws not-implemented is `📋`, and saying so is the doc's whole value.

### 4. Read the branch that is true

The default branch is often not where the work lands. Establish which branch is true before reading
a line, and prove it:

```bash
git -C <repo> fetch --all
git -C <repo> branch -r --sort=-committerdate | head
for b in main master develop stable; do
  printf '%s %s\n' "$b" "$(git -C <repo> rev-list --count origin/$b 2>/dev/null)"
done
```

Then count the files that carry meaning for this project — schema directory, interface definitions,
route table — on each candidate. A branch holding one schema file where another holds twenty-one is
not a stylistic difference, and reading the wrong one produces an atlas that is confidently, fully
wrong. Record branch and commit in every doc's frontmatter. Units may be true on different branches;
say so per unit rather than forcing one global answer.

### 5. Spec and code disagreeing is a finding

When a written spec contradicts the code, record both and name the contradiction. Never silently
pick whichever makes a tidier doc. That sentence is usually the most valuable one in the atlas.

### 6. One fact, one place

A table lives in exactly one file; every other file links to it. A copied table is wrong within a
month, and the agent reading the copy has no way to know. Links run one direction, so a reader
always knows which way to walk:

```
features  →  services  →  flows  →  code
```

### 7. Chunked and indexed

- **One file answers one question.** Target 400 lines. A file past that is two files.
- **`index.md` is a router, never content**: a table of "need → path". A router that starts
  explaining is a file the next agent must read entirely before it can decide anything.
- **The root index stays under 200 lines.** Every agent pays it every session. When a registry row
  swells into a paragraph, the detail moves into that unit's own index and the row becomes a link.
- **Frontmatter on every doc**, so `verify` can tell stale from wrong:

```yaml
---
unit: <repo or package name>
branch: <branch>
commit: <sha>
written: <YYYY-MM-DD>
status: ✅ | 🟡 | 📋
---
```

### 8. The ledger is the handoff

A codebase outgrows any context window. `_progress.md` at the atlas root holds the state of the
work and is the only thing that survives between sessions and between subagents.

- **Start:** read `_progress.md`. Never re-document a target marked done; extend it.
- **One session, one target:** one unit, one flow, one explainer. "Document the backend" is a table
  of sessions, not a session.
- **End, always, even when stopping early:** update the status cells that changed, add one
  session-log row, rewrite the two-line `## Now`, record any cross-cutting open question. An
  unwritten ledger row costs the next session a full re-discovery pass.

## Workflow, every mode

1. Read `_scout.md` for the truth-source table, the branch decision and the unit list. Missing: run
   `scout` first, whatever was asked for.
2. Read `_progress.md` for what is already done.
3. Check out the true branch for the units in scope. Record branch and commit.
4. Read written specs first where they exist, since they name the intent the code may have missed.
   Then read the code.
5. Write the doc. Update the router index and the ledger.
6. Report: file written, status tag counts, open questions. Claim a target is documented only for
   the code actually read.
