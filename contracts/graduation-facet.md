# GraduationFacet

Graduates tokens from bonding curve to Prism-Dex(UniV3).

**Facet Address**: [`0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE`](https://megaeth-testnet-v2.blockscout.com/address/0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE)

## Overview

GraduationFacet handles:
- Creating Prism-Dex(UniV3) pools
- Adding initial liquidity
- Managing LP positions
- Collecting and distributing LP fees

## Prism-Dex(UniV3) Configuration

| Setting | Value |
|---------|-------|
| `POOL_FEE` | 0.3% (3000) |
| Position Range | Full range (-887220 to 887220) |
| Deadline | block.timestamp + 300 seconds |

## External Dependencies

| Contract | Address |
|----------|---------|
| Prism-Dex(UniV3) Factory | `0x94996d371622304f2eb85df1eb7f328f7b317c3e` |
| Position Manager | `0x1279f3cbf01ad4f0cfa93f233464581f4051033a` |
| WETH | `0x4200000000000000000000000000000000000006` |

## Graduation Process (Price Continuity)

The bonding curve constant K is calibrated so that selling 684k tokens raises ~30 ETH:
- Final BC price ≈ 0.0000946 ETH
- DEX price = (30 ETH - 0.1 fee) / 316k tokens ≈ 0.0000946 ETH
- **Price difference: ~0.03%** (essentially zero)

When 30 ETH raised OR 684k tokens sold:

1. **Trading Closes** - Bonding curve trading disabled (`isOpen = false`)
2. **Fee Deducted** - 0.1 ETH graduation fee taken
3. **Calculate DEX Tokens** - Tokens needed to match final BC price (~316k)
4. **Burn Excess** - Remaining tokens burned to `0x...dEaD` (only if >1% of remaining)
5. **Pool Created** - Prism-Dex(UniV3) pool initialized with ~316k tokens + ~29.9 ETH
6. **Liquidity Minted** - Full-range position created at 0.3% fee tier
7. **Position Locked** - NFT held by Diamond contract (non-withdrawable)

## Functions

### graduate

Graduate token to Prism-Dex(UniV3). Usually auto-called when target met.

```solidity
function graduate(address token) external returns (address pool, uint256 positionId)
```

**Requirements:**
- Token must have reached target (trading closed)
- Token must not already be graduated

**Returns:**
- `pool` - Prism-Dex(UniV3) pool address
- `positionId` - NFT position ID

---

### collectFees

Collect LP fees and auto-distribute to creator.

```solidity
function collectFees(address token) external returns (uint256 amount0, uint256 amount1)
```

Anyone can call this. Fees are automatically distributed:
- 50% Creator rewards (claimable via `claimCreatorRewards()`)
- 30% Platform
- 10% Bad Bunnz
- 10% Buyback

---

### isTokenGraduated

Check if a token has graduated.

```solidity
function isTokenGraduated(address token) external view returns (bool)
```

---

## View Functions

### getPoolAddress

```solidity
function getPoolAddress(address token) external view returns (address)
```

### getPositionDetails

```solidity
function getPositionDetails(address token) external view returns (
    uint256 positionId,
    uint128 liquidity,
    uint256 tokensOwed0,
    uint256 tokensOwed1
)
```

### getGraduationStatus

```solidity
function getGraduationStatus(address token) external view returns (
    bool graduated,
    address pool,
    uint256 positionId,
    uint128 liquidity
)
```

---

## Price Initialization

When creating a new pool, the initial price is calculated from deposited amounts:

```solidity
price = (ethAmount × 1e18) / tokenAmount
sqrtPriceX96 = sqrt(price × 2^192)
```

This ensures the pool starts at a fair price based on the bonding curve's final state.

## Liquidity Position

The LP position is:
- **Full range** (-887272 to 887272 ticks)
- **Owned by Diamond** contract
- **Non-withdrawable** (liquidity is locked forever)
- **Fee-generating** (0.3% on all swaps)

---

## Events

### TokenGraduated

```solidity
event TokenGraduated(
    address indexed token,
    address indexed pool,
    uint256 positionId,
    uint256 tokenAmount,
    uint256 ethAmount
);
```

### FeesCollected

```solidity
event FeesCollected(
    address indexed token,
    uint256 amount0,
    uint256 amount1
);
```

### PoolCreated

```solidity
event PoolCreated(
    address indexed token,
    address indexed pool
);
```
