---
name: agent-serve
description: "Audit whether AI agents can buy and use your SaaS product. Scores 5 dimensions, finds blockers."
when_to_use: "agent customers, API onboarding, programmatic purchasing, agent-friendly auth, MCP server, agentic commerce, self-serve for agents"
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# agent-serve — Can Agents Buy and Use Your Product?

Audit SaaS products for agent-customer readiness.

## Philosophy

The question isn't "is my site discoverable by agents?" (that's agent-web).
The question is: "can an agent sign up, pay, and use my product without a human touching anything?"

If the answer is no, you're invisible to the agentic economy. Agents will route spend to competitors that let them self-serve.

## Detect Mode

If `$ARGUMENTS` contains a URL:
→ **URL mode**: Fetch the live product (marketing site, docs, API reference). Audit what's publicly visible about the agent experience.

If `$ARGUMENTS` is empty:
→ **Repo mode**: Audit the current codebase. Look at API routes, auth middleware, billing integration, docs.

## Step 1: Understand the Product

Before scoring, determine:
- What does this product do?
- Who are the current customers (humans? developers? both?)
- Is there an existing API?
- What billing system is used? (Stripe, custom, none visible)
- Is there documentation? Where?

## Step 2: Audit Each Dimension

Score each dimension 0-10. Read [scoring-rubrics.md](references/scoring-rubrics.md) for detailed criteria and anchor points.

1. **Onboarding** — Can an agent create an account without a browser?
2. **Authentication** — Can an agent prove identity without human ceremony?
3. **Purchasing** — Can an agent select a plan and pay programmatically?
4. **Usage Monitoring** — Can an agent track its own consumption?
5. **Self-Management** — Can an agent change plans or cancel without a human?

## Step 3: Score

```
## Agent-Serve Audit: [product]
Overall: X/50

### Onboarding: X/10
[specific findings]

### Authentication: X/10
[specific findings]

### Purchasing: X/10
[specific findings]

### Usage Monitoring: X/10
[specific findings]

### Self-Management: X/10
[specific findings]

## Blockers (things that completely prevent agent use)
- [list any hard blockers: CAPTCHA, email-only, contact-sales gates]

## Quick Wins (highest impact, lowest effort)
1. [specific recommendation]
2. [specific recommendation]
3. [specific recommendation]
```

## Step 4: Advise

For each dimension scoring below 5, read [best-practices.md](references/best-practices.md) and provide:
- **Current state** — what exists today
- **What blocks agents** — the specific friction
- **Gold standard** — who does this well
- **Recommended fix** — specific code/infra changes. Effort: hours / days / weeks

## Step 5: Roadmap

Prioritized list ordered by:
1. Hard blockers first (things that completely prevent agent use)
2. Quick wins (high impact, low effort)
3. Full maturity (complete agent-serve stack)

For each item: effort, dependency, and reference company.

## Important Rules

- This is about PRODUCT changes, not marketing files. You're recommending API endpoints, auth patterns, billing integrations.
- Be specific. "Add an API" is useless. "Add POST /v1/accounts with email+password, return API key in response, no email verification required" is useful.
- Reference real implementations. Stripe's API, Cloudflare's agent provisioning, Twilio's usage records.
- Don't recommend an MCP server unless the product already has a solid API. MCP is a layer on top of APIs, not a replacement.
- Acknowledge business constraints. "Remove CAPTCHA" has security implications. Note the tradeoff, suggest alternatives (Web Bot Auth, proof-of-work, IP reputation).
- If the product is pre-API (dashboard-only), say so clearly. The roadmap is longer but the direction is the same.
