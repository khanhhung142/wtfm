# Command: verify — does the manual still match the code

Documentation rots quietly. Nothing in a repository fails when a doc goes wrong, so the only signal is
a deliberate check. This mode produces that signal and nothing else: it reports, and updates the
ledger. Fixing is a `map` or `flow` session against the findings.

## Run this on a trigger, not on a schedule

If the manual was written to the one rule, it holds almost nothing a commit can falsify: addresses,
cross-file invariants, vocabulary, decisions. Addresses are caught mechanically, decisions cannot go
stale, and what is left needs a model only when the *design* changed — which is rare and visible.

| Trigger | Scope |
|---|---|
| The citation lint failed | Only the units with broken addresses |
| A unit was restructured, renamed or split | That unit |
| A documented invariant's guard test was deleted | That invariant |
| Somebody says a doc misled them | That doc — then ask what *class* of claim it was |
| Handing the codebase to a new team | Everything, once |

**Repeat findings are a content bug.** The third time one doc is Wrong, the fix is not a better verify
pass: that doc carries a fast-decaying claim. Move it into a comment, turn it into a test, or delete
it. A manual that needs frequent verification is a manual carrying sentences it should never have
written.

## Three questions, and only one needs a model

| # | Question | Cost | How |
|---|---|---|---|
| 1 | Do the addresses still resolve? | free | `check-citations.sh` |
| 2 | Does any doc contain banned content? | free | `check-content.sh` |
| 3 | Are the surviving claims still true? | a session | read the changed code |

Run 1 and 2 first. They are greps, they need no network and no model, and they catch the two failures
that account for most rot. Whatever is left is question 3, scoped to code that actually changed.

## Question 1: the citation lint

```bash
#!/usr/bin/env bash
# Resolve every `path/file.ext#Symbol` citation in the manual. Exit 1 if any is broken.
# Usage: ./check-citations.sh <manual-dir> <repo-root>
set -uo pipefail
manual="${1:-.}"; repo="${2:-.}"

broken=$(grep -rhoE '`[A-Za-z0-9_./-]+\.[A-Za-z]+#[A-Za-z_][A-Za-z0-9_.:-]*`' "$manual" \
  --exclude-dir=_run \
  | tr -d '`' | sort -u | while IFS='#' read -r file symbol; do
      if [ ! -f "$repo/$file" ]; then
        echo "GONE   $file#$symbol — file does not exist"
      elif ! grep -qF "$symbol" "$repo/$file"; then
        echo "GONE   $file#$symbol — symbol not found"
      fi
    done)

[ -z "$broken" ] && exit 0
echo "$broken"
exit 1
```

A green run proves an address exists. It says nothing about the prose around it.

## Question 2: the content lint

The banned content in SKILL.md is not a matter of taste — most of it has a shape a grep can find. This
catches a doc drifting back toward restating code **on the day it is written**, which is worth more
than catching its consequences a year later.

```bash
#!/usr/bin/env bash
# Flag content the manual is not supposed to contain. Advisory: exit 1 on hits, review each.
# Usage: ./check-content.sh <manual-dir>
set -uo pipefail
manual="${1:-.}"
ex="--exclude-dir=_run --exclude-dir=explain --include=*.md"

hit=0
flag() { # <label> <pattern>
  out=$(grep -rnE $ex "$2" "$manual" 2>/dev/null) || return 0
  [ -z "$out" ] && return 0
  echo "== $1"; echo "$out"; hit=1
}

flag "payload restated — cite the schema instead"      '^\s*[\"'\'']?[a-z_]+[\"'\'']?\s*:\s*[\"{[]'
flag "behaviour column — locators carry addresses"     '^\|.*\| *(Does|What it does|Behaviour|Logic) *\|'
flag "line-number citation — use path#Symbol"          '`[A-Za-z0-9_./-]+\.[A-Za-z]+:[0-9]+'
flag "version number — cite the lockfile"              '^\|.*\|[^|]*\b[0-9]+\.[0-9]+\.[0-9]+\b'
flag "derived count — give the command instead"        '\b([0-9]{2,}) (endpoints|routes|tables|handlers|keys)\b'
flag "expiring marker — use 📋 not built at <sha>"      '(🟡|partial|✅ implemented|verified on)'

exit $hit
```

Every hit is a line to look at, not a verdict: a fenced example may legitimately contain a payload, and
a stack table may need one pinned version. Advisory by design — the point is that the writer sees it,
not that a build breaks. `explain/` is excluded because an explainer's job is prose with real numbers
in it.

Offer both scripts at gate 1 alongside the commit question. Never install either unasked: they are
files in somebody's repository and possibly steps in their CI.

## Question 3: the judgement pass

What is left is what no script can do: the link resolves, the anchor is right there, and the claim
above it is false now.

**Use guards to prioritise, not to prove.** Invariants written by `map` end in `Guarded by:` — a test,
or `nothing`. A named test is evidence only after confirming it exercises the claim and runs in CI.

Read unguarded claims first, then claims whose guard disappeared or whose code changed since the doc's
revision. Review guarded claims only when either the test or the code it covers moved.

## Method

1. Read every doc's frontmatter. Compare `read:` against the unit's current head. Unmoved unit, skip.
2. For each moved unit: `git -C <repo> diff --name-only <that-sha>..HEAD`
3. Run both lints where installed.
4. Check the structural claims, which rot without any citation moving: the authored-sources table
   against the current tooling, the layer rules against current import direction.
5. Review changed files intersecting documented invariants and flow ordering constraints. The same
   symbol can survive while its behaviour changes.
6. Look for what was added and never indexed — a new entry point or boundary absent from the manual.
7. Check every `📋 not built at <sha>` row: built since means the doc is out of date, not wrong.

## Freshness, and what a stale doc is still good for

Staleness is not binary and not a verdict. One number decides how much of a doc to trust:

```bash
git -C <repo> log --oneline <doc-sha>..HEAD -- <unit path> | wc -l
```

| Commits behind | Read the doc as |
|---|---|
| 0 | A snapshot of that revision. Critical claims still need confirming in code |
| 1–20 | A map. Use the addresses, then review the changed files |
| Many, or a rename between | A hypothesis. Use it to know what to open |

A doc's addresses usually outlive its details, which is why an old doc still navigates. Say what
remains useful instead of stamping one binary "stale".

## Findings

| Kind | Meaning | Action |
|------|---------|--------|
| **Wrong** | The code changed, the claim is now false | Re-run `map` or `flow` for that area |
| **Gone** | The cited file or symbol no longer exists | Re-run, and check whether the feature was removed |
| **Banned** | A doc contains content the one rule forbids | Cut it, or move it into the code |
| **Missing** | Code exists that no doc mentions | New target in the ledger |

**Wrong outranks Missing.** An absent doc makes an agent read the code. A false doc makes it skip the
code and act on the falsehood.

**Banned outranks Wrong**, because it is the cause. A Wrong finding is one sentence to fix; a Banned
finding is a sentence that will be Wrong again next quarter.

## Report

````markdown
## verify — <YYYY-MM-DD>

| Unit | Doc revision | Head | Gone | Banned | Wrong | Missing |
|------|-------------|------|------|--------|-------|---------|

### Banned
- `services/api/index.md:41` — locator table has a `Does` column. Cut the column.

### Wrong
- `services/api/index.md` invariant 2 claims every read is tenant-scoped.
  `internal/db/hooks.go#BeforeQuery` now skips scoping for admin contexts. Changed in `a1b2c3d`.

### Missing
- `internal/http/refund.go` — three endpoints, no doc.
````

Then write it into `_run/progress.md`: flip affected cells to `🔄`, add the session-log row, and put
each Wrong finding in the open-questions table so it is citable and cannot be quietly dropped.

## Rules

- **Never skip `decisions/`, and never age it.** An ADR describes a moment, not a state; a commit
  moving underneath it does not make it wrong. Code that has diverged from an accepted decision *is*
  the finding: either the decision was quietly abandoned — write the superseding one — or the code
  drifted from a rule somebody still relies on. Both beat any broken-link fix in this report.
- **The glossary rots silently and matters most.** A renamed type leaves the old word in every doc
  written before it, and a stale word is worse than a broken link: a broken link is visible, a stale
  word gets used.
- **Classify the document before resolving a conflict.** Implementation prose disagreeing with code is
  **Wrong**. An approved spec disagreeing with code is drift. Verify reports it and adjusts neither.
- **An invariant whose guard was deleted outranks a broken link.** The doc still reads as true, CI
  stopped disagreeing, and nothing is checking it now.
- Report counts, not adjectives. "Nine of forty-one claims wrong in one unit" is actionable; "some
  drift" is not.
- **Never re-stamp a doc's `read:` revision without checking its claims.** That launders a stale doc
  into a fresh-looking one, which is worse than leaving it visibly old.
- Verify reports; fixing content is a focused `map` or `flow` session afterwards.
