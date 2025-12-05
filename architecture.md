# Architecture

## Why This Architecture?

### The Problem with Traditional Token Launches

Traditional launchpads deploy a new ERC-20 contract per token. This has significant drawbacks:

1. **Gas Waste** - Deploying a full ERC-20 costs ~1-2M gas per token
2. **Chain Bloat** - Thousands of abandoned tokens pollute blockchain state forever
3. **Redundant Code** - Every token deploys identical bytecode
4. **Fragmented Liquidity** - Each token exists in isolation

### Our Solution: ERC-6909 Multi-Token Standard

Instead of deploying separate contracts, we use **ERC-6909** - a multi-token standard where all tokens exist within a single contract:

```
Traditional Approach:              Our Approach:
┌─────────────┐                   ┌─────────────────────────────┐
│ Token A     │ (separate)        │      Diamond Contract       │
│ ERC-20      │                   │  ┌─────┬─────┬─────┬─────┐  │
└─────────────┘                   │  │ ID:1│ ID:2│ ID:3│ ... │  │
┌─────────────┐                   │  │TokenA│TokenB│TokenC│    │  │
│ Token B     │ (separate)        │  └─────┴─────┴─────┴─────┘  │
│ ERC-20      │                   │      (single contract)      │
└─────────────┘                   └─────────────────────────────┘
```

**Benefits:**
- **90%+ Gas Savings** - Creating a new token is just a storage write (~50k gas vs 1-2M)
- **No Chain Bloat** - One contract holds unlimited tokens
- **Shared Infrastructure** - Trading, fees, graduation logic shared across all tokens
- **Upgradeable** - Diamond pattern allows adding features without migration

### ERC-20 Compatibility via Wrappers

ERC-6909 tokens aren't directly compatible with Prism-Dex(UniV3) and other DeFi protocols. We solve this with **minimal proxy wrappers** (EIP-1167):

```
User wants to trade on Prism-Dex(UniV3)
            │
            ▼
┌─────────────────────────┐
│   ERC-20 Wrapper        │  ◄── Thin proxy (~45 bytes)
│   (Minimal Proxy)       │
└───────────┬─────────────┘
            │ delegates to
            ▼
┌─────────────────────────┐
│   Diamond Contract      │  ◄── Actual token logic
│   (ERC-6909 storage)    │
└─────────────────────────┘
```

Each wrapper is only ~45 bytes of bytecode (vs ~5KB for a full ERC-20). **For users and token holders, nothing changes** - every token has a standard ERC-20 address that works everywhere.

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Diamond Proxy                          │
│                 (Single Entry Point)                        │
├─────────────────────────────────────────────────────────────┤
│  TokenFacet    │  TradingFacet   │  GraduationFacet        │
│  FeeFacet      │  SecurityFacet  │  AdminFacet             │
│  ERC6909Facet  │  WrapperFacet   │  DiamondLoupe/Cut       │
├─────────────────────────────────────────────────────────────┤
│                   Shared Storage (Libraries)                │
│  LibToken │ LibTrading │ LibFee │ LibSecurity │ LibDEX     │
└─────────────────────────────────────────────────────────────┘
```

### Facets

| Facet | Purpose |
|-------|---------|
| **DiamondCutFacet** | Add, replace, or remove facets |
| **DiamondLoupeFacet** | Introspection (query facets, selectors) |
| **TokenFacet** | Token creation, ERC-6909 transfers, metadata |
| **TradingFacet** | Bonding curve buy/sell, price calculations |
| **GraduationFacet** | Prism-Dex(UniV3) pool creation, LP fee collection |
| **FeeFacet** | Fee distribution, creator rewards, withdrawals |
| **SecurityFacet** | Anti-sniper protection, fair launch limits |
| **AdminFacet** | Owner controls, emergency functions |
| **ERC6909Facet** | Extended token metadata, graduation status |
| **WrapperFacet** | ERC-20 wrapper deployment and management |

## Token Lifecycle

### Phase 1: Creation

When a user creates a token:
1. Creator calls `createToken()` with metadata, fee configuration, and optional fair launch settings
2. Diamond mints **1 million ERC-6909 tokens** internally (just storage writes, very cheap)
3. ERC-20 wrapper deployed instantly via minimal proxy (EIP-1167)
4. All 1M tokens deposited to wrapper, held by Diamond for bonding curve sale
5. Admin must call `initializeToken(wrapper)` to open trading
6. Token is now live and tradeable

### Phase 2: Bonding Curve Trading

- Users call `buyWithETH()` to purchase tokens (max 1 ETH per tx)
- Price follows x^1.5 curve: 0.00001 ETH → 0.0000946 ETH (9.5x increase)
- 1.2% fee deducted: 0.2% platform (fixed) + 1.0% split between creator, Bad Bunnz, buyback (configurable at creation)
- **Sells are disabled by default** - admin must call `setSellsEnabled(token, true)`
- **684,000 tokens** available for bonding curve sale
- Trading continues until **30 ETH raised** OR all 684k tokens sold

### Phase 3: Graduation

Triggered automatically when 30 ETH raised or 684k tokens sold:
1. Bonding curve trading closes (`isOpen = false`)
2. 0.1 ETH graduation fee deducted
3. Calculate tokens needed for DEX to match final BC price (~316k tokens)
4. Burn excess tokens to dead address (maintains price continuity)
5. Create Prism-Dex(UniV3) pool with ~316k tokens + ~29.9 ETH
6. Full-range liquidity position minted at 0.3% fee tier
7. Position NFT held by Diamond contract (liquidity locked forever)
8. Token wrapper now tradeable on DEX at same price as final BC price (~0.03% difference)

### Phase 4: DEX Trading

- Token trades freely on Prism-Dex(UniV3) - no restrictions
- LP fees accumulate in the position
- Anyone can call `collectFees(token)` to harvest LP fees
- Fees auto-distribute: 20% platform (fixed), 80% adjustable (default: 50% creator, 25% Bad Bunnz, 25% buyback)
- Creator calls `claimCreatorRewards()` anytime to withdraw their share

## Contract Interactions

```
User                    Diamond                         Prism-Dex(UniV3)
 │                        │                                   │
 │── createToken() ──────►│                                   │
 │◄── (tokenId, wrapper) ─│                                   │
 │                        │                                   │
 │── buyWithETH() ───────►│                                   │
 │◄── tokens ─────────────│                                   │
 │                        │                                   │
 │     [30 ETH Raised - Auto Graduation]                      │
 │                        │── createPool() ──────────────────►│
 │                        │◄── pool address ─────────────────│
 │                        │── mintPosition() ────────────────►│
 │                        │◄── positionId ───────────────────│
 │                        │                                   │
 │── collectFees() ──────►│── collect() ─────────────────────►│
 │                        │◄── fees ─────────────────────────│
 │◄── (auto-distributed) ─│                                   │
```

## Fee Flow

### Bonding Curve Fees
```
User buys/sells
      │
      ▼
1.2% fee deducted
      │
      ├──► 0.2% Platform (fixed)
      │
      └──► 1.0% Adjustable:
              ├──► Creator rewards (claimable)
              ├──► Bad Bunnz
              └──► Buyback
```

### DEX LP Fees
```
Trades on Prism-Dex(UniV3)
      │
      ▼
Anyone calls collectFees(token)
      │
      ▼
Auto-distributed:
      ├──► 50% Creator rewards (claimable)
      ├──► 30% Platform
      ├──► 10% Bad Bunnz
      └──► 10% Buyback
```
