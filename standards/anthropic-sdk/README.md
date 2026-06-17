# Anthropic SDK — Deep Reference

The **Anthropic SDK** is the official SDK for building applications with Claude, Anthropic's family of AI models. It provides access to Claude's unique capabilities including extended thinking, prompt caching, computer use tools, and multi-modal understanding.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest version** | Anthropic SDK 0.40+ (2025-2026) |
| **Governance** | Anthropic (commercial) |
| **License** | Open Source (MIT) |
| **Language** | Python / TypeScript |
| **Primary use case** | Claude-native app development; reasoning-intensive tasks; multi-modal applications; agent development |
| **Status** | Production; actively developed; Anthropic's recommended SDK |
| **Related standards** | MCP, OpenTelemetry GenAI, Anthropic API |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| Anthropic API v1 | 2023 | Initial API; Claude 1.0/2.0; basic chat completion |
| Anthropic SDK 0.20 | 2024 | Claude 3.0; tool use; vision; system prompts |
| Anthropic SDK 0.30 | 2024 | Claude 3.5 Sonnet; computer use; extended thinking (beta) |
| Anthropic SDK 0.40 | 2025 | Claude 3.7 Sonnet; prompt caching (GA); extended thinking (GA); MCP support |
| Anthropic SDK 0.50 (expected) | Q3 2026 | Planned: A2A integration, advanced agent framework, payment hooks |

## Architecture & API Surface

### Core API

The Anthropic SDK provides direct access to Claude's capabilities:

| Feature | Description | Model Support |
|---------|-------------|---------------|
| **Chat completion** | Standard message-based completion | All models |
| **Tool use** | Function calling with JSON schemas | Claude 3.0+ |
| **Vision** | Image understanding and analysis | Claude 3.0+ |
| **Extended thinking** | Chain-of-thought reasoning with visible reasoning steps | Claude 3.7 Sonnet |
| **Prompt caching** | Cache long prompts to reduce latency and cost | Claude 3.5+ |
| **Computer use** | Control a computer (mouse, keyboard, screenshots) | Claude 3.5 Sonnet (computer use) |
| **PDF support** | Native PDF understanding | Claude 3.5+ |
| **Batch processing** | Process large batches of requests asynchronously | All models |
| **Streaming** | Real-time token streaming | All models |

### Python API

```python
from anthropic import Anthropic

client = Anthropic(api_key="...")

# Basic chat
message = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Explain quantum computing"}
    ]
)

# Extended thinking
message = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    thinking={
        "type": "enabled",
        "budget_tokens": 1024
    },
    messages=[
        {"role": "user", "content": "Solve this complex math problem..."}
    ]
)

# Tool use
message = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    tools=[
        {
            "name": "get_weather",
            "description": "Get weather for a location",
            "input_schema": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                }
            }
        }
    ],
    messages=[
        {"role": "user", "content": "What's the weather in San Francisco?"}
    ]
)
```

### TypeScript API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({ apiKey: '...' });

const message = await client.messages.create({
  model: 'claude-3-7-sonnet-20250219',
  max_tokens: 1024,
  messages: [
    { role: 'user', content: 'Explain quantum computing' }
  ]
});
```

### Prompt Caching

Prompt caching allows reusing long context windows across multiple requests:

```python
# Cache a long system prompt
message = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "<very long system prompt>",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": "Question 1"}
    ]
)

# Subsequent requests reuse the cached prompt
message2 = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "<very long system prompt>",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": "Question 2"}
    ]
)
```

**Caching benefits**:
- 90% cost reduction on cached tokens
- 50%+ latency reduction on subsequent requests
- Cache lifetime: ~5 minutes of inactivity

### Computer Use

Claude can control a computer by taking screenshots, moving the mouse, and typing:

```python
from anthropic import Anthropic

client = Anthropic()

# Computer use requires a specific model and tool configuration
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=4096,
    tools=[
        {
            "type": "computer_20241022",
            "name": "computer",
            "display_width_px": 1024,
            "display_height_px": 768,
            "display_number": 1
        }
    ],
    messages=[
        {"role": "user", "content": "Open Chrome and search for 'laptops'"}
    ]
)
```

## Known Implementations & Integrations

| Integration | Description |
|-------------|-------------|
| **MCP** | Anthropic developed MCP; the SDK has first-class MCP support |
| **Claude Desktop** | Desktop app with built-in MCP server integration |
| **Claude Code** | CLI coding assistant with agent capabilities |
| **OpenTelemetry** | Built-in tracing with OpenTelemetry export |
| **LangChain** | Community integration; LangChain adapter for Claude |
| **CrewAI** | Community integration; CrewAI can use Claude as a crew member |
| **Vercel AI SDK** | Provider package for multi-provider apps |
| **Streamlit** | Community integration; UI for Claude apps |
| **Gradio** | Community integration; UI for Claude apps |

## Interoperability Notes

### Anthropic SDK + MCP
- Anthropic created MCP; the SDK has the most mature MCP integration
- Claude Desktop can connect to any MCP server via stdio or HTTP
- The SDK supports both MCP client and server roles
- MCP tool results are seamlessly integrated into Claude's reasoning

### Anthropic SDK + A2A
- A2A integration is available via community adapters
- Claude agents can advertise their capabilities as A2A Agent Cards
- Anthropic has not officially endorsed A2A but the community has built bridges

### Anthropic SDK + OpenTelemetry GenAI
- Built-in tracing exports OpenTelemetry-compatible traces
- All Claude interactions are instrumented with `gen_ai.*` semantic conventions
- Traces include thinking steps, tool calls, and token usage

### Anthropic SDK + x402 / UCP
- The SDK does not have built-in payment protocol integration
- Developers can implement x402 or UCP payments as custom tools
- Anthropic's focus is on reasoning and safety, not commerce

### Anthropic SDK + Extended Thinking
- Extended thinking is a Claude-specific feature; not available in other models
- The thinking process is visible in the API response, allowing developers to inspect Claude's reasoning
- This is unique among major LLM providers and is a key differentiator

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Chat completion | Generate text from a prompt | API error; model not available; rate limit exceeded |
| Tool use | Execute tool and return result | Tool schema mismatch; wrong argument types; tool execution failure |
| Vision | Analyze an image | Image format not supported; image too large; vision not enabled |
| Extended thinking | Enable thinking and receive reasoning | Thinking budget exceeded; thinking not enabled for model; wrong model |
| Prompt caching | Cache prompt and reuse | Cache miss; cache expired; wrong cache_control format |
| Computer use | Control computer via screenshots | Computer tool not configured; wrong display dimensions; OS compatibility |
| Streaming | Stream tokens in real-time | Latency spikes; buffering; client disconnect |
| PDF support | Process a PDF document | PDF too large; format not supported; text extraction fails |
| Multi-modal | Combine text + image + PDF in one request | Token limit exceeded; format incompatibility; model limitation |

## Known Issues & Sharp Edges

1. **Anthropic-only**: The SDK is exclusively for Anthropic models. There is no multi-provider support. This is a fundamental design choice, not a limitation.

2. **Extended thinking cost**: Extended thinking significantly increases token usage and cost. The thinking tokens are billed at the same rate as output tokens. For complex reasoning tasks, costs can be 2-5x higher than standard completion.

3. **Prompt caching limitations**: Cache hits are not guaranteed. The cache expires after ~5 minutes of inactivity. If the prompt changes even slightly (e.g., different whitespace), the cache is invalidated.

4. **Computer use complexity**: Computer use requires a specific environment (display server, browser, etc.). Setting up the environment is complex and not well-documented. The feature is experimental and has security implications.

5. **Rate limiting**: Anthropic has aggressive rate limits, especially for new accounts. The SDK handles rate limit retries but developers may need to implement their own backoff strategies.

6. **No built-in agent framework**: Unlike the OpenAI Agents SDK, the Anthropic SDK does not have a built-in agent framework (handoffs, guardrails, multi-agent orchestration). Developers must build these themselves or use third-party frameworks (LangChain, CrewAI).

7. **Vendor lock-in**: The SDK is deeply tied to Anthropic's ecosystem. The unique features (extended thinking, prompt caching, computer use) are not available from other providers. Migrating away from Anthropic means losing these capabilities.

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 95% | Claude models are highly capable; SDK is reliable; all features work as documented |
| **Latency** | Provider-dependent | SDK overhead is minimal; latency is determined by Claude API; prompt caching improves this |
| **Token Economics** | High cost | Extended thinking and long contexts are expensive; prompt caching helps |
| **Scale Behavior** | Good | Stateless; batch processing available; rate limits are the main constraint |
| **Ops Burden** | Low | Easy setup; excellent documentation; strong support |
| **Developer Experience** | Excellent | Clean API; excellent docs; unique features (thinking, caching) are well-designed |
| **Data Sovereignty** | Low | Anthropic-only; no self-hosting; API-only access |

**Composite Score (2026-06):** 85/100 — The best SDK for Claude-specific capabilities. Unmatched for reasoning-intensive tasks, but the Anthropic-only limitation makes it unsuitable for multi-provider strategies.

## Links

- **Documentation**: https://docs.anthropic.com
- **GitHub (Python)**: https://github.com/anthropics/anthropic-sdk-python
- **GitHub (TypeScript)**: https://github.com/anthropics/anthropic-sdk-typescript
- **MCP**: https://modelcontextprotocol.io (Anthropic-created)
- **API Reference**: https://docs.anthropic.com/en/api/getting-started

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
