# ariadne

Theseus did not memorise the labyrinth. He laid a thread.

`ariadne` is a Claude Code skill that walks into a codebase nobody has documented and lays down an
**atlas**: many small files, each answering one question, reachable through a router index, every
claim carrying a `file:line` citation. It is written for the agent that arrives next week with no
context and a budget of three file reads.

It also writes the `AGENTS.md` section that makes that agent read the atlas before it reads code.

```
/ariadne run
```

One command. It scouts the project, asks you to confirm the plan, builds the atlas skeleton,
documents the first unit for you to review as a template, then fans the rest out across subagents
and drives it to completion from a ledger.

## Why not just ask an agent to write docs

Because you get a plausible essay. The three failures are always the same:

- **It documents what the code appears to intend**, not what it does. Half a live codebase is stubs,
  and a stub described as working is worse than no doc at all.
- **It reads the wrong branch.** The default branch is frequently not where the work lands.
- **It writes one long file.** The next agent has to read all of it to find out that none of it was
  relevant.

ariadne is mostly the rules that stop those three. Cite or omit. Status tags on every section. Prove
which branch is true before reading a line. One file, one question. `index.md` is a router, never
content.

## Modes

| Mode | Does |
|------|------|
| `run` | Drives all of the below to completion. The default |
| `scout` | Recon. Units, branches, truth sources, vocabulary, a plan. Writes one file |
| `bootstrap` | Creates the atlas, symlinks it into the repos, patches `AGENTS.md` |
| `map <unit>` | Documents one repo, service or package |
| `flow <name>` | Traces one request end to end, across units |
| `explain <topic>` | A plain-language walkthrough for a human |
| `verify` | Re-resolves every citation, reports what drifted |

## What it produces

```
docs/atlas/
├── index.md          # router: need → path. Under 200 lines, forever
├── _scout.md         # units, branches, truth sources, vocabulary
├── _progress.md      # the ledger: what is done, what is next, open questions
├── services/<unit>/  # index, architecture, layers, data, api, events, config
├── flows/            # sequence diagram + hop-by-hop table, every hop cited
├── explain/          # for humans, not agents
└── decisions/
```

For a workspace of several repos, the atlas lives in a sibling directory and each repo gets a
symlink to its own slice, so the docs belong to no single repo and rot in none of them.

## Install

```
/plugin marketplace add hungphan/ariadne
/plugin install ariadne
```

Or copy `skills/ariadne/` into `~/.claude/skills/`.

## The parts that matter

**The ledger.** `_progress.md` is the state of the work and the only thing that survives between
sessions. Every session reads it first and writes it last. An unwritten ledger row costs the next
session a full re-discovery pass.

**Fan-out with one writer.** Context exhaustion, not difficulty, is why the second unit documented
in a session is thinner than the first. So each target gets a fresh subagent, each writes only
inside its own folder, none of them touch the ledger, and the parent banks every wave. Parallel
writers to one index lose each other's work.

**Two human gates.** After the scout, where you correct what only you know: which service is dead,
which branch really ships, what the acronym means. And after the first unit, reviewed as a template
rather than as content, because every flaw in it is about to be repeated sixty times.

**Status tags.** `✅ implemented`, `🟡 partial`, `📋 spec-only`. A doc with no status tags is
claiming everything works, which is a claim nobody checked.

## Prior art

Distilled from a hand-built documentation vault covering nine services across eight release phases,
and the project-specific skill that produced it. This is that skill with the project taken out and
the manual prompting replaced by a loop.

## License

MIT
