# Stage: scout — recon before anything is written

The first landing. Everything later stages assume is decided here: which units exist, which revision is
being read, where the authored contracts live, and which vocabulary needs a human. Every hour here is
repaid by every later doc; every guess here is repeated in all of them.

**Output is exactly one file: `_run/scout.md`.** No unit docs, no diagrams, no skeleton. Scout ends at
a human gate, and writing the manual before the plan is approved wastes the gate.

## Read order

Cheap and broad first. Stop expanding a branch as soon as it stops answering a question below.

1. **The perimeter.** `README`, `CONTRIBUTING`, `Makefile`, `justfile`, `docker-compose.yml`, package
   scripts, CI workflows. These name the units, the commands and the services before any source.
2. **The unit list.** Anything that builds or deploys independently: its own manifest or Dockerfile.
3. **Dependency manifests.** Language, framework, which units depend on which. Those edges are the
   first draft of the architecture.
4. **Authored sources.** Schemas, interface definitions, approved specs, migrations, route tables,
   config templates. Record what each is authoritative for, and what is generated from it — the codegen
   commands from step 1 tell you.
5. **Entry points.** `main`, server bootstrap, route registration, job registration, consumers, CLI
   commands. An entry point no unit calls is a dead unit; say so.
6. **Boundaries.** Where units talk: HTTP, RPC, queue, shared database, shared library. Each edge is a
   candidate flow.
7. **Tests.** What is covered tells you what the authors considered load-bearing, and test names are
   the cheapest source of domain vocabulary in the repo.
8. **Existing docs.** Classify before trusting: approved spec, ADR, glossary, implementation prose, or
   navigation. Approval matters for specs; historical evidence for ADRs; age only for the last two.

## Branch, evidenced not assumed

Follow the git rules in SKILL.md for every unit: read-only, never switch, record per unit. When
candidates disagree, count the files that matter with `git ls-tree` against each — no checkout, no
fetch — and put the counts in the report. A line saying `develop has 21 schema files, main has 1` is
what stops the next agent repeating the mistake. Scout proposes; gate 1 decides.

## The vocabulary pass

Collect the terms the codebase uses for its own concepts, from type names, table names, test names and
route names. Two symptoms matter more than the list:

- **One concept, several names** across units. A seam where two teams met, and where flow docs will go
  wrong.
- **One name, several concepts.** Worse. Name both and flag it at the gate; only a human can say which
  one owns the word.

This pass is why the glossary is possible, and the glossary has the longest half-life of anything in
the manual. Spend the time here.

## `_run/scout.md`

````markdown
---
scouted: <YYYY-MM-DD>
---

# Scout report

*This is a run artifact. Durable content graduates into `system.md` and `glossary.md` at the end of
the run; nothing outside `_run/` should link here.*

## What this project is
Three lines. What it does, who uses it, what shape it is.

## Units
| Unit | Path | Language / framework | Read | Depends on | Size | Priority |
|------|------|----------------------|------|------------|------|----------|

Size: rough file count of hand-written source, so waves can be balanced.
Priority: what a new agent needs first. The unit most others depend on ranks above the biggest one.

## Authored sources
| Question | Source | Authority | Approval / revision | Generated from it, do not cite |
|---|---|---|---|---|

**The table that graduates into `system.md`.** Get it right here and it is the most durable thing in
the manual.

## Boundaries
| From | To | Mechanism | Where it is defined |
|---|---|---|---|

## Vocabulary
| Term | Candidate meaning | Provenance | Also called | Conflict |
|---|---|---|---|---|

## Commands
Build, test, run, codegen — copied from the Makefile or scripts, not invented. Say which you ran.

## Branch evidence
| Unit | Candidates | Counts | Recommendation |
|---|---|---|---|

## Proposed plan
| Wave | Targets | Depends on |
|---|---|---|

Plus target count, and where the manual should live.

## Open questions for the human
Numbered. Only what the code cannot answer: which unit is dead, which branch really ships, which of two
names for one concept is right, what the acronym stands for.
````

## Rules

- **Read only, git included.** No `fetch`, `checkout`, `switch`, `commit`. Scout does not create the
  manual and does not fix anything it finds. Whatever branch is checked out is what scout reads, and
  the record says which.
- **Do not read every file.** Scout answers "what is here, and where should later work look", not "how
  does it work". A scout that reads deeply runs out of context before the last unit.
- Everything uncertain goes in Open questions rather than being resolved by assumption. The gate exists
  to spend the human's knowledge, and a question there costs one line instead of a wrong doc.
