# FeeManager

Manages all fee collection and distribution.

**Address**: `0x194c622e62FA1C91A8A678A898F4b10E138124cD`

## Overview

FeeManager handles:
- Trading fee distribution (bonding curve)
- DEX LP fee distribution
- Graduation fee collection
- Creator reward claims

## Fee Structure

### Trading Fees (Bonding Curve)

**Total Fee**: 1.2% per trade

| Recipient | Amount | Configurable |
|-----------|--------|--------------|
| **Platform** | 0.2% (fixed) | No |
| Creator | 0.5% (default) | Yes |
| Bad Bunnz | 0.25% (default) | Yes |
| Buyback | 0.25% (default) | Yes |

The remaining 1.0% is split between Creator, Bad Bunnz, and Buyback (must sum to 100% of that 1.0%).

### Graduation Fee

**Fixed**: 0.1 ETH (deducted from raised funds, not paid by creator)

### DEX LP Fees

Configurable per token (must sum to 100%):

| Recipient | Default | Description |
|-----------|---------|-------------|
| Platform | 30% | Platform revenue |
| Creator | 50% | Token creator |
| Bad Bunnz | 10% | NFT holders |
| Buyback | 10% | Buyback fund |

## Functions

### setBondingCurveFeePercentages

Set custom fee split for bonding curve trading.

```solidity
function setBondingCurveFeePercentages(
    address _token,
    uint256 _creatorFeePercentage,
    uint256 _badBunnzFeePercentage,
    uint256 _buybackFeePercentage
) external
```

**Requirements:**
- Caller must be factory or token creator
- Percentages must sum to 100

---

### setDEXFeePercentages

Set custom fee split for DEX LP fees.

```solidity
function setDEXFeePercentages(
    address _token,
    uint256 _platformFeePercentage,
    uint256 _creatorFeePercentage,
    uint256 _badBunnzFeePercentage,
    uint256 _buybackFeePercentage
) external
```

---

### claimCreatorRewards

Creators claim their accumulated trading fee rewards.

```solidity
function claimCreatorRewards() external
```

**Note:** Transfers all pending rewards to caller.

---

## View Functions

### getCreatorRewards

```solidity
function getCreatorRewards(address _creator) external view returns (uint256)
```

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
function getFeeBreakdown(address _token) external view returns (
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee,
    uint256 totalFee
)
```

### getDEXFeeBreakdown

```solidity
function getDEXFeeBreakdown(address _token) external view returns (
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee,
    bool hasCustomFees
)
```

### getTokenBuybackFees

```solidity
function getTokenBuybackFees(address _token) external view returns (uint256)
```

---

## Admin Functions

### withdrawPlatformFees

```solidity
function withdrawPlatformFees(uint256 _amount) external onlyOwner
```

### withdrawBadBunnzFees

```solidity
function withdrawBadBunnzFees(uint256 _amount) external onlyOwner
```

### withdrawBuybackFees

```solidity
function withdrawBuybackFees(uint256 _amount) external onlyOwner
```

### withdrawGraduationFees

```solidity
function withdrawGraduationFees(uint256 _amount) external onlyOwner
```

### updateGlobalDEXFeeConfig

```solidity
function updateGlobalDEXFeeConfig(
    uint256 _platformFeePercentage,
    uint256 _creatorFeePercentage,
    uint256 _badBunnzFeePercentage,
    uint256 _buybackFeePercentage
) external onlyOwner
```
