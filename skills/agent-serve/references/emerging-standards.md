# Emerging Standards

1. **Stripe Projects** — Agent provisioning + billing protocol. Curated marketplace with scoped API keys and one-time-use virtual cards. Invitation-only.
2. **Web Bot Auth** (IETF draft) — Cryptographic agent identity. WAFs can whitelist verified agents without CAPTCHA. Amazon Bedrock AgentCore Browser supports this.
3. **MCP** (Model Context Protocol) — Anthropic-originated, now Linux Foundation. Standard for agent-tool interaction.
4. **Google A2A** (Agent-to-Agent) — Discovery + interaction protocol for agents talking to agents.
5. **x402** (HTTP 402) — Micropayment protocol. Pay-per-request without subscription.

---

## WebMCP (Emerging)

HTML form attributes that turn forms into agent tools:
- `toolname` — names the tool
- `tooldescription` — describes what it does
- `toolautosubmit` — allows agent to submit without confirmation

More token-efficient than screenshots. Agents operate forms directly without parsing visual layout. Signals the direction: HTML forms become agent interfaces automatically.
