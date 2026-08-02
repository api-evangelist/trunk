---
name: Triage flaky tests
description: Find unhealthy/quarantined tests in a repo, inspect a test case, and link a tracking ticket.
api: openapi/trunk-openapi.yml
operations: [listUnhealthyTests, listQuarantinedTests, getTestDetails, linkTicketToTestCase]
---

# Triage flaky tests with the Trunk Flaky Tests API

Use the Trunk Flaky Tests API to surface unhealthy tests, inspect a specific
test case, and attach a Jira/Linear ticket for follow-up.

## Auth
- Header `x-api-token: <ORG_API_TOKEN>` on every request (Settings > Organization > General > API).
- Base URL: `https://api.trunk.io/v1`.

## Steps
1. **List unhealthy tests** — `listUnhealthyTests` (POST `/flaky-tests/list-unhealthy-tests`).
   Body: `repo` ({host, owner, name}), `org_url_slug`, `status` (`FLAKY` or `BROKEN`), and
   `page_query` ({page_size ≤ 100, page_token}). Page with `page.next_page_token`.
2. **List quarantined tests** (optional) — `listQuarantinedTests` (POST `/flaky-tests/list-quarantined-tests`)
   with the same `repo` / `org_url_slug` / `page_query`.
3. **Inspect a test case** — `getTestDetails` (POST `/flaky-tests/get-test-details`) with
   `repo`, `org_url_slug`, and the `test_id` (uuid) from step 1/2. Read `status`, failure
   rates, `most_common_failures`, `codeowners`, and `quarantined`.
4. **Link a ticket** — `linkTicketToTestCase` (POST `/flaky-tests/link-ticket-to-test-case`)
   with `repo`, `test_case_id` (uuid), and `external_ticket_id` (e.g. `KAN-123`).

## Conventions & errors
- Cursor pagination: send `page_query.page_token`, read `page.next_page_token`.
- Errors are `{ "message": string }` (see errors/trunk-problem-types.yml); `401` = bad/missing token.
- No idempotency key is documented (conventions/trunk-conventions.yml).
