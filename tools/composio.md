# Composio

| Attribute | Value |
|-----------|-------|
| Tier | B |
| Type | Integration Platform |
| License | Commercial |
| Stars | ~N/A (100K+ developers) |
| URL | https://composio.dev |

## One-line summary

An AI-native integration platform connecting agents to 900+ enterprise tools through managed authentication, pre-built connectors, and a Universal MCP Gateway, backed by $29M from Lightspeed.

## Architecture

Composio operates as a three-layer platform:

1. **Connector Layer**: 900+ pre-built, production-ready connectors to enterprise SaaS tools (Salesforce, HubSpot, Slack, GitHub, etc.). Each connector handles OAuth 2.0 flows, API key management, token refresh, and rate limiting automatically.

2. **Tool Abstraction Layer**: Provides clean, type-safe Python and TypeScript interfaces that let agents call external APIs through a unified programming model. The abstraction normalizes different API patterns (REST, GraphQL, SOAP) into a consistent agent-callable interface.

3. **MCP Gateway**: The Universal MCP Gateway exposes Composio's connectors as MCP servers, allowing any MCP-compatible client (Claude, VS Code, ChatGPT, etc.) to access all 900+ integrations through the standard MCP protocol.

The platform also includes built-in observability for agent workflows, tracking tool calls, latencies, and error rates across the connector fleet.

## Key features

- 900+ managed connectors with automatic auth handling
- Universal MCP Gateway — any MCP client can access all integrations
- Type-safe Python and TypeScript SDKs
- Managed authentication (OAuth 2.0, API keys, token refresh)
- Built-in observability for agent workflows
- $29M Series A from Lightspeed Venture Partners
- 100,000+ developers on the platform

## Benchmark status

| Dimension | Status | Notes |
|-----------|--------|-------|
| Accuracy | Not run | Connector reliability varies by API provider; no independent conformance testing |
| Latency | Not run | Gateway + connector overhead adds ~100-500ms per tool call (estimated) |
| Token Economics | Not run | Free tier available; paid plans per tool call volume |
| Scale Behavior | Not run | 900+ connectors but no published federation stress data |
| Ops Burden | Not run | Managed auth reduces OAuth burden significantly vs. DIY integrations |
| Developer Experience | Not run | Type-safe SDKs and managed auth are UX advantages |
| Data Sovereignty | Not run | Cloud-hosted platform; no self-hosted option known |

## Ops reality

- **Setup**: Sign up for Composio account, install SDK, configure connectors. The managed auth eliminates the typical OAuth 2.0 setup headache (consent screens, token storage, refresh logic).
- **Connector reliability**: Composio manages the connectors but the underlying APIs (Salesforce, HubSpot, etc.) can still change. The platform handles drift detection but there may be lag between upstream API changes and connector updates.
- **MCP Gateway**: This is the most interesting feature for the protocol landscape. It bridges the "every app needs its own integration" problem by making 900+ tools available through MCP. However, it's a commercial gateway — you pay for what you use, and the tool surface is enormous (which can be overwhelming for agents).
- **Rate limiting**: Composio handles rate limits for many APIs, but some enterprise APIs have their own aggressive limits. For high-throughput agent workloads, the platform may become a bottleneck.
- **Security posture**: Managed auth means Composio holds tokens for many integrations. This is a high-value target. The platform claims security measures but independent verification has not been conducted.

## Cost model

- **Free tier**: Limited tool calls and connectors
- **Paid plans**: Per tool call volume; enterprise pricing for high-throughput use cases
- **MCP Gateway**: Usage-based pricing when accessed through MCP
- **No open-source core**: The platform is proprietary; the MCP Gateway is a commercial offering

## When to use / when to avoid

**Use when:**
- You need agents to interact with 10+ enterprise SaaS tools and don't want to build 10+ OAuth integrations
- You are using MCP-compatible clients (Claude, VS Code, ChatGPT) and need access to business tools
- You need managed authentication (token refresh, rate limiting, drift detection) without building it yourself

**Avoid when:**
- You only need 1-2 integrations (DIY is cheaper and gives you more control)
- You have strict data residency requirements (cloud-hosted platform)
- You need open-source tooling (Composio is proprietary)
- You are building a simple agent that doesn't need enterprise integrations
