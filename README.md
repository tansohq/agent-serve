# agent-serve

Can AI agents buy and use your product without a human? Find out.

## Install

```bash
npx skills add katrinalaszlo/agent-serve
```

## The thesis

Agents are becoming buyers. If they can't sign up, authenticate, pay, and use your product programmatically, you're invisible to the agentic economy.

This skill audits your SaaS product's readiness for agent customers.

## Two modes

```bash
/agent-serve https://example.com    # Audit a live product
/agent-serve                        # Audit from codebase
```

## Five dimensions (scored 0-10)

| Dimension | What it checks |
|-----------|---------------|
| **Onboarding** | Can an agent create an account without a browser? |
| **Authentication** | API keys? OAuth client credentials? Or CAPTCHA hell? |
| **Purchasing** | Can an agent select a plan and pay programmatically? |
| **Usage Monitoring** | Can an agent track its own consumption via API? |
| **Self-Management** | Can an agent upgrade, downgrade, cancel without a human? |

## What blocks agents (anti-patterns)

- CAPTCHA / reCAPTCHA
- Email verification loops
- SMS OTP
- Browser-only OAuth consent
- "Contact sales" gates
- PDF-only documentation
- Dashboard-only configuration

## Gold standards

- **Stripe Projects** — Agents get scoped API keys, select services from catalog, pay via Stripe Payments API (invitation-only, 32 launch partners)
- **Cloudflare** — Agents create accounts, buy domains, deploy apps. Zero manual steps.
- **Twilio** — Full API for everything. Usage Records API, programmatic number provisioning.
- **OpenAI** — Usage endpoints, rate limit headers on every response, auto-tier promotion.

## What good looks like

A fully agent-serve product has:
1. API-based account creation (no browser required)
2. OAuth Client Credentials or API key auth (no human in the loop)
3. Programmatic plan selection + payment (Stripe-powered or equivalent)
4. Usage/billing API (real-time consumption data)
5. Self-serve plan changes via API (upgrade, downgrade, cancel)
6. MCP server as the agent-facing product interface

## Output

The skill produces a scorecard with specific recommendations for each dimension, referencing how gold-standard companies solve each problem.

## Author

Kat Laszlo — [@katlaszlo](https://x.com/Katlaszlo)
