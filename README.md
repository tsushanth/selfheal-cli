# selfheal-cli

MVP for "self-maintaining APIs": given a dependency's breaking-change
release notes and a target repo, generate a migration patch and open a
real PR for human review.

## How it works

```
./selfheal.sh <target-repo-dir> <changelog-file>
```

1. Creates a branch in the target repo.
2. Hands the changelog + the target codebase to `claude -p` (headless) with
   instructions to find and update every affected call site, conservatively.
3. Commits whatever Claude changed, pushes the branch, opens a PR via `gh`.
4. If nothing changed, aborts and cleans up the branch — no empty PRs.

## Proof of concept

Ran end-to-end against
[selfheal-demo-target](https://github.com/tsushanth/selfheal-demo-target), a
small owned repo seeded with a deliberately outdated SDK call and a
simulated v2 changelog. Result:
[PR #1](https://github.com/tsushanth/selfheal-demo-target/pull/1) — found
the one affected call site, updated it to the new API shape.

Deliberately validated against an **owned** repo, not a real third-party
OSS project — an unreviewed auto-migration tool opening PRs on other
people's repos is a good way to get flagged as spam, and the fix quality
isn't trustworthy enough yet for that (see gap below).

## Known gap (not yet fixed)

The demo PR missed that the changelog said the new API **returns a
Promise** — it patched the call site to the new shape but didn't propagate
async correctness to the caller. A real self-maintaining-API product needs
a verification stage (type-check / test run / lint) before opening the PR,
not just "a diff was produced." That verification stage is the actual
hard, valuable part of this problem — the patch generation is the easy 80%.

## What's not built yet

- No mechanism for actually *watching* a dependency for new releases —
  this MVP takes the changelog as a manual input.
- No verification/test-run gate before opening the PR (see above).
- No handling for migrations that need multi-file or config changes beyond
  simple call-site rewrites.
