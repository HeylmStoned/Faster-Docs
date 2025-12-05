# Faster Launchpad

Welcome to the **Faster Launchpad** documentation - a gas-efficient memecoin launchpad built on MegaETH using the EIP-2535 Diamond Standard and ERC-6909 multi-token architecture.

## What is Faster Launchpad?

A gas-efficient memecoin launchpad built on the EIP-2535 Diamond Standard with ERC-6909 multi-tokens.

## Quick Stats

| Parameter | Value |
|-----------|-------|
| **Total Supply** | 1,000,000 tokens (fixed) |
| **Bonding Curve Sale** | 684,000 tokens (68.4%) |
| **DEX Liquidity** | 316,000 tokens (31.6%) |
| **Initial Price** | 0.00001 ETH |
| **Final Price** | ~0.0000946 ETH (9.5x) |
| **Graduation Target** | 30 ETH raised |
| **Trading Fee** | 1.2% |
| **Graduation Fee** | 0.1 ETH |
| **Max Buy per TX** | 1 ETH |

## Key Features

- **ERC-6909 Multi-Token Standard** - All tokens exist within a single Diamond contract
- **ERC-20 Wrappers** - Each token gets a compatible ERC-20 address via minimal proxy
- **Bonding Curve Trading** - x^1.5 price curve with 1.2% trading fee
- **Auto-Graduation** - Tokens graduate to Prism-Dex(UniV3) when 30 ETH raised
- **90%+ Gas Savings** - ~50k gas to create a token vs 1-2M traditional
- **Price Continuity** - DEX opens at same price as final bonding curve (~0.03% difference)

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Diamond Proxy                          │
│                 (Single Entry Point)                        │
├─────────────────────────────────────────────────────────────┤
│  TokenFacet    │  TradingFacet   │  GraduationFacet        │
│  FeeFacet      │  SecurityFacet  │  AdminFacet             │
│  ERC6909Facet  │  WrapperFacet   │  DiamondLoupe/Cut       │
└─────────────────────────────────────────────────────────────┘
```

## Quick Links

- [Architecture Overview](architecture.md)
- [Getting Started](integration/getting-started.md)
- [Contract Reference](contracts/diamond.md)
- [Deployed Addresses](reference/addresses.md)

## Network

**MegaETH Testnet v2** (Chain ID: 6343)  
**RPC**: https://timothy.megaeth.com/rpc  
**Block Explorer**: [megaeth-testnet-v2.blockscout.com](https://megaeth-testnet-v2.blockscout.com)

| Contract | Address |
|----------|---------|
| **Diamond (Main Entry)** | [`0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e`](https://megaeth-testnet-v2.blockscout.com/address/0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e) |

[View all contract addresses →](reference/addresses.md)
