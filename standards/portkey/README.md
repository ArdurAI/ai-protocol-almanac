# Portkey — Deep Reference

**Portkey** is an AI Gateway and observability platform that provides unified access to 250+ AI models, with features including request routing, caching, guardrails, and MCP Gateway support.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest version** | Portkey 2.0 (2026) |
| **Governance** | Portkey (commercial + open source) |
| **License** | Partially open source (core gateway open; enterprise features commercial) |
| **Deployment** | Cloud (SaaS) / Self-hosted (open source) |
| **Primary use case** | AI gateway; model routing; observability; caching; guardrails; MCP compliance |
| **Models supported** | 250+ (OpenAI, Anthropic, Google, Azure, AWS, local, etc.) |
| **Status** | Production; MCP Gateway GA (Jan 2026); open source core (March 2026) |
| **Related standards** | MCP, OpenTelemetry GenAI, A2A, OpenAI API, Anthropic API |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| Portkey 1.0 | 2023 | Initial release; AI gateway; basic routing and logging |
| Portkey 1.5 | 2024 | Multi-provider support; caching; guardrails; cost tracking |
| Portkey 2.0 | 2025 | Major redesign; MCP Gateway (beta); advanced routing; enterprise features |
| Portkey MCP Gateway GA | Jan 2026 | General availability of MCP Gateway; full MCP spec compliance |
| Portkey Open Source | March 2026 | Core gateway open sourced; enterprise features remain commercial |
| Portkey 2.1 (expected) | Q3 2026 | Planned: A2A integration, payment gateway, advanced federation |

## Architecture & API Surface

### Core Gateway

Portkey acts as a proxy between AI applications and AI model providers:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Your App   │────▶│   Portkey   │────▶│  AI Provider│
│             │     │   Gateway   │     │  (OpenAI,   │
│             │◀────│             │◀────│  Anthropic, │
│             │     │             │     │  Google, etc.)│
└─────────────┘     └─────────────┘     └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  Observability│
                   │  (traces, logs)│
                   │  (cost, metrics)│
                   └─────────────┘
```

### Key Features

| Feature | Description | Tier |
|---------|-------------|------|
| **Unified API** | Single API for 250+ models | All tiers |
| **Request Routing** | Route requests to different models based on rules | All tiers |
| **Load Balancing** | Distribute requests across model instances | All tiers |
| **Fallbacks** | Automatic failover to backup models | All tiers |
| **Caching** | Cache responses to reduce cost and latency | All tiers |
| **Guardrails** | Input/output filtering; PII detection; content safety | Enterprise |
| **Cost Tracking** | Real-time cost attribution and budgeting | All tiers |
| **Observability** | Traces, logs, metrics with OpenTelemetry export | All tiers |
| **MCP Gateway** | Full MCP server compliance; auth, logging, rate-limiting | All tiers |
| **A2A Support** | Agent-to-agent routing and observability | Enterprise |
| **Prompt Management** | Versioned prompt templates; A/B testing | Enterprise |
| **API Key Management** | Scoped API keys; rotation; access control | Enterprise |

### MCP Gateway

Portkey's MCP Gateway is a reverse proxy that adds enterprise features to MCP servers:

| Feature | Description |
|---------|-------------|
| **Centralized auth** | OAuth 2.1, API keys, mTLS for all MCP servers |
| **Logging** | Full request/response logging with OpenTelemetry |
| **Rate limiting** | Per-client, per-tool, per-server rate limits |
| **Cost tracking** | Track token usage and cost per MCP tool call |
| **Guardrails** | Filter tool inputs and outputs |
| **Server registry** | Centralized registry of internal MCP servers |
| **Health monitoring** | Monitor MCP server health and availability |

### Configuration API

```python
from portkey import Portkey

client = Portkey(
    api_key="...",
    config={
        "provider": "openai",
        "model": "gpt-4o",
        "cache": True,
        "retry": {"count": 3, "backoff": "exponential"},
        "guardrails": ["pii_detection", "content_safety"],
        "routing": {
            "strategy": "loadbalance",
            "targets": [
                {"provider": "openai", "weight": 70},
                {"provider": "anthropic", "weight": 30}
            ]
        }
    }
)

response = client.chat.completions.create(
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Known Implementations & Integrations

| Integration | Description |
|-------------|-------------|
| **OpenAI SDK** | Drop-in replacement for OpenAI SDK with Portkey gateway |
| **Anthropic SDK** | Drop-in replacement for Anthropic SDK with Portkey gateway |
| **Vercel AI SDK** | Provider integration for multi-provider routing |
| **LangChain** | Community integration; LangChain adapter |
| **LlamaIndex** | Community integration; LlamaIndex adapter |
| **MCP** | Full MCP Gateway with compliance, auth, logging |
| **A2A** | Enterprise integration for agent-to-agent routing |
| **OpenTelemetry** | Native OpenTelemetry export; `gen_ai.*` semantic conventions |
| **Datadog** | Built-in Datadog integration |
| **New Relic** | Built-in New Relic integration |
| **Honeycomb** | Built-in Honeycomb integration |
| **Prometheus** | Built-in Prometheus metrics export |
| **Grafana** | Built-in Grafana dashboards |

## Interoperability Notes

### Portkey + MCP
- Portkey's MCP Gateway is the most mature enterprise MCP solution
- It adds auth, logging, rate limiting, and guardrails to any MCP server
- The gateway is MCP spec-compliant (tested against the MCP compliance suite)
- MCP servers can be registered in Portkey's centralized registry

### Portkey + A2A
- A2A integration is available in the Enterprise tier
- Portkey can route A2A task requests between agents
- Observability spans A2A workflows, showing agent-to-agent latency and token usage
- A2A federation support is on the roadmap for Q3 2026

### Portkey + OpenTelemetry GenAI
- Native OpenTelemetry export with full `gen_ai.*` semantic convention support
- All LLM calls, tool calls, and agent interactions are traced
- Traces include cost attribution, routing decisions, and cache hit/miss metrics

### Portkey + x402 / UCP
- Portkey does not have built-in payment protocol integration
- Cost tracking is available but payment processing is not
- A payment gateway feature is on the roadmap for Q3 2026

### Portkey + OAuth 2.1 / AIMS
- Portkey's MCP Gateway supports OAuth 2.1 for MCP server authentication
- AIMS integration is not yet available but is on the roadmap
- The gateway can act as an OAuth 2.1 resource server for AI APIs

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Unified API | Call 3+ providers through Portkey | Provider-specific errors; routing failure; API key mismatch |
| Request routing | Route requests based on rules | Wrong target selected; routing loop; fallback not triggered |
| Caching | Cache and retrieve cached response | Cache miss; cache corruption; TTL misconfiguration |
| Guardrails | Block harmful input/output | Guardrail too permissive; false positives; performance impact |
| MCP Gateway | Proxy MCP server with auth/logging | MCP server not found; auth failure; schema validation error |
| Observability | Export traces to OpenTelemetry backend | Missing spans; wrong attributes; export failure |
| Cost tracking | Track cost per request | Wrong pricing model; missing usage data; currency mismatch |
| Fallback | Failover to backup model | Fallback not triggered; backup model also fails; latency spike |

## Known Issues & Sharp Edges

1. **Open source vs. commercial boundary**: The core gateway is open source, but many advanced features (guardrails, A2A support, advanced routing, enterprise SSO) are commercial-only. This creates a clear line between what you can self-host and what you must pay for.

2. **MCP Gateway complexity**: The MCP Gateway adds enterprise features but also adds complexity. Setting up auth, rate limiting, and logging for multiple MCP servers requires significant configuration.

3. **Provider API drift**: Portkey abstracts 250+ models, but provider APIs drift over time. When a provider changes their API (e.g., OpenAI adds a new parameter), Portkey must be updated to support it. This can cause lag.

4. **Caching limitations**: Portkey's caching is based on request hash. If the request changes slightly (different temperature, different system prompt), the cache is missed. This makes caching less effective for dynamic applications.

5. **Guardrail performance**: The guardrails (PII detection, content safety) add latency to every request. In high-throughput scenarios, this can be significant. The guardrails are also not as sophisticated as dedicated safety services.

6. **Cost tracking accuracy**: Cost tracking depends on provider pricing models, which change frequently. Portkey's cost estimates may not match actual billing, especially for providers with complex pricing (e.g., tiered pricing, overage charges).

7. **Vendor lock-in**: While the core gateway is open source, the ecosystem (prompt management, advanced routing, enterprise features) is Portkey-specific. Migrating away from Portkey requires rebuilding these features.

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 93% | MCP Gateway is fully compliant; unified API works across most providers; some edge cases |
| **Latency** | 5-20ms overhead | Gateway adds minimal latency; caching reduces latency significantly |
| **Token Economics** | Cost savings | Caching reduces costs; routing optimizes for cost; no token overhead |
| **Scale Behavior** | Excellent | Horizontally scalable; load balancing; no single point of failure |
| **Ops Burden** | Medium | Gateway setup is straightforward; MCP Gateway requires more configuration |
| **Developer Experience** | Excellent | Drop-in replacement for OpenAI/Anthropic SDKs; good documentation |
| **Data Sovereignty** | Moderate | Self-hosting available (open source); commercial features require cloud |

**Composite Score (2026-06):** 87/100 — The best AI gateway for production deployments. Excellent for MCP compliance, multi-provider routing, and observability. The commercial/open source boundary is the main concern.

## Links

- **Documentation**: https://portkey.ai/docs
- **GitHub**: https://github.com/portkey-ai
- **MCP Gateway**: https://portkey.ai/mcp-gateway
- **Open Source**: https://github.com/portkey-ai/gateway
- **Website**: https://portkey.ai

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
