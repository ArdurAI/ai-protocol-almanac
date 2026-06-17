# Microsoft Agent Framework

| Attribute | Value |
|-----------|-------|
| Tier | B |
| Type | Agent Platform |
| License | Open Source |
| Stars | ~N/A (Microsoft internal metric) |
| URL | https://github.com/microsoft/agent-framework |

## One-line summary

Microsoft's unified agent framework (AutoGen + Semantic Kernel) that brings native MCP and A2A support to .NET and Python ecosystems, targeting enterprise multi-agent orchestration on Azure.

## Architecture

The Microsoft Agent Framework merges two previously separate Microsoft agent libraries into a single coherent SDK:

- **AutoGen**: Microsoft's original multi-agent conversation framework, focused on LLM-driven agent chat and tool orchestration
- **Semantic Kernel**: Microsoft's orchestration layer for AI integrations, with connectors, planners, and memory abstractions

The unified framework provides a single SDK surface that supports both patterns. Under the hood, it uses an agent orchestration engine that handles:

- **Agent lifecycle**: Creation, activation, handoff, and termination of agent instances
- **Message routing**: Routing messages between agents with deterministic and LLM-driven routing logic
- **Tool binding**: MCP server integration as first-class tools, with schema discovery and type-safe invocation
- **A2A task delegation**: Native support for Agent Cards and task delegation across the A2A protocol
- **Azure integration**: Native hooks into Azure OpenAI Service, Azure AI Search, and Azure Functions for deployment

The framework supports both .NET (C#) and Python, with the .NET path being the primary target for enterprise Azure customers.

## Key features

- AutoGen + Semantic Kernel unified in one SDK (GA April 2026)
- Native MCP server support — MCP tools attach without adapter code
- Native A2A task delegation — agents can discover and delegate to other agents via Agent Cards
- .NET and Python SDKs with feature parity
- Azure-native deployment patterns (Azure Functions, Container Apps, AKS)
- Multi-agent conversation patterns with built-in group chat and round-robin orchestration
- LLM-driven planners that can dynamically select tools and agent paths

## Benchmark status

| Dimension | Status | Notes |
|-----------|--------|-------|
| Accuracy | Not run | No formal conformance testing conducted yet |
| Latency | Not run | MCP tool discovery overhead expected; A2A handshake latency unmeasured |
| Token Economics | Not run | Azure OpenAI token costs apply; framework overhead unmeasured |
| Scale Behavior | Not run | Enterprise claims but no independent federation stress tests |
| Ops Burden | Not run | Azure lock-in expected for full feature set; .NET setup requires Visual Studio or VS Code |
| Developer Experience | Not run | Documentation quality reportedly high; Microsoft Learn ecosystem |
| Data Sovereignty | Not run | Open source core but Azure-centric deployment patterns |

## Ops reality

- **Setup**: The .NET SDK requires the .NET 8+ runtime and Visual Studio 2022 or VS Code with the C# Dev Kit. Python SDK requires Python 3.10+.
- **Azure dependency**: Many advanced features (Azure AI Search memory, Azure Functions deployment, Azure OpenAI Service connectors) require Azure subscriptions. The framework is usable without Azure but loses significant functionality.
- **Version convergence**: Since this is a unification of two previously separate libraries, there may be migration friction from AutoGen v0.2 or Semantic Kernel v1.x to the unified framework.
- **MCP maturity**: MCP support is native but the framework is relatively new. Edge cases in MCP server compatibility (especially non-standard transports or auth schemes) may require manual intervention.
- **A2A adoption**: A2A support is present but the ecosystem of A2A agents outside Google's ecosystem is still small. Cross-vendor A2A interoperability is unproven.

## Cost model

- **Framework**: Open source (MIT license) — free to use
- **Azure OpenAI**: Token costs at Azure rates (often comparable to OpenAI API rates, sometimes lower with commitment discounts)
- **Azure infrastructure**: Azure Functions, Container Apps, or AKS costs apply for deployment
- **No managed tier**: Unlike Claude Managed Agents or OpenAI's hosted solutions, Microsoft provides the SDK but not a managed runtime — you bring your own compute

## When to use / when to avoid

**Use when:**
- You are building on Azure and need multi-agent orchestration
- You need both .NET and Python support in the same organization
- You want MCP + A2A in a single framework without multiple SDKs
- You are already invested in AutoGen or Semantic Kernel and need a migration path

**Avoid when:**
- You are not on Azure and want a cloud-agnostic framework (Vercel AI SDK or CrewAI may be better)
- You need a fully managed runtime (Microsoft only provides the SDK)
- Your team is JavaScript/TypeScript-only (the .NET/Python focus may be a mismatch)
- You need rapid prototyping without Azure setup overhead
