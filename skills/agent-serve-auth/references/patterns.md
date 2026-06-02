# Authentication Patterns

## Gold Standard: OAuth 2.0 Client Credentials Grant

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

## Gold Standard: Stripe API Keys

Simple, effective:
- Secret key for server-side (full access)
- Restricted keys with granular permissions
- Key rotation via API
- Test mode keys for sandbox

## Emerging: Web Bot Auth (IETF Draft)

Cryptographic agent identity. WAFs can whitelist verified agents without CAPTCHA. Amazon Bedrock AgentCore Browser supports this.

## What Works
- API keys with scoped permissions (read-only, billing-only, admin)
- Key rotation endpoint (`POST /v1/keys/rotate`)
- Multiple keys per account (different agents, different scopes)
- Client Credentials OAuth for enterprise/multi-tenant

## What Blocks Agents

| Anti-Pattern | Why It Kills Agents | Fix |
|---|---|---|
| Magic links | Requires email inbox + browser click | API key or Client Credentials |
| SMS OTP | Requires phone | TOTP (agent can compute) |
| Browser-only OAuth consent | Click "Allow" screen | Client Credentials Grant |
| Device fingerprinting | Headless browsers get flagged | Web Bot Auth, allowlist verified agents |
| IP allowlisting without API | Can't self-manage access | API to manage IP allowlist |
| Session cookies only | No API token alternative | Add bearer token auth path |

## Hierarchy (best to worst for agents)
1. API keys with scopes (simplest, most agents support)
2. OAuth Client Credentials (enterprise-grade, short-lived tokens)
3. Service accounts with key pairs (GCP model, secure but complex)
4. OAuth Authorization Code + PKCE (requires browser, but possible with agent browsers)
5. Magic links / email OTP (agent-hostile)
6. CAPTCHA-gated (agent-impossible without workarounds)
