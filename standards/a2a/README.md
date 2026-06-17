# Agent-to-Agent (A2A) Protocol — Deep Reference

The **Agent-to-Agent (A2A) Protocol** is an open protocol developed by Google and donated to the **Linux Foundation** in April 2025. It is the leading standard for agent-to-agent coordination, enabling heterogeneous AI agents to discover, negotiate, and collaborate on tasks.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest spec** | A2A v1.0 (May 2025) |
| **Governance** | Linux Foundation (since April 2025) |
| **License** | Apache 2.0 |
| **Transport** | HTTP, Server-Sent Events (SSE), JSON-RPC over HTTP |
| **Serialization** | JSON |
| **Primary use case** | Multi-agent task coordination; agent discovery and delegation |
| **Production organizations** | 150+ (as of June 2026) |
| **Status** | Active; growing ecosystem |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| A2A v0.1 | February 2025 | Google internal preview; agent task delegation |
| A2A v1.0 | May 2025 | Public release; Agent Cards; task streaming; artifact delivery |
| A2A v1.1 (expected) | Q3 2026 | Planned: payment integration (x402), MCP bridge, enhanced auth |
| ACP → A2A | August 2025 | IBM's ACP (Agent Communication Protocol) merged into A2A |

## Transport & Schema Details

### Core Protocol

A2A uses **HTTP with JSON payloads** as its primary transport. Server-Sent Events (SSE) is used for streaming task updates and artifact delivery.

| Transport | Use case | Notes |
|-----------|----------|-------|
| **HTTP + JSON** | Synchronous task requests, Agent Card discovery | Default; most widely supported |
| **SSE** | Streaming task status, artifact push, real-time updates | Optional; client subscribes to `/tasks/{id}/status` |
| **WebSocket** | Community extension; bidirectional streaming | Not in official spec; some implementations support it |
| **JSON-RPC** | Alternative transport for some SDKs | Less common than HTTP+JSON |

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Agent Card** | JSON document describing an agent's capabilities, skills, endpoint URL, and authentication requirements. Discovered via `/.well-known/agent.json` or manual configuration. |
| **Task** | The unit of work. A task has an ID, status, messages, and artifacts. Created via `tasks/send` or `tasks/sendSubscribe`. |
| **Message** | A turn in a conversation between agents. Contains role (user/agent), parts (text, file, data), and metadata. |
| **Artifact** | Output produced by an agent during task execution. Can be text, files, or structured data. Delivered via SSE or in the task response. |
| **Skill** | A capability advertised in the Agent Card. Skills have IDs, descriptions, and input/output schemas. |

### API Surface

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/.well-known/agent.json` | GET | Agent Card discovery — capabilities, endpoint, auth requirements |
| `/tasks/send` | POST | Send a task synchronously; receive final response |
| `/tasks/sendSubscribe` | POST | Send a task with streaming updates via SSE |
| `/tasks/get` | GET | Retrieve a task by ID |
| `/tasks/cancel` | POST | Cancel an in-flight task |
| `/tasks/{id}/status` | GET (SSE) | Subscribe to task status updates (SSE stream) |

### Task Lifecycle

```
1. Agent discovery: Client fetches Agent Card from /.well-known/agent.json
2. Task creation: Client POSTs to /tasks/send or /tasks/sendSubscribe
3. Task execution: Server processes the task, produces messages and artifacts
4. Status updates: SSE stream pushes status changes (submitted → working → input-required → completed/failed)
5. Result delivery: Final response contains all artifacts and messages
6. Cleanup: Optional cancel via /tasks/cancel; tasks may expire server-side
```

### Task Status States

| Status | Meaning |
|--------|---------|
| `submitted` | Task received but not yet started |
| `working` | Agent is actively processing |
| `input-required` | Agent needs more information from the client |
| `completed` | Task finished successfully |
| `failed` | Task failed with an error |
| `canceled` | Task was canceled by the client |

### Message Schema

```json
{
  "role": "user" | "agent",
  "parts": [
    { "type": "text", "text": "..." },
    { "type": "file", "file": { "name": "...", "mimeType": "...", "bytes": "base64..." } },
    { "type": "data", "data": { "key": "value" } }
  ]
}
```

## Known Implementations & SDKs

| SDK/Framework | Language | Maintainer | Notes |
|---------------|----------|------------|-------|
| A2A Python SDK | Python | Google / community | Official SDK; HTTP + SSE |
| A2A TypeScript SDK | TypeScript | Google / community | Official SDK; widely used |
| A2A Java SDK | Java | Google / community | Official SDK; enterprise focus |
| BeeAI (IBM) | Python | IBM | Agent platform; formerly ACP-based; now A2A-based |
| OpenAI Agents SDK | Python/TypeScript | OpenAI | Supports A2A for agent delegation |
| Vercel AI SDK | TypeScript | Vercel | Multi-provider; can wrap A2A agents |
| Google Cloud Agents | Various | Google | Production A2A agents on GCP |
| LangChain | Python/TypeScript | LangChain | Community A2A adapter |
| CrewAI | Python | CrewAI | Community A2A integration |

## Interoperability Notes

### A2A + MCP
- A2A agents can wrap MCP servers as tools, but this requires an adapter layer
- The A2A spec does not natively understand MCP capabilities; the adapter translates `tools/list` into A2A Skills
- A2A v1.1 is expected to add an MCP bridge specification

### A2A + x402
- Payment negotiation is not in A2A v1.0 but is planned for v1.1 via x402 integration
- The UCP (Universal Commerce Protocol) is designed to work alongside A2A for commerce workflows
- Some A2A implementations use custom headers for payment negotiation as a workaround

### A2A + WIMSE / AIMS
- Authentication in A2A is currently transport-level (OAuth 2.0, API keys, mTLS)
- IETF AIMS (draft-klrc-aiagent-auth-01) proposes an `agent_assertion` grant type that would integrate with A2A Agent Cards
- WIMSE workload identity is relevant for cloud-native A2A deployments

### A2A Federation
- The federation test (multi-agent routing across heterogeneous implementations) shows mixed results
- Some implementations don't support SSE streaming, causing fallback to polling
- Agent Card discovery is inconsistent: some use `.well-known`, others require manual configuration

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Agent Card discovery | GET `/.well-known/agent.json` | 404; wrong content-type; missing required fields |
| Task creation | POST `/tasks/send` with valid payload | Missing `taskId`; wrong message part types |
| Task streaming | POST `/tasks/sendSubscribe` with SSE client | SSE stream disconnects; no heartbeat; client timeout |
| Task status polling | GET `/tasks/{id}` | Task expires before retrieval; status out of sync |
| Task cancellation | POST `/tasks/{id}/cancel` | Server ignores cancel; task continues running |
| Artifact delivery | Receive artifact in task response or SSE | Missing `mimeType`; blob encoding errors |
| Skill schema validation | Validate skill input/output against advertised schema | Schema mismatch; optional fields treated as required |
| Error handling | Send malformed request | Error response not JSON; wrong HTTP status code |

## Known Issues & Sharp Edges

1. **Agent Card inconsistency**: The `.well-known/agent.json` path is not universally adopted. Some agents require manual configuration of the Agent Card, defeating the discovery purpose.

2. **SSE fragility**: The SSE streaming transport is the most powerful feature but also the most fragile. Network interruptions, proxy timeouts, and load balancer idle timeouts cause frequent stream drops. Many implementations fall back to polling.

3. **No built-in auth spec**: A2A v1.0 does not define an authentication protocol. It leaves auth to transport-level mechanisms (OAuth, API keys, mTLS). This creates interoperability gaps when agents from different vendors try to authenticate each other.

4. **Task state persistence**: The spec does not mandate task state persistence. Some implementations keep tasks for hours; others drop them after minutes. This causes "task not found" errors in long-running workflows.

5. **Message part type ambiguity**: The `data` part type is essentially a free-form JSON object. Different implementations use different conventions for structured data, causing parsing failures.

6. **Skill schema optional fields**: The spec says skill input/output schemas are "advisory," but many implementations treat them as mandatory, rejecting tasks that don't match the schema exactly.

7. **ACP merge artifacts**: The ACP → A2A merger (August 2025) left some implementations with dual-mode behavior. Some IBM-derived agents still use ACP-style REST endpoints alongside A2A endpoints, causing confusion.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| Agent Card refresh rate | No refresh mechanism in spec; adapter caches for 5 min | Documented as limitation |
| Task expiration time | Not specified; adapter assumes 1 hour default | Varies by implementation |
| `input-required` re-engagement | Spec says client sends follow-up; doesn't define how | Adapter uses same task ID with new message |
| Artifact streaming vs. batch | Some servers stream artifacts; others batch them | Adapter handles both |
| SSE reconnection | No reconnection spec; adapter implements exponential backoff | Community best practice |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 91% | Core API passes; SSE streaming has edge cases; Agent Card discovery inconsistent |
| **Latency** | 45ms median (task creation) / 200ms+ (SSE streaming) | HTTP is fast; SSE adds latency overhead |
| **Token Economics** | Moderate overhead | JSON envelope + HTTP headers; no compression spec |
| **Scale Behavior** | Degrades at 50+ concurrent tasks | SSE connections consume server resources; no multiplexing |
| **Ops Burden** | Medium-High | Agent Card management is manual; auth is transport-dependent |
| **Developer Experience** | Good | SDKs are well-documented; examples are plentiful; community is growing |
| **Data Sovereignty** | Strong | Open spec; Apache 2.0; Linux Foundation governance |

**Composite Score (2026-06):** 79/100 — Strong for agent coordination, but SSE fragility and auth gaps are the biggest pain points.

## Links

- **Spec**: https://a2a.org/specification (or Google A2A documentation)
- **GitHub**: https://github.com/google/a2a
- **SDKs**: Python, TypeScript, Java (official); LangChain, CrewAI (community)
- **IBM ACP archive**: https://github.com/ibm/acp (redirected to A2A)

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
