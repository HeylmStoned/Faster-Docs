# DEXManager

Handles Uniswap V3 integration for graduated tokens.

**Address**: `0x98e57d776226cE97C0c850E2Af33669aa9F99416`

## Overview

DEXManager handles:
- Creating Uniswap V3 pools
- Adding initial liquidity
- Managing LP positions
- Collecting and distributing LP fees

## Uniswap V3 Configuration

| Setting | Value |
|---------|-------|
| Pool Fee Tier | 0.3% (3000) |
| Position Range | Full range (-887272 to 887272) |
| Slippage Protection | 2% |

## External Dependencies

| Contract | Address |
|----------|---------|
| Uniswap V3 Factory | `0x94996d371622304f2eb85df1eb7f328f7b317c3e` |
| Position Manager | `0x1279f3cbf01ad4f0cfa93f233464581f4051033a` |
| WETH | `0x4200000000000000000000000000000000000006` |

## Functions

### createPoolAndPosition

Creates a Uniswap V3 pool and adds initial liquidity. Called by Factory during graduation.

```solidity
function createPoolAndPosition(
    address _token,
    uint256 _tokenAmount,
    uint256 _ethAmount
) external payable onlyFactory returns (address pool, uint256 positionId)
```

**Parameters:**
- `_token` - Token address
- `_tokenAmount` - Tokens to add as liquidity
- `_ethAmount` - ETH to add as liquidity

**Returns:**
- `pool` - Uniswap V3 pool address
- `positionId` - NFT position ID

**Process:**
1. Create pool if doesn't exist
2. Initialize with price based on token/ETH ratio
3. Convert ETH to WETH
4. Mint full-range liquidity position
5. Store position NFT in contract

---

### collectFees

Collect accumulated LP fees from a position.

```solidity
function collectFees(address _token) external onlyFactory 
    returns (uint256 amount0, uint256 amount1)
```

Fees are automatically distributed via FeeManager.

---

## View Functions

### getPoolAddress

```solidity
function getPoolAddress(address _token) external view returns (address)
```

### isTokenGraduated

```solidity
function isTokenGraduated(address _token) external view returns (bool)
```

### getPositionDetails

```solidity
function getPositionDetails(address _token) external view returns (
    uint256 positionId,
    uint128 liquidity,
    uint256 tokensOwed0,
    uint256 tokensOwed1
)
```

### getGraduationStatus

```solidity
function getGraduationStatus(address _token) external view returns (
    bool graduated,
    address pool,
    uint256 positionId,
    uint128 liquidity
)
```

## Price Initialization

When creating a new pool, the initial price is calculated from the deposited amounts:

```solidity
price = (ethAmount × 1e18) / tokenAmount
sqrtPriceX96 = sqrt(price × 2^192)
```

This ensures the pool starts at a fair price based on the bonding curve's final state.

## Liquidity Position

The LP position is:
- **Full range** (-887272 to 887272 ticks)
- **Owned by DEXManager** contract
- **Non-withdrawable** (liquidity is locked)
- **Fee-generating** (0.3% on all swaps)
