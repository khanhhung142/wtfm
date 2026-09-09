# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM.

`wtfm` is an agent skill — plain `SKILL.md`, no runtime, works anywhere skills are read — that walks
into an undocumented codebase and writes the manual: many small files, each answering one question,
behind a router index, every claim carrying a `file:line` citation. Written for the agent that
arrives next week with no context and a budget of three file reads. It also patches `AGENTS.md` so
that agent reads the manual before it reads code.

```
/wtfm run
```

Scouts the repo, asks you to settle what code cannot answer, documents one unit for you to review as
a template, fans the rest out across subagents, drives it to done. Session dies halfway? Same
command, same place. No slash commands in your tool? Say *"use the wtfm skill, run"*.

## Commands

| Command | Does |
|---------|------|
| `run` | Everything, driven to done. The one you want |
| `system` | Stack, layers and conventions of the whole repo |
| `map <unit>` | One repo, service or package |
| `flow <name>` | One request, traced end to end across units |
| `explain <topic>` | A plain-language walkthrough for a human |
| `verify` | Re-resolve every citation, report what drifted |

## What you get

```
docs/manual/
├── index.md          # router: need → path. Under 200 lines, forever
├── system.md         # stack, shape, layers, conventions. The whole repo
├── _goal.md          # objective, scope, definition of done. You own this one
├── _scout.md         # units, branches, truth sources, vocabulary
├── _progress.md      # the ledger: done, next, open questions
├── services/<unit>/  # index, architecture, layers, data, api, events, config
├── flows/            # sequence diagram + hop-by-hop table, every hop cited
├── explain/          # for humans, not agents
└── decisions/
```

Several repos? The manual lives in a sibling directory and each repo gets a symlink to its own
slice, so the docs belong to no single repo and rot in none of them.

## Install

Any agent that reads the [Agent Skills](https://agentskills.io) format:

```
npx skills add khanhhung142/wtfm
```

Lands in whatever the tool in front of you expects — `.claude/skills/`, `.agents/skills/`,
`.codex/skills/`, `~/.cursor/skills/`. `--agent <name>` to pick one, `--list` to look first.

<details>
<summary>Other ways</summary>

**Claude Code plugin**, which adds the `/wtfm` command:

```
/plugin marketplace add khanhhung142/wtfm
/plugin install wtfm
```

**By hand** — copy `skills/wtfm/` into your agent's skills directory.

**Claude apps** — zip `skills/wtfm/`, upload at Settings → Capabilities → Skills.

</details>

## Rules it follows

- **Cite or omit.** Every technical claim names `file:line`. Anything untraceable goes in open
  questions, not in the doc.
- **Status tags.** `✅ implemented`, `🟡 partial`, `📋 spec-only`. A doc with no tags claims
  everything works, which is a claim nobody checked.
- **One file, one question.** `index.md` routes, never explains.
- **Your git stays yours.** Read-only commands only. No fetch, no checkout, no commit unless you
  said yes to that command.
- **Two gates.** After the scout, and after the first unit — reviewed as a template, because every
  flaw in it is about to be repeated sixty times.
- **It survives dying.** Targets claimed before dispatch, banked one at a time. The ledger holds the
  position, so resuming is the same command.

## License

MIT
