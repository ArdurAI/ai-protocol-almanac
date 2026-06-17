# AIMS (AI Agent Authentication and Authorization) — Deep Reference

**AIMS** is an IETF draft standard for **AI agent authentication and authorization**. It defines how AI agents prove their identity, request permissions, and obtain authorization tokens in a standardized, interoperable way.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest draft** | draft-klrc-aiagent-auth-01 (Q1 2026) |
| **Governance** | IETF (Internet Engineering Task Force) |
| **License** | IETF Draft (open; no patent restrictions claimed) |
| **Transport** | HTTP (OAuth 2.0 / OIDC extension) |
| **Serialization** | JSON, JWT |
| **Primary use case** | AI agent authentication; agent-to-service and agent-to-agent authorization |
| **Status** | Active draft; early implementations emerging |
| **Related standards** | OAuth 2.0, OIDC, WIMSE, SCIM for Agents, MCP auth |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| Initial proposal | 2024 | Problem statement: AI agents need standardized auth |
| draft-klrc-aiagent-auth-00 | Q4 2025 | Initial draft; `agent_assertion` grant type; agent identity model |
| draft-klrc-aiagent-auth-01 | Q1 2026 | Updated draft; delegation chains; consent framework; WIMSE integration |
| draft-klrc-aiagent-auth-02 (expected) | Q4 2026 | Planned: A2A integration, payment authorization, multi-domain federation |
| RFC target | 2028+ | Expected as Informational or Standards Track RFC |

## Transport & Schema Details

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Agent Identity** | A unique, verifiable identity for an AI agent (e.g., `agent://example.com/shopping-agent`) |
| **Agent Assertion** | A cryptographically signed statement by an agent about its identity, capabilities, or intent |
| **`agent_assertion` Grant Type** | An OAuth 2.0 extension where the agent presents an assertion (instead of a user password) to obtain a token |
| **Delegation Chain** | A chain of agent delegations: user → agent A → agent B → service. Each link is signed and auditable. |
| **Consent Framework** | How an agent obtains user consent to act on their behalf; modeled after OAuth 2.0 consent but adapted for autonomous agents |
| **Capability Grant** | A token that grants specific capabilities (e.g., "can read orders, cannot modify payments") |

### The `agent_assertion` Grant Type

AIMS extends OAuth 2.0 with a new grant type specifically for AI agents:

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=agent_assertion
&agent_assertion=<JWT>
&scope=read_orders write_reviews
&audience=https://api.example.com
```

The `agent_assertion` is a JWT containing:

```json
{
  "iss": "agent://example.com/shopping-agent",
  "sub": "agent://example.com/shopping-agent",
  "aud": "https://auth.example.com",
  "iat": 1718500000,
  "exp": 1718503600,
  "agent_assertion": {
    "agent_id": "shopping-agent-123",
    "agent_version": "1.2.0",
    "platform": "vercel-ai-sdk",
    "capabilities": ["product_search", "cart_management", "checkout"],
    "delegation_chain": [
      {
        "delegator": "user://alice@example.com",
        "delegate": "agent://example.com/shopping-agent",
        "scope": "read_orders write_reviews",
        "consent_id": "consent-abc-123",
        "consent_timestamp": "2026-01-15T10:00:00Z"
      }
    ],
    "attestation": {
      "method": "wimse",
      "evidence": "sha256:..."
    }
  }
}
```

### Delegation Chain

```
User (Alice)
  ├── delegates to → Shopping Agent (agent://example.com/shopping-agent)
  │     ├── [consent: "read_orders, write_reviews"]
  │     ├── delegates to → Product Search Agent (agent://example.com/product-agent)
  │     │     ├── [scope: "read_orders"]
  │     │     └── calls → API (read product catalog)
  │     └── delegates to → Checkout Agent (agent://example.com/checkout-agent)
  │           ├── [scope: "write_reviews"]
  │           └── calls → API (submit review)
  └── each delegation is signed and auditable
```

### Consent Framework

| Consent Type | Description | Use case |
|-------------|-------------|----------|
| **Explicit Consent** | User actively approves each agent action | High-risk actions (payments, data deletion) |
| **Scoped Consent** | User pre-approves a scope of actions | Medium-risk (shopping, booking) |
| **Delegated Consent** | User delegates consent to another agent | Multi-agent workflows |
| **Revocable Consent** | User can revoke consent at any time | All consent types |

### Capability Grant Token

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read_orders write_reviews",
  "agent_id": "shopping-agent-123",
  "capabilities": {
    "product_search": { "max_results": 100, "filters": ["price", "rating"] },
    "cart_management": { "max_items": 50, "max_value": 10000 },
    "checkout": { "max_value": 5000, "payment_methods": ["x402", "credit_card"] }
  },
  "delegation_chain": [ ... ],
  "consent_id": "consent-abc-123"
}
```

## Known Implementations & SDKs

| SDK/Platform | Language | Maintainer | Notes |
|-------------|----------|------------|-------|
| AIMS Python SDK | Python | Community | Early implementation; OAuth 2.0 extension |
| AIMS TypeScript SDK | TypeScript | Community | Early implementation; integrates with Vercel AI SDK |
| Stytch AIMS Adapter | Various | Stytch | Auth platform with AIMS-inspired agent auth |
| Scalekit OAuth for AI | Various | Scalekit | B2B agent auth; OBO tokens; delegation support |
| Aembit MCP Auth | Various | Aembit | MCP authorization with OAuth 2.1, PRMs, workload identity |
| WIMSE + AIMS Bridge | Various | Community | Integration layer between WIMSE workload identity and AIMS agent auth |

## Interoperability Notes

### AIMS + OAuth 2.1 / OIDC
- AIMS is an extension of OAuth 2.0, not a replacement
- Existing OAuth 2.1 servers can add `agent_assertion` support with minimal changes
- The MCP spec (Nov 2025) mandates OAuth 2.1 for remote server auth; AIMS is the natural extension for agent-specific flows
- OIDC can be extended with AIMS claims for agent identity in identity tokens

### AIMS + WIMSE
- WIMSE provides workload identity ("this is a real agent running on a real platform")
- AIMS provides application identity ("this agent has permission to do X")
- The two are complementary: WIMSE attestation can be included in the `agent_assertion` JWT as proof of platform legitimacy
- The IETF is coordinating between the two working groups

### AIMS + A2A
- A2A Agent Cards can include `authRequirements` that reference AIMS-capable auth servers
- A2A agents can use `agent_assertion` to authenticate to other agents in a federation
- The A2A v1.1 draft is expected to formalize AIMS integration in the Agent Card schema

### AIMS + SCIM for Agents
- SCIM for Agents (draft-abbey-scim-agent-extension) manages agent lifecycle (create, update, delete, list)
- AIMS manages agent authentication and authorization
- The two standards are designed to work together: SCIM provisions the agent → AIMS authenticates the agent

### AIMS + x402 / UCP
- AIMS can authorize an agent to make payments via x402 or UCP
- The capability grant can include payment-specific limits (max amount, allowed payment methods)
- This is the bridge between agent identity and agent commerce

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| `agent_assertion` grant | POST `/token` with `grant_type=agent_assertion` | Server doesn't support grant type; wrong assertion format |
| Agent identity verification | Verify `iss` and `sub` in assertion | Agent ID not registered; assertion signature invalid |
| Delegation chain validation | Verify each link in delegation chain | Missing link; scope escalation; expired consent |
| Consent verification | Verify `consent_id` exists and is valid | Consent not found; consent revoked; wrong scope |
| Capability grant | Token includes correct capabilities | Missing capabilities; overly broad capabilities |
| Token refresh | Refresh token works for agent tokens | Refresh denied; scope changed without consent |
| Revocation | Revoked agent or consent is rejected | No revocation endpoint; cached token accepted |
| WIMSE attestation integration | Assertion includes valid WIMSE attestation | Missing attestation; attestation expired; wrong platform |

## Known Issues & Sharp Edges

1. **Draft instability**: AIMS is in early draft (draft-klrc-aiagent-auth-01). The `agent_assertion` format, delegation chain structure, and consent model are all subject to change. Implementing AIMS now means accepting future breaking changes.

2. **No production implementations**: As of June 2026, there are no widely deployed production implementations of AIMS. The existing SDKs are early community efforts. Stytch, Scalekit, and Aembit have AIMS-inspired features but are not fully compliant.

3. **Delegation chain complexity**: Multi-hop delegation chains (user → agent A → agent B → service) are theoretically elegant but practically complex. Each hop adds latency, signature verification overhead, and potential failure points. Debugging a failed delegation chain is difficult.

4. **Consent UX gap**: The spec defines consent models but doesn't specify how consent is presented to users. In practice, consent UI is often a simple OAuth-style approval screen, which doesn't adequately explain what an agent will do. This is a security risk.

5. **Scope granularity**: The spec doesn't define a standard scope taxonomy. "`read_orders`" means different things on different platforms. This creates interoperability problems when agents cross platform boundaries.

6. **Token size**: Delegation chains with multiple hops produce large JWTs. These may exceed HTTP header size limits or cause performance issues in high-throughput systems.

7. **Revocation at scale**: Revoking a consent or an agent identity requires propagating the revocation to all services that accepted the agent's tokens. In a large federation, this is operationally challenging.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| `agent_assertion` JWT format | Draft-01 defines structure; some implementations use different field names | Adapter follows draft-01 exactly |
| Delegation chain depth limit | Not specified; adapter supports up to 10 hops | Arbitrary limit; real-world chains are usually 2-3 hops |
| Consent format | Not specified; adapter uses OAuth 2.0 consent screen model | Platform-specific |
| Capability schema | No standard schema; adapter uses flat key-value pairs | Community convention |
| Token lifetime for agents | Not specified; adapter uses 1 hour default | Should be shorter than human tokens (agents are more autonomous) |
| Cross-domain agent identity | Not specified; adapter assumes single trust domain | Multi-domain federation is unresolved |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 65% | Draft is clear but not yet implemented; community SDKs are partial |
| **Latency** | 50-100ms (token issuance) | Delegation chain verification adds overhead |
| **Token Economics** | Moderate overhead | JWT with delegation chains is large; ~2-5KB per token |
| **Scale Behavior** | Untested | No production deployments at scale |
| **Ops Burden** | High | Draft status means frequent changes; no migration guide exists |
| **Developer Experience** | Poor | SDKs are early; docs are draft-quality; examples are sparse |
| **Data Sovereignty** | Excellent | IETF governance; open draft; no vendor lock-in |

**Composite Score (2026-06):** 62/100 — Critically important for the future of agent authentication, but too early for production adoption. The draft needs more implementations and real-world testing before it can be evaluated fairly.

## Links

- **IETF Draft**: https://datatracker.ietf.org/doc/draft-klrc-aiagent-auth/
- **IETF AIMS Interest**: https://datatracker.ietf.org/group/aiagent/
- **OAuth 2.0 spec**: https://tools.ietf.org/html/rfc6749
- **MCP auth spec**: https://modelcontextprotocol.io/specification/authorization
- **Stytch**: https://stytch.com
- **Scalekit**: https://scalekit.com
- **Aembit**: https://aembit.io

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
