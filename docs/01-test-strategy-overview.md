# Test Strategy Overview

## What This Is

A test strategy is the set of decisions that guide *how* we test. It's not a detailed test plan (that lives per-feature). This doc captures our team-level approach: what we automate, what we don't, where we invest effort, and why.

## QA in Modern Teams

### What QA Means Now

QA is not a phase. It's not a gate at the end of a sprint. In CI/CD teams, QA is an **activity that everyone owns** — devs, QA, PMs, even support. The dedicated QA role exists to lead quality strategy, not to be the only person who tests.

The old model: dev writes code → throws it over the wall → QA finds bugs → dev fixes → repeat. That's slow, expensive, and creates adversarial dynamics. It doesn't work when you're deploying multiple times a day.

### From Manual QA to Quality Engineering

Most teams start with "manual QA" — someone clicks through the app before release. That scales terribly. As you grow, the QA role shifts:

- **Level 1 — Manual testing only.** No automation. Releases are slow. QA is a bottleneck.
- **Level 2 — Some automation.** Smoke tests are automated. QA still does most regression manually. Better, but regression still eats sprint time.
- **Level 3 — Quality engineering.** Automation covers regression. QA focuses on test strategy, exploratory testing, test infrastructure, and shifting left. Devs write unit and integration tests as part of development.

Most orgs are stuck between Level 2 and 3. The goal is Level 3, but it takes time, investment, and buy-in. Be honest about where you are.

### Role of QA in Agile Teams

The QA person on a squad should be:

- **The quality advocate** — asking "what happens when X fails?" during refinement, not after coding starts.
- **The risk analyst** — helping the team decide what to test and what to skip.
- **The automation enabler** — building frameworks, coaching devs, reviewing test code.
- **The explorer** — spending time on exploratory testing of new features, not scripting every test case in advance.

A QA person who spends most of their time writing manual test cases in Jira is being misused. Push back.

### Speed vs. Quality — The Real Trade-off

"Speed and quality are not in conflict" sounds good in a keynote but misses the point. In practice:

- **You can have both, but not instantly.** Building quality in takes upfront investment (test infrastructure, CI pipeline, cultural shift). During that transition, speed may dip.
- **Shortcuts have compounding interest.** Skipping tests to hit a deadline today means more bugs and slower velocity next sprint. The debt compounds.
- **The real question is: what quality level is acceptable?** Not every feature needs aerospace-grade testing. A prototype or internal tool can ship with less rigor than a payment flow. Be intentional about the difference.

Opinion: The teams that ship fast consistently are the ones that invested in quality early. The teams that cut quality to ship fast end up slower in 3 months. But this is a hard sell to a PM who needs a feature out by Friday. Pick your battles.

## Guiding Principles

- **Risk drives effort.** We don't test everything equally. We test based on likelihood and impact of failure.
- **Speed matters.** Slow feedback loops kill adoption. Tests must be fast enough to run before merge.
- **Automation is a tool, not a religion.** Some things are better tested manually (exploratory, visual, complex workflows). Know the difference.
- **Tests are code.** They rot, need refactoring, and have maintenance cost. Treat them accordingly.

## Test Levels (How We Think About Them)

We loosely follow the testing pyramid but don't worship it. In practice:

| Level | Goal | Who Owns | Typical Feedback |
|---|---|---|---|
| **Unit** | Catch logic errors fast | Devs | Seconds |
| **Integration** | Service/API contracts, DB interactions | Devs + QA | Minutes |
| **E2E** | Critical user journeys | QA | 10-30 min |
| **Manual/Exploratory** | UX, edge cases, new features | QA | Hours |

**Reality check:** Most orgs have a "testing ice-cream cone" (too many E2E, not enough unit). We actively push back on that. If you find yourself writing a new E2E test, ask: *"Can this be covered at a lower level?"*

## Risk-Based Testing (The Real Decision Framework)

We don't have time to test everything. Here's the prioritization:

1. **Critical path** — login, checkout, payments, data loss scenarios. Always covered.
2. **High traffic** — features used by 80%+ of users. High regression risk.
3. **Regression-prone** — areas that break often. Invest in automation here.
4. **New/Changed** — whatever was just shipped. Smoke test + targeted edge cases.
5. **Everything else** — tested based on gut feel, time available, and historical bugs.

This is not a fixed formula. It changes per sprint, per release, per mood of the product manager. Accept it.

## Automation Principles

- **Prefer API tests over UI tests.** UI tests are brittle and slow. Use them sparingly.
- **Test data should be explicit.** Tests that depend on "whatever is in the DB" are flaky. Seed what you need.
- **Don't chase 100% coverage.** It's a vanity metric. Focus on coverage of risky paths.
- **Flaky tests must be fixed or removed.** A flaky suite destroys trust. If a test can't be stabilized, delete it.

## What's NOT Covered Here

- Step-by-step test case formats (see test plan templates)
- CI/CD pipeline config (see ci-cd-integration docs)
- Specific tool recommendations (those go in tooling docs per squad)

## Trade-offs We Live With

- **Speed vs. confidence.** Faster runs mean fewer tests. More tests mean slower feedback. We lean fast.
- **Coverage vs. maintenance.** Every test is a liability. Write fewer, better tests.
- **Consistency vs. autonomy.** Different teams test differently. That's fine. We share patterns, not mandates.

## When to Update This Doc

- When we change our automation approach significantly
- When a principle stops serving us
- When new team members keep asking the same questions (write the answer down)
