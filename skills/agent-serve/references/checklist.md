# Checklist: Fully Agent-Serve Ready

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
