# GraduationFacet

Graduates tokens from bonding curve to Uniswap V3 onm Prism.

**Facet Address**: [`0xe0955cB7Eb76aa7F5a070B8b2cE60fF93000821C`](https://mega.etherscan.io/address/0xe0955cB7Eb76aa7F5a070B8b2cE60fF93000821C)

## Overview

GraduationFacet handles:
- Creating Uniswap V3 onm Prism pools
- Adding initial liquidity
- Managing LP positions
- Collecting and distributing LP fees

**All functions that take a `token` parameter expect the ERC-20 wrapper address** (from `TokenCreated`).

## Uniswap V3 onm Prism Configuration

| Setting | Value |
|---------|-------|
| `POOL_FEE` | 0.3% (3000) |
| Position Range | Full range (-887220 to 887220) |
| Deadline | block.timestamp + 300 seconds |

## External Dependencies

| Contract | Address |
|----------|---------|
| UniswapV3Factory | `0xef349aa6cc5e87559e716ac293845a48cadf30d5` |
| PositionManager | `0x9feaf944c518164d5d0c45f28255758acff8e987` |
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
5. **Pool Created** - Uniswap V3 onm Prism pool initialized with ~316k tokens + ~29.9 ETH
6. **Liquidity Minted** - Full-range position created at 0.3% fee tier
7. **Position Locked** - NFT held by Diamond contract (non-withdrawable)

## Functions

### graduate

Graduate token to Uniswap V3 onm Prism. Usually auto-called when target met.

```solidity
function graduate(address token) external returns (address pool, uint256 positionId)
```

**Requirements:**
- Token must have reached target (trading closed)
- Token must not already be graduated

**Returns:**
- `pool` - Uniswap V3 onm Prism pool address
- `positionId` - NFT position ID

---

### collectFees

Collect LP fees and auto-distribute to creator.

```solidity
function collectFees(address token) external returns (uint256 amount0, uint256 amount1)
```

Anyone can call this. Fees are automatically distributed:
- **20% Platform** (fixed)
- **80% Adjustable** (default split):
  - 50% Creator rewards (claimable via `claimCreatorRewards()`)
  - 25% Bad Bunnz
  - 25% Buyback

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
    uint256 positionId
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
