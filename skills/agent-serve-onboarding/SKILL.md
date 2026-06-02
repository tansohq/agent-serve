---
name: agent-serve-onboarding
description: "Finds what blocks AI agents from creating accounts on a SaaS product — checks for programmatic signup endpoints, email verification loops, CAPTCHA, phone requirements, and deploy-first-claim-later patterns. Use when evaluating agent onboarding friction or planning a self-serve signup flow."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Can an agent create an account without a browser?

If signup requires a browser, email click, or phone number, agents can't onboard. They'll route to competitors that let them self-serve.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the marketing site, docs, and API reference. Look for signup endpoints, registration flows, and onboarding documentation.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look for signup routes, account creation endpoints, verification middleware, CAPTCHA integration.

## What to Check
- Does a `POST /signup` or `POST /accounts` endpoint exist?
- Does it require browser interaction (CAPTCHA, email click, phone verification)?
- How many steps from zero to "has an account with API access"?
- Is there a sandbox/trial mode accessible without approval?
- Can agents provision immediately and claim later (deploy-first-claim-later)?
- Is there instant sandbox access for dev tools?

## What Blocks Agents
- CAPTCHA / reCAPTCHA (browser fingerprinting)
- Email verification loops (agent has no inbox)
- SMS OTP / phone number requirement
- Manual approval queues ("we'll review your application")
- Multi-step browser wizards
- Device fingerprinting that flags headless browsers

## Deep Patterns
Read [patterns.md](references/patterns.md) for gold standards (Cloudflare, Stripe Projects), deploy-first-claim-later examples (Netlify, Prisma), and sandbox provisioning patterns.

## Report

```
### Onboarding
**Today:** [what exists — signup flow, verification steps, time to API access]
**Blocks agents:** [specific friction — which of the above blockers are present]
**Build:** [specific fix — endpoint to add, verification to bypass, effort estimate]
```

## Rules
- Be specific. "Add an API" is useless. "Add `POST /v1/accounts` with email+password, return API key in response, no email verification required" is useful.
- Acknowledge security tradeoffs. "Remove CAPTCHA" has implications — suggest alternatives (Web Bot Auth, proof-of-work, IP reputation).
- Reference real implementations — Cloudflare's agent provisioning, Netlify's anonymous deploy, Stripe Projects.
- If the product is pre-API (browser-only signup), say so clearly and recommend the first endpoint to build.
