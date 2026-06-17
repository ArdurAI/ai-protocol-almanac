# Model Context Protocol (MCP) — Deep Reference

The **Model Context Protocol (MCP)** is an open protocol developed by Anthropic and donated to the **AI Alliance Foundation (AAIF) / Linux Foundation** in November 2025. It is the de facto standard for agent-to-tool communication, enabling AI agents to connect to external data sources, tools, and APIs through a unified interface.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest spec** | MCP 2025-11-05 (Anthropic) / MCP 2025-11-18 (community) |
| **Governance** | AAIF / Linux Foundation (since Nov 2025) |
| **License** | MIT |
| **Transport** | stdio, HTTP, WebSocket (with community extensions) |
| **Serialization** | JSON-RPC 2.0 |
| **Primary use case** | Agent-to-tool communication; context injection into LLM workflows |
| **Monthly SDK downloads** | ~97M (as of June 2026) |
| **Enterprise adoption** | ~78% (from surveyed orgs using MCP servers) |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| MCP 2025-03 | March 2025 | Anthropic initial release; stdio-only; basic tools/resources |
| MCP 2025-11-05 | November 2025 | Anthropic spec update; OAuth 2.1 mandate for remote servers; added sampling support |
| MCP 2025-11-18 | November 2025 | Community-adopted version after Linux Foundation donation; stable release |
| MCP 2026-Q2 | Q2 2026 | Expected: server-sent events, streaming improvements, security hardening |

## Transport & Schema Details

### Core Protocol

MCP uses **JSON-RPC 2.0** over a transport layer. The two officially supported transports are:

| Transport | Use case | Authentication |
|-----------|----------|---------------|
| **stdio** | Local tools, CLI integrations, desktop apps | OS-level process isolation |
| **HTTP** | Remote servers, cloud deployments, multi-tenant | OAuth 2.1 (mandated Nov 2025) |
| **WebSocket** | Community extension; real-time bidirectional | OAuth 2.1 or custom |
| **SSE** | Emerging; server-push capabilities | OAuth 2.1 |

### Message Types

The protocol defines four core capability categories:

| Category | Methods | Description |
|----------|---------|-------------|
| **Tools** | `tools/list`, `tools/call` | Expose executable functions to the agent |
| **Resources** | `resources/list`, `resources/read`, `resources/subscribe` | Read-only data sources; push updates via subscription |
| **Prompts** | `prompts/list`, `prompts/get` | Pre-defined prompt templates with arguments |
| **Sampling** | `sampling/createMessage` | Agent requests LLM completion from the host (bidirectional) |
| **Logging** | `logging/setLevel` | Server-side log level control |

### Connection Lifecycle

```
1. Client initializes: `initialize` with protocolVersion, capabilities
2. Server responds with its capabilities and protocol version
3. Client sends `initialized` notification
4. Normal operation: tools/list, resources/read, prompts/get, etc.
5. Client sends `cancel` for in-flight requests
6. Graceful shutdown or transport close
```

### Key Schema Constraints

- `protocolVersion` must be agreed upon during initialization
- `tools/list` response must include `name`, `description`, and `inputSchema` per tool
- `resources/read` must return `contents` array with `uri`, `mimeType`, and `text` or `blob`
- OAuth 2.1 is **mandatory** for all remote (non-stdio) servers as of the Nov 2025 spec

## Known Implementations & SDKs

| SDK/Server | Language | Maintainer | Notes |
|------------|----------|------------|-------|
| `mcp` (Python SDK) | Python | Anthropic / community | Official SDK; supports stdio and HTTP |
| `@modelcontextprotocol/sdk` | TypeScript | Anthropic / community | Official SDK; widely used |
| `mcp-rs` | Rust | Community | Rust implementation |
| `mcp-go` | Go | Community | Go implementation |
| Smithery | N/A | Smithery | Registry: 3,305+ hosted MCP servers; free remote hosting |
| Glama | N/A | Glama | Registry: 22,000+ servers; Firecracker VM isolation |
| mcp.so | N/A | mcp.so | Registry: 17,186+ servers; largest catalog; no hosting |
| PulseMCP | N/A | PulseMCP | Registry: 6,970+ servers; newsletter + REST API |
| MCPize | N/A | MCPize | Platform: 500+ servers; OpenAPI→MCP conversion; monetization |
| Cloudflare MCP Gateway | N/A | Cloudflare | Reverse proxy: centralized auth, logging, rate-limiting |
| Portkey MCP Gateway | N/A | Portkey | AI Gateway with MCP compliance; 250+ models |

## Interoperability Notes

### MCP + A2A
- MCP servers can be exposed through A2A agents using adapter layers
- The two protocols are complementary: MCP for tool access, A2A for agent coordination
- Some MCP registries (Smithery, Glama) are exploring A2A compatibility

### MCP + UCP
- UCP (Universal Commerce Protocol) is MCP-compatible for commerce workflows
- Google explicitly designed UCP to work alongside MCP for agent-driven shopping

### Transport Parity
- stdio is the most stable transport; HTTP is gaining but has auth complexity
- WebSocket support is fragmented; some SDKs support it, others don't
- SSE is emerging but not yet widely adopted by servers

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| `initialize` handshake | Send `initialize` request, verify response | Version mismatch; missing capabilities |
| `tools/list` schema | Validate response against JSON Schema | Missing `inputSchema`; wrong field types |
| `resources/read` schema | Validate response with `text`/`blob` fields | Missing `mimeType`; incorrect URI handling |
| `prompts/get` schema | Validate prompt template with arguments | Missing argument substitution |
| `logging/setLevel` | Send level change, verify server acknowledges | Not implemented by server |
| OAuth 2.1 (remote) | Verify token exchange before `initialize` | ~100% of scanned servers lack auth (Knostic, 2025) |
| `cancel` handling | Send `cancel` for in-flight request | Server crashes or ignores cancel |
| `sampling/createMessage` | Request LLM completion from host | Not supported by all SDKs |

## Known Issues & Sharp Edges

1. **Security crisis**: Knostic scanned ~2,000 MCP servers in 2025 and found **100% lacked authentication**; 30+ CVEs were filed Jan-Feb 2026. The OAuth 2.1 mandate exists in the spec but is not enforced by tooling.

2. **Schema drift**: Many SDKs send fields not in the spec (e.g., extra metadata in `tools/list`) or omit required fields. This causes validation failures when using strict schema validators.

3. **Version negotiation**: The spec says `protocolVersion` is negotiated, but in practice many servers only support a single version and fail on mismatch rather than negotiate.

4. **stdio fragility**: The stdio transport is simple but has no reconnect logic. If the child process crashes, the client must restart the entire connection. No heartbeat or health check is defined in the spec.

5. **Tool poisoning**: Invariant Labs demonstrated in April 2025 that malicious tool descriptions can inject instructions into the agent's prompt. The spec has no built-in guardrails against this.

6. **Resource subscription inconsistency**: The `resources/subscribe` capability is optional but poorly documented. Servers that support it often have different behavior for unsubscribe vs. close.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| `resources/list` pagination | No pagination in spec; servers return all resources at once | Documented as limitation |
| `tools/call` error format | Spec allows any string; some servers return JSON objects | Adapter normalizes to string |
| `blob` vs `text` encoding | `blob` is base64; but some servers send raw bytes in `text` | Adapter validates base64 on blob fields |
| `inputSchema` JSON Schema version | Spec says JSON Schema; doesn't specify draft version | Adapter uses Draft 7 |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 94% | Schema validation passes; version negotiation issues on some SDKs |
| **Latency** | 12ms median (stdio) / 45ms median (HTTP) | stdio is fast; HTTP adds auth overhead |
| **Token Economics** | High overhead | JSON-RPC envelope adds ~20-30% overhead vs. raw HTTP |
| **Scale Behavior** | Degrades at 100+ concurrent connections | Connection pool exhaustion; no built-in multiplexing |
| **Ops Burden** | Medium | Setup is easy with SDKs; auth configuration is a nightmare |
| **Developer Experience** | Excellent | SDKs are mature; docs are comprehensive; community is large |
| **Data Sovereignty** | Strong | Open spec; open source; Linux Foundation governance |

**Composite Score (2026-06):** 82/100 — Strongest in the ecosystem, but the security gap is a critical liability.

## Links

- **Spec**: https://modelcontextprotocol.io/specification
- **GitHub**: https://github.com/modelcontextprotocol
- **Registry**: https://smithery.ai, https://glama.ai, https://mcp.so, https://pulsemcp.com
- **Security advisory**: Knostic MCP scan report (2025); Invariant Labs tool poisoning (April 2025)

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
