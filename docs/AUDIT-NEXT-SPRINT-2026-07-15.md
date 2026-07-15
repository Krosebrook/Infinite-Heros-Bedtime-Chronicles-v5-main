# Audit & Next-Sprint Proposal — 2026-07-15

This is a companion note to the full audit done in the canonical repo:
`chaosclubco/infinite-heros-bedtime-chronicles-v5` PR #353
(`docs/AUDIT-NEXT-SPRINT-2026-07-15.md` there — a private repo, so no live
link here; see that repo directly). This note covers the two findings
specific to **this** repo.

## Finding 1: this repo never actually converged to the canonical tree

This repo's own `CLAUDE.md` and the canonical repo's `CLAUDE.md` both
describe the 2026-07-13 three-repo merge as leaving this repo with "no
application source ... contributed nothing beyond receiving the merged tree
so all three repos converge on identical content." **That's not true of this
repo's actual `main`.** Checked directly (this repo's live GitHub state, not
a possibly-stale local checkout):

- `main` is still the **pre-merge** codebase: Firebase Admin auth (not
  Supabase), Expo SDK 54 / React Native 0.85, plaintext parent-controls PIN,
  `server/routes.ts` as one ~550-line file, no content-safety guard, no
  cloud sync, no COPPA consent screen.
- `krosebrook/bedtime_chronicles-v2` (the other sibling repo) got a real
  convergence PR (#3, "adopt canonical v5 super-version tree") — this repo
  never got the equivalent.

## Finding 2: CI is broken on `package-lock.json` drift (fixed on this branch)

The most recent CI run on `main` (run `29221776371`, 2026-07-13) fails at
the "Verify package-lock.json is in sync with package.json" step — before
lint, test, typecheck, or `npm audit` even run, so this repo has had **zero
CI visibility** since then. Reproduced locally with the exact command CI
uses (`npm install --package-lock-only --ignore-scripts && git diff --exit-code
package-lock.json`): it fails because `package-lock.json` is missing `libc`
metadata fields that the installed npm version regenerates. This is the
exact drift class this repo's own `.claude/CLAUDE.md`-equivalent gotcha
already warns about.

**This branch includes the fix**: `package-lock.json` regenerated via
`npm install --package-lock-only --ignore-scripts` (matching CI's own
verification step exactly). The diff is an 18-line removal of stale `libc`
fields — mechanical, no dependency version changes.

## Recommendation

1. Merge the lockfile fix on this branch first, so CI actually runs on this
   repo again.
2. Then run this repo through the same convergence process `v2` went
   through in PR #3 — replacing `main`'s tree with the canonical repo's, per
   the documented (but not yet executed, for this repo) merge plan.

Full WSJF scoring and the rest of the cross-repo picture is in the canonical
repo's audit doc referenced above.
