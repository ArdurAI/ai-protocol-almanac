# HashiCorp Vault — Deep Reference

**HashiCorp Vault** is the enterprise standard for secrets management, providing secure storage, dynamic secrets, secret rotation, and encryption-as-a-service. It is widely used in AI infrastructure for managing API keys, credentials, and sensitive configuration.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest version** | Vault 1.18+ (2025-2026) |
| **Governance** | HashiCorp (IBM, since 2024 acquisition) |
| **License** | Open Source (BSL 1.1) + Enterprise (commercial) |
| **Deployment** | Self-hosted (on-prem, cloud, Kubernetes) / HCP Vault (managed) |
| **Primary use case** | Secrets management; dynamic secrets; encryption; PKI; identity |
| **Status** | Production; enterprise standard; widely deployed |
| **Related standards** | WIMSE, SPIFFE/SPIRE, OAuth 2.0, Kubernetes, AWS IAM |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| Vault 0.1 | 2015 | Initial release; basic secrets storage |
| Vault 1.0 | 2018 | Production-ready; dynamic secrets; PKI; AWS/GCP/Azure integration |
| Vault 1.10 | 2022 | Kubernetes integration; OIDC auth; Transform secrets engine |
| Vault 1.15 | 2024 | AI/ML secrets support; enhanced Kubernetes integration; WIMSE alignment |
| Vault 1.18 | 2025 | AI agent secrets management; dynamic API key generation; MCP auth integration |
| Vault 1.19 (expected) | Q3 2026 | Planned: AIMS integration, agent identity management, enhanced rotation |
| IBM acquisition | 2024 | HashiCorp acquired by IBM; Vault continues as open source + enterprise |

## Architecture & API Surface

### Core Secrets Engines

| Secrets Engine | Description | AI Use Case |
|---------------|-------------|-------------|
| **KV (Key-Value)** | Static secret storage | API keys, configuration, certificates |
| **Dynamic Secrets** | On-demand credential generation | Database credentials, cloud IAM, API keys |
| **PKI** | Certificate management | mTLS, WIMSE X.509 SVIDs, service mesh |
| **Transit** | Encryption-as-a-service | Encrypt sensitive data, PII, model outputs |
| **Identity** | Identity management | OIDC, JWT, SAML authentication |
| **AWS/GCP/Azure** | Cloud IAM integration | Dynamic cloud credentials for AI workloads |
| **Kubernetes** | K8s service account integration | Workload identity for AI agents in K8s |
| **Transform** | Data transformation (tokenization, encryption) | PII protection, data masking |
| **SSH** | SSH certificate management | Secure access to AI training infrastructure |

### AI-Specific Features

| Feature | Description | Vault Version |
|---------|-------------|---------------|
| **API Key Management** | Store and rotate AI provider API keys (OpenAI, Anthropic, etc.) | 1.15+ |
| **Dynamic API Keys** | Generate temporary API keys with scoped permissions | 1.18+ |
| **MCP Auth Integration** | OAuth 2.1 token issuance for MCP servers | 1.18+ |
| **Agent Identity** | X.509 certificate issuance for agent identity (WIMSE) | 1.15+ |
| **Secret Rotation** | Automatic rotation of API keys and credentials | 1.10+ |
| **Encryption of Model Outputs** | Encrypt sensitive model outputs at rest | 1.10+ |
| **Audit Logging** | Comprehensive audit of all secret access | 1.0+ |

### Dynamic Secrets for AI

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  AI Agent   │────▶│   Vault     │────▶│  AI Provider│
│             │     │             │     │  (OpenAI)   │
│             │◀────│             │◀────│             │
│             │     │  1. Request │     │  2. Generate│
│             │     │     dynamic │     │     temp key│
│             │     │     API key │     │  3. Return  │
│             │     │  4. Lease   │     │     key     │
│             │     │     expires │     │             │
│             │     │  5. Key     │     │             │
│             │     │     revoked │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

### Vault API for AI Secrets

```bash
# Store an AI API key
vault kv put secret/ai/openai api_key="sk-..."

# Retrieve an API key
vault kv get secret/ai/openai

# Generate a dynamic AWS credential for an AI workload
vault write aws/creds/ai-workload ttl=1h

# Issue an X.509 certificate for an agent
vault write pki/issue/agent-common-name \
  common_name="agent-001.ai.example.com" \
  ttl=24h

# Encrypt sensitive model output
vault write transit/encrypt/ai-data \
  plaintext=$(base64 <<< "sensitive model output")
```

## Known Implementations & Integrations

| Integration | Description |
|-------------|-------------|
| **Kubernetes** | Vault Agent Sidecar; CSI driver; external secrets operator |
| **Terraform** | Vault provider for infrastructure secrets management |
| **Consul** | Service mesh integration; mTLS with Vault-issued certificates |
| **Nomad** | Workload identity; dynamic secrets for batch jobs |
| **AWS** | AWS IAM dynamic credentials; AWS Secrets Manager sync |
| **GCP** | GCP service account dynamic credentials |
| **Azure** | Azure service principal dynamic credentials |
| **GitHub Actions** | Vault Action for CI/CD secrets |
| **Jenkins** | Vault plugin for CI/CD secrets |
| **Spinnaker** | Vault integration for deployment secrets |
| **MCP** | Community integration; MCP server for Vault secrets access |
| **A2A** | Community integration; agent identity via Vault PKI |
| **WIMSE** | Vault can issue WIMSE-compatible X.509 SVIDs |

## Interoperability Notes

### Vault + MCP
- Vault can store and manage MCP server credentials
- Dynamic API keys can be generated for MCP servers with scoped permissions
- The MCP OAuth 2.1 mandate can be fulfilled using Vault's OIDC/identity engine
- Community MCP servers exist for accessing Vault secrets from AI agents

### Vault + A2A
- Vault's PKI engine can issue X.509 certificates for A2A agent identity
- Agent Cards can reference Vault-issued certificates for authentication
- The delegation chain in A2A can be anchored to Vault-issued identities

### Vault + WIMSE
- Vault is the most mature solution for WIMSE workload identity deployment
- The PKI engine can issue SPIFFE-compatible X.509 SVIDs
- The Kubernetes integration can automatically issue identities to pods
- Vault can integrate with SPIRE (SPIFFE Runtime Environment) for attestation

### Vault + AIMS
- Vault's OIDC/identity engine can support the `agent_assertion` grant type
- Agent identities can be stored and managed in Vault's identity engine
- The delegation chain in AIMS can be verified against Vault's audit logs
- AIMS integration is on the roadmap for Vault 1.19

### Vault + OAuth 2.1
- Vault can act as an OAuth 2.0/OIDC authorization server
- It can issue tokens for AI APIs with scoped permissions
- Token rotation and revocation are managed automatically
- The OAuth 2.1 PKCE flow is supported

### Vault + Kubernetes
- The Kubernetes auth method allows pods to authenticate to Vault using their service account
- The Vault Agent Sidecar automatically injects secrets into pod environments
- The CSI driver mounts secrets as files in the pod filesystem
- This is the primary deployment pattern for AI workloads in Kubernetes

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Secret storage | Store and retrieve a secret | Storage backend failure; ACL misconfiguration; encryption error |
| Dynamic secrets | Generate temporary credentials | Provider API failure; TTL misconfiguration; lease not renewed |
| Secret rotation | Rotate a secret automatically | Rotation script failure; downstream system not updated; old secret still valid |
| PKI certificate issuance | Issue an X.509 certificate | CA not configured; role misconfiguration; TTL too long |
| Encryption | Encrypt and decrypt data | Key not found; algorithm mismatch; corrupted ciphertext |
| Audit logging | Verify all access is logged | Audit device not enabled; log loss; incorrect log format |
| Authentication | Authenticate with multiple auth methods | Auth method not enabled; token expired; MFA failure |
| Authorization | Verify ACL policies | Policy misconfiguration; path mismatch; permission denied |
| High availability | Failover to standby node | Raft quorum lost; network partition; data replication lag |

## Known Issues & Sharp Edges

1. **Operational complexity**: Vault is powerful but complex. Setting up a production Vault cluster requires expertise in storage backends (Raft, Consul, etc.), HA configuration, seal/unseal procedures, and disaster recovery. This is not a plug-and-play solution.

2. **BSL license concern**: Vault switched from MPL to BSL (Business Source License) in 2023. This means the open source version has usage restrictions for competitive managed services. The IBM acquisition has raised questions about the future of the open source edition.

3. **Kubernetes integration complexity**: While Vault integrates well with Kubernetes, the setup (CSI driver, sidecar injection, service account mapping) is complex. Misconfigurations can lead to pods failing to start or secrets not being injected.

4. **Dynamic secrets provider lag**: Dynamic secrets require Vault to call the provider's API (e.g., AWS IAM, GCP IAM). If the provider's API is slow or rate-limited, secret generation can be delayed, causing application failures.

5. **Secret rotation disruption**: Automatic secret rotation can disrupt running applications if they don't handle credential refresh gracefully. This requires applications to implement lease renewal and credential refresh logic.

6. **PKI CA management**: Managing a PKI CA (root CA, intermediate CAs, CRL, OCSP) is complex. Vault automates much of this but still requires careful planning for certificate lifetimes, revocation, and trust distribution.

7. **Audit log volume**: In high-throughput environments, Vault generates massive audit logs. Storage, retention, and analysis of these logs can be challenging and expensive.

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 96% | Core features are rock-solid; PKI, encryption, dynamic secrets all work reliably |
| **Latency** | 5-50ms (secret retrieval) / 100-500ms (dynamic secrets) | KV is fast; dynamic secrets depend on provider API |
| **Token Economics** | Minimal overhead | No token overhead; cost is infrastructure-dependent |
| **Scale Behavior** | Excellent | HA mode; Raft consensus; horizontally scalable read replicas |
| **Ops Burden** | High | Complex setup; requires expertise; ongoing maintenance |
| **Developer Experience** | Good | Good API; excellent documentation; strong community; but complex |
| **Data Sovereignty** | Excellent | Self-hosted; open source core; no cloud dependency |

**Composite Score (2026-06):** 89/100 — The gold standard for secrets management in AI infrastructure. Unmatched capabilities, but the operational complexity is significant.

## Links

- **Documentation**: https://developer.hashicorp.com/vault/docs
- **GitHub**: https://github.com/hashicorp/vault
- **HCP Vault (Managed)**: https://cloud.hashicorp.com/products/vault
- **Vault Learn**: https://developer.hashicorp.com/vault/tutorials
- **SPIFFE/SPIRE**: https://spiffe.io

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
