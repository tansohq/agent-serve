---
name: agent-serve-purchasing
description: "Finds what blocks AI agents from selecting a plan and paying for a SaaS product programmatically — checks for JSON plan catalogs, Stripe Subscriptions API without browser checkout, saved payment methods, and machine-readable pricing.json. Use when evaluating agent purchasing readiness or implementing programmatic billing."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Can an agent select a plan and pay via API?

If purchasing requires a browser checkout page or "contact sales," agents can't buy. They need plan catalog + payment method + subscription creation — all via API.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the pricing page, docs, API reference, and billing documentation. Look for plan catalogs, checkout flows, Stripe integration, and pricing visibility.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at billing routes, Stripe integration, plan management, checkout flows.

## What to Check
- Is there a `GET /plans` or equivalent returning plan IDs, prices, features, limits as JSON?
- Can subscriptions be created via API (`POST /subscriptions` with plan_id + payment_method)?
- Does the checkout require a browser redirect (Stripe Checkout Sessions) or work via API (Payment Intents)?
- Can agents use saved payment methods (`pm_...`) without browser interaction?
- Is pricing visible and machine-readable, or hidden behind "contact sales"?
- Is there usage-based billing with automatic metering?

## What Blocks Agents
- "Contact sales" gates for any plan
- Browser-only Stripe Checkout (redirect flow)
- Manual invoice / PO process
- Pricing hidden behind demo request
- Enterprise-only tiers with no self-serve option
- Annual-only billing (agents need monthly or usage-based for flexibility)

## Deep Patterns
Read [patterns.md](references/patterns.md) for the Stripe Setup Intent + Subscriptions API pattern, tokenized payments (saved `pm_...` vs. SPTs vs. virtual cards), and Stripe Projects.

Read [pricing-json.md](references/pricing-json.md) for the `pricing.json` schema — machine-readable pricing that agents can discover, evaluate, and compare across vendors.

## Report

```
### Purchasing
**Today:** [what exists — billing system, plan visibility, checkout flow]
**Blocks agents:** [specific friction — which blockers are present]
**Build:** [specific fix — endpoints to add, Stripe integration changes, effort estimate]
```

## Rules
- Most SaaS already uses Stripe. The fix is usually "expose what Stripe supports as API endpoints" not "build a new billing system."
- Be specific about the Stripe API pattern: Setup Intent for saving payment methods, Payment Intents for charges, Subscriptions API for recurring.
- Don't recommend Stripe Projects unless the product is dev-infra with an existing solid API. It's GA (since April 2026, 32 providers) but additive — most should start with Setup Intent + Subscriptions API.
- pricing.json is the quick win for discoverability — recommend publishing it at the domain root.
