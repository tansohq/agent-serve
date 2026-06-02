---
name: agent-serve-usage
description: "Finds what blocks AI agents from tracking their own consumption on a SaaS product — checks for usage API endpoints, rate limit headers, billing APIs, quota visibility, and threshold webhooks. Use when evaluating agent usage monitoring or building consumption visibility."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Can an agent track its own consumption?

If usage data lives only in a dashboard or monthly email invoices, agents can't self-throttle, budget, or react to limits. They need real-time consumption data via API.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the docs, API reference, and billing documentation. Look for usage endpoints, rate limit headers, billing APIs, and webhook documentation.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at usage tracking, rate limiting middleware, billing integration, webhook configuration.

## What to Check
- Do API responses include rate limit headers (`x-ratelimit-remaining-*`, `x-ratelimit-reset-*`)?
- Is there a dedicated usage API (`GET /v1/usage`) with current-period data?
- Is there a billing/invoice API?
- Can agents see their quotas and how close they are to limits?
- Are there webhooks for threshold events (80% quota, overage, billing failure)?
- Is cost-per-request visible in response headers or metadata?

## What Blocks Agents
- Dashboard-only usage (no API equivalent)
- Monthly email invoices as only billing visibility
- No rate limit headers (agent can't self-throttle)
- Usage data only available after billing cycle ends
- No webhook/alert mechanism for thresholds

## Deep Patterns
Read [patterns.md](references/patterns.md) for gold standards (OpenAI rate limit headers, Twilio Usage Records API) and the minimum viable usage API template.

## Report

```
### Usage Monitoring
**Today:** [what exists — usage visibility, rate limiting, billing access]
**Blocks agents:** [specific friction — which blockers are present]
**Build:** [specific fix — headers to add, endpoints to create, effort estimate]
```

## Rules
- Rate limit headers on every response is the single highest-impact, lowest-effort fix. Recommend it first.
- The minimum viable usage API is small: period, requests, limit, cost, projected cost. Don't over-scope it.
- Webhooks for threshold events are the difference between "agent can see usage" and "agent can react to usage."
- Reference OpenAI (per-request headers + usage endpoint) and Twilio (Usage Records API with filtering).
