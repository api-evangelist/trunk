---
name: Control the merge queue
description: Submit a pull request to the flake-aware merge queue, poll its state, and cancel or restart it.
api: openapi/trunk-openapi.yml
operations: [submitPullRequest, getSubmittedPullRequest, restartTestsOnPullRequest, cancelPullRequest]
---

# Control the Trunk Merge Queue

Drive the flake-aware parallel Merge Queue: submit a PR, watch its state, and
restart or cancel it.

## Auth
- Header `x-api-token: <ORG_API_TOKEN>` on every request.
- Base URL: `https://api.trunk.io/v1`.

## Steps
1. **Submit** — `submitPullRequest` (POST `/submitPullRequest`) with a `PullRequestRef`:
   `repo` ({host, owner, name}), `targetBranch` (e.g. `main`), and `pr` ({number}).
   Response is a `PullRequestState`.
2. **Poll state** — `getSubmittedPullRequest` (POST `/getSubmittedPullRequest`) with the same
   `PullRequestRef`. `state` progresses through `NOT_READY → PENDING → TESTING →
   TESTS_PASSED → MERGED`, or `FAILED` / `CANCELLED` / `PENDING_FAILURE`.
3. **Restart tests** (on flake) — `restartTestsOnPullRequest` (POST `/restartTestsOnPullRequest`)
   with the `PullRequestRef`.
4. **Cancel** — `cancelPullRequest` (POST `/cancelPullRequest`) with the `PullRequestRef`.

## Conventions & errors
- Errors are `{ "message": string }` (errors/trunk-problem-types.yml); `401` = bad/missing token.
- Subscribe to `pull_request.*` webhooks (asyncapi/trunk-webhooks.yml) instead of tight polling.
- Merge Queue health is exposed as Prometheus metrics via `getMergeQueueMetrics`.
