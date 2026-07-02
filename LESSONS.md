# Lessons: breaking and fixing the CI/CD flow

A Week 0 walkthrough of intentionally turning the CI build **red**, then **green**
again on the starter service — the cheapest possible demonstration of the CI
feedback loop.

## What we did

1. **Broke CI** — one-line logic bug in `app/main.py`: the failing-tests penalty
   in `deployment_risk_score` changed `0.5 → 0.4`. This fails
   `test_failing_tests_add_risk` (which expects the score to jump by exactly 0.5
   when tests are failing). Local result: `1 failed, 6 passed`.
2. **Committed & pushed** on a branch `break-ci-demo` → the `pytest -q` step in
   `.github/workflows/ci.yml` failed and the CI check went **red**.
3. **Reverted** the bug (`0.4 → 0.5`), confirmed `7 passed`, committed and pushed
   → CI went **green**.

The branch keeps a realistic `red commit → green fix` history.

## Lessons learned

- **A logic bug beats a syntax error for teaching a red build.** The app still
  imports and runs, so the failure surfaces in the test layer — where CI is
  meant to catch it — instead of blowing up at parse time.
- **Push the failing state as its own commit.** The `red → green` trail on one
  branch is the realistic story of catching and fixing a regression, and it's
  what shows up on the Actions / PR timeline.
- **Test the pure function, break the pure function.** `deployment_risk_score`
  is deterministic and unit-tested, so a tiny change gives a precise, explainable
  failure (`0.4 == 0.5`) rather than a flaky or vague one.
- **`on: push: branches: ["**"]`** means every branch runs CI, so you can see the
  check without opening a PR — though a PR also shows the status inline.
- **Cheap by design.** The whole loop cost only ~0.1s pytest runs plus a couple
  of free GitHub Actions minutes — no Docker build, no API calls, no billing.

## Cleanup

```bash
git checkout main && git branch -D break-ci-demo && git push origin --delete break-ci-demo
```
