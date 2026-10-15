---
name: lgtm
description: "LGTM = let's get this merged. Take the current branch from finished code to a merged, prod-verified PR. Rates the PR into a risk tier (T0 trivial to T3 high-risk), then runs the steps that tier needs: review with /simplify and /code-review, settle every finding, get CI green, smoke test the preview, merge, smoke test prod and check the error tracker, opening a revert PR on a regression. User-invoked only (/lgtm)."
disable-model-invocation: true
---

# /lgtm: let's get this merged

Review, merge, verify.

This flow **is** the review: nothing downstream is counted on to catch what it misses. How much runs depends on the PR's tier. Every step the tier calls for runs, in order, and a step that cannot finish stops the flow with the exact blocker. Never skip a step to get a merge through.

Base branch is `main` unless the user names another.

## Tiers

Rate the PR right away, and re-rate after every push. If the user named a tier (`/lgtm T1`), use it.

| Tier | Name | In one line |
| --- | --- | --- |
| **T0** | trivial | Nothing that ships to a user changes (docs, tests only, agent config) |
| **T1** | small | Up to ~150 lines in one project, no shared library, no dependency change, far from money and data |
| **T2** | standard | Everything else |
| **T3** | high-risk | Touches money, stored data, auth, orders, or how things deploy, at any size |

**Two passes.** First a mechanical score from paths and size (`git diff --stat origin/main...HEAD`). Then read the diff and **raise** the tier (never lower it) when the change:

- computes a price, total, discount, tax, fee or refund: T3
- changes order or payment state: T3
- changes who can see or do something (auth, roles, scoping): T3
- deletes or rewrites stored data, or changes what a cron/queue writes: T3
- removes a feature flag, or changes a shared contract (API shape, exported type): T2

Tip: list your repo's actual high-risk paths (migrations, payments, auth, CI workflows) here, so pass 1 is mechanical.

| Step | T0 | T1 | T2 | T3 |
| --- | --- | --- | --- | --- |
| 1 errors in mind | skip | yes | yes | yes |
| 2 open PR | ready | ready | draft | draft |
| 3 review | skip | `/code-review --comment` | `/simplify`, then `/code-review --comment` | `/simplify`, then `/code-review high --comment` |
| 4 settle findings | skip | yes | yes | yes |
| 5 CI green | yes | yes | yes | yes, plus typecheck dependents locally |
| 6 preview smoke | skip | skip | yes | yes; no preview means a local smoke |
| 7 merge | yes | yes | yes | yes |
| 8 prod verify | skip | confirm deploy + errors | full | full, then check errors again 30 min later |

## Step 1: build it with errors in mind

Read the diff for how its failures would surface in prod. New code that can fail must report to your error tracker (Sentry or similar), not only log to stdout, or step 8 cannot see it.

## Step 2: open the PR

Reuse the branch's PR if one exists. Otherwise push and open it with a HEREDOC body so markdown survives the shell:

```bash
git push -u origin HEAD
gh pr create --draft --title "<type>(<area>): <description>" --body "$(cat <<'BODY'
## Changes

- <one bullet per meaningful change>

Part of #<issue>
BODY
)"
```

**Link the issue with `Part of #<n>`, never `Closes`/`Fixes`.** An issue often carries several PRs, and closing is `/close`'s job once everything has shipped.

## Step 3: review

1. `/simplify`: applies reuse and cleanup fixes. Commit and push what it changes.
2. `/code-review --comment`: posts its findings as inline PR comments.

## Step 4: settle every finding

Each finding is either fixed (commit + push) or answered with a reason it does not apply (reply on the comment). None left silent. A non-trivial fix gets `/code-review --comment` again. Then `gh pr ready <pr>` and a comment like `reviewed (T2): simplify / code-review, all settled`.

## Step 5: CI green

```sh
gh pr checks <pr>
gh pr view <pr> --json mergeable,mergeStateStatus
```

- Failed check: read it (`gh run view <run> --log-failed`). Flaky or infra: rerun once (`gh run rerun <run> --failed`). Caused by this PR: fix and push. Broken on `main`: stop and report.
- `CONFLICTING`: merge `origin/main` in, resolve, push. Stop if a conflict needs a real decision.
- Pending is not green.

## Step 6: smoke test the preview

Try the change on the preview deploy the way a user would (see `/drive`): load the page, exercise the changed flow, read console and network, screenshot the result. App: run the Maestro flow. API: call the endpoint. No preview: say so and smoke test locally. A bug found here gets fixed and goes back through step 5.

## Step 7: merge

Re-check CI and `mergeable` right before merging:

```sh
gh pr merge <pr> --squash --delete-branch --subject "<title> (#<pr>)"
```

Keep closing keywords out of the squash title and body too.

If merging is not the release for this project (mobile app needing an OTA or store build, a service deployed by hand), say so right after the merge, `Merged, not shipped yet: <what is left>`, and ship it or name who must.

## Step 8: smoke test prod and watch errors

1. **Confirm it deployed.** Find the deploy run for the merge commit and check it finished. Merged is not deployed. Verify against what prod actually serves.
2. **Repeat step 6 on prod.**
3. **Check for new errors** since the merge in your error tracker and logs. Get a baseline before calling anything an incident.

**On a regression, open a revert PR right away**, without waiting to be asked:

```sh
git fetch origin
git switch -c revert/<original-branch> origin/main
git revert --no-edit <merge-sha>
git push -u origin HEAD
gh pr create --base main --title 'revert: <original title>' --body '<what broke, evidence>'
```

## Output

Lead with the outcome: merged and verified, merged but not shipped (with what is left), reverted, or blocked. Then: PR link, tier with a one-line reason, findings and how each was settled, CI state, preview and prod smoke results with evidence, error tracker state. When blocked: the exact blocker and what would unblock it.
