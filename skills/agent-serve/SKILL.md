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

Five dimensions define agent-serve readiness:
1. **Onboarding** — Can an agent create an account without a browser?
2. **Authentication** — Can an agent prove identity without human ceremony?
3. **Purchasing** — Can an agent select a plan and pay programmatically?
4. **Usage Monitoring** — Can an agent track its own consumption?
5. **Self-Management** — Can an agent change plans or cancel without a human?

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

### Onboarding (0-10)

Check for:
- API-based account creation endpoint (`POST /signup`, `POST /accounts`)
- Whether signup requires browser interaction (CAPTCHA, email click, phone verification)
- How many steps from zero to "has an account"
- Whether there's a programmatic trial/sandbox mode

| Score | Meaning |
|-------|---------|
| 0 | Browser-only signup with CAPTCHA |
| 3 | API signup exists but requires email verification loop |
| 5 | API signup with instant access, but limited (needs manual upgrade) |
| 8 | Full API onboarding, instant access, sandbox/trial included |
| 10 | Agent provisions itself completely (Cloudflare model) |

### Authentication (0-10)

Check for:
- API key generation (programmatic or self-serve dashboard)
- OAuth 2.0 Client Credentials Grant support
- Service account pattern (GCP model)
- Blockers: magic links, SMS OTP, browser-only OAuth consent, device fingerprinting

| Score | Meaning |
|-------|---------|
| 0 | SSO-only or magic-link-only, no API auth path |
| 3 | API keys exist but require human to generate in dashboard |
| 5 | API keys + scoped permissions, but no machine-to-machine OAuth |
| 8 | Client Credentials Grant + scoped API keys + key rotation API |
| 10 | Full programmatic identity with cryptographic attestation |

### Purchasing (0-10)

Check for:
- Programmatic plan selection endpoint
- Payment without browser redirect (tokenized cards, Stripe API)
- Transparent pricing (machine-readable plans, limits, rates)
- Whether upgrade/plan-change requires "talk to sales"

| Score | Meaning |
|-------|---------|
| 0 | "Contact sales" only |
| 3 | Self-serve pricing page but payment requires browser checkout |
| 5 | Stripe Checkout with direct link (semi-programmatic) |
| 8 | Full billing API (select plan, attach payment, activate) |
| 10 | Agent marketplace model (Stripe Projects / Cloudflare) |

### Usage Monitoring (0-10)

Check for:
- Usage/consumption API endpoint
- Rate limit headers on responses (`x-ratelimit-remaining-*`)
- Billing/invoice API
- Quota/limits visibility
- Alerting/webhook for threshold events

| Score | Meaning |
|-------|---------|
| 0 | Dashboard-only, no API for usage data |
| 3 | Basic rate limit headers but no usage API |
| 5 | Usage API exists but limited (daily aggregates only) |
| 8 | Real-time usage + billing API + threshold webhooks |
| 10 | Full observability (OpenAI model: per-request tokens + cost) |

### Self-Management (0-10)

Check for:
- Plan change via API (upgrade, downgrade)
- Cancel via API
- Configuration changes via API (not dashboard-only)
- Team/seat management via API
- Data export/deletion via API

| Score | Meaning |
|-------|---------|
| 0 | All management requires dashboard or support ticket |
| 3 | Some settings via API, but plan changes require human |
| 5 | Plan changes via API, but cancel requires support |
| 8 | Full lifecycle management via API |
| 10 | Complete self-serve + MCP server for agent interaction |

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

For each dimension scoring below 5, provide:
- **Current state** — what exists today
- **What blocks agents** — the specific friction
- **Gold standard** — who does this well (use this mapping: Onboarding→Cloudflare, Auth→OAuth Client Credentials + Stripe keys, Purchasing→Stripe Projects, Usage→OpenAI, Self-Management→Stripe Subscriptions API)
- **Recommended fix** — specific code/infra changes needed. Effort: hours (ship today), days (requires planning), weeks (requires design + implementation)

Scores between anchor points (0, 3, 5, 8, 10): interpolate based on how many criteria of the higher anchor are met.

See [best-practices.md](best-practices.md) for detailed patterns and references.

## Step 5: Roadmap

Output a prioritized list of changes ordered by:
1. Hard blockers first (things that completely prevent agent use)
2. Quick wins (high impact, low effort)
3. Full maturity (complete agent-serve stack)

For each item, note:
- Effort: hours / days / weeks
- Dependency: what else needs to change first
- Reference: company that solved this well

## Important Rules

- This is about PRODUCT changes, not marketing files. Unlike agent-web, you're not generating robots.txt. You're recommending API endpoints, auth patterns, billing integrations.
- Be specific. "Add an API" is useless. "Add POST /v1/accounts with email+password, return API key in response, no email verification required" is useful.
- Reference real implementations. Stripe's API, Cloudflare's agent provisioning, Twilio's usage records.
- Don't recommend an MCP server unless the product already has a solid API. MCP is a layer on top of APIs, not a replacement.
- Acknowledge business constraints. "Remove CAPTCHA" has security implications. Note the tradeoff, suggest alternatives (Web Bot Auth, proof-of-work, IP reputation).
- If the product is pre-API (dashboard-only), say so clearly. The roadmap is longer but the direction is the same.
