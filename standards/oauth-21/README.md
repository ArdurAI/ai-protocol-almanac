# OAuth 2.1 + PKCE — Deep Reference

**OAuth 2.1 + PKCE** (Proof Key for Code Exchange) is the dominant authentication and authorization pattern for AI agents, remote services, and web applications. It is the mandated authentication mechanism for the MCP protocol and is the foundation upon which AIMS and other agent-specific auth standards are built.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest spec** | OAuth 2.1 (RFC 6819 + updates, 2025) |
| **Governance** | IETF (Internet Engineering Task Force) |
| **License** | IETF Standard (RFC) |
| **Transport** | HTTP (redirects, token endpoint) |
| **Serialization** | JSON, URL-encoded form data |
| **Primary use case** | User authorization for agent actions; remote server authentication; API access delegation |
| **Status** | Mature standard; widely deployed; mandated by MCP (Nov 2025 spec) |
| **Related standards** | OAuth 2.0 (RFC 6749), OIDC (OpenID Connect), AIMS (agent extension), PKCE (RFC 7636) |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| OAuth 1.0 | 2007 | Early token-based authorization; now deprecated |
| OAuth 2.0 (RFC 6749) | 2012 | Current standard; authorization code, implicit, client credentials, password grants |
| PKCE (RFC 7636) | 2015 | Extension for OAuth 2.0; mitigates authorization code interception attacks |
| OAuth 2.0 Security Best Practices (RFC 6819) | 2013 | Security guidance for OAuth 2.0 implementations |
| OAuth 2.1 (draft) | 2020-2025 | Consolidated spec: OAuth 2.0 + PKCE + security best practices + deprecates implicit/password |
| OAuth 2.1 (RFC, expected) | 2025-2026 | Final RFC; PKCE is mandatory for public clients; implicit/password grants deprecated |
| MCP OAuth 2.1 mandate | Nov 2025 | MCP specification mandates OAuth 2.1 for all remote (non-stdio) MCP servers |

## Transport & Schema Details

### Core Protocol

OAuth 2.1 + PKCE is a authorization framework that allows users to grant agents limited access to their resources without sharing passwords.

### The Authorization Code + PKCE Flow

```
┌─────────┐                                    ┌─────────┐
│  User   │                                    │  Agent  │
│ (human) │                                    │  (AI)   │
└────┬────┘                                    └────┬────┘
     │                                              │
     │ 1. Agent requests authorization              │
     │──────────────────────────────────────────────▶│
     │                                              │
     │ 2. Agent generates PKCE code_verifier        │
     │    + code_challenge (SHA256 hash)             │
     │                                              │
     │ 3. Agent redirects user to Auth Server        │
     │    /authorize?client_id=...&code_challenge=...│
     │◀─────────────────────────────────────────────│
     │                                              │
     │ 4. User authenticates and consents            │
     │    ("Allow agent to access my orders?")       │
     │                                              │
     │ 5. Auth Server redirects back with code       │
     │──────────────────────────────────────────────▶│
     │                                              │
     │ 6. Agent exchanges code for token             │
     │    POST /token with code_verifier            │
     │◀─────────────────────────────────────────────│
     │                                              │
     │ 7. Agent receives access_token + refresh_token│
     │──────────────────────────────────────────────▶│
     │                                              │
     │ 8. Agent uses access_token to call API        │
     │──────────────────────────────────────────────▶│
     │                                              │
     │ 9. Agent refreshes token when expired         │
     │◀─────────────────────────────────────────────│
```

### PKCE Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `code_verifier` | High-entropy random string (43-128 chars) | `dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk` |
| `code_challenge` | Base64URL(SHA256(code_verifier)) | `E9Melhoa2OwvFrEMT...` |
| `code_challenge_method` | Always `S256` in OAuth 2.1 | `S256` |

### Token Response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
  "scope": "read_orders write_reviews"
}
```

### Token Usage

```
GET /api/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

### MCP-Specific OAuth 2.1 Requirements

The MCP specification (Nov 2025) adds MCP-specific requirements to OAuth 2.1:

| Requirement | Description |
|-------------|-------------|
| **Remote server auth** | All MCP servers exposed over HTTP must use OAuth 2.1 + PKCE |
| **Scope format** | MCP scopes are `mcp:<capability>` (e.g., `mcp:tools/list`, `mcp:resources/read`) |
| **Token introspection** | MCP servers must support token introspection for verification |
| **Consent UI** | MCP servers must provide a consent UI that explains what tools/resources the agent will access |
| **Refresh token rotation** | Refresh tokens must be rotated on every use (security best practice) |

## Known Implementations & SDKs

| SDK/Platform | Language | Maintainer | Notes |
|-------------|----------|------------|-------|
| OAuth 2.1 / OIDC certified servers | Various | Various | Auth0, Okta, Keycloak, Authentik, AWS Cognito, Azure AD, Google Identity |
| `oauthlib` (Python) | Python | Community | OAuth 2.0/2.1 client and server library |
| `node-oauth` | JavaScript | Community | OAuth 2.1 client for Node.js |
| `spring-security-oauth` | Java | Spring | OAuth 2.1 support in Spring Security |
| Stytch | Various | Stytch | Auth platform with OAuth 2.1 + MCP auth server support |
| Scalekit | Various | Scalekit | B2B agent auth with OAuth 2.1 + OBO tokens |
| Aembit | Various | Aembit | MCP authorization with OAuth 2.1, PRMs, workload identity |
| Authed | Various | Authed | Agent-to-agent auth; dynamic tokens; no static credentials |
| HashiCorp Vault | Various | HashiCorp | Secret management + OAuth 2.0 token issuance |

## Interoperability Notes

### OAuth 2.1 + MCP
- The MCP spec mandates OAuth 2.1 for remote servers, but Knostic's 2025 scan found **100% of ~2,000 MCP servers lacked authentication**
- This is a critical gap: the spec exists but the ecosystem is not implementing it
- The MCP community is working on reference implementations and tooling to close this gap

### OAuth 2.1 + AIMS
- AIMS extends OAuth 2.1 with the `agent_assertion` grant type
- This allows agents to authenticate without user interaction (autonomous agent authentication)
- The two are backward compatible: existing OAuth 2.1 servers can add `agent_assertion` support

### OAuth 2.1 + WIMSE
- WIMSE provides workload identity (mTLS, X.509 SVIDs)
- OAuth 2.1 provides application authorization (scopes, consent)
- Together: WIMSE authenticates the workload → OAuth 2.1 authorizes the agent's actions
- This is the recommended stack for high-security agent deployments

### OAuth 2.1 + A2A
- A2A agents can use OAuth 2.1 to authenticate to other agents' endpoints
- The Agent Card's `authRequirements` field can specify OAuth 2.1 as the required auth method
- The A2A spec does not mandate a specific auth method, leaving it to the Agent Card

### OAuth 2.1 + SCIM for Agents
- SCIM for Agents can provision OAuth 2.1 client credentials as part of the `authCredentials` field
- This allows automated agent provisioning with pre-configured OAuth 2.1 access

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| PKCE enforcement | Client uses `code_challenge` + `code_verifier` | Server doesn't enforce PKCE; client skips PKCE |
| Authorization code exchange | POST `/token` with valid code + verifier | Wrong verifier; expired code; code reuse |
| Token refresh | POST `/token` with `refresh_token` | Refresh token expired; rotation not enforced; scope changed |
| Scope validation | Token includes correct scopes | Overly broad scopes; scope escalation; missing required scopes |
| Token introspection | Server supports `token_info` or introspection endpoint | Introspection not implemented; response format wrong |
| Consent UI | Consent screen explains requested capabilities | Generic consent; no capability breakdown; auto-approve |
| Refresh token rotation | New refresh token issued on every refresh | Same refresh token reused; rotation not enforced |
| Token expiry | Token expires after `expires_in` | Token doesn't expire; expiry too long; no expiry |

## Known Issues & Sharp Edges

1. **MCP auth gap**: The MCP spec mandates OAuth 2.1, but ~100% of scanned MCP servers lack authentication. This is the biggest security issue in the MCP ecosystem. The community is working on it, but adoption is slow.

2. **Implicit grant deprecation**: OAuth 2.1 deprecates the implicit grant. Many existing implementations still use it. Migrating to authorization code + PKCE requires client-side changes.

3. **Password grant deprecation**: OAuth 2.1 deprecates the password grant. This breaks some legacy integrations that relied on direct username/password auth.

4. **Scope granularity**: There is no standard scope taxonomy for AI agents. "`read_orders`" means different things on different platforms. This creates interoperability issues.

5. **Consent fatigue**: Users may be asked to consent frequently if agents request new scopes often. This leads to consent fatigue and users clicking "approve" without reading.

6. **Token leakage**: Bearer tokens are vulnerable to leakage (logs, headers, browser storage). If a token is leaked, an attacker can use it until expiry. Short expiry + rotation helps but doesn't eliminate the risk.

7. **Refresh token storage**: Refresh tokens are long-lived and powerful. Storing them securely in agent environments is challenging. Compromised refresh tokens allow indefinite access.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| MCP scope format | Spec says `mcp:<capability>`; implementations vary | Adapter uses `mcp:tools/list`, `mcp:resources/read` |
| Consent screen content | Not specified; adapter assumes capability list + description | Platform-specific |
| Token introspection response | OAuth 2.0 introspection RFC defines format; MCP doesn't extend it | Adapter uses standard introspection response |
| Client authentication for public clients | OAuth 2.1 says no client_secret for public clients; some servers require it | Adapter follows OAuth 2.1 spec |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 95% | OAuth 2.1 is mature and well-tested; PKCE is widely implemented |
| **Latency** | 200-500ms (full flow) / 10-50ms (token refresh) | Authorization flow is slow; refresh is fast |
| **Token Economics** | Minimal overhead | JWT tokens are small; HTTP overhead is negligible |
| **Scale Behavior** | Excellent | Stateless token verification; auth servers scale horizontally |
| **Ops Burden** | Medium | OAuth 2.1 servers are mature; MCP auth gap is the main pain point |
| **Developer Experience** | Excellent | Extensive documentation, SDKs, tutorials, community support |
| **Data Sovereignty** | Excellent | IETF standard; open spec; no vendor lock-in |

**Composite Score (2026-06):** 91/100 — The gold standard for agent authentication. The only issue is the MCP ecosystem's failure to implement it.

## Links

- **OAuth 2.1 Draft**: https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/
- **OAuth 2.0 (RFC 6749)**: https://tools.ietf.org/html/rfc6749
- **PKCE (RFC 7636)**: https://tools.ietf.org/html/rfc7636
- **OAuth 2.0 Security Best Practices (RFC 6819)**: https://tools.ietf.org/html/rfc6819
- **MCP Auth Spec**: https://modelcontextprotocol.io/specification/authorization
- **Auth0**: https://auth0.com
- **Okta**: https://okta.com
- **Keycloak**: https://keycloak.org

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
