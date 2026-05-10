# Test Design Techniques

## Why This Matters

You don't need to test every possible input. That's impossible and pointless. Test design techniques help you pick the few inputs that find the most bugs. This doc covers the techniques we actually use day-to-day — not the full ISTQB glossary.

## Boundary Value Analysis (BVA)

Most bugs live at the edges. If a field accepts values 1–100, the bug is probably at 0, 1, 100, or 101, not at 42. Test the boundaries.

### Login — Password length

Password field says 8–128 characters:

| Input | Expected | Why |
|---|---|---|
| 7 chars | Rejected | Just below min |
| 8 chars | Accepted | Exactly min |
| 9 chars | Accepted | Just above min |
| 127 chars | Accepted | Just below max |
| 128 chars | Accepted | Exactly max |
| 129 chars | Rejected | Just above max |

Don't bother testing 50 chars. It tells you nothing new.

### Payments — Amount field

Amount field with min $1.00, max $10,000.00:

- $0.99 → rejected (below min)
- $1.00 → accepted (exact min)
- $1.01 → accepted (above min)
- $9,999.99 → accepted (below max)
- $10,000.00 → accepted (exact max)
- $10,000.01 → rejected (above max)

**Gotcha:** Check decimal handling. Some systems truncate, some round, some reject. $10,000.001 is a classic miss.

**Pro shortcut:** In a rush, just test min-1, min, max, max+1. Skip everything in between. If you're really pressed, test min and max only. That catches 90% of boundary bugs. Not perfect, but good enough for a Friday afternoon deploy.

### API — Pagination

`GET /api/orders?page=1&limit=50`

- `limit=0` → what happens? (most APIs handle this wrong)
- `limit=1` → works? (edge of valid)
- `limit=50` → works (max)
- `limit=51` → does it clamp or error?
- Negative page numbers → try it. You'd be surprised.

## Equivalence Partitioning

Group inputs into buckets that should behave the same. Test one from each bucket. If 7 chars fails and 8 works, you don't need to test 6, 5, 4 — they're in the same "too short" bucket.

### Real example: Age field (18–120)

**Valid partitions:** 18–120 (test: 25)
**Invalid partitions:** < 18 (test: 15), > 120 (test: 150), non-numeric (test: "abc"), empty

That's 4 tests instead of infinite. Same coverage.

### Payment — Card expiry

- Past month this year → rejected
- Current month/year → accepted
- Future month this year → accepted
- Future year → accepted
- Invalid format (XX/YY, "abcd") → rejected

One test per partition. Done.

### API — Status codes by response family

- 2xx → success path (test: 200)
- 4xx → client error (test: 400, 401, 403, 404, 422)
- 5xx → server error (test: 500, 502, 503)

Don't test all 50 possible 4xx codes. The application probably handles them the same way. Test the ones that actually appear in your codebase.

## Exploratory Testing

This is where you stop following a script and start looking for trouble.

### When to Do It

- Before a release (after automation passes)
- When a new feature hits staging
- When you have a gut feeling something is off
- When you're bored with regression scripts (trust this instinct)

### How We Approach It

1. **Set a timebox.** 30–60 minutes. No more. Stop when the timer goes off.
2. **Know the feature.** Read the AC, look at the code changes if you can.
3. **Start with the happy path.** Make sure it works. Then break it.
4. **Follow the weird.** Click things you wouldn't normally click. Submit empty forms. Hit the back button after a payment. Open two tabs and submit the same form twice. Resize the window mid-flow.
5. **Take notes.** Not formal test cases. Just enough to reproduce what you found.

### My Session Notes Template

```
Feature: Checkout flow
Time: 45 min

What I tried:
- Added item to cart, logged out, logged back in → cart still there? (yes)
- Removed item from cart → empty cart message shown? (yes, but broken layout on mobile)
- Clicked "Place Order" twice fast → double charge? (NO — no idempotency key! logged P1 bug)
- Changed quantity to 999 → shows $13,977.00 total (formats correctly? yes)
```

**Note:** The double-click on Place Order found a real P1 bug. The formal test cases never would have caught it. That's the point.

### Common Heuristics

- **Empty states:** What happens with no data, no results, no items?
- **Error states:** What does the error message say? Is it helpful or confusing?
- **Loading states:** Is there a spinner? Can you trigger race conditions?
- **Edge of data:** Very long names, special characters, emoji in text fields
- **Concurrency:** Two users editing the same thing. Payment timing out and retrying.

## When NOT to Automate

Automation is expensive. Every automated test has a maintenance cost. Here's when it's not worth it.

### 1. One-time validation

Testing a migration, a data fix, or a hotfix that will never run again. Don't automate it. Run it manually, verify, move on.

### 2. Visual/design checks

"Is the button the right shade of blue?" or "Is the logo centered?" — pixel-level assertions are brittle and break on every browser update. Use visual regression tools if you must, but know they have a high false-positive rate.

### 3. Complex workflows that change constantly

If the flow is rewritten every other sprint, your tests will break faster than you can fix them. Automate the stable parts and test the rest manually until things settle.

Exception: Critical paths (checkout, login) should be automated even if they change. The cost of not catching a regression is higher.

### 4. Exploratory territory

You can't automate "what happens if I click this button 5 times fast then refresh?" Exploratory testing and automation complement each other. Don't try to replace one with the other.

### 5. Tests that need human judgment

"Is this error message useful?" — no assertion can answer that. Get a human to read it.

### The Decision Framework

```
Is this a critical/stable path?         → Automate
Is this a one-time check?               → Do it manually
Does it need human judgment?            → Do it manually
Will this test survive >3 months?       → Automate
Is it cheaper to fix bugs than to test? → Don't test at all (yes, seriously)
```

That last one is controversial but honest. Some edge cases are cheaper to fix in production than to build and maintain automation for. Use sparingly.

## When You Don't Have Time for This

Let's be real. In a sprint, you often don't have time to methodically run BVA on every field. Here's the shortcut:

- **Test boundaries on anything that handles money, auth, or data.** Payment amounts, password lengths, pagination limits. Those matter.
- **Skip BVA on display-only fields.** If it's just showing text on a page, boundaries are unlikely to find a bug.
- **Equivalence partition by instinct.** After a while, you just know that "empty list" and "list with one item" are the two cases that matter. Trust the gut.
- **Do exploratory testing even if you skip everything else.** 15 minutes of clicking around the changed feature finds more bugs than running through a checklist.

The techniques in this doc are the ideal. In practice, you'll use them about 60% of the time. The other 40% is instinct, experience, and making educated guesses about where bugs hide. That's normal.
