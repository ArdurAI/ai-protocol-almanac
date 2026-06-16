# Project Intent & Philosophy

## Why this almanac exists

The AI protocol landscape is fragmented, fast-moving, and poorly understood. Every month, a new protocol claims to be the "universal standard" for agent interoperability, a new SDK promises seamless integration, and a blog post declares that protocol X is now the industry standard. But **nobody independently verifies these claims**. The compliance numbers are self-reported, the interoperability demos are scripted, and the "protocol comparison" posts are vendor marketing.

This almanac is the **public record of independent protocol verification**. It exists because integration engineers and platform architects need a single source of truth that answers:

- Does this protocol actually work across heterogeneous agent implementations?
- What's the real cost of protocol adoption (handshake overhead, schema complexity, transport negotiation)?
- How does it fail when agents speak different protocol versions or non-standard extensions?
- What's the total cost of interoperability at scale across a federation of agents?
- Can I trust the vendor's compliance numbers?

## Core principles

### 1. Frozen methodology before results

The harness, judge model, prompts, conformance test suite, and scoring rubric are **fixed and published before any protocol is tested**. This prevents "cherry-picking" the methodology that favors a particular protocol or vendor. If a protocol doesn't fit the harness, we adapt the adapter — not the rules.

### 2. Interop-first evaluation

Most protocol benchmarks measure conformance to a single specification. We measure **what an integration engineer actually lives with**:
- Time from reading the spec to the first successful cross-vendor handshake
- Schema negotiation failures when two agents support different protocol versions
- Transport layer incompatibilities (WebSocket vs. SSE vs. HTTP streaming)
- Upgrade pain when protocol version N → N+1 breaks existing connections
- Cost of maintaining compliance across a multi-agent fleet

### 3. Raw data is always published

Every benchmark run produces a JSON file with every message exchanged, every handshake attempted, every schema validation result, every latency measurement. These raw files are published alongside the summary. If you disagree with a ranking, you can re-analyze the data yourself.

### 4. No protocol is above criticism

Every protocol on the roster has been through a smoke gate. Every protocol has non-compliant edge cases, undocumented extensions, or version negotiation bugs. We document them honestly. A vendor relationship or standards-body membership does not influence rankings. The only way a protocol improves its score is by actually improving.

### 5. Living document, not a static snapshot

The almanac is updated monthly. Protocols enter and exit the roster. Scores change as protocols improve or degrade. The "founding edition" is a snapshot; the current edition is the truth.

## Design philosophy

### The two-bar test

Every protocol must clear two bars to justify its existence:
1. **Beat the naive baseline** on interoperability and compliance (e.g., does it work better than ad-hoc REST/JSON between agents?)
2. **Beat the full-capability baseline** on adoption overhead, complexity, and ecosystem lock-in (e.g., is it worth adopting over a simpler custom protocol?)

If a protocol can't do both, it has no reason to exist as a standard. A protocol that is 5% more compliant but requires 10x more SDK complexity than a simple HTTP/JSON approach is not worth adopting.

### The seven dimensions

We score every protocol on seven dimensions because no single number captures "good infrastructure":

| Dimension | Protocol-Specific Meaning | Why it matters |
|-----------|--------------------------|---------------|
| **Accuracy** | Protocol compliance percentage: how strictly the implementation follows the published specification (schema validation, required fields, message sequencing, error code correctness) | A non-compliant protocol implementation is worse than no protocol at all — it creates silent failures and data corruption |
| **Latency** | Handshake and message round-trip time: connection establishment, capability negotiation, schema exchange, and per-message overhead | Protocol overhead must be negligible compared to the actual AI work being done |
| **Token Economics** | Protocol efficiency: bytes-on-wire per semantic unit, compression ratio, unnecessary metadata overhead, retransmission cost | Inefficient protocols multiply bandwidth and compute costs at scale |
| **Scale Behavior** | Multi-agent federation performance: how handshake overhead, connection state, and broadcast costs grow with agent count | A protocol that works for 2 agents but collapses at 100 is not production-ready |
| **Ops Burden** | Time to integrate, version negotiation complexity, breaking-change frequency, monitoring difficulty | Protocols are infrastructure — if they consume your team's life, they fail |
| **Developer Experience** | SDK quality, spec clarity, error message usefulness, documentation completeness, debugging tooling | A protocol nobody can implement correctly is a protocol nobody uses |
| **Data Sovereignty** | Self-hosting viability, protocol transparency, open-spec availability, vendor-neutral governance | A protocol controlled by a single vendor is not a standard — it's a product |

### The adapter pattern

Every protocol is tested through a **ProtocolAdapter** — a frozen interface that the protocol implementation must satisfy. The adapter handles connection initialization, capability negotiation, message exchange, schema validation, and teardown. This means:
- Protocols are tested identically
- The adapter is the only thing that changes per protocol
- New protocols can be added without changing the harness
- The adapter is published and open for review

### The canary

Every benchmark batch starts with a **no-protocol baseline** (the "canary"). If the benchmark leaked answers anywhere, the canary would score above zero. The canary must score exactly zero — this is a hard invariant. If it doesn't, the entire batch is invalid.

## Who this is for

- **Integration engineers** evaluating which protocol to adopt for multi-agent systems
- **Platform architects** making protocol adoption decisions with actual data
- **Protocol designers** who want independent benchmarking of their specification
- **Researchers** studying the AI interoperability landscape
- **Standards bodies** and **vendors** who want to improve their protocols based on real evidence
- **CTOs/CIOs** making build-vs-adopt decisions for agent interoperability frameworks

## What this is NOT

- Not a marketing site for any protocol, SDK, or standards body
- Not a "best of" list based on GitHub stars, funding rounds, or conference talks
- Not a tutorial on how to implement any protocol
- Not a replacement for your own conformance testing and due diligence
- Not a static document that never changes

## The "Quest"

The "Integration Engineer's Quest for Interoperability" is the ongoing effort to test, measure, and rank every protocol on the roster. It's not a one-time effort. It's a continuous process of:
1. **Discovery** — finding new protocols via research, standards bodies, community, and submissions
2. **Triage** — deciding if a protocol is serious enough to enter the roster (real spec, real implementations, real adoption)
3. **Smoke gate** — running every protocol through an identical 3-turn scenario to catch bugs (initialize connection, exchange message, verify compliance)
4. **Benchmark** — running standard conformance tests + custom interoperability tests + federation stress tests
5. **Publication** — publishing raw message logs + summary + per-protocol deep-dives
6. **Iteration** — re-testing as protocols update, as specs evolve, as new interoperability challenges emerge

## How to challenge a result

If you believe a ranking or score is wrong:
1. Check the **raw results JSON** — the message logs and validation results are public
2. Check the **adapter implementation** — the adapter code is public
3. Check the **judge prompts** — the prompts are frozen and public
4. File an issue with a specific claim and evidence (e.g., "MCP schema validation incorrectly fails on optional fields")
5. We'll re-run the test or update the methodology if warranted

## Governance

- **ArdurAI** maintains the almanac and runs the Quest
- **Methodology changes** require a public RFC and at least one edition cycle of notice
- **Protocol additions/removals** are decided by the triage criteria (spec maturity, real implementations, community activity, seriousness)
- **Benchmark results** are machine-generated; summaries are human-reviewed for fairness
- **Conflicts of interest** are disclosed (e.g., ArdurAI contributes to some protocols on the roster); mitigation is identical harness for all

## License

Content: CC BY 4.0  
Harness code: MIT  
Raw data: CC BY 4.0
