# Stage: scout — recon before anything is written

The first landing. Everything later modes assume gets decided here: which units exist, which
revision is being inspected, where authored contracts live, and which vocabulary needs human
confirmation. Every hour spent here is repaid by every subsequent doc; every guess made here is
repeated in all of them.

**Output is exactly one file: `_scout.md`.** No unit docs, no diagrams, no manual skeleton. Scout
ends at a human gate, and writing the manual before the plan is approved wastes the gate.

## Read order

Cheap and broad first. Stop expanding a branch as soon as it stops answering a question below.

1. **The perimeter.** `README`, `CONTRIBUTING`, `Makefile`, `justfile`, `docker-compose.yml`,
   `package.json` scripts, CI workflow files. These name the units, the commands and the services
   before any source is read.
2. **The unit list.** Directories that build or deploy independently: a repo, a service, a package,
   a workspace member. Anything with its own manifest or its own Dockerfile is a unit.
3. **Dependency manifests.** Language, framework, major versions, and which units depend on which.
   The dependency edges are the first draft of the architecture.
4. **Authored sources.** Find schema files, interface or API definitions, approved specs,
   migrations, route tables and config templates. Record what each is authoritative for, and find
   what is generated from it by reading the codegen commands in step 1.
5. **Entry points.** `main`, server bootstrap, route registration, job registration, message
   consumers, CLI commands. An entry point that no unit calls is a dead unit; say so.
6. **Boundaries.** Where do units talk to each other: HTTP, RPC, queue, shared database, shared
   library. Each edge is a candidate flow.
7. **Tests.** What is covered tells you what the authors considered load-bearing. Test names are the
   cheapest source of domain vocabulary in the repo.
8. **Existing docs.** Classify before trusting: approved spec, ADR, glossary, implementation prose
   or navigation. Approval matters for specs; historical evidence matters for ADRs; age against the
   code matters only for implementation prose and navigation.

## Branch, evidenced not assumed

Follow the two git rules in SKILL.md for every unit: gather read-only, never switch, and record the
result per unit in the table. When candidates disagree, count the files that matter with
`git ls-tree` against each candidate — no checkout, no fetch — and put the counts in `_scout.md`. A
one-line note saying `develop has 21 schema files, main has 1` is what stops the next agent from
repeating the mistake. Scout proposes; gate 1 decides which branch is stable and who switches.

## The vocabulary pass

Collect the terms the codebase uses for its own concepts, from type names, table names, test names
and route names. For each, note the definition and the synonyms the project avoids. Two symptoms
matter more than the list itself:

- **One concept, several names** across units. That is a seam where two teams met, and it is where
  flow docs will go wrong.
- **One name, several concepts.** Worse. Name both and flag it at the gate; only a human can say
  which one owns the word.

## `_scout.md`

````markdown
---
scouted: <YYYY-MM-DD>
---

# Scout report

## What this project is
Three lines. What it does, who uses it, what shape it is (monolith, N services, library plus app).

## Units

| Unit | Path | Language / framework | Branch inspected | Commit | Depends on | Size | Priority |
|------|------|----------------------|---------------------|--------|------------|------|----------|

Size: rough file count of hand-written source, so waves can be balanced.
Priority: what a new agent needs first. The unit most other units depend on ranks above the one with
the most code.

## Authored sources

| Question | Source | Authority | Approval / revision | Generated from it, do not cite |
|---|---|---|---|---|

## Boundaries

| From | To | Mechanism | Where it is defined |
|---|---|---|---|

## Vocabulary

| Term | Candidate meaning | Provenance | Also called | Conflict |
|---|---|---|---|---|

## Commands
Build, test, run, codegen. Copy them from the Makefile or scripts rather than inventing them, and
say which ones you actually ran.

## Proposed plan

| Wave | Targets | Depends on |
|---|---|---|

Target count and the manual location (see bootstrap: in-repo or sibling).

## Open questions for the human
Numbered. Only what the code cannot answer: which unit is dead, which branch really ships, which of
two names for one concept is correct, what the acronym stands for.
````

## Rules

- **Read only, git included.** Scout runs no `fetch`, no `checkout`, no `switch`, no `commit` — see
  SKILL.md. It does not create the manual and does not fix anything it finds. Whatever branch the
  tree is on is the branch scout reads; the record says which one that was.
- **Do not read every file.** Scout answers "what is here and where should later work look", not
  "how does it work".
  Depth is `map`'s job, and a scout that reads deeply runs out of context before it reaches the
  last unit.
- Everything uncertain goes in Open questions rather than being resolved by assumption. The gate
  exists to spend the human's knowledge, and a question asked there costs one line instead of a
  wrong doc.
