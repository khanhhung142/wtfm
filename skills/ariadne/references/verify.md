# Mode: verify — does the atlas still match the code

Documentation rots quietly. Nothing in a repository fails when a doc goes wrong, so the only signal
is a deliberate check. This mode produces that signal, and produces nothing else: it reports and
updates the ledger. Fixing is a `map` or `flow` session against the findings.

Run it after a merge that touched a documented unit, before trusting the atlas for a large piece of
work, and on a schedule if the project is moving fast.

## Method

1. Read every doc's frontmatter. Compare `commit` against the unit's current head on the recorded
   branch. A unit whose head has not moved cannot have stale citations; skip it entirely, which is
   what makes this cheap enough to run often.
2. For each moved unit, get the changed files:
   ```bash
   git -C <repo> diff --name-only <doc-commit>..HEAD
   ```
3. Extract every `path:line` citation from that unit's docs. Check each one that lands in a changed
   file: does the line still exist, and does it still say what the doc claims?
4. Check the structural claims too, since they rot without any citation moving: an endpoint list
   against the current interface definitions, a table list against the current migrations, a config
   table against the current template.
5. Look for what was added and never documented. A new endpoint or table absent from the docs is a
   gap, and gaps are the finding most likely to mislead, because the doc looks complete.

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
- `services/api/api.md:88` claims `POST /orders` requires admin. `internal/http/order.go:23` now
  checks membership only. Changed in `a1b2c3d`.

### Missing
- `internal/http/refund.go` — three endpoints, no doc.
````

Then write it into `_progress.md`: flip the affected cells to `🔄`, add the session-log row, and put
each **Wrong** finding in the open-questions table so it is citable and cannot be quietly dropped.

## Rules

- Verify does not edit docs, beyond fixing a **Moved** line number in place. Everything else is a
  documenting session, and doing it here means doing it without having read the surrounding code.
- Report counts, not adjectives. "Nine of forty-one citations wrong in one unit" is actionable where
  "some drift" is not.
- Never re-stamp a doc's frontmatter commit without checking its citations. Doing so launders a
  stale doc into a fresh-looking one, which is worse than leaving it visibly old.
