---
name: agent-serve-dev-ready
description: "Audits whether a SaaS product's API is good to build against — checks error responses, idempotency, pagination, versioning, OpenAPI spec, llms.txt, SDKs, test mode, MCP server quality, A2A readiness, and agent toolkit design. Good DX serves humans today and agents tomorrow. Use when evaluating integration quality or planning API improvements."
disable-model-invocation: true
user-invocable: true
allowed-tools: WebFetch Read(*) Write(*) Bash(find *) Bash(grep *) Bash(ls *)
argument-hint: "[url-or-nothing]"
---

# Is your API good to build against?

Access gates (signup, auth, billing) are necessary but not sufficient. If the API has vague errors, no retry safety, missing pagination, or no test mode, agents fail at integration even after they get in. Developers give up. Both route to competitors with better DX.

Good DX serves humans today and agents tomorrow. Agents have zero tolerance for ambiguity, undocumented behavior, and human-only flows. The same improvements that help agents self-correct on errors also help developers debug faster.

## Detect Mode

If `$ARGUMENTS` contains a URL:
Read the docs, API reference, SDK pages, quickstarts, changelog, llms.txt, and OpenAPI spec. Evaluate the integration experience from zero to first successful API call.

If `$ARGUMENTS` is empty:
Audit the current codebase. Look at error responses, pagination patterns, idempotency handling, API versioning, test mode support, SDK/docs quality, MCP server implementation, and agent toolkit design.

## What to Check

### Error Responses
- Do errors return structured JSON (not HTML 500 pages)?
- Is the format RFC 9457 Problem Details, or at minimum: type/code, human message, machine-parseable code, which parameter failed?
- Do errors include agent-actionable fields: `is_retriable`, `retry_after_seconds`, `documentation_url`?
- Is there a request ID on every response (`X-Request-Id`) for tracing?
- Do 429 responses include `Retry-After` with specific backoff?
- Are error codes documented with explanations?

### Idempotency
- Do mutating endpoints (POST/PATCH/DELETE) accept an `Idempotency-Key` header?
- What's the dedup window? (24 hours minimum)
- Are responses identical on replay (same status code, same body)?
- Is there a reconciliation path (lookup-by-client-reference)?

### Pagination
- Do list endpoints paginate?
- Is pagination cursor-based (stable across mutations) or offset-based (breaks when items change)?
- Does the response include `has_more` or equivalent?
- Is pagination consistent across all list endpoints?

### Versioning
- Is the API versioned?
- Can clients pin a version (header or URL)?
- Is there a deprecation policy with timeline (minimum 12 months)?
- Do breaking changes get announced before they ship?
- Is there a changelog with RSS/webhook feed?

### OpenAPI Spec
- Is there a published OpenAPI spec? Is it complete?
- Are all enum values, field constraints, and parameter formats documented?
- Are operation IDs descriptive and action-oriented (`createInvoice` not `postData`)?
- Is the spec versioned alongside the API?

### Documentation & Discoverability
- Does `/llms.txt` exist with curated technical content?
- Does `/llms-full.txt` exist with complete embedded documentation?
- Are docs available in markdown (not just rendered HTML)?
- Do docs include runnable code examples (not pseudocode)?
- Is there an API explorer / playground?

### SDKs & Time-to-First-Call
- Are there official SDKs? In which languages?
- Is there a quickstart: zero to first working request in under 5 minutes?
- How many steps from install to first successful API call?
- Is there a CLI scaffolding tool (`npx create-*` pattern)?

### Test Mode / Sandbox
- Is there a test mode with separate keys (`sk_test_*` pattern)?
- Are there test fixtures / magic values (test card numbers, test phone numbers)?
- Can test environments be provisioned and torn down programmatically?
- Is test data isolated from production?

### MCP Server Quality (if product ships one)
- Are tool names prefixed with service name (`stripe_create_payment` not `create_payment`)?
- Are tool descriptions written for AI agents ("Use this when..." phrasing)?
- Do tools report errors via `isError: true` in content (not protocol-level errors)?
- Is pagination cursor-based with `has_more`/`next_cursor`?
- Are tool annotations present (`readOnlyHint`, `destructiveHint`, `idempotentHint`)?
- Is the tool surface curated (10-15 tools, not full API dump)?
- Does the server support structured output (`outputSchema`)?
- Does authentication use OAuth 2.1 with PKCE for remote use?

### A2A Readiness (if product exposes agent-to-agent capabilities)
- Does the product expose an Agent Card at a well-known URL?
- Does the Agent Card declare capabilities, skills, and auth schemes?
- Are tasks resumable through interrupted states (`INPUT_REQUIRED`, `AUTH_REQUIRED`)?
- Are errors structured with machine-readable codes and recovery paths?
- Is streaming supported via SSE with proper lifecycle events?

### Agent Toolkit (if product ships one)
- Does it support multiple frameworks (LangChain, OpenAI Agents SDK, Vercel AI SDK)?
- Can developers scope agent permissions (read-only, specific operations)?
- Is the tool surface curated (10-15 operations, not full API dump)?
- Does it work in sandbox/test mode?

## What Blocks Integration

- HTML error pages instead of structured JSON
- `{"error": "something went wrong"}` — no actionable info
- No request IDs (can't trace failures)
- No idempotency (agents can't safely retry — retries cause duplicates)
- Offset-based pagination (breaks under concurrent mutations)
- No API versioning (updates break integrations silently)
- No test mode (must use production to experiment)
- Rate limiting with no `Retry-After` (agents guess and get banned)
- No OpenAPI spec (agents can't auto-discover endpoints)
- MCP server with 100+ tools (agents can't select the right one)
- llms.txt that's marketing copy instead of technical content
- Quickstart that takes more than 10 steps

## Deep Patterns
Read [patterns.md](references/patterns.md) for gold standards (Stripe error objects, idempotency keys, date-based versioning), MCP server design patterns, A2A Agent Card structure, agent toolkit design, and the time-to-first-call benchmark.

## Report

```
### Dev Readiness
**Today:** [what exists — error quality, idempotency, pagination, versioning, docs, SDKs, test mode, MCP, A2A]
**Blocks integration:** [specific friction — which of the above are missing or broken]
**Build:** [specific fix — what to add/change, effort estimate, priority order]
```

## Rules
- This is about QUALITY, not access. The other five focused skills cover access gates. This one covers "is it actually good once you're in."
- Good DX serves humans today and agents tomorrow. Frame recommendations as benefiting both.
- Time-to-first-call is the north star metric. How many steps from zero to a working API request?
- Idempotency is non-negotiable for agent use. Agents retry on network errors. Without idempotency keys, retries cause duplicate charges, double-creates, data corruption.
- Don't recommend "add SDKs in 7 languages." Start with OpenAPI spec — SDKs auto-generate from spec.
- Don't recommend an MCP server with 100 tools. A curated set of 10-15 well-designed tools beats a full API dump.
- Don't recommend A2A unless the product already has solid APIs and MCP. A2A is a layer on top.
- Reference real implementations: Stripe (errors, idempotency, versioning, agent toolkit), Twilio (quickstarts, error codes), CopilotKit (scaffolding UX), Cloudflare (MCP Code Mode, sandboxes).
