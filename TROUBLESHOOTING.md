# Troubleshooting & Debugging

How to understand the protocol codebase, debug issues, and resolve common problems when working with the almanac.

## Table of Contents

1. [Understanding the Codebase](#understanding-the-codebase)
2. [Common Issues](#common-issues)
3. [Debugging the Data Pipeline](#debugging-the-data-pipeline)
4. [Debugging Benchmark Runs](#debugging-benchmark-runs)
5. [FAQ](#faq)
6. [Getting Help](#getting-help)

---

## Understanding the Codebase

### High-level flow

```
Research Agents → Research Output (Markdown) →
  Python Script → roster.json (Structured Data) →
    Manual Review → Edition Markdown →
      Git Commit → GitHub Publication
```

### Key files and their roles

| File | Role | When to read it |
|------|------|-----------------|
| `README.md` | Project overview, quick reference | First thing you read |
| `INTENT.md` | Philosophy, why we do things this way | When you disagree with a decision |
| `IMPLEMENTATION.md` | How things are built, how to add protocols | When you want to contribute |
| `TESTING.md` | Benchmark methodology, scoring | When you want to reproduce or challenge a result |
| `TROUBLESHOOTING.md` | This file | When something is broken |
| `architecture.md` | Stack architecture diagram | When you want to understand the big picture |
| `editions/YYYY-MM.md` | Monthly snapshot of the landscape | When you want historical data |
| `data/roster.json` | Machine-readable catalog | When you want to query or analyze the data |
| `methodology/benchmark-harness.md` | Harness specification | When you want to build an adapter or run benchmarks |
| `standards/<key>/README.md` | Per-standard deep reference | When you want spec details for a specific protocol |

### The data model

The almanac is fundamentally a **directed graph** of data:

```
Research findings → Protocol metadata → Roster JSON → Edition Markdown → README
                                          ↓
                                   Benchmark results → Per-protocol pages
```

- **Research findings** are the raw output of the research swarm. They're saved in `research/` (not in the public repo).
- **Protocol metadata** is extracted from research and stored in `data/roster.json`.
- **Roster JSON** is the single source of truth. Everything else derives from it.
- **Edition markdown** is human-written based on the roster and research.
- **README** is auto-generated from the roster and the latest edition.

### Understanding `data/roster.json`

This is the most important file in the repo. It is the single source of truth.

**Structure**:
```json
{
  "meta": { ... },
  "standards": {
    "standard-key": {
      "name": "...",
      "description": "...",
      "estimated_total": N,
      "protocols": [
        { "name": "...", "type": "...", "license": "...", "tier": "A|B|C", "transport": "...", "governance": "...", "notes": "..." }
      ]
    }
  }
}
```

**How to query it**:
```bash
# Find all Tier A protocols in a standard
jq '.standards."mcp".protocols[] | select(.tier == "A") | .name' data/roster.json

# Count protocols by tier
jq '.standards | to_entries[] | .value.protocols | group_by(.tier) | map({tier: .[0].tier, count: length})' data/roster.json

# Find all MIT-licensed protocols
jq '.. | objects | select(.license == "MIT") | .name' data/roster.json

# Find all WebSocket-based protocols
jq '.. | objects | select(.transport | contains("WebSocket")) | .name' data/roster.json
```

### The edition markdown

Editions are **human-written** summaries, not machine-generated. They are based on the roster but include analysis, interpretation, and narrative that a machine can't produce.

**How editions are structured**:
1. Front matter: date, research method, context (e.g., "MCP 2025-11 spec released")
2. Landscape at a glance: summary table (protocols by tier, transport, governance)
3. Per-standard sections: findings, roster, analysis, interoperability notes
4. Quest diary: what was tested this month (conformance tests, interop bake-offs, federation stress)
5. Cross-standard findings: patterns that span standards (e.g., "WebSocket adoption growing across all protocols")

### The benchmark harness (separate repo)

The actual benchmark code lives in a separate repository. The almanac repo contains:
- The methodology specification
- The results (JSON + markdown)
- The adapters (interface definitions)

The harness repo contains:
- The runner code
- The adapter implementations (MCP compliance adapter, A2A interop adapter, custom protocol test adapter)
- The schema validator integration
- The telemetry collector

**Why separate?** Because the harness is code that runs, and the almanac is data that is published. They have different lifecycles and different audiences.

## Common Issues

### Issue: `roster.json` is invalid JSON

**Symptoms**:
- `jq` fails to parse it
- GitHub Actions fails on JSON validation
- Python `json.load()` raises `JSONDecodeError`

**Diagnosis**:
```bash
python3 -c "import json; json.load(open('data/roster.json'))"
```

**Resolution**:
1. Find the line with the error: `python3 -m json.tool data/roster.json`
2. Common causes: trailing commas, unescaped quotes, incorrect nesting
3. Fix the JSON and re-validate
4. Consider using a JSON linter in your editor

### Issue: Edition markdown has broken links

**Symptoms**:
- Links to protocols return 404
- Links to benchmarks don't exist yet
- Relative links work locally but break on GitHub

**Diagnosis**:
```bash
# Check all links in the repo
find . -name "*.md" -exec grep -oP '\[.*?\]\(.*?\)' {} + | grep -v "http" | grep -v "mailto"
```

**Resolution**:
1. For internal links, use relative paths: `../data/roster.json`
2. For external links, verify the URL is correct (spec URLs change frequently)
3. For protocols without a per-protocol page yet, link to their spec or GitHub repo
4. Run a link checker as part of CI

### Issue: Tier assignment is wrong

**Symptoms**:
- A protocol is Tier A but has no production implementations
- A protocol is Tier C but is widely adopted
- A protocol's tier changed without explanation

**Diagnosis**:
1. Check the tier assignment rules in `IMPLEMENTATION.md`
2. Verify the protocol's adoption, implementation count, and community health
3. Check the edition notes for the rationale

**Resolution**:
1. File an issue with evidence (implementation count, production references, GitHub stars)
2. The tier will be reviewed in the next edition cycle
3. Tiers are not changed mid-edition; they are updated at edition boundaries

### Issue: Benchmark results can't be reproduced

**Symptoms**:
- Running the harness produces different results
- The adapter fails with a different protocol version
- The schema validator is unavailable

**Diagnosis**:
1. Check the `results.json` metadata for the exact commit, seed, and hardware
2. Check if the protocol version or SDK version has changed since the published run
3. Verify the schema validator is accessible
4. Check if the transport layer (WebSocket, SSE, HTTP) is behaving differently

**Resolution**:
1. Use the exact commit and dependencies from the results metadata
2. If the protocol version changed, the results are for a different version — this is expected
3. If the schema validator changed, that's a methodology issue — file an issue
4. If transport behavior differs, check network configuration and proxy settings

### Issue: Adapter fails on protocol version negotiation

**Symptoms**:
- `negotiate()` crashes with a version mismatch error
- `await_ready()` times out because the server rejects the client version
- `query()` returns a version error instead of the expected response

**Diagnosis**:
1. Check the protocol version supported by the adapter vs. the implementation
2. Check the spec version history — was there a breaking change?
3. Check if the implementation supports version negotiation or only accepts a single version

**Resolution**:
1. Update the adapter to support the implementation's version
2. If the implementation is non-compliant, document it and file a bug with the vendor
3. If the spec is ambiguous, document the ambiguity and propose a clarification

### Issue: Benchmark results inconsistent due to spec ambiguity

**Symptoms**:
- Same protocol, same test, different compliance scores across runs
- Schema validation fails intermittently on the same message
- Two team members get different results testing the same protocol

**Diagnosis**:
1. Check if the spec defines the behavior precisely or leaves it implementation-defined
2. Check if the schema validator is strict or lenient on optional fields
3. Check if the protocol has non-compliant extensions that the adapter treats differently

**Resolution**:
1. Document the spec ambiguity in the protocol's deep-dive page
2. If the spec is ambiguous, the adapter should follow the most common implementation behavior
3. File an issue with the spec authors to clarify the ambiguous behavior
4. Record the ambiguity in the benchmark results metadata

### Issue: Transport layer differences cause test failures

**Symptoms**:
- Protocol works over HTTP but fails over WebSocket
- SSE stream behaves differently from HTTP polling
- Message ordering differs across transports

**Diagnosis**:
1. Check if the spec requires transport-agnostic behavior or allows transport-specific behavior
2. Check if the adapter's transport abstraction is leaking transport details
3. Check if the implementation has transport-specific bugs

**Resolution**:
1. If the spec is transport-agnostic, the implementation should behave identically — file a bug
2. If the spec allows transport-specific behavior, document it and update the adapter
3. Run the transport abstraction test to measure parity across transports
4. Document transport-specific behavior in the protocol's deep-dive page

### Issue: Research agent missed a protocol

**Symptoms**:
- A well-known protocol is not in the roster
- A protocol from a specific region or standards body is missing
- A newly launched protocol is not in the latest edition

**Diagnosis**:
1. Check if the protocol meets triage criteria in `IMPLEMENTATION.md`
2. Check if it was added in a previous edition and later removed
3. Check if it falls outside the search scope (e.g., a proprietary protocol with no public spec)

**Resolution**:
1. File an issue with the protocol name, spec URL, and evidence of adoption/activity
2. The protocol will be triaged for the next edition
3. If it meets criteria, it will be added

### Issue: Monthly update cron failed

**Symptoms**:
- No new edition was published on the 15th
- The cron job is missing from the scheduler
- The research agent timed out

**Diagnosis**:
```bash
# Check cron status
cron status

# Check the cron job list
# (use the Kimi Work cron interface)
```

**Resolution**:
1. Check if the cron job is still registered
2. Check if the research agent timed out (increase timeout if needed)
3. Manually trigger the update if the cron missed a cycle
4. Check the workspace path in the cron job configuration

### Issue: GitHub push fails

**Symptoms**:
- `git push` returns 403 or 401
- The remote is not configured
- The branch is behind origin

**Diagnosis**:
```bash
git remote -v
git status
git log --oneline -5
```

**Resolution**:
1. Verify the remote URL is correct: `git remote set-url origin https://github.com/ArdurAI/...`
2. Verify GitHub CLI auth: `gh auth status`
3. If behind origin, pull first: `git pull origin main`
4. If there are conflicts, resolve them manually

## Debugging the Data Pipeline

### Research output → roster.json

**Problem**: Research agents produce markdown, but the roster JSON is incomplete or wrong.

**Debug steps**:
1. Read the research output files in `research/` (local workspace, not in the repo)
2. Check if the Python extraction script correctly parsed the protocol tables
3. Check if protocols were dropped during triage (check the triage log)
4. Verify the JSON schema is correct

**Common bugs**:
- Protocol names with special characters break JSON parsing → Escape them properly
- Protocols with no tier get dropped → Default to Tier C if unsure
- Protocols with no notes get empty strings → Add a minimal note
- Protocols with no transport field get dropped → Add "Unknown" as default

### roster.json → edition markdown

**Problem**: The edition doesn't reflect the roster.

**Debug steps**:
1. Compare the protocol counts in the roster vs. the edition
2. Check if the edition was written before the roster was updated
3. Check if protocols were manually edited in the edition but not in the roster

**Resolution**:
1. The edition should be derived from the roster, not the other way around
2. If the edition has manual additions, ensure they are also in the roster
3. The edition is a human-readable summary; the roster is the source of truth

### Edition markdown → README

**Problem**: The README roster-at-a-glance doesn't match the latest edition.

**Debug steps**:
1. Check which edition is referenced in the README
2. Check if the README was updated after the edition was published

**Resolution**:
1. The README should always reference the latest edition
2. Update the README when a new edition is published
3. Consider automating README updates from the roster JSON

## Debugging Benchmark Runs

### The adapter fails

**Symptoms**:
- `setup()` crashes
- `negotiate()` throws a version mismatch exception
- `query()` returns nothing or a non-compliant response
- `validate()` fails schema checks on a message the spec says is valid

**Debug steps**:
1. Run the adapter in isolation (without the harness)
2. Check the protocol's spec for setup requirements and version support
3. Check if environment variables are set (API keys, auth tokens, endpoint URLs)
4. Check if the protocol version matches what the adapter expects
5. Check if the transport layer (WebSocket, SSE, HTTP) is configured correctly

**Common fixes**:
- Missing transport daemon (e.g., WebSocket server) → Start the transport
- Missing API key or auth token → Set the environment variable
- Wrong protocol version → Update the adapter or pin the version
- Dependency conflict → Use a virtual environment or container
- Spec ambiguity → Document the ambiguity and choose the most common behavior
- Transport-specific bug → Switch to a different transport or document the limitation

### The canary fails

**Symptoms**:
- The no-protocol baseline scores above zero
- The abstention score is not 1.000

**Debug steps**:
1. Check if the benchmark workload has leaked answers
2. Check if the grading pipeline has a bug
3. Check if the schema validator is incorrectly marking empty responses as valid
4. Check if the random seed was set

**Resolution**:
1. If the workload leaked answers, redesign the workload
2. If the grader has a bug, fix the grader and rerun all tests
3. This is a critical failure — the entire batch is invalid

### Results are inconsistent

**Symptoms**:
- Same protocol, same test, different results across runs
- Scores vary by more than the confidence interval
- Handshake timing varies by orders of magnitude

**Debug steps**:
1. Check if the protocol has non-deterministic behavior (async timing, network-dependent negotiation)
2. Check if the hardware was different between runs
3. Check if the protocol version changed between runs
4. Check if the transport layer has variable latency (e.g., WebSocket vs. HTTP)

**Resolution**:
1. Pin protocol versions and record them in results metadata
2. Record hardware specs and network configuration in results metadata
3. Run multiple warm-up handshakes before measurement to stabilize connection state
4. If transport variability is expected, document it and measure per-transport

## FAQ

### Q: Why is protocol X not in the roster?

A: Either it doesn't meet triage criteria, it hasn't been discovered yet, or it was removed for inactivity. File an issue with evidence and we'll triage it.

### Q: Why did protocol X's score change?

A: Either the protocol was updated, the methodology was refined, or we found a bug in our previous test. All three are valid reasons. Check the edition notes for the rationale.

### Q: Can I run the benchmarks myself?

A: Yes. The harness is published separately. Clone it, install dependencies, and run the adapter for the protocol you want to test. See `TESTING.md` for reproducibility instructions.

### Q: How do I challenge a ranking?

A: File an issue with specific evidence. Check the raw results JSON, the adapter code, and the schema validator. If you find a real problem, we'll re-run or update the methodology.

### Q: Can I add a protocol to the roster?

A: Yes. See `CONTRIBUTING.md` for instructions. The protocol must meet triage criteria and pass the smoke gate.

### Q: Why are there separate standard almanacs?

A: Each standard area (MCP, A2A, auth, etc.) is deep enough to warrant its own dedicated documentation with spec details, implementation notes, and compliance checklists. The parent almanac is the master catalog.

### Q: How often are benchmarks re-run?

A: Standard conformance: every quarter for each protocol. Stress: annually. Integration: quarterly. If a protocol releases a major version or spec update, we may re-run early.

### Q: What's the difference between Tier A, B, and C?

A: Tier A = de facto standard or strongest technical merit. Tier B = solid option, specific use cases. Tier C = niche, early-stage, or specialized. See `IMPLEMENTATION.md` for full rules.

### Q: Can vendors or standards bodies sponsor the almanac?

A: No. The almanac is independently funded. Sponsorship would compromise the core mission. Vendors and standards bodies can improve their scores by actually improving their protocols.

### Q: What do I do when a spec is ambiguous?

A: Document the ambiguity in the protocol's deep-dive page, choose the most common implementation behavior for the adapter, and file an issue with the spec authors. The benchmark results should note the ambiguity.

### Q: How do I handle protocol version mismatches?

A: The adapter should support version negotiation if the spec defines it. If the implementation is non-compliant, document the failure and assign a lower compliance score. If the spec is unclear, document the ambiguity.

### Q: What if a protocol works on one transport but not another?

A: Run the transport abstraction test. If the spec requires transport-agnostic behavior, file a bug with the implementation. If the spec allows transport-specific behavior, document it and measure per-transport.

## Getting Help

### File an issue

GitHub issues are the primary support channel. Use the appropriate template:

- **Protocol request**: "Add [Protocol Name] to [Standard]"
- **Data correction**: "[Protocol Name] metadata is wrong: [what's wrong]"
- **Benchmark challenge**: "Challenge [Protocol Name] ranking on [Dimension]: [evidence]"
- **Bug report**: "[Bug description] in [file/process]"
- **Feature request**: "[Feature description] for [use case]"
- **Spec ambiguity**: "[Protocol Name] spec is ambiguous on [behavior]: [details]"

### Discussion

GitHub Discussions are for:
- General questions about the almanac
- Sharing experiences with protocols on the roster
- Proposing methodology changes
- Community announcements
- Spec interpretation questions

### Email

For private or sensitive inquiries: Use the contact info in the ArdurAI org profile.

## License

Content: CC BY 4.0
