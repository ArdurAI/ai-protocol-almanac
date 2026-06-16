# Implementation Guide

How the protocol almanac is built, how to add a protocol, how to update an edition, and how the data pipeline works.

## Table of Contents

1. [Repository Structure](#repository-structure)
2. [The Data Pipeline](#the-data-pipeline)
3. [Adding a New Protocol](#adding-a-new-protocol)
4. [Updating an Edition](#updating-an-edition)
5. [The Roster JSON Schema](#the-roster-json-schema)
6. [Directory Conventions](#directory-conventions)
7. [Building the Adapter](#building-the-adapter)
8. [Automation](#automation)

---

## Repository Structure

```
ai-protocol-almanac/
├── README.md                          # Project overview + roster at a glance
├── INTENT.md                          # Philosophy, design principles, governance
├── IMPLEMENTATION.md                  # This file
├── TESTING.md                         # Benchmark methodology, harness details
├── TROUBLESHOOTING.md                 # Common issues, debugging, FAQ
├── CONTRIBUTING.md                    # How to contribute
├── architecture.md                    # Stack architecture + protocol stack diagram
├── SETUP.md                           # How to push to GitHub
├── .gitignore
│
├── standards/                         # Per-standard deep references
│   ├── mcp/README.md                  # Model Context Protocol deep reference
│   ├── a2a/README.md                  # Agent-to-Agent protocol deep reference
│   ├── ucp/README.md                  # Universal Commerce Protocol reference
│   ├── x402/README.md                 # x402 payments protocol reference
│   ├── wimse/README.md              # WIMSE (IETF) reference
│   └── ...
│
├── editions/                          # Monthly editions
│   └── 2026-06.md                   # Founding edition
│
├── benchmarks/                        # Benchmark results (rolling)
│   └── (populated as results land)
│
├── methodology/
│   └── benchmark-harness.md         # Detailed harness spec for protocol testing
│
├── data/
│   └── roster.json                  # Machine-readable catalog (46 protocols)
│
├── tools/                             # Per-protocol deep-dive pages (placeholder)
│   └── (populated as deep-dives are written)
│
└── assets/                            # Sequence diagrams, conformance charts, spec comparison tables
    └── (populated by editions)
```

## The Data Pipeline

The almanac data flows through four stages:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Discovery      │────▶│  Triage         │────▶│  Research       │────▶│  Publication    │
│  (find protocols│     │  (decide entry) │     │  (deep dive)    │     │  (write edition)│
│   + SDKs)       │     │                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Stage 1: Discovery

Protocols and SDKs are discovered through:
- **Monthly research swarm**: 8-10 parallel agents search for new protocols, SDK releases, and spec updates
- **Community submissions**: Issues, PRs, email, social media, standards-body announcements
- **Vendor announcements**: Major SDK releases, protocol version bumps, breaking changes
- **GitHub trending**: New repos implementing emerging protocols (e.g., MCP servers, A2A agents)
- **Standards tracking**: IETF drafts, W3C working groups, industry consortia publications
- **Conference talks**: Papers, demos, interoperability bake-offs from major events

### Stage 2: Triage

A protocol enters the roster if it meets ALL of these criteria:
1. **Seriousness**: Not a toy/demo spec. Must have a real specification document, real implementation(s), or real adoption.
2. **Activity**: Last spec update, SDK release, or implementation push within 6 months. Exceptions for "stable/mature" standards (e.g., SCIM, OAuth 2.0).
3. **Documentation**: Must have a spec, README, or at least a landing page explaining what the protocol does and how to implement it.
4. **Accessibility**: Must be accessible to test (open source spec, free SDK, or evaluation license available).
5. **Scope**: Must fit the category definition. A general-purpose HTTP library doesn't enter a protocol category.

A protocol is **excluded** if:
- It's a re-branding of another protocol with no meaningful divergence
- It's a wrapper SDK around another protocol with no added interoperability value
- It has no implementations, no community, and no evidence of real-world use
- It requires an enterprise-only license with no evaluation path
- The specification is not publicly available

### Stage 3: Research

For each new protocol, we collect:
- Name, type (standard, SDK, framework), license, language, spec URL, GitHub URL, stars
- Last push date, release cadence, spec version
- Key features and differentiators (transport layers, auth schemes, schema formats)
- Known bugs and sharp edges (from smoke gate: version negotiation failures, schema drift, non-compliant extensions)
- Community health (issues, PRs, maintainer responsiveness, standards-body participation)
- Ecosystem adoption (number of implementations, major users, SDK availability)

This data is stored in `data/roster.json` and summarized in the edition.

### Stage 4: Publication

The edition is a markdown file that includes:
- Landscape at a glance table (protocols by tier, transport type, governance model)
- Per-standard findings and trends (MCP ecosystem growth, A2A adoption, new IETF drafts)
- New protocols added and protocols removed
- Notable spec releases and breaking changes
- Quest diary (what was tested this month: conformance tests run, interoperability bake-offs, federation stress tests)

## Adding a New Protocol

### Step 1: Verify the protocol meets triage criteria

Check: seriousness, activity, documentation, accessibility, scope.

### Step 2: Add to the roster JSON

Edit `data/roster.json` and add the protocol to the appropriate category:

```json
{
  "name": "ProtocolName",
  "type": "Standard|SDK|Framework",
  "license": "License",
  "region": "Region",
  "tier": "A|B|C",
  "transport": "WebSocket|SSE|HTTP|gRPC|Multiple",
  "governance": "Vendor|Consortium|IETF|W3C|Open Community",
  "notes": "One-line description and key differentiators"
}
```

**Tier assignment rules**:
- **Tier A**: De facto standard, widest adoption, or strongest technical merit. Must have multiple independent implementations and real production usage. Examples: MCP, A2A, Anthropic SDK.
- **Tier B**: Solid option, actively maintained, but not the dominant standard. Good for specific use cases or emerging ecosystems. Examples: ACP, ANP, x402.
- **Tier C**: Niche, early-stage, or specialized. Worth knowing about but not a default choice. Examples: AGEX, OAN, NANDA.

### Step 3: Update the edition

Add the protocol to the appropriate section in `editions/YYYY-MM.md`. If the protocol is Tier A, add it to the roster-at-a-glance table in the README.

### Step 4: Update the standard deep-reference

If the protocol is Tier A, create or update `standards/<protocol-key>/README.md` with a deep reference covering:
- Spec summary and version history
- Transport and schema details
- Known implementations and SDKs
- Interoperability notes
- Compliance checklist

### Step 5: Run the smoke gate

Before the protocol is officially "in," it must pass the smoke gate (see TESTING.md). If it fails, document the failure in the edition and assign it to Tier C with a note about the blocker.

## Updating an Edition

### Monthly update checklist

```
□ Check for new protocols and SDKs (discovery phase)
□ Triage new protocols (add to roster or reject)
□ Update metadata for existing protocols (spec versions, SDK releases, last push)
□ Flag protocols for removal (dead/abandoned specs)
□ Run smoke gate for new protocols
□ Run benchmark updates for re-tested protocols
□ Draft the edition markdown
□ Update README roster-at-a-glance
□ Update standard deep-references
□ Commit and push
```

### Edition markdown template

```markdown
# Edition YYYY-MM — [Title]

*Research conducted YYYY-MM-DD. [Context about this month: e.g., "MCP 2025-11 spec released, A2A adoption accelerates"].*

## The landscape at a glance

| Standard | Tool Count | New This Month | Notable Changes |
|----------|-----------|----------------|-----------------|

## [Standard] — [Theme]

### Tier A roster
[table]

### Findings
[bullets]

## Quest diary — [Month] [Year]

- [what was done: conformance tests, interoperability bake-offs, spec reviews]

## Coming next month

[what's planned]

## License
Content is licensed CC BY 4.0.
```

## The Roster JSON Schema

```json
{
  "meta": {
    "name": "AI Protocol Almanac Roster",
    "version": "YYYY-MM",
    "generated_at": "ISO-8601 timestamp",
    "total_protocols": number,
    "standards": number,
    "research_method": "description"
  },
  "standards": {
    "standard-key": {
      "name": "Human-readable name",
      "description": "What this standard covers",
      "estimated_total": number,
      "protocols": [
        {
          "name": "Protocol Name",
          "type": "Standard|SDK|Framework",
          "license": "License",
          "region": "Region",
          "tier": "A|B|C",
          "transport": "WebSocket|SSE|HTTP|gRPC|Multiple",
          "governance": "Vendor|Consortium|IETF|W3C|Open Community",
          "notes": "Description"
        }
      ]
    }
  }
}
```

**Field definitions**:
- `name`: The protocol's common name. Use the name the protocol calls itself.
- `type`: What kind of artifact is it? (e.g., "Standard", "SDK", "Framework", "Implementation")
- `license`: The primary license. Use SPDX identifiers where possible.
- `region`: Where the protocol is primarily developed (US, EU, China, Global, etc.)
- `tier`: A, B, or C (see tier rules above)
- `transport`: Primary transport mechanism(s) used by the protocol
- `governance`: Who controls the specification (vendor, consortium, standards body, open community)
- `notes`: One-line description with key differentiators. Keep under 100 chars.

## Directory Conventions

### `editions/`
- One file per month: `YYYY-MM.md`
- Never delete old editions. The history is part of the record.
- New editions are appended; old editions are never rewritten.

### `data/`
- `roster.json` is the single source of truth for the protocol catalog.
- It is machine-generated from the research process.
- It should be valid JSON at all times.

### `benchmarks/`
- One file per benchmark run: `<standard>-<suite>-<date>.md`
- Raw JSON files alongside the markdown: `<standard>-<suite>-<date>.json`
- Raw data is never deleted. It is the audit trail.
- Protocol-specific logs: message traces, handshake dumps, schema validation results

### `tools/`
- One file per protocol implementation: `<name>.md`
- Contains deep-dive analysis: spec compliance notes, benchmark results, interoperability findings, bug notes, comparison with peers
- Populated as deep-dives are written (not all protocols have a page immediately)

### `standards/`
- One directory per major standard: `<standard-key>/README.md`
- Contains: spec summary, version history, transport details, schema format, known implementations, compliance checklist, interoperability notes
- Updated when specs change or new implementations emerge

### `assets/`
- Sequence diagrams, conformance charts, spec comparison tables, interoperability matrices
- Named descriptively: `mcp-conformance-2026-06.png`, `a2a-interop-matrix-2026-06.png`, `protocol-landscape-2026-06.png`

### `methodology/`
- The protocol benchmark harness specification
- Frozen before any results are generated
- Changes require an RFC and a public announcement

## Building the Adapter

When a new protocol is added to the roster and is ready for benchmarking, an adapter must be built. The adapter is the bridge between the protocol's API/SDK and the harness's fixed interface.

### The ProtocolAdapter contract

```python
class ProtocolAdapter:
    def setup(self) -> None:
        """Initialize the protocol connection (server, client, or both)."""
        pass
    
    def negotiate(self) -> None:
        """Perform capability negotiation and version handshake."""
        pass
    
    def load(self, workload) -> None:
        """Ingest the workload into the protocol (schemas, messages, contexts)."""
        pass
    
    def await_ready(self) -> None:
        """Wait for async negotiation to complete. Measure handshake lag."""
        pass
    
    def query(self, query) -> Response:
        """Run the test message exchange and return the response."""
        pass
    
    def validate(self, response) -> ValidationResult:
        """Validate the response against the protocol specification."""
        pass
    
    def teardown(self) -> None:
        """Clean up connections, measure resource usage."""
        pass
```

### Adapter rules

1. The adapter must be **pure** — it should not modify the protocol's behavior, only interface with it.
2. The adapter must be **documented** — every step should be explainable in plain English (e.g., "We send a capabilities request and expect a JSON-RPC response with these fields").
3. The adapter must be **reproducible** — running it twice on the same machine should produce the same handshake.
4. The adapter must be **isolated** — it should not depend on other protocols' adapters.
5. The adapter code is **published** in the benchmark harness repo (separate from the almanac repo).

### Adapter types for protocols

| Adapter Type | Purpose | Example |
|-------------|---------|---------|
| **MCP Compliance Adapter** | Tests conformance to the Model Context Protocol specification | Validates `tools/list`, `resources/read`, `prompts/get` request/response schemas |
| **A2A Interop Adapter** | Tests Agent-to-Agent interoperability across implementations | Exchanges task messages between two different A2A SDKs and validates schema compliance |
| **Custom Protocol Test Adapter** | Tests a bespoke or emerging protocol against its own spec | Validates message sequencing, auth negotiation, and error handling for a non-standard protocol |
| **Transport Abstraction Adapter** | Tests protocol behavior across different transport layers | Runs the same protocol over WebSocket, SSE, and HTTP to measure transport-specific behavior |

### Example adapter (pseudocode)

```python
class MCPAdapter(ProtocolAdapter):
    def __init__(self, sdk_name, config):
        self.sdk = sdk_name
        self.config = config
        self.client = None
    
    def setup(self):
        if self.sdk == "anthropic-mcp":
            self.client = MCPClient(self.config)
            self.server = MCPServer(stdio=self.config["command"])
        elif self.sdk == "smithery-mcp":
            self.client = SmitheryClient(self.config)
    
    def negotiate(self):
        self.capabilities = self.client.initialize(
            protocol_version="2025-11-18",
            capabilities={"tools": {}, "resources": {}}
        )
    
    def load(self, schemas):
        for schema in schemas:
            self.server.register_tool(schema)
    
    def await_ready(self):
        # Wait for server to be ready to accept requests
        while not self.server.is_ready():
            time.sleep(0.1)
    
    def query(self, request):
        return self.client.send_request(request)
    
    def validate(self, response):
        return validate_mcp_schema(response, self.capabilities.protocol_version)
    
    def teardown(self):
        self.client.close()
        self.server.stop()
```

## Automation

### Monthly update cron

The monthly update is run by a scheduled job:
- **Trigger**: `cron` expression `0 7 15 * *` (monthly, 15th at 7:00 AM)
- **Action**: Runs a research agent to discover new protocols, update metadata, and draft the next edition
- **Output**: Commits to the repo with the updated roster and new edition

### GitHub Actions (optional)

For automatic metadata refresh (GitHub stars, spec versions, last push dates), a GitHub Actions workflow can be configured:

```yaml
name: Monthly Protocol Metadata Refresh
on:
  schedule:
    - cron: '0 7 1 * *'
  workflow_dispatch:
jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Refresh metadata
        run: python scripts/refresh_metadata.py
      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "Monthly protocol metadata refresh: $(date +%Y-%m)" || echo "No changes"
          git push
```

## License

Content: CC BY 4.0  
Code: MIT
