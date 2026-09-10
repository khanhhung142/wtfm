# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM.

Most docs explain what the code already does. A few weeks later you have two sources of truth, and the docs are the one that went stale: they still say `POST /orders` needs admin while the code checks membership. Your agent reads both and has no way to tell which is right.

wtfm fixes that by not writing the sentence in the first place.

**Anything derivable from a file in the repo gets cited, never copied.** No payloads, no field lists, no endpoint counts, no version numbers, no paraphrase of what a function does. One test decides: could this line become false without any file being renamed? If yes, it stays out.

What is left is the half code cannot state about itself:

- **where things are**: the door you open to change this, cited by symbol
- **what must not break across files**: the ORM hook that scopes every query by tenant, the two tables that need one transaction
- **what the words mean here**: an `Order` is unpaid until a `Payment` attaches, and what the warehouse calls an order is a `Shipment`
- **what was already rejected**, so the next agent stops re-proposing it

Those change when the design changes, not when the code moves. A doc built from them still holds a year later, and looks thin the day it is written.

Traps that live in one file get written as a comment beside that line instead, with your permission. A comment is reviewed in the same pull request as the code and moves with it. The same sentence in a doc is reviewed by nobody.

Approved specs still state intent, and decision records still keep the why. When those disagree with the code, the manual records the drift instead of blending them into one tidier answer.

```
/wtfm run
```

Scouts the repo, asks you the eight questions code cannot answer, documents one unit for you to review as a template, then writes the rest. If the session dies halfway, run the same command again. No slash commands in your tool? Say *use the wtfm skill, run*.

wtfm uses read-only git commands unless you approve a specific write. Adding trap comments to your source is one of the eight questions, and no is a valid answer.

| Command | What it does |
|---------|----------------|
| `run` | The full job, start to finish |
| `system` | Stack, layers, conventions, authored sources, glossary |
| `map <unit>` | Document one repo, service, or package |
| `flow <name>` | Trace one request across units |
| `explain <topic>` | A plain-language walkthrough, for a human |
| `decide <topic>` | Record why the code is this way, and what was rejected |
| `verify` | Two free greps, then a scoped read of what changed |

`verify` runs when a unit gets restructured or a link breaks, not on a schedule. It ships two lints that need no model: one resolves every `path#Symbol` citation, the other greps for content the manual is not supposed to contain, so a doc drifting back toward restating code gets caught the day it is written.

## Install

```
npx skills add khanhhung142/wtfm
```

Works with any agent that reads the [Agent Skills](https://agentskills.io) format. You can also copy `skills/wtfm/` into your agent's skills directory.

MIT
