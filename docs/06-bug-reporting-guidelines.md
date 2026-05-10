# Bug Reporting Guidelines

## Why This Matters

A bad bug report wastes everyone's time. The developer can't reproduce it, so they ask questions. You go back and forth for three days. By then, the sprint is over and the bug is still open.

A good bug report is a gift. The developer reads it, reproduces it, fixes it, and moves on. No follow-up needed. That's the goal.

## What Engineers Actually Need

Engineers don't care about your opinion on the UI. They care about:

1. **Can I reproduce it?** — Exact steps. Starting state. Environment. Data conditions.
2. **What did you expect vs. what happened?** — Clear before and after.
3. **Where is it?** — URL, component, API endpoint, error log.
4. **How bad is it?** — Who is affected and how. Not your emotional state about it.
5. **Evidence.** — Screenshot, video, network tab, console logs, stack trace, database state.

If you provide these five things, the developer will fix your bug faster than anyone else's.

## How to Write Effective Bug Reports

### Title

Describe the problem, not your theory about the problem.

| Bad | Good |
|---|---|
| "Login is broken" | "Login returns 500 error when password contains special characters" |
| "Payment page crashes" | "Checkout page throws TypeError when coupon code is applied on mobile Safari" |
| "UI looks wrong" | "User avatar overflows container on profile page at 768px viewport width" |

The good title includes: what, where, and under what condition. The developer should know what the bug is about without opening the ticket.

### Description Template

```
**Steps to reproduce:**
1. Go to [URL/page]
2. Enter [specific data/state]
3. Click [button/action]
4. Observe [result]

**Expected:** [what should happen]
**Actual:** [what actually happens]

**Environment:**
- Browser/OS: [Chrome 120 / macOS 14.2]
- Device: [iPhone 15, Desktop, etc.]
- User role: [admin, customer, guest]
- Feature flags: [if relevant]

**Frequency:** [always, intermittent (approx 1 in 5), only on prod]

**Attachments:** [screenshot, video, HAR file, console logs, API response]
```

### Common Mistakes

- **Skipping step 1.** "Go to the dashboard" — but what dashboard? What URL? What user? Be specific.
- **Combining multiple bugs in one ticket.** "The form has validation issues" — if there are 3 different validation bugs, write 3 tickets. They'll be fixed independently.
- **Using vague language.** "It seems like..." "I think it might be..." Don't guess. State facts. If you're unsure, say what you observed and what you inferred.
- **No expected behavior.** You described the bug but not what should happen. The developer doesn't know if the fix is to change the backend, change the UI, or change the business logic.
- **Opinions as bug reports.** "The button should be blue" is a design request, not a bug. Don't inflate preferences into defects.

## Severity vs. Priority

This gets butchered in every org. Here's the practical version.

### Severity (How Bad Is the Technical Impact)

| Level | Meaning | Example |
|---|---|---|
| **Critical** | Data loss, security breach, core feature completely broken for all users | Users can't complete payment. All API calls return 500. |
| **Major** | Core feature broken for some users, or significant data incorrect | Search returns wrong results for logged-in users. Report export is missing columns. |
| **Minor** | Feature works but has cosmetic or edge case issues | Button alignment off on mobile. Error message is confusing. |
| **Trivial** | Low impact, low visibility | Typo in tooltip. Missing period in footer. |

### Priority (How Fast Should We Fix It)

| Level | Meaning |
|---|---|
| **P0** | Drop everything. Fix now. Blocking release or production outage. |
| **P1** | Fix this sprint. Important but not an outage. |
| **P2** | Fix next sprint. Shouldn't block current work. |
| **P3** | Fix when we have time. Backlog. Icebox. |

### The Critical Insight

Severity and priority are NOT the same thing.

- **High severity, low priority:** A bug in a feature that's being deprecated. Technically severe, but we're never going to fix it.
- **Low severity, high priority:** A typo on the homepage that the CEO noticed. It's trivial but it's getting fixed today.
- **Critical severity, P3:** A data corruption bug in a tool only used by the ops team internally. It's bad, but ops has a workaround and the feature is being rewritten next quarter.

Severity is assigned by QA (technical assessment). Priority is assigned by the PM (business decision). Don't argue priority as QA — provide the severity and let the PM decide what to do with it.

### Severity Inflation

Everyone marks their bugs as Critical/P0. If everything is critical, nothing is critical. Be honest. If you mark a typo as Critical, you're teaching the team to ignore your severity labels.

### The Politics of Severity

Here's the thing nobody says out loud: sometimes a bug is technically Minor but you mark it Major because that's the only way it gets fixed this sprint. And sometimes a bug is technically Critical but everyone knows it's never getting fixed because the feature is being deprecated.

This is political, not technical. Be aware of it. If you inflate severity to get attention, you lose credibility over time. If you're honest and the team still doesn't fix things, that's a process problem, not a bug report problem.

### What Happens When You Find a Bug Mid-Sprint

You're testing a feature on day 3 of a 2-week sprint. You find a Major bug. Dev stops working on their current task to fix it. Now that task is at risk. This is the sprint reality.

The right call depends on context:
- **If the bug blocks the feature** — it gets fixed now. The other task slips.
- **If the bug is in an unrelated area** — log it, prioritize in the next sprint planning. Don't derail the current sprint.
- **If the bug is in the feature being tested but has a workaround** — document the workaround, log the bug, let the PM decide if it blocks the release.

You won't always agree with the decision. That's fine. Your job is to provide clear information about the bug and its impact. The PM's job is to decide what to do about it.

## Examples: Good vs. Bad

### Bad Bug Report

```
Title: Checkout broken

Description: I tried to buy something and it didn't work. Please fix ASAP.

Severity: Critical
Priority: P0
```

**Problems:** No steps, no environment, no error message, no evidence. "It didn't work" tells me nothing. This ticket will sit for 3 days while someone asks you clarifying questions.

### Good Bug Report

```
Title: Checkout fails with "Invalid coupon" error when coupon code contains uppercase letters

Steps:
1. Go to /products and add any item to cart
2. Go to /cart and click "Checkout"
3. Enter coupon code "SAVE20" (uppercase)
4. Click "Apply"
5. See error: "Invalid coupon code"

Expected: Coupon "SAVE20" should be accepted. The system should be case-insensitive for coupon codes.
Actual: Error message "Invalid coupon code" is shown. Coupon codes with lowercase letters work fine (e.g., "save20").

Environment:
- Browser: Chrome 121 on macOS 14.3
- User: yasar@example.com
- Staging environment

Frequency: 100% reproducible

Attachments:
- Screenshot showing the error
- API response: POST /api/coupons/apply returned 400 { "error": "coupon not found" }
- Console: No errors
```

**Why it's good:** Exact steps, clear expected vs actual, environment details, evidence including the actual API response. The developer knows the bug is in the coupon lookup logic (case sensitivity in the DB query or comparison), not in the checkout flow.

### Bad Bug Report

```
Title: App crashes on mobile

Description: The app keeps crashing on my phone. I think it's a memory issue.

Severity: Critical
Priority: P1
```

**Problems:** "My phone" — what phone? What OS? What version of the app? What were you doing when it crashed? No crash log, no screen recording, no steps.

### Good Bug Report

```
Title: App crashes when uploading image > 5MB on iOS 17+

Steps:
1. Open app on iPhone 15 Pro, iOS 17.4
2. Go to Profile > Edit Profile
3. Tap "Change Avatar"
4. Select a photo from gallery (tested with 6.2MB JPEG)
5. App crashes immediately after selection

Expected: Photo should upload, or user should see a file size limit error.
Actual: App crashes to home screen. No error message shown.

Environment:
- Device: iPhone 15 Pro, iOS 17.4
- App version: 3.2.1 (build 204)
- Network: WiFi

Frequency: 100% reproducible with files > 5MB. Works fine with files < 5MB.

Attachments:
- Screen recording showing the crash
- Crash log from Xcode: [link]
- Device console output: [link]
- Test image used: [link]
```

### The "Good Enough" Bug Report

Sometimes you find a bug during exploratory testing and you're in the middle of a flow. You don't want to stop and write a perfect report because you'll lose context. What do you do?

Write a "good enough" report:
- Title captures the issue
- Steps are brief but reproduceable
- Screenshot attached
- Mark it as "needs cleanup" or add a TODO tag

Come back to it after your session and fill in the details. If you don't come back to it, at least the developer has something to work with. A so-so bug report today is better than a perfect bug report never.

### When Bugs Don't Get Fixed

Not all bugs get fixed. Some get triaged out. Some sit in the backlog for 18 months. Some are "won't fix" because the effort outweighs the impact.

This is frustrating, especially for junior QA. You found a real bug. You wrote a great report. And the team is going to ship without fixing it.

Here's the reality check:

- **Won't fix doesn't mean it's not a bug.** It means the team decided other things are more important right now. That's their call to make.
- **Document it and move on.** If it's truly important, it will come up again. If it doesn't come up, it wasn't as important as you thought.
- **Learn the difference between "this matters" and "this matters right now."** A cosmetic bug on an internal admin page might never be worth fixing. That doesn't make you wrong for reporting it. It makes you wrong if you keep pushing it.

## Tips for Better Bug Reports

- **Reproduce it twice.** If you can't reproduce it consistently, say so. "Happened once on prod, couldn't reproduce on staging" is honest and saves the developer from wasting time trying to repro.
- **Include the "before" state.** "User has 3 items in cart, is logged in, and has an active subscription." The starting state matters.
- **Note workarounds.** "Refreshing the page resolves it temporarily." or "Clearing the cache fixes it." This helps the developer understand the root cause and helps support answer customers.
- **Link related issues.** "This might be related to bug #142 — same error message." Even if you're wrong, it gives context.
- **Call out regressions.** "This worked in v3.1.0, broken in v3.2.0." Regression information is gold. It narrows the search space immediately.
- **Tag bugs by source.** "Found during exploratory testing" vs "found in production" vs "found while writing automated tests." This helps the team see where their testing gaps are. If most bugs come from production monitoring, your test coverage is weak. If most come from exploratory testing, your automation is missing edge cases. Patterns matter.
