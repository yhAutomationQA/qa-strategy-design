# API Testing Strategy

## Why API Testing Is the Highest ROI

API tests sit at the sweet spot. They're faster than UI tests, more reliable, and exercise the same logic an end user would hit — just without the UI getting in the way. A good API test suite catches backend bugs before they ever reach the frontend.

In most teams, API tests catch more regressions than unit and UI tests combined. Invest here first.

## Coverage Approach

You don't need to test every endpoint exhaustively. You need to test every endpoint *intelligently*.

### What Every Endpoint Needs

| Test Type | What It Covers | Priority |
|---|---|---|
| **Happy path** | Returns 200, correct response body, correct status | Required |
| **Validation** | Missing fields, wrong types, out of range, malformed input | Required |
| **Auth** | No token, expired token, wrong role, invalid token format | Required |
| **Edge cases** | Empty results, pagination boundaries, large payloads | High |
| **Error handling** | 4xx, 5xx responses, error message format, idempotency | High |
| **Performance** | Response time under load, timeout behavior | Medium |

### Skip These

- Testing every HTTP method on every endpoint if they're generated (OpenAPI spec handles this)
- Re-testing the framework (you don't need to verify that Express/Spring/Django returns 404 for unknown routes — trust your framework)
- Testing things that never change (static config endpoints, health checks beyond "is it alive")

### Real-World Example: `/api/orders`

**Happy path:** POST with valid payload → 201 + order object

**Validation tests:**
- Missing `customerId` → 400
- `customerId` with invalid format → 400
- `items` array empty → 400 or 422
- `total` negative → 400
- `total` with too many decimals → does it round, truncate, or reject?
- Extra unknown fields → does it ignore or reject? (depends on your API philosophy)

**Auth tests:**
- No auth header → 401
- Expired JWT → 401
- Valid JWT for different user → 403 (different customer's orders)
- Valid JWT but deleted user → varies by design (401 or 403)

**Edge cases:**
- POST with 1000 items → does it accept? time out? hit a limit?
- GET with no orders → returns `[]` or `{"orders": []}`?
- Pagination: `?page=-1`, `?limit=0`, `?limit=10000`
- Concurrent POST for same resource (race condition)

**Error tests:**
- Downstream payment service times out → does the API return 502 or 500?
- Database unavailable → returns 503 with a useful message (not a stack trace)

## Authentication Handling

Auth is where most API test suites are weak because it's a pain to set up. But it's also where most security bugs live.

### JWT

If your API uses JWTs, you need to test:

- **Token validity** — expired, not-yet-valid (wrong `iat`/`nbf`), malformed signature, tampered payload
- **Token scope** — can a token for the "read:orders" scope call a write endpoint?
- **Token refresh** — does the refresh flow return a new valid token? Can you use a refresh token twice? (you shouldn't be able to)
- **Anonymous access** — endpoints that should be public (/health, /docs) work without a token. Endpoints that shouldn't be public actually reject anonymous calls.

**Common miss:** Teams test auth once and assume it applies to all endpoints. It doesn't. New endpoints get added without auth middleware. Test auth on every endpoint, even if it's just one test per endpoint.

### OAuth

With OAuth flows, test:

- **Authorization code flow** — does the redirect work? does the token exchange succeed?
- **Client credentials flow** — machine-to-machine, used for internal services
- **Token introspection** — resource server validates token with authorization server
- **Error flows** — what happens when the user denies consent? what if the auth server is down?

**Strategic note:** Don't try to test the OAuth provider itself (Google, Auth0, Okta). They handle their own testing. Test *your integration* — the redirect handling, the token storage, the session creation after successful auth. Mock the provider response and test your code.

### API Key Auth

For internal/machine-to-machine APIs:

- Revoked key → rejected
- Key with wrong permissions → rejected with clear error
- Key that expired → rejected
- Rate limiting per key → blocked after N requests

### Common Auth Anti-Patterns to Catch

- **Hardcoded tokens in test suites** — tests that pass because they use a "magic" token that bypasses auth. These pass in CI but fail in production.
- **Auth tested only in the auth service** — the gateway validates the token, but the downstream service trusts whatever comes through. Test that downstream services also validate.
- **Assuming 401 means "auth failed"** — some APIs return 401 for "not authenticated" and 403 for "not authorized." Others use 401 for both. Check your convention and make sure it's documented.

## Schema Validation

This is the most underrated API testing practice. Most teams test that the response contains the fields they expect. Few teams validate the actual schema.

### Why It Matters

- APIs change. Someone adds a new field and doesn't tell the frontend. Someone renames a field and the mobile app crashes.
- API responses silently change types. A field that was a string becomes a number. An optional field becomes required.
- Error response formats are inconsistent. Some endpoints return `{error: "message"}`, others return `{message: "error"}`.

### What to Validate

- **Response structure** — all expected fields present, no unexpected fields (if you want strict mode)
- **Field types** — string, number, boolean, array, object, null
- **Field constraints** — min/max length, pattern (email, UUID, date format), enum values
- **Nullability** — fields that should never be null vs. fields that are legitimately nullable
- **Error response format** — consistent structure across all endpoints

### How We Do It

- Use OpenAPI/Swagger specs as the source of truth. Validate responses against the spec in CI. If the spec and the response don't match, the test fails.
- Add schema validation to every API test. It's one line of assertion and catches a huge class of regressions.
- When the spec changes, update the tests. When the implementation changes without updating the spec, flag it.

**Gotcha:** Don't validate the schema against the spec on every single test. Validate it once per endpoint in a dedicated test. Your happy path tests check the response body logic; your schema test checks the contract.

## The Reality of API Test Environments

The docs above assume you have a clean test environment with controlled data. In real life:

- **Staging is unreliable.** Someone is running a migration, or the test data got wiped, or the environment is down. Your API tests fail because of environment, not because of bugs.
- **You can't cleanly separate test data.** Other teams are also testing on the same staging environment. Their test runs delete your test records.
- **The API behaves differently in staging vs production.** Different config, different data volume, different caching behavior. Tests that pass in staging might fail in prod, and vice versa.

### How to Deal with It

- Make your tests create their own data. Don't assume pre-seeded data exists.
- Run API tests in CI against isolated environments (ephemeral environments, test containers).
- If you can't isolate, at least tag your test data so you can identify it when it gets corrupted.
- Accept that some API test failures are environment issues, not code issues. Build your triage process around this reality — first check if the environment is healthy, then investigate the code.

### Debugging in Production

Sometimes you have to. The staging environment doesn't have the same data volume. The bug only repros with real users and real data. You add logging, run a test against prod (read-only, on a test account), and catch the issue.

This is not ideal. It's also reality. If you're doing this, be careful:
- Never modify production data
- Never test against real user accounts
- Use feature flags to isolate test traffic
- Clean up any test data you create

The goal is to not need this. But sometimes you do. That's honest.

## Performance Considerations

API tests serve double duty. They verify correctness *and* give you a baseline for performance.

### Response Time Expectations

Every API test should have a loose response time assertion:

- Internal API (same data center): < 200ms
- External API (over internet): < 500ms
- Batch/reporting endpoint: < 5s (or async)

These should be loose enough to not be flaky, tight enough to catch performance regressions. If a test that usually takes 50ms suddenly takes 2s, something changed.

### What to Test

- **Baseline performance** — single request, normal load, average response time
- **Concurrent requests** — 10-50 parallel requests to the same endpoint
- **Payload size** — small payload vs. large payload response times
- **Caching behavior** — first request vs. second request (cached) times
- **Rate limiting** — what happens when you exceed the limit? 429 response? After how many requests?

### What NOT to Test in Your Functional Suite

- Full load tests (1000s of concurrent users) — that's dedicated performance testing
- Soak tests (running for hours) — different tool, different goal
- Stress tests (beyond breaking point) — important, but separate from your API regression

Keep performance *observations* in your functional suite. Keep dedicated performance *testing* separate.

### Real-World Gotchas

- **N+1 queries** — API returns quickly with 10 items, but slows to a crawl with 1000. Test with realistic data volumes.
- **Unindexed queries** — works fine in dev with 100 rows, falls over in prod with 10M. Can't catch this with functional tests alone, but you can flag it when response times spike.
- **Serialization overhead** — returning 1000 items with full nested objects is slow. Check if the API supports field selection or pagination.
- **Third-party latency** — your API chains calls to three downstream services. Response time = sum of all three. Test timeouts and circuit breakers.
