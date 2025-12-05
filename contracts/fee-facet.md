# FeeFacet

Manages all fee collection and distribution.

**Facet Address**: [`0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb`](https://megaeth-testnet-v2.blockscout.com/address/0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb)

## Overview

FeeFacet handles:
- Trading fee distribution (bonding curve)
- DEX LP fee distribution
- Graduation fee collection
- Creator reward claims

## Fee Structure

### Trading Fees (Bonding Curve)

**Total Fee**: 1.2% per trade (`TOTAL_TRADING_FEE = 120` basis points)

| Recipient | Amount | Configurable |
|-----------|--------|--------------|
| **Platform** | 0.2% (`PLATFORM_FEE = 20` bps) | No |
| Creator | 0.5% (default 50% of adjustable) | Yes |
| Bad Bunnz | 0.25% (default 25% of adjustable) | Yes |
| Buyback | 0.25% (default 25% of adjustable) | Yes |

The adjustable portion is 1.0% (`ADJUSTABLE_FEE = 100` bps), split between Creator, Bad Bunnz, and Buyback (percentages must sum to 100).

### Graduation Fee

**Fixed**: 0.1 ETH (deducted from raised funds, not paid by creator)

### DEX LP Fees

**Platform Fee**: 20% (fixed, not configurable)

The remaining 80% is adjustable and split between Creator, Bad Bunnz, and Buyback:

| Recipient | Default Split | Effective Share |
|-----------|---------------|-----------------|
| **Platform** | 20% (fixed) | 20% |
| Creator | 50% of 80% | 40% |
| Bad Bunnz | 25% of 80% | 20% |
| Buyback | 25% of 80% | 20% |

The adjustable portion percentages (Creator, Bad Bunnz, Buyback) must sum to 100. Platform fee is automatically applied on top.

## Functions

### claimCreatorRewards

Creators claim their accumulated trading fee rewards.

```solidity
function claimCreatorRewards() external
```

**Note:** Transfers all pending rewards (from both bonding curve and DEX LP fees) to caller.

---

### getCreatorRewards

Check claimable balance for a creator.

```solidity
function getCreatorRewards(address creator) external view returns (uint256)
```

---

## View Functions

### getTradingFeePercentage

```solidity
function getTradingFeePercentage() external pure returns (uint256)
```

Returns `120` (1.2% as basis points × 100).

### getGraduationFee

```solidity
function getGraduationFee() external pure returns (uint256)
```

Returns `0.1 ether`.

### getPlatformStats

```solidity
function getPlatformStats() external view returns (
    uint256 platformFees,
    uint256 badBunnzFees,
    uint256 buybackFees,
    uint256 graduationFees,
    uint256 creatorFees
)
```

### getFeeBreakdown

```solidity
function getFeeBreakdown(address token) external view returns (
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee,
    uint256 totalFee
)
```

---

## Admin Functions

### withdrawPlatformFees

```solidity
function withdrawPlatformFees(uint256 amount) external onlyOwner
```

### withdrawBadBunnzFees

```solidity
function withdrawBadBunnzFees(uint256 amount) external onlyOwner
```

### withdrawBuybackFees

```solidity
function withdrawBuybackFees(uint256 amount) external onlyOwner
```

### setFeeWallets

```solidity
function setFeeWallets(address platform, address buyback) external onlyOwner
```

---

## Events

### FeesDistributed

```solidity
event FeesDistributed(
    address indexed token,
    address indexed creator,
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee
);
```

### CreatorRewardsClaimed

```solidity
event CreatorRewardsClaimed(
    address indexed creator,
    uint256 amount
);
```

### PlatformFeesWithdrawn

```solidity
event PlatformFeesWithdrawn(uint256 amount);
```
