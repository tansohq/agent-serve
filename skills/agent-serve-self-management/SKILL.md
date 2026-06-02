---
name: agent-serve-self-management
description: "Finds what blocks AI agents from changing plans, canceling, and configuring a SaaS product via API — checks for subscription lifecycle endpoints, configuration APIs, team management, data export, and MCP server as management interface. Use when evaluating agent self-management friction."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Can an agent change plans or cancel without a human?

If plan changes require "contact support" or settings live only in a dashboard, agents can't manage their own lifecycle. Full self-management means every action available via API.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the docs, API reference, and account management documentation. Look for subscription management endpoints, configuration APIs, and self-serve capabilities.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at subscription management routes, settings endpoints, team management, data export capabilities.

## What to Check
- Can plans be upgraded/downgraded via API (`PATCH /v1/subscription`)?
- Can subscriptions be canceled via API (immediate or end-of-period)?
- Is configuration changeable via API (not dashboard-only settings)?
- Is there team/seat management via API?
- Is there a data export endpoint?
- Is there an account deletion endpoint (GDPR compliance)?
- Does the product ship an MCP server for agent interaction?

## What Blocks Agents
- "Contact support to cancel"
- "Contact support to downgrade"
- Settings only accessible via dashboard UI
- Team management requires admin dashboard
- No data portability API
- Plan changes require contract amendment
- PDF-only documentation (not parseable as tool schemas)

## Deep Patterns
Read [patterns.md](references/patterns.md) for the Stripe Subscriptions API lifecycle, MCP server as management interface (with tool schema example), and which companies ship MCP servers today.

## Report

```
### Self-Management
**Today:** [what exists — subscription management, config access, MCP server]
**Blocks agents:** [specific friction — which blockers are present]
**Build:** [specific fix — endpoints to add, MCP tools to expose, effort estimate]
```

## Rules
- Don't recommend an MCP server unless the product already has a solid REST API. MCP is a layer on top of APIs, not a replacement.
- Stripe Subscriptions API is the reference implementation for lifecycle management — `POST`, `PATCH`, `DELETE` on subscriptions.
- Data export and account deletion are GDPR requirements anyway — frame them as compliance + agent-readiness.
- If settings are dashboard-only, the fix is usually a REST API for the same operations, not rebuilding the dashboard.
