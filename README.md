# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM.

`wtfm` is a Claude Code skill that walks into a codebase nobody has documented and writes the
manual: many small files, each answering one question, reachable through a router index, every claim
carrying a `file:line` citation. The reader it is written for is the agent that arrives next week
with no context and a budget of three file reads.

It also writes the `AGENTS.md` section that makes that agent read the manual before it reads code.

```
/wtfm run
```

One command. It scouts the project, asks you to confirm the plan, builds the skeleton, documents the
first unit for you to review as a template, then fans the rest out across subagents and drives it to
completion. If the session dies halfway, run the same command again and it picks up where it stopped.

## Why not just ask an agent to write docs

Because you get a plausible essay. The three failures are always the same:

- **It documents what the code appears to intend**, not what it does. Half a live codebase is stubs,
  and a stub described as working is worse than no doc at all.
- **It reads the wrong branch.** The default branch is frequently not where the work lands.
- **It writes one enormous file.** The next agent reads all of it to discover that none of it was
  relevant.

`wtfm` is mostly the rules that stop those three. Cite or omit. Status tags on every section. Prove
which branch is true before reading a line. One file, one question. `index.md` is a router, never
content.

## What you type

| Command | Does |
|---------|------|
| `/wtfm run` | Everything below, driven to completion. The one you want |
| `/wtfm map <unit>` | Just this repo, service or package |
| `/wtfm flow <name>` | Just this request, traced end to end across units |
| `/wtfm explain <topic>` | A plain-language walkthrough for a human |
| `/wtfm verify` | Re-resolve every citation, report what drifted |

Two more stages exist, and `run` performs both. You never type them. **Scout** is the read-only
recon pass that works out the units, the true branch per unit, the truth sources and the vocabulary.
**Bootstrap** creates the skeleton and wires it into `AGENTS.md`. Scout is not optional: everything
downstream reads its output, and skipping it means every later doc guesses at which branch is real.

## What it produces

```
docs/manual/
├── index.md          # router: need → path. Under 200 lines, forever
├── _goal.md          # objective, scope, definition of done. You own this one
├── _scout.md         # units, branches, truth sources, vocabulary
├── _progress.md      # the ledger: what is done, what is next, open questions
├── services/<unit>/  # index, architecture, layers, data, api, events, config
├── flows/            # sequence diagram + hop-by-hop table, every hop cited
├── explain/          # for humans, not agents
└── decisions/
```

For a workspace of several repos, the manual lives in a sibling directory and each repo gets a
symlink to its own slice, so the docs belong to no single repo and rot in none of them.

## Install

```
/plugin marketplace add khanhhung142/wtfm
/plugin install wtfm
```

Or copy `skills/wtfm/` into `~/.claude/skills/`.

## The parts that matter

**Three files, three owners.** `_goal.md` is the objective and the definition of done, and you own
it. `_scout.md` is what the project is, written once. `_progress.md` is the ledger, rewritten every
target. Keeping the goal out of the ledger is what lets you steer between waves — drop a unit, stop
before flows, change the depth — without interrupting a session or re-explaining the project.

**Fan-out with one writer.** Context exhaustion, not difficulty, is why the second unit documented
in a session is thinner than the first. So each target gets a fresh subagent, each writes only inside
its own folder, none of them touch the ledger, and the parent banks every report as it lands.
Parallel writers to one index lose each other's work.

**It survives dying.** Targets are claimed in the ledger before dispatch, results are banked one at
a time, and the manual is committed once per wave. Run out of credit mid-run and the next `wtfm run`
reads the ledger, finds the rows still marked in-progress, checks each against what is actually on
disk, and continues. There is no resume command, because a second state file is a second thing that
can disagree with reality.

**Two human gates.** After the scout, where you correct what only you know: which service is dead,
which branch really ships, what the acronym means. And after the first unit, reviewed as a template
rather than as content, because every flaw in it is about to be repeated sixty times.

**Status tags.** `✅ implemented`, `🟡 partial`, `📋 spec-only`. A doc with no status tags is
claiming everything works, which is a claim nobody checked.

## Prior art

Distilled from a hand-built documentation vault covering nine services across eight release phases,
and the project-specific skill that produced it. This is that skill with the project taken out and
thirty copy-pasted prompts replaced by a loop.

## License

MIT
