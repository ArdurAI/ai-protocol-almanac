# Universal Commerce Protocol (UCP) — Deep Reference

The **Universal Commerce Protocol (UCP)** is an open protocol developed by **Google** and **Shopify** for agent-driven commerce. It enables AI agents to discover products, negotiate terms, and complete purchases autonomously across retail platforms.

## Spec Summary

| Attribute | Value |
|-----------|-------|
| **Latest spec** | UCP v1.0 (2025) |
| **Governance** | Google + Shopify (co-developed); no independent standards body yet |
| **License** | Apache 2.0 |
| **Transport** | HTTP, MCP-compatible (stdio/HTTP), A2A-compatible |
| **Serialization** | JSON |
| **Primary use case** | Agent-driven commerce: product discovery → checkout |
| **Retail partners** | 20+ (as of June 2026) |
| **Status** | Early production; growing adoption |

## Version History

| Version | Date | Notes |
|---------|------|-------|
| UCP v0.1 | Mid-2025 | Google + Shopify internal collaboration; commerce API for agents |
| UCP v1.0 | Late 2025 | Public release; MCP and A2A compatibility layers; 20+ retail partners |
| UCP v1.1 (expected) | Q3 2026 | Planned: payment integration (x402), return handling, subscription commerce |

## Transport & Schema Details

### Core Protocol

UCP is designed as a **layered protocol** that can operate over multiple transports:

| Transport Mode | Use case | Notes |
|---------------|----------|-------|
| **Native HTTP** | Direct UCP server communication | Full protocol; all capabilities |
| **MCP wrapper** | UCP as an MCP tool/server | Reduced capabilities; agent uses MCP to access UCP |
| **A2A wrapper** | UCP as an A2A skill | Agent-to-agent commerce negotiation |

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Merchant** | A retailer or seller offering products via UCP |
| **Product Catalog** | JSON-LD structured product data; discoverable via UCP endpoints |
| **Cart** | Agent-managed shopping cart; can be modified across multiple merchants |
| **Checkout** | Finalization of purchase; payment, shipping, confirmation |
| **Order** | Post-checkout entity; tracking, status, returns |
| **Consent** | Agent must obtain user consent before purchase; UCP defines consent flows |

### API Surface (Native HTTP)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/catalog/search` | POST | Search products by query, filters, merchant |
| `/catalog/product/{id}` | GET | Product details, pricing, availability |
| `/cart/create` | POST | Create a new cart for the agent's user |
| `/cart/{id}/add` | POST | Add product to cart |
| `/cart/{id}/remove` | POST | Remove product from cart |
| `/cart/{id}/checkout` | POST | Initiate checkout (requires consent) |
| `/order/{id}` | GET | Order status and tracking |
| `/consent/request` | POST | Request user consent for a purchase |
| `/consent/verify` | GET | Verify consent status |

### Product Schema (JSON-LD)

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "...",
  "description": "...",
  "brand": { "@type": "Brand", "name": "..." },
  "offers": {
    "@type": "Offer",
    "price": "...",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock"
  },
  "merchant": { "@type": "Organization", "name": "..." }
}
```

### Consent Flow

```
1. Agent discovers product via /catalog/search
2. Agent adds to cart via /cart/{id}/add
3. Agent requests consent via /consent/request
4. User (human) receives notification and approves/denies
5. Agent verifies consent via /consent/verify
6. If approved, agent proceeds to checkout via /cart/{id}/checkout
7. Payment is processed (via x402, AP2, or traditional payment)
8. Order confirmation returned
```

## Known Implementations & SDKs

| SDK/Platform | Language | Maintainer | Notes |
|-------------|----------|------------|-------|
| UCP TypeScript SDK | TypeScript | Google / Shopify | Official SDK; HTTP native |
| UCP Python SDK | Python | Google / Shopify | Official SDK |
| Shopify UCP Integration | N/A | Shopify | Shopify merchants can expose catalogs via UCP |
| Google Cloud UCP | N/A | Google | GCP-hosted UCP endpoints for merchants |
| MCP→UCP Adapters | Various | Community | Wrappers that expose UCP as MCP tools |

## Interoperability Notes

### UCP + MCP
- UCP can be wrapped as an MCP server, exposing `/catalog/search` as `tools/searchProducts` and `/cart/{id}/checkout` as `tools/checkout`
- The MCP wrapper loses some fidelity (e.g., JSON-LD product metadata is flattened into tool arguments)
- MCP's OAuth 2.1 mandate aligns with UCP's consent requirements

### UCP + A2A
- A2A agents can negotiate commerce via UCP by advertising UCP skills in their Agent Cards
- An A2A agent could delegate a shopping task to a UCP-enabled agent
- The payment layer (x402, AP2, MPP) is the bridge between UCP and A2A

### UCP + x402 / AP2 / MPP
- UCP handles the commerce layer (product discovery, cart, checkout)
- Payment is handled by a separate payment protocol (x402, AP2, or MPP)
- The checkout endpoint returns a payment URI that the agent presents to the user or processes via the payment protocol
- This separation is by design: UCP doesn't mandate a specific payment method

## Compliance Checklist

| Requirement | Test Method | Common Failure Mode |
|-------------|-------------|-------------------|
| Catalog search | POST `/catalog/search` with query | Empty results for valid queries; pagination missing |
| Product details | GET `/catalog/product/{id}` | Missing JSON-LD fields; wrong schema.org types |
| Cart management | POST `/cart/create`, `/cart/{id}/add` | Cart expiration; session loss; merchant-specific cart behavior |
| Consent flow | POST `/consent/request`, GET `/consent/verify` | Consent timeout; no notification mechanism; silent approval |
| Checkout | POST `/cart/{id}/checkout` | Payment failure; no retry mechanism; order not created |
| Order tracking | GET `/order/{id}` | Order not found; status out of sync with merchant |
| JSON-LD compliance | Validate product schema against schema.org | Missing `@context`; wrong `@type`; incomplete `offers` |
| Merchant federation | Search across multiple merchants | Results de-duplication; pricing inconsistency; availability mismatch |

## Known Issues & Sharp Edges

1. **Consent mechanism gap**: The UCP consent flow is well-defined in the spec but poorly implemented in practice. Many merchants auto-approve consent without explicit user confirmation, creating security and liability risks.

2. **Payment protocol fragmentation**: UCP deliberately does not mandate a payment protocol. This means merchants may support x402, AP2, MPP, or traditional credit card payments. Agents must negotiate payment methods dynamically, which adds complexity.

3. **Merchant federation inconsistency**: Searching across multiple merchants yields inconsistent result formats. Some merchants return full JSON-LD; others return simplified JSON. The adapter must normalize.

4. **Cart state persistence**: Cart state is merchant-dependent. Some merchants persist carts for days; others expire them in minutes. The spec does not define a standard cart TTL.

5. **Return/refund ambiguity**: UCP v1.0 does not define return or refund flows. The spec focuses on purchase; post-purchase support is left to merchant-specific APIs.

6. **Privacy concerns**: Agents passing user preferences and purchase history across merchants raises privacy questions. The spec has no built-in privacy controls or data minimization requirements.

7. **Limited to retail**: UCP is designed for physical goods commerce. Digital goods, services, and subscriptions are not well-supported in v1.0.

## Spec Ambiguities

| Ambiguity | Current Adapter Behavior | Status |
|-----------|-------------------------|--------|
| Cart TTL | No standard in spec; adapter polls cart every 30 seconds | Merchant-specific |
| Payment protocol negotiation | Agent checks supported payment methods before checkout | UCP endpoint returns `supportedPaymentMethods` array |
| Consent notification mechanism | Spec says "notify user"; doesn't define how | Adapter assumes push notification or email |
| Multi-merchant cart | Spec allows one cart per merchant; federation is unclear | Adapter creates separate carts per merchant |
| Product image URLs | Optional in schema.org; some merchants omit them | Adapter handles missing images gracefully |

## Benchmark Results (2026-06 Founding Edition)

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Accuracy / Compliance** | 78% | Core API works; JSON-LD compliance varies; consent flow is inconsistent |
| **Latency** | 120ms median (search) / 500ms+ (checkout) | Search is fast; checkout adds payment latency |
| **Token Economics** | Moderate overhead | JSON-LD is verbose; product metadata is large |
| **Scale Behavior** | Untested at scale | Federation complexity grows with merchant count |
| **Ops Burden** | Medium-High | Multi-merchant integration is complex; payment protocol fragmentation |
| **Developer Experience** | Good | SDKs are well-documented; examples are clear |
| **Data Sovereignty** | Moderate | Open spec; but Google/Shopify co-development raises governance questions |

**Composite Score (2026-06):** 72/100 — Promising for commerce agents, but the consent gap and payment fragmentation are significant barriers to production adoption.

## Links

- **Spec**: https://developers.google.com/ucp (or Shopify/ Google developer documentation)
- **GitHub**: Not yet independently published (expected Q3 2026)
- **SDKs**: TypeScript, Python (Google / Shopify official)
- **Partner list**: 20+ retail partners (Google Cloud / Shopify)

## License

Content: CC BY 4.0 — share and adapt with attribution to **ArdurAI / Context Protocols, Authentication & Integration Almanac**.
