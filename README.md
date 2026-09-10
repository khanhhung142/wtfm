# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM.

Most docs try to explain what the code already does. After a few weeks you have two sources of truth, and the docs are the one that silently goes stale: they still say `POST /orders` needs admin while the code only checks membership.

Code already explains itself. wtfm writes a companion to that code, not a second copy of it. The manual is a set of small files, each answering one question, reached through a short index. It gets the next agent to the right place (`internal/http/order.go#Create`) instead of restating those lines. Words go to what reading the file will not tell you: which routes are stubs, which traps span more than one file, and which decisions were rejected.

Approved specs still state intent, and decision records still keep the why. When those disagree with the code, the manual records the drift. It does not blend them into one tidier answer.

```
/wtfm run
```

That command scouts the repo, asks you the questions code cannot answer, documents one unit for you to review as a template, then writes the rest. If the session dies halfway, run the same command again. If your tool has no slash commands, say: *use the wtfm skill, run*.

wtfm only uses read-only git commands unless you approve a specific write.

| Command | What it does |
|---------|----------------|
| `run` | The full job, start to finish |
| `system` | Stack, layers, conventions, and glossary for the whole repo |
| `map <unit>` | Document one repo, service, or package |
| `flow <name>` | Trace one request across units |
| `explain <topic>` | A plain-language walkthrough for a person |
| `decide <topic>` | Record why the code is this way, and what was rejected |
| `verify` | Check that links still point at real code, and review changed files |

## Install

```
npx skills add khanhhung142/wtfm
```

Works with any agent that reads the [Agent Skills](https://agentskills.io) format. You can also copy `skills/wtfm/` into your agent's skills directory.

MIT
