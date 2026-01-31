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
- **Auto-Graduation** - Tokens graduate to Uniswap V3 when 30 ETH raised
- **90%+ Gas Savings** - ~50k gas to create a token vs 1-2M traditional
- **Price Continuity** - DEX opens at same price as final bonding curve (~0.03% difference)

## Using tokens (for integrators and users)

**Treat the wrapper address as a normal ERC-20 token.** You get the wrapper address from the `TokenCreated` event when a token is created. From then on:

- **Wallets** – Add the wrapper address as a custom token (e.g. MetaMask). Balances and transfers work like any ERC-20.
- **Transfers** – Use `transfer(to, amount)` and `transferFrom(from, to, amount)` on the wrapper contract. No Diamond calls needed for simple sends.
- **Approvals** – Use `approve(spender, amount)` on the wrapper for DEXs, vaults, or other protocols.
- **DEXs / DeFi** – After graduation, use the wrapper address as the token in Uniswap V3 or any ERC-20–compatible protocol.

No special SDK or Diamond ABI is required for day-to-day token use. Use the Diamond only for launchpad actions (create, buy/sell on bonding curve, graduate, collect fees, claim rewards). See [ERC-20 Wrappers](contracts/wrappers.md) for details.

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

| For | Start here |
|-----|------------|
| **Integrate (frontend / bot)** | [Getting Started](integration/getting-started.md) → [Code Examples](integration/examples.md) |
| **Use tokens like ERC-20** | [ERC-20 Wrappers](contracts/wrappers.md) – wrapper address = token address |
| **Contract API** | [Diamond](contracts/diamond.md) → [TokenFacet](contracts/token-facet.md), [TradingFacet](contracts/trading-facet.md), etc. |
| **Addresses & ABIs** | [Deployed Addresses](reference/addresses.md) |
| **Events** | [Events](reference/events.md) |
| **Architecture** | [Architecture](architecture.md) |

## Network

**MegaETH Mainnet** (Chain ID: 4326)  
**RPC**: https://mainnet.megaeth.com/rpc  
**Block Explorer**: [mega.etherscan.io](https://mega.etherscan.io)

| Contract | Address |
|----------|---------|
| **Diamond (Main Entry)** | [`0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B`](https://mega.etherscan.io/address/0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B) |

[View all contract addresses →](reference/addresses.md)
