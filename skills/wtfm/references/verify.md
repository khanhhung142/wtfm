# Command: verify — does the manual still match the code

Documentation rots quietly. Nothing in a repository fails when a doc goes wrong, so the only signal
is a deliberate check. This mode produces that signal, and produces nothing else: it reports and
updates the ledger. Fixing is a `map` or `flow` session against the findings.

## Two questions, and only one needs a model

Verifying a manual feels expensive because it is usually done as one pass answering two questions
that have nothing in common:

1. **Do the citations still resolve?** Purely mechanical. A string, a file, a line. No judgement, no
   context, no reading.
2. **Are the claims still true?** Judgement. Requires reading the code and knowing what the doc
   meant.

Run them together and the second one's cost gets paid on every citation in the manual, most of which
only needed the first. Split them, and the shape changes completely: question 1 becomes a lint that
runs free on every commit, and question 2 becomes a small pass over the handful of places the lint
flagged.

**Do not run a `verify` session until the lint is red.** A green lint on an unchanged unit means
there is nothing for a model to look at.

## Question 1: the lint

Write this once into the manual as `check-citations.sh`. It needs no model, no network and no
dependency beyond `grep` and `sed`, and it is the reason `verify` stops being a chore.

```bash
#!/usr/bin/env bash
# Resolve every `path/file.ext:LINE Symbol` citation in the manual. Exit 1 if any is stale.
# Usage: ./check-citations.sh <manual-dir> <repo-root>
set -uo pipefail
manual="${1:-.}"; repo="${2:-.}"

stale=$(grep -rhoE '`[A-Za-z0-9_./-]+\.[A-Za-z]+:[0-9]+ [A-Za-z_][A-Za-z0-9_]*`' "$manual" \
  | tr -d '`' | sort -u | while read -r loc sym; do
      file="${loc%:*}"; line="${loc##*:}"
      if [ ! -f "$repo/$file" ]; then
        echo "GONE   $loc $sym — file does not exist"
      elif sed -n "${line}p" "$repo/$file" | grep -qF "$sym"; then
        :                                          # anchor still on the cited line
      elif found=$(grep -nF "$sym" "$repo/$file" | head -1 | cut -d: -f1) && [ -n "$found" ]; then
        echo "MOVED  $file:$line -> $file:$found $sym"
      else
        echo "GONE   $loc $sym — anchor not in file"
      fi
    done)

[ -z "$stale" ] && exit 0
echo "$stale"
exit 1
```

Wire it wherever the project already fails builds — a pre-commit hook, a CI step, a `make` target.
Every **Moved** and every **Gone** is now caught by a machine the day it happens, for free, and each
one prints the replacement line number.

Offer this at bootstrap; never install it without asking. It is the one part of this skill that adds
a file to somebody's build.

## Question 2: the judgement pass

What is left is the part no script can do: the citation resolves, the anchor is right there, and the
claim above it is nonetheless false now. This is the `verify` session, and it should be small.

**Read the guards before reading the code.** Traps written by `map` end with `Guarded by:` — either a
test, or `nothing`. A claim guarded by a passing test has a machine checking it on every CI run; it
does not need you. A claim guarded by `nothing` has nobody checking it at all, which is where a
silent falsehood lives.

So the reading order for this pass is: unguarded claims first, then claims whose named guard no
longer exists — a deleted test is a claim that lost its guard without anyone noticing — and only then
anything else, if there is budget. A `verify` session that reads a unit top to bottom is one that
spent most of its context confirming what CI already confirmed.

## Method

1. Read every doc's frontmatter. Compare `commit` against the unit's current head on the recorded
   branch. A unit whose head has not moved cannot have stale citations; skip it entirely, which is
   what makes this cheap enough to run often.
2. For each moved unit, get the changed files:
   ```bash
   git -C <repo> diff --name-only <doc-commit>..HEAD
   ```
3. Run `check-citations.sh` and take its output as given. Do not re-resolve citations by hand — that
   is the work you just automated. If the lint is not installed, do it by hand this once, in this
   order:
   ```bash
   sed -n '34p' internal/user/register.go          # is the anchor still on the cited line?
   grep -n 'Register' internal/user/register.go    # only if it is not: where did it go?
   ```
   Anchor on the line and unchanged → nothing to do. Anchor found elsewhere in the file → **Moved**,
   fix the number in place, no reading required. Anchor gone from the file → **Gone**, and now go
   read. Anchor there but the surrounding code no longer does what the doc says → **Wrong**.

   This is the whole reason citations carry an anchor. Without one, every citation below an inserted
   line is silently off and the only way to tell is to re-read the file; with one, the common case is
   a `grep` and a number.
4. Check the structural claims too, since they rot without any citation moving: an endpoint list
   against the current interface definitions, a table list against the current migrations, a config
   table against the current template.
5. Look for what was added and never documented. A new endpoint or table absent from the docs is a
   gap, and gaps are the finding most likely to mislead, because the doc looks complete.

## Freshness, and what a stale doc is still good for

Staleness is not binary and it is not a verdict on the doc. One number decides how much of a doc to
trust, and it costs one command:

```bash
git -C <repo> log --oneline <doc-commit>..HEAD -- <unit path> | wc -l
```

| Commits behind | Read the doc as |
|---|---|
| 0 | Fact. Citations resolve; act on them |
| 1–20 | Map. Paths and structure hold, line numbers are suspect — jump by anchor, not by number |
| Many, or a rename in between | Hypothesis. Use it to know what to open, verify before quoting |

A doc's paths outlive its line numbers, and its shape outlives its paths. That is why an old doc is
still worth reading: it is wrong about details long before it is wrong about where to look. Say this
in the report rather than marking a unit "stale", which reads as "ignore this file" and throws away
the part that still works.

## Findings

Four kinds, and they are not equally urgent:

| Kind | Meaning | Action |
|------|---------|--------|
| **Moved** | The line shifted, the claim holds | Fix the number in place |
| **Wrong** | The code changed, the claim is now false | Re-run `map` or `flow` for that area |
| **Gone** | The cited file or symbol no longer exists | Re-run, and check whether the feature was removed |
| **Missing** | Code exists that no doc mentions | New target in the ledger |

**Wrong** outranks **Missing**. An absent doc makes an agent go read the code. A false doc makes it
skip reading the code and act on the falsehood.

## Report

````markdown
## verify — <YYYY-MM-DD>

| Unit | Doc commit | Head | Citations | Moved | Wrong | Gone | Missing |
|------|-----------|------|-----------|-------|-------|------|---------|

### Wrong
- `services/api/api.md:88` claims `POST /orders` requires admin. `internal/http/order.go:23 Create`
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
  drifted from a rule somebody still relies on. Both are worth more than any citation fix in this
  report.
- **The glossary rots silently and matters most.** A renamed type leaves the old word in every doc
  written before it, and a wrong word is worse than a wrong line number: a stale line number gets
  noticed on the first read, a stale word gets used.
- **When a doc and the code disagree, the code is right.** Record the doc as **Wrong** and move on.
  Never file a finding against the code on a doc's authority, and never adjust the code to match.
  This mode reports on documentation only.
- Verify does not edit docs, beyond fixing a **Moved** line number in place. Everything else is a
  documenting session, and doing it here means doing it without having read the surrounding code.
- **A trap whose guard was deleted outranks a wrong line number.** The doc still reads as true, CI
  stopped disagreeing, and nothing anywhere is checking it now.
- Report counts, not adjectives. "Nine of forty-one citations wrong in one unit" is actionable where
  "some drift" is not.
- Never re-stamp a doc's frontmatter commit without checking its citations. Doing so launders a
  stale doc into a fresh-looking one, which is worse than leaving it visibly old.
