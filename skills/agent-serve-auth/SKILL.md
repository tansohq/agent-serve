---
name: agent-serve-auth
description: "Finds what blocks AI agents from authenticating with a SaaS product — checks for API keys with scoped permissions, OAuth Client Credentials Grant, key rotation endpoints, and blockers like magic links, SMS OTP, and browser-only OAuth consent. Use when evaluating agent authentication friction."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Can an agent prove identity without human ceremony?

If auth requires a browser click, email link, or phone call, agents can't authenticate. Machine-to-machine auth must work without human interaction.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the docs, API reference, and auth documentation. Look for API key management, OAuth flows, and auth requirements.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at auth middleware, token generation, OAuth configuration, session management.

## What to Check
- Do API keys exist? Can they be generated programmatically or only via dashboard?
- Are keys scoped (read-only, billing-only, admin)?
- Is there a key rotation endpoint?
- Does the product support OAuth Client Credentials Grant (machine-to-machine)?
- Is there a service account pattern?
- Are there multiple keys per account for different agents/scopes?

## What Blocks Agents
- Magic links (requires email inbox + browser click)
- SMS OTP (requires phone)
- Browser-only OAuth consent screens (click "Allow")
- Device fingerprinting (headless browsers get flagged)
- IP allowlisting without API to manage it
- Session cookies without API token alternative

## Deep Patterns
Read [patterns.md](references/patterns.md) for the auth hierarchy (best to worst for agents), OAuth Client Credentials flow, Stripe API key model, and Web Bot Auth (IETF draft).

## Report

```
### Authentication
**Today:** [what exists — auth methods, key types, OAuth support]
**Blocks agents:** [specific friction — which blockers are present]
**Build:** [specific fix — auth endpoint to add, key scoping to implement, effort estimate]
```

## Rules
- Reference the auth hierarchy: API keys with scopes > Client Credentials > Service accounts > Auth Code + PKCE > Magic links > CAPTCHA.
- Don't recommend removing human auth — recommend ADDING a machine-to-machine path alongside it.
- Key rotation is table stakes. If keys exist but can't be rotated via API, flag it.
- Web Bot Auth (profile of RFC 9421, IETF WG chartered early 2026) is live at Cloudflare, AWS WAF, Vercel, Shopify, Akamai (as of early 2026). Recommend it as a current fix for WAF/CAPTCHA friction, not just a future direction.
