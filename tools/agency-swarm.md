# Agency Swarm

| Attribute | Value |
|-----------|-------|
| Tier | C |
| Type | Agent Framework |
| License | MIT |
| Stars | ~4,100 |
| URL | https://github.com/VRSEN/agency-swarm |

## One-line summary

A lightweight Python framework for building collaborative AI agent teams modeled after organizational structures, built on top of the OpenAI Agents SDK with role-based agent definitions and task dependency resolution.

## Architecture

Agency Swarm is an open-source Python framework that abstracts multi-agent collaboration through an organizational metaphor. The architecture is built on top of the OpenAI Agents SDK and provides three layers:

1. **Agent Definition Layer**: Developers define agents with distinct personas (CEO, Developer, Researcher, etc.), each with tailored instructions, tools, and capabilities. Agents are defined using Pydantic models for type-safe tool schemas and automatic argument validation.

2. **Crew Assembly Layer**: Agents are assembled into "crews" (teams) with tasks, dependencies, and execution order. The framework handles task dependency resolution — if Task B depends on Task A's output, Agency Swarm ensures execution order.

3. **Orchestration Layer**: The framework manages the agent conversation loop, tool execution, handoffs between agents, and result aggregation. It uses the OpenAI Agents SDK under the hood for the actual LLM calls and tool invocations.

## Key features

- Role-based agent definitions (CEO, Developer, Researcher, etc.) with tailored instructions
- Crew assembly with task dependency resolution and execution ordering
- Type-safe tools via Pydantic models with automatic argument validation
- OpenAPI schema import for tool definitions
- Built on OpenAI Agents SDK (inherits MCP, sandbox, and tracing capabilities)
- Python 3.12+ required
- MIT license, open source

## Benchmark status

| Dimension | Status | Notes |
|-----------|--------|-------|
| Accuracy | Not run | Inherits OpenAI Agents SDK accuracy; no independent testing |
| Latency | Not run | Role-playing overhead adds extra LLM calls for "staying in character" |
| Token Economics | Not run | Extra LLM calls for role-playing increase token costs |
| Scale Behavior | Not run | No published multi-crew federation data |
| Ops Burden | Not run | Fast setup for role-based agents; debugging multi-agent interactions is opaque |
| Developer Experience | Not run | Intuitive organizational metaphor; Python-only; less control than graph-based alternatives |
| Data Sovereignty | Not run | Open source; self-hostable; OpenAI API dependency for LLM calls |

## Ops reality

- **Setup**: Install via pip (`pip install agency-swarm`), define agents with natural language role descriptions, assemble into crews. Setup is reportedly fast due to the intuitive organizational metaphor.
- **Role-playing overhead**: The framework's core abstraction is that agents "stay in character" as their role (CEO, Developer, etc.). This requires extra LLM calls to maintain the persona, adding latency and token cost. For latency-sensitive systems, this overhead matters.
- **Debugging opacity**: Multi-agent interactions are hard to trace. When Agent A delegates to Agent B, the reasoning chain is not always transparent. The framework provides some logging but the mental model of "why did this agent hand off to that agent" is opaque.
- **OpenAI dependency**: The framework is built on OpenAI Agents SDK, so it requires OpenAI API access. While the OpenAI Agents SDK supports 100+ non-OpenAI models (as of April 2026), Agency Swarm's integration with these models is unproven.
- **Python only**: No TypeScript/JavaScript SDK. Teams with full-stack JS/TS stacks will need to bridge to Python.
- **CrewAI comparison**: Agency Swarm is similar in concept to CrewAI but smaller in community (4,100 stars vs. CrewAI's 52,400). CrewAI has native MCP and A2A support; Agency Swarm inherits MCP support from OpenAI Agents SDK but A2A support is unclear.
- **Version 1.8.0**: Latest release (February 2026), requires Python 3.12+. Active development but smaller community than major frameworks.

## Cost model

- **Framework**: Open source (MIT) — free
- **OpenAI API**: Token costs at OpenAI rates (or other model provider rates if using non-OpenAI models)
- **No managed tier**: Self-hosted; you bring your own compute and API keys
- **Role-playing overhead**: Budget for 10-30% more LLM calls due to persona maintenance prompts

## When to use / when to avoid

**Use when:**
- You want a lightweight, role-based multi-agent framework for Python
- The organizational metaphor (CEO, Developer, Researcher) maps naturally to your problem
- You are already using OpenAI Agents SDK and want a higher-level crew abstraction
- You need rapid prototyping of multi-agent workflows without complex graph orchestration

**Avoid when:**
- You need latency-sensitive systems (role-playing overhead adds LLM calls)
- You need fine-grained control over execution flow (graph-based frameworks like LangGraph or Microsoft Agent Framework are better)
- You are building in JavaScript/TypeScript (Python-only)
- You need production-grade observability and debugging (debugging is reportedly opaque)
- You need the largest community and ecosystem (CrewAI is significantly larger)
