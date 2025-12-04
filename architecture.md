# Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      MemecoinFactory                            │
│  - Token creation                                               │
│  - Buy/Sell routing                                             │
│  - Auto-graduation to DEX                                       │
└─────────────────┬───────────────────────────────────────────────┘
                  │
    ┌─────────────┼─────────────┬─────────────┐
    ▼             ▼             ▼             ▼
┌───────────┐ ┌───────────┐ ┌───────────┐ ┌─────────────────┐
│BondingCurve│ │FeeManager │ │DEXManager │ │SecurityManager  │
│           │ │           │ │           │ │                 │
│- Pricing  │ │- Fee dist │ │- Uniswap  │ │- Fair launch    │
│- Buy/Sell │ │- Creator  │ │  V3 pools │ │- Sniper protect │
│- Trading  │ │  rewards  │ │- LP mgmt  │ │- Pause control  │
└───────────┘ └───────────┘ └───────────┘ └─────────────────┘
                  │
                  ▼
         ┌───────────────┐
         │MemecoinToken  │
         │ (ERC20)       │
         │- Metadata     │
         │- Trade stats  │
         └───────────────┘
```

## Token Lifecycle

### Phase 1: Creation

When a user creates a token:
1. `MemecoinToken` contract deployed (1M total supply)
2. 800k tokens sent to `BondingCurve`
3. Trading data initialized
4. Optional: Fair launch / sniper protection enabled

### Phase 2: Bonding Curve Trading

- Users buy/sell through `MemecoinFactory`
- Price follows exponential curve: `price = 0.00001 + (K × sold^1.5)`
- 1.2% trading fee distributed to creator, platform, buyback
- Max 1 ETH per transaction

### Phase 3: Graduation

Triggered automatically when either:
- **400,000 tokens** sold, OR
- **30 ETH** raised

Graduation process:
1. Trading closed on bonding curve
2. 0.1 ETH graduation fee deducted
3. Remaining tokens + ETH sent to `DEXManager`
4. Uniswap V3 pool created with full-range liquidity

### Phase 4: DEX Trading

- Token trades freely on Uniswap V3
- 0.3% pool fee tier
- LP fees collected and distributed via `FeeManager`

## Contract Interactions

```
User                Factory              BondingCurve         DEXManager
 │                    │                      │                    │
 │── createToken() ──►│                      │                    │
 │                    │── initializeToken() ─►│                    │
 │                    │◄─────────────────────│                    │
 │◄── token address ──│                      │                    │
 │                    │                      │                    │
 │── buyWithETH() ───►│                      │                    │
 │                    │── buyWithETH() ─────►│                    │
 │                    │◄── tokens, ethSpent ─│                    │
 │◄── tokens ─────────│                      │                    │
 │                    │                      │                    │
 │     [Target Reached - Auto Graduation]    │                    │
 │                    │── closeTrading() ───►│                    │
 │                    │── withdrawForGrad() ─►│                    │
 │                    │◄── tokens, ETH ──────│                    │
 │                    │── createPool() ──────────────────────────►│
 │                    │◄── pool address ─────────────────────────│
```

## Fee Flow

```
Trading Fee (1.2%)
       │
       ▼
┌──────────────────┐
│   FeeManager     │
└──────┬───────────┘
       │
       ├──► Creator (configurable, default 50%)
       ├──► Bad Bunnz (configurable, default 25%)
       └──► Buyback (configurable, default 25%)
```
