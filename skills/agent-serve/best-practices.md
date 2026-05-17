# Agent-Serve Best Practices

## Onboarding Patterns

### Gold Standard: Cloudflare (2026)

Agents create accounts, buy domains, start paid subscriptions, deploy apps. Zero manual steps beyond ToS acceptance. Protocol:
- REST/JSON catalog for service discovery
- Identity attestation + OAuth for auth
- Tokenized payment with configurable spend cap ($100/month default)

### Gold Standard: Stripe Projects

Agents get scoped API keys per project with explicit capabilities (read billing, create charges up to threshold). 32 launch partners.

### What Works
- `POST /v1/accounts` — returns account ID + API key in one call
- Instant sandbox/trial access (no approval queue)
- Programmatic ToS acceptance (checkbox equivalent via API param)
- Tiered access: start restricted, unlock via verification

### What Blocks Agents
- Email verification loops (agent has no inbox by default)
- CAPTCHA (invisible reCAPTCHA still uses browser fingerprinting)
- Phone number requirement (SMS OTP)
- Manual approval queues ("we'll review your application")
- Multi-step browser wizards

### Practical Fix
If you can't remove email verification entirely:
1. Offer API key auth that bypasses email for programmatic access
2. Or: accept a verified domain as identity proof
3. Or: support OAuth Client Credentials (identity comes from the OAuth provider, not email)

---

## Authentication Patterns

### Gold Standard: OAuth 2.0 Client Credentials Grant

Machine-to-machine auth. No user interaction. Short-lived tokens. Supported by Auth0, Okta, every major identity provider.

Flow:
```
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=AGENT_ID
&client_secret=AGENT_SECRET
&scope=read:billing write:config
```

Returns short-lived access token. Agent refreshes as needed.

### Gold Standard: Stripe API Keys

Simple, effective:
- Secret key for server-side (full access)
- Restricted keys with granular permissions
- Key rotation via API
- Test mode keys for sandbox

### Emerging: Web Bot Auth (IETF Draft)

Cryptographic agent identity. WAFs can whitelist verified agents without CAPTCHA. Amazon Bedrock AgentCore Browser supports this.

### What Works
- API keys with scoped permissions (read-only, billing-only, admin)
- Key rotation endpoint (`POST /v1/keys/rotate`)
- Multiple keys per account (different agents, different scopes)
- Client Credentials OAuth for enterprise/multi-tenant

### What Blocks Agents
- Magic links (requires email inbox + browser click)
- SMS OTP (requires phone)
- Browser-only OAuth consent screens (click "Allow")
- Device fingerprinting (headless browsers get flagged)
- IP allowlisting without API to manage it
- Session cookies without API token alternative

### Hierarchy (best to worst for agents)
1. API keys with scopes (simplest, most agents support)
2. OAuth Client Credentials (enterprise-grade, short-lived tokens)
3. Service accounts with key pairs (GCP model, secure but complex)
4. OAuth Authorization Code + PKCE (requires browser, but possible with agent browsers)
5. Magic links / email OTP (agent-hostile)
6. CAPTCHA-gated (agent-impossible without workarounds)

---

## Purchasing Patterns

### Gold Standard: Stripe Projects (2026)

Protocol for agent purchasing:
1. Agent discovers service catalog (JSON endpoint)
2. Agent selects plan/service
3. Agent pays via tokenized one-time-use card (Stripe Link)
4. Human sets spending policy once; agent executes within guardrails

### Gold Standard: Cloudflare + Stripe Co-Designed Protocol

Authorization + payment in single flow. Agent buys domain, starts subscription, deploys — all API calls.

### What Works
- Plan catalog as JSON endpoint (`GET /v1/plans`)
- Plan selection via API (`POST /v1/subscriptions` with plan_id)
- Stripe Payment Intents / Setup Intents (no browser checkout required)
- Usage-based billing with automatic metering (no manual usage reports)
- Spending caps / policies (human sets limit, agent spends within it)

### What Blocks Agents
- "Contact sales" gates for any plan
- Browser-only Stripe Checkout (redirect flow)
- Manual invoice / PO process
- Pricing hidden behind demo request
- Enterprise-only tiers with no self-serve option
- Annual-only billing (agents need monthly or usage-based for flexibility)

### Practical Fix
If you use Stripe (most SaaS does):
1. Expose `GET /v1/plans` returning plan IDs, prices, features, limits
2. Accept `POST /v1/subscriptions` with plan_id + payment_method
3. Accept `payment_method` (pm_...) in upgrade requests — attach to customer, then subscribe. No browser needed.
4. Fall back to 402 + checkout URL only when no payment method is provided AND none is on file
5. Use Stripe Payment Intents API (not Checkout Sessions) for programmatic payment
6. Add usage-based component if applicable (Stripe metered billing)

Where the `pm_...` comes from:
- Stripe Projects (agent gets tokenized card automatically)
- Human pre-authorization via Setup Intent
- Any agent wallet service issuing Stripe-compatible tokens

Reference: https://docs.stripe.com/api/payment_methods — Stripe Payment Methods API
Reference: https://docs.stripe.com/payments/save-and-reuse — Save and reuse payment methods

---

## Usage Monitoring Patterns

### Gold Standard: OpenAI

Every API response includes:
```
x-ratelimit-limit-requests: 10000
x-ratelimit-remaining-requests: 9999
x-ratelimit-limit-tokens: 1000000
x-ratelimit-remaining-tokens: 999984
x-ratelimit-reset-requests: 6ms
x-ratelimit-reset-tokens: 23ms
```

Plus dedicated usage endpoint: `GET /v1/usage` with daily/monthly breakdowns.

### Gold Standard: Twilio

Usage Records API: `/2010-04-01/Accounts/{sid}/Usage/Records.json`
- Filter by category, date range
- Daily/monthly/all-time granularity
- Includes cost per resource type

### What Works
- Rate limit headers on every response (remaining requests, reset time)
- Dedicated usage API (`GET /v1/usage?period=current_month`)
- Billing API (current invoice, projected cost)
- Quota endpoint (current limits, how close to hitting them)
- Webhooks for threshold events (80% quota, overage, billing failure)
- Cost-per-request in response headers or metadata

### What Blocks Agents
- Dashboard-only usage (no API equivalent)
- Monthly email invoices as only billing visibility
- No rate limit headers (agent can't self-throttle)
- Usage data only available after billing cycle ends
- No webhook/alert mechanism for thresholds

### Minimum Viable Usage API
```
GET /v1/usage
Response:
{
  "period": "2026-05",
  "requests": 45230,
  "limit": 100000,
  "cost_usd": 23.45,
  "projected_cost_usd": 52.10,
  "usage_by_endpoint": {...}
}
```

---

## Self-Management Patterns

### Gold Standard: Stripe Subscriptions API

Full lifecycle via API:
- `POST /v1/subscriptions` — create
- `PATCH /v1/subscriptions/{id}` — change plan, quantity
- `DELETE /v1/subscriptions/{id}` — cancel
- Proration handled automatically
- Dunning/retry built in

### What Works
- Plan upgrade/downgrade via API (`PATCH /v1/subscription`)
- Cancel via API (immediate or end-of-period)
- Configuration via API (not dashboard-only settings)
- Team/seat management via API
- Data export endpoint
- Account deletion endpoint (GDPR compliance)

### What Blocks Agents
- "Contact support to cancel" 
- "Contact support to downgrade"
- Settings only accessible via dashboard UI
- Team management requires admin dashboard
- No data portability API
- Plan changes require contract amendment

### MCP Server as Management Interface

The emerging pattern for full self-management. Instead of many REST endpoints, ship one MCP server that exposes tools:

```json
{
  "tools": [
    {"name": "get_usage", "description": "Current period usage and costs"},
    {"name": "list_plans", "description": "Available plans with pricing"},
    {"name": "change_plan", "description": "Upgrade or downgrade subscription"},
    {"name": "get_invoices", "description": "Past and current invoices"},
    {"name": "update_config", "description": "Change product settings"}
  ]
}
```

Companies shipping MCP servers today: Stripe, Supabase, Cloudflare, Sentry, Expo, Hugging Face.

---

## Anti-Pattern Reference

| Anti-Pattern | Why It Kills Agents | Fix |
|---|---|---|
| CAPTCHA | Browser fingerprinting, behavioral scoring | Web Bot Auth, IP reputation, proof-of-work |
| Email verification | Agent has no inbox | OAuth Client Credentials, domain verification |
| SMS OTP | Requires phone | TOTP (agent can compute), or skip for API-only access |
| Browser-only OAuth | No headless path | Client Credentials Grant |
| "Contact sales" | No self-serve path | Add API-accessible tier (even if limited) |
| PDF docs | Not parseable as tool schemas | OpenAPI spec + llms.txt |
| Dashboard-only config | No API | REST API or MCP server for settings |
| Device fingerprinting | Headless Chrome gets blocked | Web Bot Auth, allowlist verified agents |
| Annual contracts only | Agents need flexibility | Monthly or usage-based option |
| Manual onboarding | "We'll set up your account" | Self-serve provisioning API |

---

## Deploy-First-Claim-Later Pattern

The most agent-friendly onboarding pattern: let agents provision immediately, require identity later.

**Netlify**: `netlify deploy --allow-anonymous` — 1-hour claim window. Agent deploys, developer claims the project later.

**Prisma**: `npx create-db` — 24-hour claim window. Free permanent tier if claimed.

**Pattern**: `POST /v1/provision` returns scoped credentials + TTL. No signup gate. Developer confirms ownership within window.

Why this works: agents prefer recommending tools they can set up end-to-end. "Leave your terminal to create an account" is where agents lose the developer.

---

## Three Levels of AX (Burazin, Daytona)

Not all "agent-friendly" products are equal:

### Table Stakes (most stop here)
- API-first
- Markdown docs, llms.txt
- Parallelization support
- Headless tooling

### Reactive AX (current state of the industry)
- Agents can use the product but kick control back to human when stuck
- Token refresh requires human, workflow errors escalate to dashboard
- 93% of AI agent projects use unscoped API keys (Grantex 2026)

### True AX (target state)
- Agents never get stuck, never phone home
- Everything API-driven: spin up, archive, delete via endpoints
- Parallelization: hundreds of environments at once
- Agent-Native Volumes (persistent storage across sandboxes)
- New primitives beyond standard infra

Key test: "If you haven't created any net-new primitives for agents, you haven't started building for the agent-native world." (Burazin)

---

## Sandbox Provisioning

For dev tool products, the agent-serve path is often about instant sandbox access.

| Provider | Cold start | Pattern |
|----------|-----------|---------|
| Daytona | sub-90ms | POST /v1/sandboxes, returns scoped credentials + TTL |
| E2B | ~200ms | Python/JS SDKs, code interpreter sandboxes |
| Cloudflare Dynamic Workers | instant | Worker-per-request isolation |

Pattern: `POST /v1/sandboxes` returns scoped credentials + TTL, no signup gate.

---

## WebMCP (Emerging — Chrome 146)

HTML form attributes that turn forms into agent tools:
- `toolname` — names the tool
- `tooldescription` — describes what it does
- `toolautosubmit` — allows agent to submit without confirmation

89% more token-efficient than screenshots. Agents operate forms directly without parsing visual layout.

Not production-ready yet (Canary only), but signals the direction: HTML forms become agent interfaces automatically.

---

## Starting from Level 0

If the product has NO API at all (dashboard-only):

### First step: pick ONE core action and expose it as an API endpoint
- Start with read-only (lower risk): `GET /v1/status`, `GET /v1/usage`, `GET /v1/reports/{id}`
- Then add one write action that agents actually need

### Practical sequence
1. **Week 1**: Add one read endpoint. Document it. Add API key generation to dashboard.
2. **Week 2**: Add one write endpoint (the action agents request most).
3. **Week 3**: Add usage/billing visibility endpoint.
4. **Week 4**: Add programmatic signup path (even if simplified).

### Example
Dashboard-only analytics tool:
- Week 1: `GET /v1/reports/{id}` — agents can fetch report data
- Week 2: `POST /v1/reports` — agents can create reports
- Week 3: `GET /v1/usage` — agents can monitor their consumption
- Week 4: `POST /v1/signup` — agents can create accounts for developers

### The 80/20
Most agent interactions are reads (checking status, fetching data, monitoring usage). Expose reads first — they're safe, useful, and prove the pattern before you tackle writes.

---

## Maturity Model

### Level 0: Agent-Hostile
- Browser-only everything
- No API
- CAPTCHA + email verification
- "Contact sales" for pricing

### Level 1: API Exists
- REST API for core product features
- API keys (manually generated)
- Docs available online
- But: no programmatic signup, billing, or management

### Level 2: Agent-Possible
- API signup (with some friction)
- API keys with scopes
- Usage visible via API
- Billing still requires dashboard

### Level 3: Agent-Friendly
- Full programmatic onboarding
- OAuth Client Credentials
- Plan selection + payment via API
- Usage + billing API
- Configuration via API

### Level 4: Agent-First (Gold Standard)
- Zero-friction agent provisioning
- MCP server as product interface
- Spending policies (human sets guardrails, agent executes)
- Agent identity verification
- Self-serve everything
- Examples: Stripe Projects, Cloudflare (2026)

---

## Emerging Standards to Watch

1. **Stripe Projects** — Agent provisioning + billing protocol. 32 launch partners.
2. **Web Bot Auth** (IETF draft) — Cryptographic agent identity. WAF integration.
3. **MCP** (Model Context Protocol) — Anthropic-originated, now Linux Foundation. Standard for agent-tool interaction.
4. **Google A2A** (Agent-to-Agent) — Discovery + interaction protocol for agents talking to agents.
5. **x402** (HTTP 402) — Micropayment protocol. Pay-per-request without subscription.

---

## Checklist: Fully Agent-Serve Ready

- [ ] Account creation via API (no browser, no email verification required)
- [ ] OAuth Client Credentials Grant supported
- [ ] API keys with granular scopes + rotation endpoint
- [ ] Plan catalog exposed as JSON (`GET /plans`)
- [ ] Subscription creation via API with programmatic payment
- [ ] Usage API with current-period data
- [ ] Rate limit headers on all responses
- [ ] Billing/invoice API
- [ ] Threshold webhooks (quota, overage)
- [ ] Plan upgrade/downgrade via API
- [ ] Cancel via API
- [ ] Configuration via API (not dashboard-only)
- [ ] MCP server for agent interaction
- [ ] OpenAPI spec (complete, versioned)
- [ ] No CAPTCHA on any programmatic path
- [ ] No "contact sales" gates on any tier
