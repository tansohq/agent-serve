---
name: agent-serve
description: "Audits a SaaS product across six areas to find what blocks AI agents from signing up, authenticating, paying, monitoring usage, self-managing, and integrating reliably — then recommends specific API endpoints, auth patterns, billing integrations, and dev-readiness improvements to build. Use when evaluating whether a product is ready for agent customers or planning an agent-readiness roadmap."
when_to_use: "agent customers, API onboarding, programmatic purchasing, agent-friendly auth, MCP server, agentic commerce, self-serve for agents"
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# agent-serve — Make Your Product Self-Serve for Agents

Full audit across all six areas. Find what's blocking agents and tell the developer exactly what to build.

## Philosophy

The question isn't "is my site discoverable by agents?" — the question is: "can an agent sign up, pay, and use my product without a human touching anything?"

If the answer is no, agents will route spend to competitors that let them self-serve.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Fetch the live product (marketing site, docs, API reference). Audit what's publicly visible.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at API routes, auth middleware, billing integration, docs.

## Step 1: Understand the Product

Before diagnosing, determine:
- What does this product do?
- Who are the current customers (humans? developers? both?)
- Is there an existing API?
- What billing system is used? (Stripe, custom, none visible)
- Is there documentation? Where?

If the product has NO API at all, read [starting-from-zero.md](references/starting-from-zero.md) for the incremental build sequence.

## Step 2: Check Each Area

For each area, find what exists and what's missing:

### Onboarding
- Does `POST /signup` or `POST /accounts` exist?
- Does it require CAPTCHA, email verification, phone, or browser?
- Is there instant sandbox/trial access?
- Can agents provision immediately (deploy-first-claim-later)?

Read [onboarding-patterns.md](references/onboarding-patterns.md) for gold standards (Cloudflare, Stripe Projects), deploy-first-claim-later examples, and sandbox provisioning patterns.

### Authentication
- Do API keys exist? Scoped? Rotatable via API?
- Is OAuth Client Credentials supported?
- Are there blockers: magic links, SMS OTP, browser-only OAuth consent?

Read [auth-patterns.md](references/auth-patterns.md) for the auth hierarchy, OAuth Client Credentials flow, Stripe API key model, and Web Bot Auth.

### Purchasing
- Is there a JSON plan catalog (`GET /plans`)?
- Can subscriptions be created via API with a saved payment method?
- Does checkout require a browser redirect?
- Is pricing machine-readable (pricing.json)?

Read [purchasing-patterns.md](references/purchasing-patterns.md) for the Stripe Setup Intent + Subscriptions API pattern and tokenized payments.
Read [pricing-json.md](references/pricing-json.md) for the machine-readable pricing schema.

### Usage Monitoring
- Do responses include rate limit headers?
- Is there a usage API with current-period data?
- Are there threshold webhooks?

Read [usage-patterns.md](references/usage-patterns.md) for gold standards (OpenAI rate limit headers, Twilio Usage Records API).

### Self-Management
- Can plans be changed/canceled via API?
- Is configuration changeable via API?
- Does the product ship an MCP server?

Read [self-management-patterns.md](references/self-management-patterns.md) for Stripe Subscriptions API lifecycle and MCP server as management interface.

### Dev Readiness
- Do errors return structured JSON with type, code, message, and failing parameter?
- Do mutating endpoints accept an idempotency key?
- Is pagination cursor-based with `has_more`?
- Is the API versioned with a deprecation policy?
- Is there a published OpenAPI spec?
- Is there a test mode with separate keys and fixtures?
- Does `/llms.txt` exist with curated technical content?
- If there's an MCP server, is the tool surface curated (10-15 tools)?
- If there's an A2A Agent Card, does it declare capabilities and auth?
- How many steps from zero to first working API call?

Read [dev-ready-patterns.md](references/dev-ready-patterns.md) for gold standards (Stripe errors, idempotency, versioning), MCP server design, A2A Agent Card, and time-to-first-call benchmarks.

## Step 3: Report

Read [scoring-rubrics.md](references/scoring-rubrics.md) for the 0-10 scoring criteria per area. Score each area using the anchor points in that file.

Use this exact format:

```
# Agent-Serve Audit: [product]

## Scorecard

| Area             | Score | Status                        |
|------------------|-------|-------------------------------|
| Onboarding       | X/10  | [one-line summary]            |
| Authentication   | X/10  | [one-line summary]            |
| Purchasing       | X/10  | [one-line summary]            |
| Usage Monitoring | X/10  | [one-line summary]            |
| Self-Management  | X/10  | [one-line summary]            |
| Dev Readiness    | X/10  | [one-line summary]            |

**Maturity: Level X — [Agent-Hostile / API Exists / Agent-Possible / Agent-Friendly / Agent-First]**

---

### Onboarding — X/10
**Today:** [what exists]
**Blocks agents:** [specific friction]
**Build:** [specific fix with effort estimate]

---

### Authentication — X/10
**Today:** [what exists]
**Blocks agents:** [specific friction]
**Build:** [specific fix]

---

### Purchasing — X/10
**Today:** [what exists]
**Blocks agents:** [specific friction]
**Build:** [specific fix]

---

### Usage Monitoring — X/10
**Today:** [what exists]
**Blocks agents:** [specific friction]
**Build:** [specific fix]

---

### Self-Management — X/10
**Today:** [what exists]
**Blocks agents:** [specific friction]
**Build:** [specific fix]

---

### Dev Readiness — X/10
**Today:** [what exists — error quality, idempotency, pagination, versioning, docs, SDKs, test mode, MCP quality, A2A]
**Blocks integration:** [specific friction]
**Build:** [specific fix]

---

## Hard Blockers
- [things that completely prevent agent use — fix these first]

## Quick Wins
1. [highest impact, lowest effort — effort estimate]
2. [next]
3. [next]

## Roadmap

| Priority | Item | Effort | Reference |
|----------|------|--------|-----------|
| 1 | [what to build] | [time] | [who does it well] |
| 2 | ... | ... | ... |
```

## Step 4: Roadmap

Prioritized build sequence:
1. Hard blockers first (things that completely prevent agent use)
2. Quick wins (high impact, low effort)
3. Full maturity (complete agent-serve stack)

For each item: what to build, effort, dependency, and who does it well today.

Read [maturity.md](references/maturity.md) for levels 0-4 (Agent-Hostile to Agent-First) and how to assess where the product sits.
Read [checklist.md](references/checklist.md) for the full readiness checklist.
Read [emerging-standards.md](references/emerging-standards.md) for standards to watch (Stripe Projects, Web Bot Auth, MCP, A2A, x402).

## Important Rules

- This is about PRODUCT changes, not marketing files. You're recommending API endpoints, auth patterns, billing integrations.
- Be specific. "Add an API" is useless. "Add POST /v1/accounts with email+password, return API key in response, no email verification required" is useful.
- Reference real implementations. Stripe's API, Cloudflare's agent provisioning, Twilio's usage records.
- Don't recommend an MCP server unless the product already has a solid API. MCP is a layer on top of APIs, not a replacement.
- Acknowledge business constraints. "Remove CAPTCHA" has security implications. Note the tradeoff, suggest alternatives (Web Bot Auth, proof-of-work, IP reputation).
- If the product is pre-API (dashboard-only), say so clearly. Read [starting-from-zero.md](references/starting-from-zero.md) for the 4-week sequence.
- When claiming middleware, auth, or any function is broken, trace it to the call site. Parameter names may not match the actual function being passed — a parameter called `ensureVisitor` might receive `ensureAuth` from the caller. Verify what function is actually bound, not just what the variable is named.
