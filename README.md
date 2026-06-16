# Context Protocols, Authentication & Integration Almanac

A living encyclopedia of AI protocols, authentication standards, and integration patterns. Updated monthly with fresh repo metadata, releases, landscape shifts, and independent benchmark results.

> Vendors publish their own benchmark numbers. Nobody reproduces them independently, and nobody evaluates tools the way a platform engineer has to live with them: ops burden, failure modes, scale curves, and cost. This almanac is the public record of that work.

## How to use this repo

| You want… | Go to |
|-----------|-------|
| The state of the landscape right now | The latest file in `editions/` |
| Everything we know about one tool | `tools/<name>.md` |
| Machine-readable roster + metadata | `data/roster.json` |
| Architecture diagrams | `architecture.md` |
| Benchmark results (rolling) | `benchmarks/` |
| How tools are tested and ranked | `methodology/benchmark-harness.md` |
| Project intent & philosophy | `INTENT.md` |
| Implementation guide & how to add tools | `IMPLEMENTATION.md` |
| Testing methodology & benchmarks | `TESTING.md` |
| Troubleshooting & debugging | `TROUBLESHOOTING.md` |
| How to contribute | `CONTRIBUTING.md` |

## The roster

**Tier A** — 13 tools: MCP (Model Context Protocol), A2A (Agent-to-Agent), UCP (Universal Commerce Protocol), x402, WIMSE (IETF), AIMS (IETF), SCIM for Agents, Vercel AI SDK, OpenAI Agents SDK, Anthropic SDK…

**Tier B** — 26 tools: ACP (Agent Communication Protocol), ANP (Agent Network Protocol), AP2 (Agent Payments Protocol), MPP (Stripe), Authed, Smithery, Glama, mcp.so, PulseMCP, MCPize…

**Tier C** — 7 tools: AGEX, OAN, NANDA, Agora Protocol, AConP (Cisco), Agent Name Service (ANS), ERC-8004 (Trustless Agents)

**Total: 46 tools**

## Methodology

Results published here come from a frozen-before-results harness:
- Standard benchmarks for comparability with published claims — every ranking ships a _published vs. reproduced_ table.
- A custom PlatformOps benchmark: testing on infrastructure work — setup, reliability, scale, cost.
- A stress suite: contradiction storms, near-duplicate floods, concurrent writers, kill-the-backing-store chaos, cost-runaway measurement.
- Seven scored dimensions: accuracy, latency, token economics, scale behavior, **ops burden**, developer experience, data sovereignty.

The judge model, prompts (SHA-256-frozen), and control variables were fixed before any tool ran. Raw results JSON is published with every ranking.

## Update cadence

One edition per month under `editions/YYYY-MM.md`: refreshed metadata, notable releases, new entrants triaged in or out, and a diary of what was tested.

## License

Content is licensed CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
