# wtfm — Write The Fine Manual

You cannot RTFM when there is no FM.

`wtfm` is an agent skill — plain `SKILL.md`, no runtime, works anywhere skills are read — that walks
into an undocumented codebase and writes the manual: many small files, each answering one question,
behind a router index, every claim carrying a `file:line Symbol` citation. Written for the agent
that arrives next week with no context and a budget of three file reads.

**It writes an index to your code, not a second copy of it.** Most docs folders quietly become a
rival source of truth, drift for two quarters, and end up making your agent more confident and more
wrong. This one is built to be the thing you check *before* opening the file, never instead of it —
and to spend its words on the two things no file can state: what your words mean, and why the code
is like this.

```
/wtfm run
```

Scouts the repo, asks you to settle what code cannot answer, documents one unit for you to review as
a template, fans the rest out across subagents, drives it to done. Session dies halfway? Same
command, same place. No slash commands in your tool? Say *"use the wtfm skill, run"*.

---

## Most docs make your agent worse. Here is why

The usual move is to write a `docs/` folder that explains what the code does, and point the agent at
it. That folder is now a **second source of truth** — and it is the one that nobody maintains. Code
ships daily; docs get touched twice a year. Nothing in your repo fails when a doc goes wrong, so the
drift is silent, and one day the doc says `POST /orders` needs admin while the code has checked
membership only since March.

Your agent reads both. It cannot tell which one shipped. Sometimes it picks the doc, skips reading
the code entirely, and confidently builds on a fact that stopped being true two quarters ago. That
is worse than having no docs at all, because with no docs it would have gone and read the code.

**wtfm refuses to be that folder.** The code is the truth. The manual is the index to it, plus the
two things no file can ever state: what the words mean here, and why the code is like this.

## What that actually changes

|  | The usual `docs/` folder | wtfm |
|---|---|---|
| **Authority** | Second source of truth. Conflicts with code; the agent guesses | Index to the code. *Code wins* — written into the docs, the index and your `AGENTS.md` |
| **Claims** | Prose nobody can check | `file:line Symbol` on every claim. Confirmed or refuted in one read |
| **Citations** | Line numbers that break the moment anyone inserts a line above them | Line *and* anchor. A rename breaks it; an edit does not, and `verify` repairs the number with one `grep` |
| **Drift** | Silent forever | A 20-line lint resolves every citation on each commit. No model, no network, no cost |
| **Upkeep** | A recurring chore nobody schedules twice | Claims chosen for slow decay, and a `Guarded by:` test on the rest. What CI already checks, no one re-reads |
| **Going stale** | Binary and fatal: trusted, then quietly wrong | Graded. One `git log` says whether to read a doc as fact, as a map, or as a hypothesis |
| **Volume** | Restates the code, then rots | Points at the authored file, spends its words on what reading it *won't* tell you |
| **Unbuilt code** | Described as though it runs | `📋 spec-only`. Half a live codebase is stubs, and the manual says which half |
| **The "why"** | Lost the day the author left | `decisions/` — the options that were rejected, and what would reverse the call |
| **Vocabulary** | Assumed shared | `glossary.md` — what an `Order` means *here*, and what it does not |
| **Reading cost** | Load it all, hope | Router index under 200 lines, forever. Open only what the task needs |
| **Humans** | Same files, badly served | `explain/` — a separate artifact with separate rules, written in prose |

## What it refuses to write

The restraint is the product. A doc that survives is a doc somebody will still trust in a year.

- **No table a script could regenerate.** If it is derivable from a schema, a route table or a config
  template, the manual points at that file instead. Where a surface really is scattered across
  thirty controllers, it builds the index — but as *addresses, never payloads*.
  `POST /orders → internal/http/order.go:23 Create, auth: member, 🟡` survives a field being added
  to the request body. A copy of that request body does not, and saves the reader a read they should
  have done anyway.
- **No trap left implicit.** The space saved goes to what the schema cannot say: the ORM hook that
  scopes every query by tenant, the two tables that must share a transaction, the three routes out
  of sixty that skip auth, the column that is nullable in the schema and required by every path that
  reads it.
- **No uncited technical claim.** Anything untraceable goes to `## Open questions`, numbered, so a
  doc can cite the gap in one token instead of quietly filling it.
- **No invented rationale.** Every other claim is checkable; "we chose Postgres for write
  contention" is not. `Options: unknown — nobody recorded them` is a correct ADR. A plausible story
  reverse-engineered from the code is a liability that reads like an asset.
- **No paraphrase of the unit doc.** If an explainer restates a table, the table was better. Delete
  the explainer.
- **No fact that belongs in one file.** A comment on the ORM hook moves with the hook and is reviewed
  in the same pull request. The manual cannot do either, so it keeps only what spans files.

## What one actually looks like

A surface doc, whole. Notice how little of it is the code, and how much of it is the part you cannot
get by reading the code faster.

````markdown
---
unit: orders-api
branch: develop
commit: 4f2a91c
written: 2026-03-14
status: 🟡
---

# orders-api — HTTP surface

The routes are registered by hand, not from a spec, so this table is the only place the surface
exists in one piece. Registration: `internal/http/router.go:52 Register`.

| Route | Entry | Auth | Status |
|-------|-------|------|--------|
| `POST /orders` | `internal/http/order.go:23 Create` | member | ✅ |
| `POST /orders/:id/refund` | `internal/http/refund.go:19 Refund` | **none** | 🟡 |
| `GET /orders/export` | `internal/http/export.go:11 Export` | admin | 📋 |

Request and response bodies are generated from `api/openapi.yaml` — read that, not this.

## Exceptions

- **`POST /orders/:id/refund` skips the auth middleware.** `internal/http/router.go:71 Register`
  registers it on the bare mux, not the authed group. It checks a signed token in the body instead
  (`internal/http/refund.go:31 verifySignature`), which is deliberate — see
  [decisions/04](../../decisions/04-refunds-out-of-band.md) — but it means the usual
  "everything under /orders is authed" reading is wrong.
  Guarded by: `internal/http/refund_test.go:88 TestRefundRejectsUnsignedBody`
- **`GET /orders/export` is a stub.** Returns `501` at `internal/http/export.go:14 Export`. The
  route exists, the client calls it, nothing is behind it.
  Guarded by: nothing.
- **Every handler here is tenant-scoped by an ORM hook**, not by its own query — see
  [architecture.md](architecture.md). A handler that builds a raw query bypasses it and returns
  another customer's rows.
  Guarded by: nothing — a raw query bypasses it silently.

## Open questions
1. Is `GET /orders/export` waiting on the reporting service, or abandoned? (blocks: nothing)
````

Six routes' worth of request bodies would have doubled the length and been wrong by April. The three
bullets under **Exceptions** are the reason the file exists, and none of them are visible in the
route table, the schema, or an OpenAPI document.

The `Guarded by:` lines are what keeps this file cheap to own. The first bullet has a test watching
it, so no future review needs to re-read it — CI will say so. The two marked `nothing` are the entire
surface anybody has to check by hand, and the third one is really a missing test in disguise.

## Why it does not become a chore

Most documentation efforts die at upkeep, not at authoring. wtfm attacks that three ways, and none of
them is "remember to run the checker".

**It splits the check in two.** Asking *do the citations resolve* and *are the claims still true* in
one pass makes you pay a model's attention for work `grep` does. So question one is a 20-line
`check-citations.sh` you wire into pre-commit or CI: it finds every moved and every dead citation the
day it happens, prints the corrected line number, and costs nothing. Question two — the one that
needs judgement — only runs where the lint went red. *(Offered at setup, never installed without
asking. The skill itself stays plain markdown.)*

**It names what already guards each claim.** Traps end with `Guarded by: <test>` or
`Guarded by: nothing`. A claim a test already covers is checked by your CI on every run, forever, and
no documentation pass ever needs to look at it. The `nothing` ones are the whole review surface — and
half the time the right fix is a missing test, not a better sentence.

**It prefers claims that decay slowly.** A line number rots on the next edit; a surface list rots on
the next feature; an invariant rots only when the design changes, and a decision never rots at all.
The manual is deliberately weighted toward the bottom of that ladder, so most of it is still true in
a year without anybody having maintained it.

## The two files code can never replace

Everything else in the manual restates something a file already says, faster and in one place. These
two hold what no file says and no file ever will — and they are the only docs here that do not rot,
because they describe a *moment*, not a state.

**`decisions/`** — what was considered instead, and why it lost. Your codebase shows the option that
won. It never shows the three that were rejected, so every agent that arrives re-proposes one of
them. wtfm harvests what survived — revert commits, `HACK` comments, dead feature flags, tests named
for a bug — then asks you for the rest, and labels anything it merely inferred so nobody acts on a
guess wearing a decision's clothes.

**`glossary.md`** — an agent can see you have a type called `Order`. What no file states is that an
order here is unpaid until a `Payment` attaches, that the warehouse's "order" is a `Shipment`, and
that `order_id` on the legacy table means something else entirely. Each of those is a bug somebody
is about to write.

## Commands

| Command | Does |
|---------|------|
| `run` | Everything, driven to done. The one you want |
| `system` | Stack, layers, conventions and glossary of the whole repo |
| `map <unit>` | One repo, service or package |
| `flow <name>` | One request, traced end to end across units |
| `explain <topic>` | A plain-language walkthrough for a human |
| `decide <topic>` | Record why the code is like this, and what was rejected |
| `verify` | Re-resolve every citation, report what drifted |

## What you get

```
docs/manual/
├── index.md          # router: need → path. Under 200 lines, forever
├── system.md         # stack, shape, layers, conventions. The whole repo
├── glossary.md       # what this project's words mean. What code cannot say, part one
├── _goal.md          # objective, scope, definition of done. You own this one
├── _scout.md         # units, branches, truth sources, vocabulary
├── _progress.md      # the ledger: done, next, open questions
├── services/<unit>/  # index, architecture, layers, data, api, events, config
├── flows/            # sequence diagram + hop-by-hop table, every hop cited
├── explain/          # for humans, not agents
├── decisions/        # numbered ADRs. What code cannot say, part two
└── check-citations.sh  # the lint, if you accepted it. No model, no network
```

Several repos? The manual lives in a sibling directory and each repo gets a symlink to its own
slice, so the docs belong to no single repo and rot in none of them.

## Built to survive the run

Documenting a real codebase is bigger than one context window, so wtfm is built as a loop rather
than a session.

- **Two gates, and it stops at them.** After the scout it puts seven decisions in front of you with
  its recommendation and the evidence already filled in, so you correct rather than compose. After
  the first unit it hands that unit back as a *template* — because every flaw in it is about to be
  copied sixty times, and this is the last cheap moment to catch one.
- **It fans out.** Three to five subagents a wave, each writing only inside its own folder, each
  spot-checked before its report is banked. A subagent that fabricated one citation fabricated more.
- **It survives dying.** Targets are claimed in the ledger *before* dispatch, banked one at a time.
  Context fills, the laptop closes, the credit runs out — resuming is the same command, and a `🔄`
  row is treated as a crash survivor, never promoted to done because its folder exists.
- **No subagents in your tool?** Same targets, one per session. The ledger is the handoff either
  way; a wave of one is slower, not different.
- **Your git stays yours.** Read-only commands only. It gathers the evidence for which branch is
  stable and shows you the counts — `develop has 21 schema files, main has 1` — then waits. No
  fetch, no checkout, no commit unless you said yes to that exact command. A wrong doc costs one
  re-read; a clobbered working tree costs your afternoon.

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

- **Code is the truth.** Where a doc and the code disagree, the code is right and the doc is a bug.
  Never reworded, never averaged. It patches your `AGENTS.md` to say so.
- **Cite or omit, with an anchor.** Every technical claim names `file:line Symbol` — the line to
  jump to, the symbol to find it again once the line has moved. Anything untraceable goes in open
  questions, not in the doc.
- **Old is not useless.** A doc is wrong about line numbers long before it is wrong about paths, and
  wrong about paths long before it is wrong about shape. `verify` reports how far behind a doc is
  and what it is still good for, instead of stamping it "stale" and throwing the working part away.
- **Status tags.** `✅ implemented`, `🟡 partial`, `📋 spec-only`. A doc with no tags claims
  everything works, which is a claim nobody checked.
- **One fact, one place.** A table lives in one file; everything else links to it. Links run one
  direction: `system → services → flows → code`.
- **One file, one question.** `index.md` routes, never explains.
- **Spec contradicting code is a finding**, recorded as a contradiction — never quietly resolved in
  favour of the tidier doc. That sentence is usually the most valuable one in the manual.

## License

MIT
