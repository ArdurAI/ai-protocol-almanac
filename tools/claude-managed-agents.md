# Claude Managed Agents

| Attribute | Value |
|-----------|-------|
| Tier | B |
| Type | Agent Platform |
| License | Commercial |
| Stars | N/A (managed service) |
| URL | https://anthropic.com/claude-managed-agents |

## One-line summary

Anthropic's fully managed agent runtime (beta, April 2026) that lets developers define, deploy, and run Claude-powered agents in Anthropic-hosted cloud containers with MCP server integration.

## Architecture

Claude Managed Agents is a hosted platform-as-a-service built on top of the Claude Agent SDK. The architecture separates three layers:

1. **Agent Definition Layer**: Developers define agents via the Claude Agent SDK — specifying model (Claude 4, 3.5, etc.), system prompt, tools, MCP servers, and guardrails. The definition is a declarative YAML/JSON configuration that Anthropic stores and versions.

2. **Container Runtime**: Anthropic runs each agent in a managed cloud container with an isolated filesystem, network policy, and resource limits. The runtime handles the agent loop (planning, tool execution, reflection, response generation) automatically.

3. **Tool/MCP Integration Layer**: Agents can attach MCP servers (via the `options.mcpServers` configuration) to access external tools, databases, and APIs. The managed runtime handles MCP server lifecycle (startup, health checks, reconnection) automatically.

Key primitives include the async `query()` generator for streaming, built-in tools (Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, AskUserQuestion), lifecycle hooks (PreToolUse, PostToolUse, Stop, SessionStart, SessionEnd, UserPromptSubmit), and subagents defined on `options.agents` with their own context windows.

## Key features

- Fully managed runtime — Anthropic hosts the container, agent loop, and infrastructure
- MCP server attachment with automatic lifecycle management
- Built-in sandboxed tool execution (filesystem, shell, web search)
- Subagent spawning with isolated context windows
- Lifecycle hooks for custom orchestration logic
- Streaming via `query()` generator
- Claude-native (only Claude models supported)

## Benchmark status

| Dimension | Status | Notes |
|-----------|--------|-------|
| Accuracy | Not run | No independent conformance testing; Claude model accuracy is high but not benchmarked in this context |
| Latency | Not run | Managed runtime latency depends on model choice and tool call chain length |
| Token Economics | Not run | Separate Agent SDK subscription credit ($20-$200/month depending on plan); standard API rates when credit exhausted |
| Scale Behavior | Not run | Beta service; no published multi-agent federation data |
| Ops Burden | Not run | Zero infrastructure burden (managed); but Claude-only model lock-in |
| Developer Experience | Not run | Claude Agent SDK is well-documented; managed UI for deployment |
| Data Sovereignty | Not run | Cloud-hosted; Anthropic processes agent data; no self-hosting option |

## Ops reality

- **Beta status**: As of July 2026, Claude Managed Agents is still in beta. Expect API changes, feature additions, and potentially pricing changes before GA.
- **Claude lock-in**: Only Claude models are supported. If you need multi-model agents (GPT-4, Gemini, Llama), this is not the platform.
- **MCP server reliability**: The managed runtime handles MCP server lifecycle, but if an MCP server crashes or becomes unresponsive, the agent may stall. There's no documented behavior for MCP server failure modes in the managed runtime.
- **Cost model change**: Starting June 15, 2026, Agent SDK usage (including managed agents) draws from a separate monthly credit ($20 on Pro, $100 on Max 5x, $200 on Max 20x). When exhausted, usage flows to standard API rates if enabled, otherwise requests stop. Unused credit does not roll over. This is a significant budget consideration for CI/scheduled workloads.
- **Sandbox limitations**: The sandboxed tool execution is powerful but constrained. Agents cannot install arbitrary packages, modify system-level configurations, or access the host network. The sandbox is designed for safe, contained execution.
- **Subagent isolation**: Subagents run with their own context windows and tools, but there's a cost overhead. Each subagent spawn is a new LLM call and context window initialization.

## Cost model

- **Subscription credit**: Separate monthly Agent SDK credit ($20-$200/month depending on Claude plan)
- **API fallback**: When credit exhausted, usage flows to standard API rates (if enabled) or stops
- **No rollover**: Unused credit does not roll over to the next month
- **Beta pricing**: Subject to change before GA

## When to use / when to avoid

**Use when:**
- You want Claude-powered agents with zero infrastructure management
- You need MCP server integration without self-hosting MCP servers
- You want sandboxed tool execution (safe, contained)
- You are already on Claude Pro/Max and have the Agent SDK credit

**Avoid when:**
- You need multi-model agents (Claude-only)
- You have strict data residency or compliance requirements (cloud-hosted, Anthropic processes data)
- You need to self-host or run on-premises
- You run high-volume CI/scheduled workloads (credit model may not scale cost-effectively)
- You need production-grade reliability (beta status)
