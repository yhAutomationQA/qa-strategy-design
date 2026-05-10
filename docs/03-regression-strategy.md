# Regression Strategy

## What Regression Actually Means

Regression testing is answering one question: *"Did we break anything that used to work?"*

That's it. It's not about finding new bugs in new features. It's about making sure the old stuff still works after you changed something. In practice, regression is where most QA time gets swallowed, so getting this right matters.

## How Regression Suites Evolve in Real Teams

No one designs a regression suite from scratch. It grows organically, and usually badly, until someone takes the time to prune it.

### The Natural Lifecycle

1. **Startup mode** — No regression suite. You test whatever you remember. Works for 2-3 developers.
2. **Bug-driven** — Every production bug becomes a test case. The suite grows without structure. This is where most teams live.
3. **Categorized** — Someone realizes the suite is unmanageable and splits it into smoke, critical, and full regression.
4. **Pruned** — Tests that haven't failed in 6 months get questioned. Flaky tests get deleted. The suite becomes intentional.

Most teams are stuck between phase 2 and 3. That's fine. The goal is phase 4.

### A Realistic Structure

```
Smoke (5-10 min)
  ├── Sanity check deploy: login, home page loads, basic read API responds
  ├── Runs on every deploy
  └── If this fails, don't bother with anything else

Critical Path (15-30 min)
  ├── Core user journeys: signup, checkout, payment, notifications
  ├── Runs nightly or before every release
  └── Must pass. Blocking.

Full Regression (1-4 hours)
  ├── Everything automated: all APIs, edge cases, integrations
  ├── Runs nightly or weekly depending on deploy frequency
  └── Failures are triaged, not necessarily blocking
```

**Reality:** Most teams don't have a clean 3-tier suite. They have what I call the "spaghetti suite" — 200 tests that take 45 minutes, half of which are flaky, and no one knows what they actually cover. If that's you, start by categorizing what exists before adding new tests.

## What Gets Automated vs. Manually Tested

### Automate These

- All API endpoints (contract, payload validation, error codes)
- Database CRUD operations
- Critical user journeys (login → action → logout)
- Integration points with external services (mock the service, test your integration)
- Data transformations (imports, exports, migrations)
- Authentication and authorization scenarios (roles, permissions, tokens)

### Keep Manual These

- Visual layout and UI consistency
- Complex multi-step workflows that change often
- Accessibility checks (screen readers, keyboard navigation)
- New features before they stabilize (automate after 2-3 sprints)
- One-time regression checks for hotfixes
- Performance/load testing (dedicated tooling, not part of functional suite)

### The 80/20 Rule

80% of your regression value comes from 20% of the tests. The critical path suite is the 20%. If you're drowning in maintenance, trim everything else and protect that 20%.

## Dealing with Time Pressure

This is where regression strategy meets reality. You will be asked to "just run a quick regression" with 2 hours before a release. Here's how to handle it.

### When You Have No Time

1. **Run smoke tests only.** If the deploy is sound, that's your baseline.
2. **Run the critical path.** If core journeys work, ship it.
3. **Skip the long tail.** Edge cases, legacy features, visual checks — those can wait.
4. **Write down what you skipped.** Tell the team: "We didn't test X, Y, Z. If something breaks there, we own it."

Documenting skipped coverage is not CYA. It's honest risk communication. The business can decide if they want to delay or accept the risk.

### When You Have Some Time

- Prioritize based on what changed. If you touched the payment service, don't run the full regression. Run payment tests + smoke.
- Use impact analysis. Changed a utility function? Run unit tests + integration. Don't touch E2E.
- Parallelize. If your tests are sequential, fix that. It's the single biggest time gain.

### When You Have "All the Time" (Never Happens)

If you somehow get a free sprint, don't just run regression. Spend it on:
- Deleting flaky tests
- Splitting slow test suites to run in parallel
- Adding tests for areas that have no coverage
- Reviewing test data for reliability

## Common Mistakes in Regression Planning

### 1. The Growing Suite Problem

Every sprint adds tests. No one removes them. After 12 months, your suite takes 6 hours and no one runs it.

**Fix:** Every time you add a test, ask: "Can I remove or combine an existing one?" Review the suite quarterly. Remove tests that haven't failed in 6 months. If they're important, they'll get added back when they break.

### 2. Testing Everything Every Time

Full regression before every release is a sign you don't trust your lower-level tests. If you're running 500 E2E tests every deploy, your test pyramid is upside down.

**Fix:** Push coverage down. More unit tests, more integration tests. Reserve E2E for critical paths only.

### 3. Treating All Tests as Equal

A login test is not as important as a payment test. But most suites treat them the same — both fail, both block the release.

**Fix:** Tag your tests by severity. Smoke, critical, normal, optional. Failures in optional tests shouldn't block a deploy. They should create a ticket.

### 4. Ignoring Test Data

The most common reason regression tests fail in CI is bad test data, not bad code. A record that was deleted, a time-sensitive coupon that expired, a rate limit that was hit.

**Fix:** Tests should create their own data or use explicitly seeded test data. Never depend on "whatever is in staging." Use fresh data per run or reset state between runs.

### 5. Chasing 100% Coverage

You can have 100% line coverage and still ship a broken experience. Coverage tells you what code was executed, not whether it works correctly.

**Fix:** Focus coverage on logic-heavy code, data transformations, error handling, and security boundaries. Don't waste time testing getters, setters, or generated code.

### 6. No Regression on Your Regression

When you change test infrastructure (new CI tool, new test framework), the regression suite itself breaks. Nobody tests that the tests work until a release is blocked.

**Fix:** Run the smoke suite after any test infrastructure change. Treat test framework upgrades like code changes — review, test, deploy.

### 7. Manual Regression Sclerosis

Teams keep doing manual regression because "that's how we've always done it." They don't realize the manual suite takes 3 days and catches nothing because the testers have gone blind from repetition.

**Fix:** Track what manual regression actually finds. If a manual test hasn't found a bug in 3 runs, automate it or remove it. Free up that person for exploratory testing.
