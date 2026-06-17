# Benchmarks

The `benchmarks/` directory contains benchmark results for the AI Protocol Almanac.

## Structure

| File Pattern | Description |
|-------------|-------------|
| `<protocol>-<suite>-<date>.md` | Markdown summary of benchmark results |
| `<protocol>-<suite>-<date>.json` | Raw benchmark data (message traces, handshake logs, schema validation results) |
| `<protocol>-<suite>-<date>-adapter.json` | Adapter configuration and metadata |
| `cross-standard-<date>.md` | Cross-standard integration test results |
| `README.md` | This file |

## Current status

**No benchmark results have been published yet.**

The first benchmark run is scheduled for **Q3 2026** (July-September). The benchmark harness will test Tier A protocols (MCP, A2A, UCP, x402, OAuth 2.1, WIMSE, AIMS, OpenTelemetry GenAI) across the seven scored dimensions:

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| Accuracy / Compliance | 25% | Protocol conformance, schema validation, message sequencing |
| Latency | 15% | Handshake time, message round-trip time |
| Token Economics | 15% | Bytes-on-wire per message, metadata overhead |
| Scale Behavior | 15% | Multi-agent federation performance |
| Ops Burden | 15% | Setup time, version negotiation, upgrade pain |
| Developer Experience | 10% | SDK quality, documentation, debugging |
| Data Sovereignty | 5% | Open spec, self-hosting, vendor neutrality |

## Benchmark schedule

| Quarter | Protocols tested | Test type |
|---------|-----------------|-----------|
| Q3 2026 | MCP, A2A, OAuth 2.1 | Standard conformance + smoke gate |
| Q4 2026 | UCP, x402, WIMSE | Standard conformance + stress suite |
| Q1 2027 | AIMS, SCIM for Agents, OpenTelemetry GenAI | Standard conformance + integration tests |
| Q2 2027 | All Tier A + selected Tier B | Full benchmark suite + cross-standard tests |

## How to reproduce

When results are published, each benchmark run will include:

1. **Raw JSON** with every message exchanged, every handshake attempted, every validation result
2. **Adapter code** used for the run (pinned commit)
3. **Harness metadata** (judge model, prompts hash, control variables, random seed, hardware spec)
4. **Reproducibility instructions** in the markdown summary

See `methodology/benchmark-harness.md` for the full methodology.

## License

Benchmark data: CC BY 4.0
Harness code: MIT
