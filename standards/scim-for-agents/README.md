# SCIM for Agents — Deep Reference

**SCIM for Agents** is an IETF draft extension to the System for Cross-domain Identity Management (SCIM) protocol, designed specifically for **AI agent lifecycle management**: provisioning, updating, deprovisioning, and auditing agents across systems.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest draft** | draft-abbey-scim-agent-extension-00 (Q1 2026) |
| **Governance** | IETF (Internet Engineering Task Force) |
| **License** | IETF Draft (open; no patent restrictions claimed) |
| **Transport** | HTTP (REST API) |
| **Serialization** | JSON (SCIM 2.0 format) |
| **Primary use case** | Agent lifecycle management; provisioning, updating, deprovisioning, auditing |
| **Status** | Active draft; early implementations emerging |
| **Related standards** | SCIM 2.0 (RFC 7643/7644), AIMS, WIMSE, OAuth 2.0 |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| SCIM 2.0 Core | 2015 | RFC 7643 (schema) / RFC 7644 (protocol); user and group provisioning |
| SCIM 2.0 Enterprise User | 2019 | RFC extension for enterprise user attributes |
| SCIM for Agents proposal | 2024 | Problem statement: agents need lifecycle management like users |
| draft-abbey-scim-agent-extension-00 | Q1 2026 | Initial draft; `Agent` and `AgenticApplication` resource types; agent provisioning API |
| draft-abbey-scim-agent-extension-01 (expected) | Q4 2026 | Planned: delegation chains, audit logging, multi-domain federation |
| RFC target | 2028+ | Expected as Standards Track RFC |

## Transport & Schema Details

### Core Protocol

SCIM for Agents extends SCIM 2.0 with two new resource types:

| Resource Type | Description | Base Path |
|--------------|-------------|-----------|
| **Agent** | Represents an AI agent: identity, capabilities, version, platform, auth credentials | `/Agents` |
| **AgenticApplication** | Represents an application that uses agents: deployment, configuration, agent assignments | `/AgenticApplications` |

### Agent Resource Schema

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Agent"],
  "id": "agent-123e4567-e89b-12d3-a456-426614174000",
  "externalId": "shopping-agent-001",
  "meta": {
    "resourceType": "Agent",
    "created": "2026-01-15T10:00:00Z",
    "lastModified": "2026-03-20T14:30:00Z",
    "version": "W/\"a330bc54f0671c9\"",
    "location": "https://scim.example.com/Agents/agent-123e4567-e89b-12d3-a456-426614174000"
  },
  "agentName": "Shopping Assistant",
  "agentId": "shopping-agent-001",
  "agentVersion": "2.1.0",
  "platform": "vercel-ai-sdk",
  "capabilities": [
    {
      "name": "product_search",
      "description": "Search products across catalogs",
      "inputSchema": { ... },
      "outputSchema": { ... }
    },
    {
      "name": "cart_management",
      "description": "Add, remove, and view cart items",
      "inputSchema": { ... },
      "outputSchema": { ... }
    }
  ],
  "status": "active",
  "authCredentials": {
    "type": "oauth2",
    "clientId": "agent-001-client",
    "scope": "read_orders write_reviews",
    "tokenEndpoint": "https://auth.example.com/token"
  },
  "delegationChain": [
    {
      "delegator": "user://alice@example.com",
      "scope": "read_orders write_reviews",
      "consentId": "consent-abc-123",
      "grantedAt": "2026-01-15T10:00:00Z",
      "expiresAt": "2026-07-15T10:00:00Z"
    }
  ],
  "auditLog": [
    {
      "timestamp": "2026-03-20T14:30:00Z",
      "event": "capability_updated",
      "actor": "admin@example.com",
      "details": { ... }
    }
  ],
  "owner": "alice@example.com",
  "parentApplication": "AgenticApplication-789"
}
```

### AgenticApplication Resource Schema

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"],
  "id": "app-789e4567-e89b-12d3-a456-426614174000",
  "meta": { ... },
  "applicationName": "E-commerce Agent Platform",
  "applicationId": "ecommerce-platform-001",
  "deployment": {
    "environment": "production",
    "region": "us-east-1",
    "platform": "kubernetes",
    "namespace": "agents"
  },
  "agents": [
    {
      "value": "agent-123e4567-e89b-12d3-a456-426614174000",
      "display": "Shopping Assistant"
    }
  ],
  "configuration": {
    "maxAgents": 10,
    "defaultAuthMethod": "oauth2",
    "allowedCapabilities": ["product_search", "cart_management", "checkout"]
  },
  "status": "active",
  "owner": "platform-team@example.com"
}
```

### API Surface

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/Agents` | GET | List all agents (with filtering, pagination) |
| `/Agents` | POST | Create a new agent |
| `/Agents/{id}` | GET | Retrieve an agent by ID |
| `/Agents/{id}` | PUT | Replace an agent entirely |
| `/Agents/{id}` | PATCH | Update specific agent attributes |
| `/Agents/{id}` | DELETE | Deprovision (delete) an agent |
| `/AgenticApplications` | GET | List all agentic applications |
| `/AgenticApplications` | POST | Create a new agentic application |
| `/AgenticApplications/{id}` | GET | Retrieve an application by ID |
| `/AgenticApplications/{id}` | PUT/PATCH | Update an application |
| `/AgenticApplications/{id}` | DELETE | Delete an application |
| `/Bulk` | POST | Bulk operations (create, update, delete multiple agents) |

### Filtering & Query

SCIM for Agents supports standard SCIM 2.0 filtering:

```
GET /Agents?filter=status eq "active" and platform eq "vercel-ai-sdk"
GET /Agents?filter=capabilities.name co "search"
GET /Agents?filter=meta.lastModified gt "2026-01-01T00:00:00Z"
GET /Agents?sortBy=meta.lastModified&sortOrder=descending
```

## Known Implementations & SDKs

| SDK/Platform | Language | Maintainer | Notes |
|-------------|----------|------------|-------|
| SCIM 2.0 Java SDK | Java | UnboundID / Ping Identity | Mature SCIM library; can be extended for Agent resources |
| SCIM 2.0 Python SDK | Python | Various | Community libraries; partial Agent support |
| SCIM 2.0 Node.js SDK | JavaScript | Various | Community libraries; partial Agent support |
| Okta SCIM | N/A | Okta | Identity platform; can provision agents as "users" |
| Azure AD SCIM | N/A | Microsoft | Enterprise provisioning; agent support via custom schemas |
| Ping Identity SCIM | N/A | Ping Identity | Enterprise identity; SCIM 2.0 compliant |
| Custom SCIM servers | Various | Various | Many organizations build custom SCIM servers for agent management |

## Interoperability Notes

### SCIM for Agents + AIMS
- SCIM for Agents provisions the agent (create, update, delete)
- AIMS authenticates the agent (obtain tokens, prove identity)
- The two are complementary: SCIM is the "directory" → AIMS is the "auth server"
- The `authCredentials` field in the Agent resource can reference the AIMS token endpoint

### SCIM for Agents + WIMSE
- The Agent resource can include WIMSE attestation evidence in the `authCredentials` section
- This allows a SCIM server to verify that an agent is running on a legitimate platform before provisioning it
- The integration is still draft-quality; no production implementations exist

### SCIM for Agents + MCP / A2A
- MCP servers and A2A agents can be registered as SCIM Agent resources
- This enables centralized management of a fleet of MCP servers and A2A agents
- The `capabilities` field maps naturally to MCP tools and A2A skills

### SCIM for Agents + Traditional SCIM
- SCIM for Agents is a superset of SCIM 2.0. Existing SCIM 2.0 servers can be extended with Agent resource support
- The `Agent` resource type is a new schema; it doesn't conflict with User or Group resources
- Backward compatibility: SCIM 2.0 clients that don't understand Agent resources will ignore them

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Agent creation | POST `/Agents` with valid Agent JSON | Missing required fields; wrong schema URI; duplicate `agentId` |
| Agent retrieval | GET `/Agents/{id}` | Agent not found; wrong ID format |
| Agent update | PATCH `/Agents/{id}` with partial update | Update rejected; version conflict; field not mutable |
| Agent deprovision | DELETE `/Agents/{id}` | Cascade delete not handled; dependent resources left behind |
| Agent listing | GET `/Agents` with filter | Filter not supported; pagination missing; wrong filter syntax |
| AgenticApplication management | CRUD on `/AgenticApplications` | Application not found; agent references invalid |
| Bulk operations | POST `/Bulk` with multiple operations | Partial failure; no rollback; inconsistent state |
| Audit logging | Verify `auditLog` is populated | Missing audit entries; wrong timestamp format |

## Known Issues & Sharp Edges

1. **Draft status**: SCIM for Agents is in very early draft (draft-abbey-scim-agent-extension-00). The resource schema, API surface, and error handling are all subject to change. No production implementations exist yet.

2. **Capability schema complexity**: The `capabilities` field in the Agent resource requires JSON Schema for input/output. This is powerful but verbose. In practice, many implementations will use simplified capability descriptions.

3. **Delegation chain storage**: The `delegationChain` field in the Agent resource stores the full chain. This can grow large for multi-hop delegations. The spec doesn't define a maximum size or a truncation policy.

4. **Auth credentials security**: The `authCredentials` field may contain sensitive information (client IDs, token endpoints). The spec doesn't define encryption or access control for this field. In practice, SCIM servers must restrict access to this field.

5. **Audit log standardization**: The `auditLog` field is a free-form array. Different implementations will log different events with different formats. This makes cross-platform audit analysis difficult.

6. **No search indexing**: The spec doesn't mandate search indexing for large agent fleets. A SCIM server with 10,000+ agents may have poor query performance without indexing.

7. **SCIM 2.0 ecosystem maturity**: SCIM 2.0 is widely deployed for user provisioning but not for agents. Many SCIM 2.0 servers will need significant changes to support Agent resources.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| Capability JSON Schema version | Not specified; adapter uses Draft 7 | Community convention |
| Delegation chain max depth | Not specified; adapter stores up to 10 hops | Arbitrary limit |
| Auth credentials encryption | Not specified; adapter assumes server-side encryption | Security gap in spec |
| Audit log event taxonomy | Not specified; adapter uses flat event names | Community convention |
| Agent status transitions | Spec defines `active`, `inactive`, `suspended`; doesn't define transition rules | Adapter assumes manual transitions |
| Multi-domain agent federation | Not specified; adapter assumes single domain | Unresolved |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 60% | Draft is clear but not implemented; no real implementations to test |
| **Latency** | Untested | No production implementations |
| **Token Economics** | Moderate | SCIM 2.0 JSON is verbose; agent resources are large (~5-10KB) |
| **Scale Behavior** | Untested | No production implementations at scale |
| **Ops Burden** | High | Draft status; no SDKs; requires custom SCIM server extension |
| **Developer Experience** | Poor | No dedicated SDKs; docs are draft-quality; examples are sparse |
| **Data Sovereignty** | Excellent | IETF governance; open draft; no vendor lock-in |

**Composite Score (2026-06):** 58/100 — Important for the future of agent lifecycle management, but far too early for production. The spec needs implementations, SDKs, and real-world testing before it can be evaluated fairly.

## Links

- **IETF Draft**: https://datatracker.ietf.org/doc/draft-abbey-scim-agent-extension/
- **SCIM 2.0 Core**: RFC 7643 (https://tools.ietf.org/html/rfc7643)
- **SCIM 2.0 Protocol**: RFC 7644 (https://tools.ietf.org/html/rfc7644)
- **SCIM 2.0 Enterprise User**: RFC extension
- **Okta SCIM**: https://developer.okta.com/docs/reference/scim/
- **Azure AD SCIM**: https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
