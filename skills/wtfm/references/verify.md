# Command: verify — does the manual still match the code

Documentation rots quietly. Nothing in a repository fails when a doc goes wrong, so the only signal
is a deliberate check. This mode produces that signal, and produces nothing else: it reports and
updates the ledger. Fixing is a `map` or `flow` session against the findings.

## Two questions, and only one needs a model

Verifying a manual feels expensive because it is usually done as one pass answering two questions
that have nothing in common:

1. **Do navigation links still resolve?** Purely mechanical. A path and optional symbol. No
   judgement or code reading.
2. **Are the claims still true?** Judgement. Requires reading the code and knowing what the doc
   meant.

Run them together and the second one's cost gets paid on every link in the manual, most of which
only needed the first. Split them, and the shape changes completely: question 1 becomes a cheap
lint, while question 2 is scoped by code changes since the recorded commit. A green lint proves
only that an address still exists; logic can change underneath the same function name.

## Question 1: the optional link lint

Offer this as `check-citations.sh` when the project benefits from it. It needs no model or network.
It checks only `path#Symbol` links; it does not verify the prose around them.

```bash
#!/usr/bin/env bash
# Resolve every `path/file.ext#Symbol` citation in the manual. Exit 1 if any is broken.
# Usage: ./check-citations.sh <manual-dir> <repo-root>
set -uo pipefail
manual="${1:-.}"; repo="${2:-.}"

broken=$(grep -rhoE '`[A-Za-z0-9_./-]+\.[A-Za-z]+#[A-Za-z_][A-Za-z0-9_.:-]*`' "$manual" \
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

Wire it into an existing pre-commit hook, CI step or `make` target only when the human opts in.
Every **Gone** address is then caught mechanically.

Offer this at bootstrap; never install it without asking. It is the one part of this skill that adds
a file to somebody's build.

## Question 2: the judgement pass

What is left is the part no script can do: the link resolves, the anchor is right there, and the
claim above it is nonetheless false now. This is the `verify` session, and it should be small.

**Use guards to prioritise, not to prove.** Traps written by `map` end with `Guarded by:` — either a
test, or `nothing`. A named test is evidence only after confirming that it exercises the claim and
runs in CI. It does not make the prose permanently true.

Read unguarded claims first, then claims whose guard disappeared or whose implementation changed
since the doc commit. Review guarded claims when either the test or the code it covers changed.

## Method

1. Read every doc's frontmatter. Compare `commit` against the unit's current head on the recorded
   branch. If a previously verified unit has not moved, skip it.
2. For each moved unit, get the changed files:
   ```bash
   git -C <repo> diff --name-only <doc-commit>..HEAD
   ```
3. Run `check-citations.sh` when installed. A missing path or symbol is **Gone**. A resolved link
   proves only that navigation still works.
4. Check the structural claims too, since they rot without any citation moving: an endpoint list
   against the current interface definitions, a table list against the current migrations, a config
   table against the current template.
5. Review changed files that intersect documented traps, invariants and flows. The same symbol can
   remain while its behaviour changes.
6. Look for what was added and never indexed. A new entry point or boundary absent from the manual
   is a gap when the manual claims to route that area.

## Freshness, and what a stale doc is still good for

Staleness is not binary and it is not a verdict on the doc. One number decides how much of a doc to
trust, and it costs one command:

```bash
git -C <repo> log --oneline <doc-commit>..HEAD -- <unit path> | wc -l
```

| Commits behind | Read the doc as |
|---|---|
| 0 | Recorded snapshot. Critical claims may still need direct confirmation |
| 1–20 | Map. Use paths and anchors, then review the changed files |
| Many, or a rename in between | Hypothesis. Use it to know what to open, verify before quoting |

A doc's paths often outlive its behavioural details. That is why an old doc may still be useful for
navigation. Say what remains useful instead of applying one binary "stale" label.

## Findings

Three kinds, and they are not equally urgent:

| Kind | Meaning | Action |
|------|---------|--------|
| **Wrong** | The code changed, the claim is now false | Re-run `map` or `flow` for that area |
| **Gone** | The cited file or symbol no longer exists | Re-run, and check whether the feature was removed |
| **Missing** | Code exists that no doc mentions | New target in the ledger |

**Wrong** outranks **Missing**. An absent doc makes an agent go read the code. A false doc makes it
skip reading the code and act on the falsehood.

## Report

````markdown
## verify — <YYYY-MM-DD>

| Unit | Doc commit | Head | Links | Wrong | Gone | Missing |
|------|-----------|------|-------|-------|------|---------|

### Wrong
- `services/api/api.md#POST-orders` claims `POST /orders` requires admin.
  `internal/http/order.go#Create`
  now checks membership only. Changed in `a1b2c3d`.

### Missing
- `internal/http/refund.go` — three endpoints, no doc.
````

Then write it into `_progress.md`: flip the affected cells to `🔄`, add the session-log row, and put
each **Wrong** finding in the open-questions table so it is citable and cannot be quietly dropped.

## Rules

- **Never skip `decisions/`, and never age it.** An ADR describes a moment, not a state, so a commit
  moving underneath it does not make it wrong. Code that has diverged from an accepted decision is
  the finding: either the decision was quietly abandoned — write the superseding one — or the code
  drifted from a rule somebody still relies on. Both are worth more than any broken-link fix in this
  report.
- **The glossary rots silently and matters most.** A renamed type leaves the old word in every doc
  written before it, and a wrong word is worse than a broken link: a broken link is visible, while
  a stale word gets used.
- **Classify the document before resolving a conflict.** An implementation map that disagrees with
  code is **Wrong**. An approved spec that disagrees with code is implementation drift. Verify
  reports the mismatch and never adjusts either side.
- Verify reports; fixing content is a focused `map` or `flow` session after reading the surrounding
  code.
- **A trap whose guard was deleted outranks a broken link.** The doc still reads as true, CI
  stopped disagreeing, and nothing anywhere is checking it now.
- Report counts, not adjectives. "Nine of forty-one claims wrong in one unit" is actionable where
  "some drift" is not.
- Never re-stamp a doc's frontmatter commit without checking its links and claims. Doing so launders a
  stale doc into a fresh-looking one, which is worse than leaving it visibly old.
