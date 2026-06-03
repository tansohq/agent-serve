# Emerging Standards

1. **Stripe Projects** — Agent provisioning + billing protocol. Curated marketplace with scoped API keys and one-time-use virtual cards. Invitation-only.
2. **Web Bot Auth** (RFC 9421 HTTP Message Signatures) — Cryptographic agent identity. Agents sign requests with a public key registered in a well-known directory; WAFs verify without CAPTCHA. No longer draft-only — Cloudflare, AWS WAF, Vercel, Shopify, Akamai have implemented support. Visa Trusted Agent Protocol and Mastercard Agent Pay build on top. Amazon Bedrock AgentCore Browser uses it to reduce CAPTCHAs.
3. **MCP** (Model Context Protocol) — Anthropic-originated, now Linux Foundation. Standard for agent-tool interaction.
4. **Google A2A** (Agent-to-Agent) — Discovery + interaction protocol for agents talking to agents.
5. **x402** (HTTP 402) — Micropayment protocol. Pay-per-request without subscription. Rails are real (Stripe integrated USDC on Base; governance moved to Linux Foundation with Coinbase, Cloudflare, Google, Visa, Stripe, AWS, Mastercard as members). Sustained commercial volume is unproven — independent analysis found ~half of reported transaction volume was wash-traded, and daily transactions dropped ~92% from Dec 2025 peak to Feb 2026. Watch, but don't build on volume claims yet.

---

## WebMCP (Emerging)

HTML form attributes that turn forms into agent tools:
- `toolname` — names the tool
- `tooldescription` — describes what it does
- `toolautosubmit` — allows agent to submit without confirmation

More token-efficient than screenshots. Agents operate forms directly without parsing visual layout. Signals the direction: HTML forms become agent interfaces automatically.
