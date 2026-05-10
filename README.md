# QA Strategy Playbook

Real-world QA thinking — no fluff, no theory.

This repo captures how we approach quality across teams. It's a living set of docs, not a handbook that gathers dust. We write down what works, what doesn't, and why we made the calls we did.

## Who This Is For

QA engineers, SDETs, developers who test, and engineering leads who want to understand how their QA team thinks. If you're new to the team, start here.

## Why This Exists

Traditional QA was a phase at the end. QA was the "no" department. That model is dead in CI/CD teams. Quality engineering means embedding testing into the entire lifecycle — from design to deploy — and everyone owns it. This playbook reflects that shift.

## What's Here

```
docs/
├── 01-test-strategy-overview.md    — Core testing philosophy, risk framework, trade-offs
├── 02-test-design-techniques.md    — BVA, equivalence partitioning, exploratory testing
├── 03-regression-strategy.md       — Suite structure, time pressure, common mistakes
├── 04-api-testing-strategy.md      — Coverage, auth, schema validation, environments
├── 05-ui-automation-strategy.md    — Selectors, flaky tests, the pyramid, cross-browser
└── 06-bug-reporting-guidelines.md  — Severity vs priority, good reports, triage reality
```

## How This Repo Works

These docs were written at different times, by different people on the team, for different audiences. Some sections are more detailed than others. Some opinions contradict each other. That's fine — it reflects how we actually think, not a sanitized version.

If something seems inconsistent, ask about it. That's usually where the interesting decisions live.

## Status

Active. Six docs covering the full QA lifecycle — from strategy to day-to-day tactics. Still evolving as the team and product change. If something's missing, open an issue or add it.
