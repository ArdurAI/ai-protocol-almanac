# Architecture: Context Protocols, Authentication & Integration

How the context protocols, authentication & integration landscape is shaped, and how the Quest tests it.

## The landscape at a glance

| Tool | Tier | License | Focus | Notes |
|------|------|---------|-------|-------|
| MCP (Model Context Protocol) | A | MIT (Open) | Protocol | 97M monthly SDK downloads; agent-to-tool communication; Anth |
| A2A (Agent-to-Agent) | A | Apache-2.0 | Protocol | 150+ orgs in production; agent-to-agent coordination; Google |
| UCP (Universal Commerce Protocol) | A | Apache-2.0 | Protocol | 20+ retail partners; Google + Shopify; agent-driven commerce |
| x402 | A | Open | Protocol | 150M+ transactions; Coinbase → Linux Foundation; HTTP 402 st |
| WIMSE (IETF) | A | IETF Draft | Standard | Active WG; workload identity; SPIFFE successor; architecture |
| AIMS (IETF) | A | IETF Draft | Standard | Q1 2026; AI agent auth/authz; draft-klrc-aiagent-auth-01; `a |
| SCIM for Agents | A | IETF Draft | Standard | Q1 2026; agent lifecycle mgmt; draft-abbey-scim-agent-extens |
| Vercel AI SDK | A | Open Source | SDK | Widely adopted; multi-provider AI apps; TypeScript; 20+ prov |
| OpenAI Agents SDK | A | Open Source | SDK | 10M+ npm/week; OpenAI-native agents; Python/TypeScript; MCP  |
| Anthropic SDK | A | Open Source | SDK | High; Claude-native apps; extended thinking, prompt caching, |
| Portkey | A | Commercial + OSS | AI Gateway | Production; AI observability + routing; 250+ models; MCP Gat |
| HashiCorp Vault | A | Open + Enterprise | Secrets Mgmt | Enterprise standard; secret rotation; dynamic secrets, multi |
| AWS Secrets Manager | A | Commercial | Secrets Mgmt | AWS-native; secret rotation; automated rotation via Lambda;  |

## How the Quest tests a tool

Same harness for all entries; the judge was frozen before any tool ran:

```
Adapter[frozen CategoryAdapter contract]
  ├── setup()    → install, configure
  ├── load()     → ingest workload
  ├── await_ready() → async barrier
  ├── query()    → run test, get response
  └── teardown() → cleanup, measure
       ↓
Telemetry: latency · tokens · $ · ops notes
       ↓
Grading: deterministic + LLM judge (frozen prompts)
       ↓
Raw results JSON (published)
```

The `await_ready()` barrier is where async designs get their cost measured instead of hidden.

## License

Content is licensed CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
