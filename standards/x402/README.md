# x402 Protocol — Deep Reference

The **x402 Protocol** (formerly the "402 Payment Protocol") is an open standard for **agent micropayments** on the internet. It repurposes HTTP 402 (Payment Required) status code to enable autonomous agents to pay for APIs, services, and resources using stablecoins and cryptocurrency.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest spec** | x402 v2.0 (2026) |
| **Governance** | Linux Foundation (donated by Coinbase, January 2026) |
| **License** | Open (permissive) |
| **Transport** | HTTP (native); HTTP 402 status code |
| **Serialization** | JSON |
| **Primary use case** | Agent micropayments; API monetization; pay-per-request |
| **Transactions processed** | 150M+ (as of June 2026) |
| **Supported chains** | 10+ (Ethereum, Base, Solana, Polygon, etc.) |
| **Status** | Production; growing adoption |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| x402 v1.0 | 2024 | Coinbase initial release; Base chain only; basic HTTP 402 flow |
| x402 v1.5 | 2025 | Multi-chain support; stablecoin expansion (USDC, USDT, DAI) |
| x402 v2.0 | January 2026 | Coinbase → Linux Foundation; 10+ chains; payment streams; subscription support |
| x402 v2.1 (expected) | Q3 2026 | Planned: A2A integration, UCP checkout bridge, fiat on-ramp |

## Transport & Schema Details

### Core Protocol

x402 is built on a simple but powerful HTTP status code mechanism:

| Step | Client Action | Server Response |
|------|---------------|-----------------|
| 1 | Client sends request to API | Server returns `402 Payment Required` with payment requirements |
| 2 | Client parses payment requirements | JSON payload: `paymentMethod`, `amount`, `currency`, `recipient` |
| 3 | Client constructs payment transaction | Signs and submits transaction on the specified blockchain |
| 4 | Client retries request with payment proof | Includes `X-402-Payment-Proof` header with transaction hash |
| 5 | Server verifies payment on-chain | Confirms transaction, grants access |
| 6 | Server returns actual response | Normal API response |

### HTTP 402 Response Schema

```json
{
  "paymentMethod": "x402",
  "version": "2.0",
  "amount": "0.001",
  "currency": "USDC",
  "chain": "base",
  "recipient": "0x1234...",
  "requiredHeaders": ["X-402-Payment-Proof"],
  "paymentUri": "https://pay.x402.org/...",
  "expiresAt": "2026-06-16T12:00:00Z"
}
```

### Payment Proof Header

```
X-402-Payment-Proof: <blockchain>:<tx_hash>:<signature>
```

Example:
```
X-402-Payment-Proof: base:0xabc123...:0xdef456...
```

### Supported Payment Methods

| Method | Description | Maturity |
|--------|-------------|----------|
| **Stablecoin (USDC)** | USDC on Base, Ethereum, Polygon | Production |
| **Stablecoin (USDT)** | USDT on Ethereum, Tron, Solana | Production |
| **Stablecoin (DAI)** | DAI on Ethereum, Polygon | Production |
| **Native ETH** | Ethereum native payments | Production |
| **SOL** | Solana native payments | Production |
| **Payment Streams** | Continuous micro-payments for long-lived sessions | Beta |
| **Subscriptions** | Pre-paid access for a time period | Beta |

### Supported Blockchains (v2.0)

| Chain | Status | Notes |
|-------|--------|-------|
| Base | Production | Coinbase L2; primary chain for x402 |
| Ethereum | Production | L1; higher gas fees |
| Solana | Production | Low fees; high throughput |
| Polygon | Production | L2; EVM-compatible |
| Arbitrum | Production | L2; low fees |
| Optimism | Production | L2; low fees |
| Tron | Beta | USDT primary chain |
| Avalanche | Beta | EVM-compatible |
| Sui | Beta | Move-based; new integration |
| Near | Beta | Rust-based; new integration |

## Known Implementations & SDKs

| SDK/Platform | Language | Maintainer | Notes |
|-------------|----------|------------|-------|
| x402 TypeScript SDK | TypeScript | Coinbase / community | Official SDK; HTTP client + server |
| x402 Python SDK | Python | Coinbase / community | Official SDK |
| x402 Go SDK | Go | Community | Server-side middleware |
| x402 Rust SDK | Rust | Community | High-performance server |
| Coinbase Agent Kit | TypeScript | Coinbase | Includes x402 payments for agents |
| Reown AppKit | Various | Reown | Wallet integration for x402 |
| A2A x402 Bridge | TypeScript | Community | A2A skill for x402 payment negotiation |

## Interoperability Notes

### x402 + UCP
- UCP checkout can trigger x402 payments for the final purchase
- The UCP `/cart/{id}/checkout` endpoint returns a payment URI that can be an x402 payment request
- This is the primary integration path for agent-driven commerce with crypto payments

### x402 + A2A
- A2A agents can negotiate payments using x402 via the Agent Card's `paymentRequirements` field
- A2A v1.1 is expected to formalize x402 integration in the Agent Card schema
- Some community implementations use custom A2A message parts for payment negotiation

### x402 + MCP
- MCP servers can be monetized via x402 by returning 402 for expensive tool calls
- The MCP `tools/call` method can trigger an x402 payment flow before executing the tool
- This enables "pay-per-inference" models where expensive LLM calls require micro-payments

### x402 + Traditional Fiat
- x402 is cryptocurrency-native. Fiat payments require on-ramp services (Coinbase, Stripe)
- The MPP (Stripe) protocol is the fiat counterpart to x402
- Some implementations support both x402 and MPP, letting the agent choose based on user preference

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| 402 response generation | Server returns 402 with valid JSON | Missing required fields; wrong currency format |
| Payment proof verification | Server verifies `X-402-Payment-Proof` | Invalid signature; wrong chain; expired transaction |
| Multi-chain support | Server accepts payments on multiple chains | Some chains not supported; gas fee estimation wrong |
| Payment stream | Continuous payments for long sessions | Stream breaks; client doesn't send proof in time |
| Subscription support | Pre-paid time-based access | Subscription expiry; renewal logic |
| Refund handling | Server processes refunds | No refund mechanism in spec; merchant-specific |
| Rate limiting | Server prevents duplicate payment proofs | Replay attacks; same proof used twice |
| Error handling | Server returns clear error for invalid payment | Generic 402 with no details; wrong HTTP status |

## Known Issues & Sharp Edges

1. **Gas fee volatility**: On-chain transaction fees vary wildly. A payment that costs $0.001 in USDC might require $0.05 in gas fees on Ethereum L1. This makes micropayments economically unviable on some chains.

2. **Transaction confirmation latency**: Blockchain confirmation times vary from seconds (Solana) to minutes (Ethereum). During confirmation, the API request is blocked. This adds latency that HTTP clients may not tolerate.

3. **No refund standard**: x402 has no standardized refund mechanism. If a payment is made but the API returns an error, getting a refund is merchant-specific and often manual.

4. **Wallet integration complexity**: Agents need access to a wallet with funds. This requires key management, which is a security risk. The spec does not define wallet integration standards.

5. **Replay attack risk**: If a server doesn't track used payment proofs, the same proof can be replayed for multiple requests. The spec warns about this but doesn't mandate a specific defense.

6. **Fiat on-ramp gap**: For users without crypto, getting funds into a wallet is a friction point. Coinbase and other on-ramps exist but add KYC/AML overhead.

7. **Tax and compliance**: Cryptocurrency payments trigger tax events in many jurisdictions. The spec does not address tax reporting, invoice generation, or compliance documentation.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| Payment expiration | `expiresAt` is optional; adapter assumes 5 minutes | Some servers don't set expiration |
| Re-request after payment | Spec says "retry with proof"; doesn't define retry count | Adapter retries 3 times with exponential backoff |
| Partial payment | Not defined; server expects exact amount | Adapter rejects partial payments |
| Payment stream interval | Not defined; adapter sends proof every 30 seconds | Community convention |
| Chain selection priority | Server suggests chain; client may choose differently | Adapter follows server suggestion |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 88% | Core 402 flow works; multi-chain support is good; payment stream is fragile |
| **Latency** | 2-30 seconds (payment) / <10ms (after payment) | Blockchain confirmation is the bottleneck |
| **Token Economics** | Very efficient | USDC payments are cheap on L2s; gas fees are the main cost |
| **Scale Behavior** | Good at scale | Stateless payment verification; servers can scale horizontally |
| **Ops Burden** | Medium | Wallet management is complex; gas fee monitoring is required |
| **Developer Experience** | Good | SDKs are simple; HTTP-native design is intuitive |
| **Data Sovereignty** | Strong | Open spec; no vendor lock-in; Linux Foundation governance |

**Composite Score (2026-06):** 80/100 — Excellent for crypto-native payments, but the fiat gap and gas fee issues limit mainstream adoption.

## Links

- **Spec**: https://x402.org/specification (or Coinbase developer documentation)
- **GitHub**: https://github.com/coinbase/x402
- **SDKs**: TypeScript, Python, Go, Rust (Coinbase / community)
- **Coinbase Agent Kit**: https://docs.cdp.coinbase.com/agentkit
- **Linux Foundation**: https://lfdecentralizedtrust.org

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
