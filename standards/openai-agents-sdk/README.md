# OpenAI Agents SDK — Deep Reference

The **OpenAI Agents SDK** is the official SDK for building OpenAI-native agents. It provides a high-level framework for agent orchestration, including handoffs, guardrails, tool calling, and tracing, with 10M+ weekly npm downloads.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest version** | Agents SDK 0.1.x (2025-2026) |
| **Governance** | OpenAI (commercial) |
| **License** | Open Source (MIT) |
| **Language** | Python / TypeScript |
| **Primary use case** | OpenAI-native agent development; multi-agent orchestration; guardrails; tracing |
| **Weekly downloads** | 10M+ npm (TypeScript) / PyPI (Python) |
| **Status** | Production; actively developed; OpenAI's recommended agent framework |
| **Related standards** | MCP, A2A, OpenTelemetry GenAI, OpenAI API |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| Assistants API | 2023 | Predecessor; OpenAI's first agent API; threads, runs, messages |
| Agents SDK 0.0.1 | 2024 | Initial release; agent loops; tool calling; handoffs |
| Agents SDK 0.1.0 | 2025 | Major update; guardrails; tracing; MCP support; multi-agent orchestration |
| Agents SDK 0.1.5 | 2026 | Enhanced MCP integration; A2A support; improved guardrails |
| Agents SDK 0.2.0 (expected) | Q3 2026 | Planned: payment integration, advanced reasoning, multi-modal agents |

## Architecture & API Surface

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Agent** | A configured LLM with instructions, tools, and handoff targets |
| **Handoff** | Transfer control from one agent to another based on the conversation context |
| **Guardrail** | A safety check that runs before or during agent execution to prevent harmful outputs |
| **Tool** | A function the agent can call; can be local functions, APIs, or MCP servers |
| **Trace** | An execution trace of the agent's reasoning, tool calls, and handoffs |
| **Run** | A single execution of an agent on a task |

### Python API

```python
from agents import Agent, Runner, handoff, guardrail

# Define an agent
agent = Agent(
    name="Shopping Assistant",
    instructions="You are a helpful shopping assistant. Help users find products and complete purchases.",
    tools=[search_products, add_to_cart, checkout],
    handoffs=[checkout_agent, support_agent],
    model="gpt-4o",
)

# Run the agent
result = await Runner.run(agent, input="I need a laptop for programming")

# Access trace
print(result.trace)
```

### TypeScript API

```typescript
import { Agent, Runner } from 'openai-agents';

const agent = new Agent({
  name: 'Shopping Assistant',
  instructions: 'You are a helpful shopping assistant...',
  tools: [searchProducts, addToCart, checkout],
  handoffs: [checkoutAgent, supportAgent],
  model: 'gpt-4o',
});

const result = await Runner.run(agent, { input: 'I need a laptop for programming' });
console.log(result.trace);
```

### Handoff System

```
User: "I need a laptop for programming"
  ↓
Shopping Agent
  ├── searches products → returns results
  ├── user asks about warranty
  │     └── handoff to → Support Agent
  │           └── answers warranty question
  │           └── handoff back to → Shopping Agent
  └── user decides to buy
        └── handoff to → Checkout Agent
              └── completes purchase
              └── handoff back to → Shopping Agent
```

### Guardrails

| Guardrail Type | Description | Example |
|---------------|-------------|---------|
| **Input guardrail** | Checks user input before agent processes it | Block harmful prompts; validate input format |
| **Output guardrail** | Checks agent output before returning to user | Filter PII; check for harmful content; validate JSON |
| **Tool guardrail** | Checks tool arguments before execution | Validate API parameters; prevent SQL injection |
| **Handoff guardrail** | Checks if handoff is appropriate | Prevent handoff to wrong agent; verify context |

### MCP Integration

```python
from agents import Agent, MCPServer

# Connect to an MCP server
mcp_server = await MCPServer.connect(
    command="npx",
    args=["-y", "@modelcontextprotocol/server-filesystem"]
)

# Use MCP tools in the agent
agent = Agent(
    name="File Assistant",
    instructions="Help users manage files.",
    tools=mcp_server.tools,
    model="gpt-4o",
)
```

### Tracing

The Agents SDK has built-in tracing that captures:
- Agent reasoning steps
- Tool calls and results
- Handoff decisions
- Guardrail checks
- Token usage
- Latency metrics

Traces can be exported to OpenTelemetry-compatible backends.

## Known Implementations & Integrations

| Integration | Description |
|-------------|-------------|
| **OpenAI API** | Native integration; all OpenAI models supported |
| **MCP** | Built-in MCP client; supports stdio and HTTP transports |
| **A2A** | Community integration; A2A Agent Card support |
| **OpenTelemetry** | Built-in tracing with OpenTelemetry export |
| **LangChain** | Can be used alongside LangChain; not a replacement |
| **CrewAI** | Community integration; CrewAI can use OpenAI Agents as crew members |
| **Streamlit** | Community integration; UI for agent apps |
| **Gradio** | Community integration; UI for agent apps |

## Interoperability Notes

### OpenAI Agents SDK + MCP
- The Agents SDK has first-class MCP support (v0.1.0+)
- MCP servers can be connected and their tools are automatically made available to agents
- The SDK handles MCP initialization, tool discovery, and schema validation
- OAuth 2.1 for remote MCP servers must be configured separately

### OpenAI Agents SDK + A2A
- A2A integration is available via community adapters
- An OpenAI agent can advertise its capabilities as an A2A Agent Card
- Handoffs between OpenAI agents can be mapped to A2A task delegation
- Full A2A integration is on the roadmap for v0.2.0

### OpenAI Agents SDK + OpenTelemetry GenAI
- Built-in tracing exports OpenTelemetry-compatible traces
- All agent steps, tool calls, and handoffs are instrumented with `gen_ai.*` semantic conventions
- Traces include token usage, latency, and model information

### OpenAI Agents SDK + x402 / UCP
- The Agents SDK does not have built-in payment protocol integration
- Developers can implement x402 or UCP payments as custom tools
- A payment tool pattern is emerging in the community

### OpenAI Agents SDK + Guardrails
- The guardrail system is SDK-specific; not interoperable with other frameworks
- Guardrails run locally and don't integrate with external guardrail services (e.g., AWS Bedrock Guardrails, Azure Content Safety)
- This is a limitation for enterprises that need centralized guardrail management

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Agent execution | Run agent on a simple task | Agent loops infinitely; wrong tool selected; task not completed |
| Tool calling | Agent calls tool with correct arguments | Tool schema mismatch; wrong argument types; tool execution failure |
| Handoff | Agent hands off to another agent at the right time | Handoff too early; handoff to wrong agent; context lost |
| Guardrails | Guardrail blocks harmful input/output | Guardrail too permissive; false positives; performance impact |
| MCP integration | Connect MCP server and use its tools | MCP server not found; auth failure; tool registration error |
| Tracing | Trace is complete and accurate | Missing steps; wrong timestamps; incomplete token usage |
| Multi-agent orchestration | Multiple agents collaborate on a task | Coordination failure; deadlock; context not shared |
| Error handling | Graceful error for invalid API key | Uncaught exceptions; retry loops; no user feedback |

## Known Issues & Sharp Edges

1. **OpenAI-only**: The SDK is optimized for OpenAI models. While it can work with other providers via adapters, the experience is not first-class. Features like extended thinking, prompt caching, and tool calling may not work correctly with non-OpenAI models.

2. **Guardrail limitations**: The built-in guardrails are basic (input/output filtering). They don't integrate with enterprise guardrail services or advanced safety models. For production use, additional guardrail layers are needed.

3. **Handoff complexity**: Handoffs are powerful but can be unpredictable. The agent decides when to hand off based on its reasoning, which can be inconsistent. There's no explicit handoff trigger mechanism (e.g., "hand off when the user mentions X").

4. **State management**: The SDK doesn't have built-in state management for long-running conversations. Developers must implement their own state persistence (e.g., using a database, Redis, or file system).

5. **No built-in caching**: Like the Vercel AI SDK, there's no built-in prompt caching or response caching. This leads to redundant LLM calls and higher costs.

6. **Vendor lock-in**: The SDK is deeply tied to OpenAI's ecosystem. Using it with other providers requires adapters and may result in a degraded experience. This is a significant concern for multi-provider strategies.

7. **Python/TypeScript parity**: The Python and TypeScript SDKs have feature parity in most areas, but there are some differences in async handling, error messages, and MCP integration. The Python SDK is more mature.

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 90% | Agent execution is reliable; tool calling works well; handoffs are sometimes inconsistent |
| **Latency** | Provider-dependent | SDK overhead is minimal; latency is determined by OpenAI API |
| **Token Economics** | Minimal overhead | SDK doesn't add token overhead; cost is OpenAI-dependent |
| **Scale Behavior** | Good | Stateless; can run in serverless environments; no built-in rate limiting |
| **Ops Burden** | Low | Easy setup with `pip install` or `npm install`; good documentation |
| **Developer Experience** | Excellent | Intuitive API; good examples; strong community; OpenAI support |
| **Data Sovereignty** | Low | OpenAI-only; vendor lock-in; no self-hosting option |

**Composite Score (2026-06):** 83/100 — The best SDK for OpenAI-native agents. Excellent for OpenAI-centric stacks, but the vendor lock-in is a significant limitation for multi-provider or self-hosted strategies.

## Links

- **Documentation**: https://platform.openai.com/docs/agents
- **GitHub**: https://github.com/openai/openai-agents-python
- **PyPI**: https://pypi.org/project/openai-agents
- **NPM**: https://www.npmjs.com/package/openai-agents
- **OpenAI API**: https://platform.openai.com

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
