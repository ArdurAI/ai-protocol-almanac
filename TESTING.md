# Testing & Benchmarking

How the almanac tests protocols, what the harness does, how scoring works, and how to reproduce results.

## Table of Contents

1. [The Three Benchmark Types](#the-three-benchmark-types)
2. [The Seven Dimensions](#the-seven-dimensions)
3. [The Harness Architecture](#the-harness-architecture)
4. [The Canary](#the-canary)
5. [Standard Benchmarks](#standard-benchmarks)
6. [Protocol Custom Benchmarks](#protocol-custom-benchmarks)
7. [Stress Suite](#stress-suite)
8. [Cross-Standard Integration Tests](#cross-standard-integration-tests)
9. [Scoring](#scoring)
10. [Reproducibility](#reproducibility)
11. [Failure Mode Taxonomy](#failure-mode-taxonomy)

---

## The Three Benchmark Types

Every protocol is tested across three types of benchmarks:

| Type | Purpose | Frequency |
|------|---------|-----------|
| **Standard benchmarks** | Verify vendor claims with published conformance test suites | Every benchmark run |
| **Protocol custom benchmarks** | Test interop reality: setup, handshake, failure modes, version negotiation | Every benchmark run |
| **Cross-standard integration tests** | Test how protocols work together in a multi-agent stack | Quarterly |

## The Seven Dimensions

Every protocol is scored 0-100 on each dimension. The final score is a weighted average, but the per-dimension scores are always published.

| Dimension | Weight | What it measures | How it's tested |
|-----------|--------|-----------------|-----------------|
| **Accuracy / Compliance** | 25% | Does the implementation follow the spec? Schema validation, message sequencing, error code correctness, required field presence | Conformance test suite + schema validation + message trace analysis |
| **Latency** | 15% | Handshake time, message round-trip time, connection establishment overhead, capability negotiation lag | Instrumented measurements; p50, p95, p99 across handshake + message exchange cycles |
| **Token Economics** | 15% | Protocol efficiency: bytes-on-wire per message, metadata overhead, compression effectiveness, unnecessary round-trips | Standardized workloads; bytes/request, overhead ratio, retransmission cost |
| **Scale Behavior** | 15% | What happens at 10, 100, 1000 concurrent agents? Connection state growth, broadcast amplification, handshake storm | Federation load tests; saturation curves; degradation points |
| **Ops Burden** | 15% | Time to first successful handshake, dependency conflicts, version negotiation complexity, upgrade pain | Measured setup time; smoke-gate sweep; dependency matrix across SDK versions |
| **Developer Experience** | 10% | Spec clarity, SDK quality, error message usefulness, documentation completeness, debugging tooling | Structured rubric; community health metrics; time-to-first-message measurement |
| **Data Sovereignty** | 5% | Open-spec availability, self-hosting viability, vendor-neutral governance, audit trails | Feature matrix; governance model assessment; EU AI Act / GDPR compliance mapping |

### Why these weights?

The weights reflect what an integration engineer actually cares about. Compliance is the most important — a protocol that doesn't follow its own spec is worse than no protocol at all. But ops burden is nearly as important because a protocol that consumes your team's life with version negotiation, schema drift, and transport debugging is not worth the compliance gain.

Weights are reviewed annually. Changes require an RFC and a public comment period.

## The Harness Architecture

```
┌─────────────────────────────────────────┐
│  ProtocolAdapter (frozen contract)      │
│  ├── setup()   → initialize connection  │
│  ├── negotiate() → capability handshake │
│  ├── load()    → ingest schemas/messages│
│  ├── await_ready() → handshake barrier  │
│  ├── query()   → run test exchange      │
│  ├── validate() → schema/spec compliance│
│  └── teardown() → cleanup, measure      │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Telemetry Collector                    │
│  ├── handshake latency (p50/p95/p99)   │
│  ├── message round-trip time            │
│  ├── bytes-on-wire & overhead           │
│  ├── connection state & memory          │
│  ├── error rate & failure mode taxonomy │
│  └── ops notes (setup time, deps, bugs) │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Grading Pipeline                       │
│  ├── Schema validator (JSON Schema, etc.)│
│  ├── Protocol judge (frozen prompts)     │
│  ├── Second pass (confidence < 0.7)      │
│  └── Failure mode taxonomy               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Results Publisher                        │
│  ├── Raw JSON (per message, per run)    │
│  ├── Summary tables (per protocol)       │
│  ├── Cross-verification analysis         │
│  └── Insight extraction                  │
└─────────────────────────────────────────┘
```

### The `await_ready()` barrier

This is where async-handshake designs get their cost measured instead of hidden. Many protocols (MCP with stdio initialization, A2A with capability discovery, WebSocket with subprotocol negotiation) claim "fast" connection paths because the actual handshake work happens in the background. The `await_ready()` barrier forces the protocol to finish all background negotiation before the message exchange is run, so the true handshake latency is measured.

### The Telemetry Collector

Every adapter call is instrumented:
- **Handshake latency**: `time.monotonic()` around `negotiate()` and `await_ready()`; p50, p95, p99 computed across all runs
- **Message round-trip**: `time.monotonic()` around `query()` for request/response pairs
- **Bytes-on-wire**: Message size in bytes, metadata overhead ratio, compression ratio
- **Connection state**: Memory usage for connection state, number of open connections, session tracking overhead
- **Errors**: Every exception, timeout, schema validation failure, or non-compliant response is logged with full message trace
- **Ops notes**: Human observations about setup friction, dependency conflicts, spec ambiguity, documentation quality

## The Canary

The first run of every batch is the **no-protocol baseline** through the identical pipeline. If the benchmark leaked answers anywhere, the canary would score above zero on answerable categories.

**Canary rules**:
- The canary must score exactly **0.000** on all answerable categories
- The canary must score exactly **1.000** on abstention categories (it should abstain on everything)
- If the canary fails, the entire batch is invalid and must be rerun
- The canary run is published alongside the real results
- The canary adapter is a no-op: setup does nothing, negotiate does nothing, load does nothing, query returns nothing, validate returns null, teardown does nothing

## Standard Benchmarks

### By standard

| Standard | Benchmark | What it tests | Source |
|----------|-----------|-------------|--------|
| MCP | MCP compliance test | Schema conformance for `tools/list`, `resources/read`, `prompts/get`, `logging/setLevel` | Custom harness |
| MCP | MCP server stress test | Server behavior under malformed requests, missing fields, invalid JSON-RPC | Custom harness |
| A2A | A2A interoperability test | Cross-SDK message exchange: task creation, streaming, artifact delivery | Custom harness |
| A2A | A2A federation test | Multi-agent task routing across heterogeneous A2A implementations | Custom harness |
| Authentication | OAuth 2.0 / OIDC compliance | Token exchange, scope validation, refresh behavior | OAuth 2.0 conformance suite |
| SCIM for Agents | SCIM compliance | User/agent provisioning, schema mapping, group management | SCIM conformance test suite |
| General | JSON Schema validation | Schema correctness, validation speed, error message quality | Custom harness |
| General | Transport abstraction test | Same protocol over WebSocket, SSE, HTTP streaming: behavior parity | Custom harness |

### Published vs. reproduced

Every standard benchmark ranking ships a table:

| Protocol | Published Claim | Our Result | Delta | Verdict |
|----------|----------------|------------|-------|---------|
| Protocol A | "100% MCP compliant" | 94% on compliance suite | -6% | ⚠️ Close |
| Protocol B | "Sub-millisecond handshake" | 12ms median | - | ⚠️ Misleading |
| Protocol C | No claim | 87% on A2A interop | N/A | — |

## Protocol Custom Benchmarks

### Setup experience

**Measured**:
- Time from reading the spec to first successful handshake
- Number of dependency conflicts when installing alongside other roster SDKs
- Time to resolve dependency conflicts
- Number of undocumented steps required (e.g., "you must set this env var that isn't in the docs")
- Time to find the answer in the docs when stuck

**Scored on**:
- Sub-5 minutes: 90-100
- 5-30 minutes: 70-89
- 30-60 minutes: 50-69
- 60+ minutes or unresolved: 0-49

### Smoke gate

Every protocol must pass an identical 3-turn scenario before entering the roster:

```
Turn 1: Initialize the protocol connection (handshake, capability negotiation, auth)
Turn 2: Exchange a test message (send request, receive response, validate schema)
Turn 3: Verify protocol compliance (check required fields, error handling, version agreement)
```

**Pass criteria**:
- No crashes, no silent failures, no connection drops
- Handshake must complete with version agreement
- Message schema must validate against the published spec
- Tool must handle the basic case without workarounds

**What the smoke gate surfaced** (from protocol testing, as examples):
- **Version negotiation bugs**: Server claims support for protocol version X but fails on version X features
- **Schema drift**: SDK sends fields not in the spec, or omits required fields the spec mandates
- **Transport fragility**: Protocol works over HTTP but fails over WebSocket with identical payloads
- **Auth negotiation failures**: OAuth handshake succeeds but subsequent messages are rejected with cryptic errors
- **Handshake spread**: Sub-millisecond to ~5 seconds for 3-turn initialization across implementations
- **Non-compliant extensions**: Vendor-specific fields that break other implementations' parsers

### Stress suite

| Test | What it does | What it reveals |
|------|-------------|---------------|
| **Version storms** | Rapidly negotiate different protocol versions | How the protocol handles version mismatch, downgrade, and upgrade |
| **Schema mismatch floods** | Send messages with extra fields, missing fields, wrong types | Parser robustness, forward/backward compatibility, error specificity |
| **Concurrent handshakes** | Multiple agents connecting simultaneously | Connection pool exhaustion, race conditions, state contamination |
| **Malformed message storms** | Send invalid JSON, wrong message types, oversized payloads | Error handling quality, DoS resistance, recovery behavior |
| **Kill-the-transport** | Crash the transport layer mid-handshake | Recovery, reconnection behavior, state integrity |
| **Federation broadcast** | Send a message to 100+ agents simultaneously | Broadcast amplification, latency tail, connection state memory growth |
| **Cost-runaway** | Run the protocol at maximum message rate for 1 hour | Bandwidth cost predictability, retransmission rate, resource leak detection |

### Upgrade path

**Tested**:
- Can you upgrade from protocol version N to N+1 without breaking existing connections?
- Are there breaking changes in the message schema or handshake sequence?
- Is there a migration guide?
- Does the protocol maintain backward compatibility within a major version?
- Do old clients still work with new servers?

### Debugging experience

**Tested**:
- When the protocol fails, can you find out why in <5 minutes?
- Are error messages clear, actionable, and spec-compliant?
- Is there a message trace or debug mode?
- Are there known interoperability issues documented?
- Can you trace the message flow end-to-end?

## Cross-Standard Integration Tests

These tests run quarterly and check how protocols work together in a realistic multi-agent stack:

| Integration | What it tests | Protocols involved |
|-------------|-------------|-------------------|
| **MCP + A2A** | Can an MCP server expose tools through an A2A agent? | MCP implementation, A2A implementation |
| **Auth + All protocols** | Does adding OAuth add <100ms latency to handshakes? | Auth standard (OAuth/OIDC), protocol implementation |
| **SCIM + MCP** | Can SCIM-provisioned agents use MCP tools correctly? | SCIM for Agents, MCP |
| **Transport + Protocols** | Do protocols behave identically over WebSocket, SSE, and HTTP? | Transport abstraction, protocol implementation |
| **x402 + A2A** | Can A2A agents negotiate payments via x402? | x402, A2A |

## Scoring

### Per-dimension scoring

Each dimension is scored 0-100 using a rubric. The rubric is published before any scoring happens.

**Example: Accuracy/Compliance rubric**

| Score | Criteria |
|-------|----------|
| 90-100 | ≥95% compliance on standard conformance suite; no critical failures in stress suite; passes all cross-standard integration tests |
| 80-89 | 85-95% compliance; minor failures in stress suite; passes core integration tests |
| 70-79 | 75-85% compliance; some stress suite failures; partial integration test success |
| 60-69 | 65-75% compliance; frequent stress suite failures; integration tests require workarounds |
| 50-59 | 55-65% compliance; significant reliability issues; integration tests fail |
| 0-49 | <55% compliance or fundamentally unreliable |

### Composite score

The composite score is a weighted average of the seven dimensions:

```
Composite = (Accuracy × 0.25) + (Latency × 0.15) + (TokenEconomics × 0.15) +
            (ScaleBehavior × 0.15) + (OpsBurden × 0.15) + (DevEx × 0.10) +
            (DataSovereignty × 0.05)
```

The composite is used for ranking, but the per-dimension scores are always published. A protocol with a high composite but low ops burden score is a warning sign — it may be technically excellent but impractical to adopt.

### Confidence intervals

Every score is reported with a confidence interval computed from the standard error across runs. If the intervals overlap between two protocols, the difference is not statistically significant.

## Reproducibility

### How to reproduce a benchmark

1. Clone the benchmark harness repo (published separately)
2. Check out the exact commit used for the run (recorded in the results JSON)
3. Install the exact dependencies (lockfile is published)
4. Run the harness with the same adapter and same seed
5. Compare your results to the published results

### What is frozen

| Element | How it's frozen | Where to find it |
|---------|---------------|------------------|
| Judge model | Pinned model name and version | `results.json` metadata |
| Judge prompts | SHA-256 hash | `methodology/benchmark-harness.md` |
| Control variables | Documented values | `results.json` metadata |
| Random seeds | Published integer | `results.json` metadata |
| Adapter code | Published in harness repo | Separate repo |
| Test workloads | Published JSON files | `benchmarks/` directory |
| Schema validators | Pinned validator version | `results.json` metadata |

### What is NOT frozen (and why)

| Element | Why it changes | How we handle it |
|---------|---------------|------------------|
| Protocol versions | Protocols update | We re-run benchmarks for new versions; old results are archived |
| SDK versions | SDKs update | Re-run for new SDK versions; note breaking changes |
| Provider pricing | Cloud pricing changes | Cost is computed at runtime using current pricing; historical results are annotated |
| Hardware | We may upgrade machines | Hardware spec is recorded in `results.json`; results are hardware-specific |

## Failure Mode Taxonomy

Every failure is classified into a taxonomy. This helps identify patterns across protocols and standards.

| Category | Failure Modes |
|----------|--------------|
| **Setup** | `install_failed`, `dependency_conflict`, `config_error`, `missing_env_var`, `docs_incomplete`, `spec_ambiguous` |
| **Handshake** | `version_mismatch`, `capability_negotiation_failed`, `auth_failure`, `transport_rejected`, `timeout` |
| **Message Exchange** | `schema_validation_failed`, `missing_required_field`, `wrong_message_type`, `encoding_error`, `size_limit_exceeded` |
| **Compliance** | `spec_noncompliant`, `undocumented_extension`, `optional_treated_as_required`, `error_code_wrong` |
| **Scale** | `connection_pool_exhaustion`, `handshake_storm`, `broadcast_amplification`, `memory_leak`, `latency_tail` |
| **Ops** | `upgrade_breaking`, `backward_compat_failure`, `debug_opacity`, `community_unresponsive`, `deprecation_unannounced` |
| **Integration** | `mcp_noncompliant`, `a2a_noncompliant`, `auth_scope_mismatch`, `protocol_mismatch`, `transport_incompatible` |
| **Security** | `auth_token_leak`, `man_in_the_middle`, `replay_attack`, `scope_escalation`, `credential_exposure` |

## License

Content: CC BY 4.0  
Code: MIT
