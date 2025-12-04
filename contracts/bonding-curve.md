# BondingCurve

Handles token pricing and trading using an exponential bonding curve.

**Address**: `0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42`

## Overview

The BondingCurve contract:
- Calculates token prices based on supply sold
- Executes buy/sell transactions
- Holds tokens and ETH during bonding curve phase
- Transfers assets to DEXManager on graduation

## Pricing Formula

```
price = INITIAL_PRICE + (K × sold^1.5) / 1e18
```

| Constant | Value | Description |
|----------|-------|-------------|
| `INITIAL_PRICE` | 0.00001 ETH | Starting price per token |
| `K` | 6.42e23 | Curve steepness (calibrated for 400k = 30 ETH) |
| `MAX_BUY_AMOUNT` | 1 ETH | Maximum per transaction |

### Price Examples

| Tokens Sold | Price per Token |
|-------------|-----------------|
| 0 | 0.00001 ETH |
| 100,000 | ~0.00002 ETH |
| 200,000 | ~0.00004 ETH |
| 300,000 | ~0.00006 ETH |
| 400,000 | ~0.000075 ETH |

## Functions

### buyWithETH

Called by Factory to execute a buy order.

```solidity
function buyWithETH(
    address _token,
    address _buyer,
    uint256 _minTokensOut
) external payable onlyFactory returns (uint256 tokensOut, uint256 ethSpent)
```

**Parameters:**
- `_token` - Token to buy
- `_buyer` - Address receiving tokens
- `_minTokensOut` - Minimum tokens (slippage protection)

**Returns:**
- `tokensOut` - Actual tokens purchased
- `ethSpent` - ETH spent (excluding fees)

---

### sellToken

Called by Factory to execute a sell order.

```solidity
function sellToken(
    address _token,
    uint256 _amount,
    address _seller,
    uint256 _minEthOut
) external onlyFactory returns (uint256 price, uint256 fee)
```

**Parameters:**
- `_token` - Token to sell
- `_amount` - Amount of tokens
- `_seller` - Address selling
- `_minEthOut` - Minimum ETH (slippage protection)

**Returns:**
- `price` - Gross ETH value
- `fee` - Fee deducted

---

### initializeToken

Initialize trading data for a new token.

```solidity
function initializeToken(address _token) external onlyFactory
```

---

### closeTrading

Close trading for a token (before graduation).

```solidity
function closeTrading(address _token) external onlyFactoryOrOwner
```

---

### withdrawForGraduation

Withdraw tokens and ETH for DEX graduation.

```solidity
function withdrawForGraduation(address _token) external onlyFactory 
    returns (uint256 remainingTokens, uint256 raisedETH)
```

---

## View Functions

### getBuyPrice

Get price quote for buying tokens.

```solidity
function getBuyPrice(address _token, uint256 _amount) external view 
    returns (uint256 price, uint256 fee)
```

### getSellPrice

Get price quote for selling tokens.

```solidity
function getSellPrice(address _token, uint256 _amount) external view 
    returns (uint256 price, uint256 fee)
```

### getCurrentPrice

Get current spot price.

```solidity
function getCurrentPrice(address _token) external view returns (uint256)
```

### estimateTokensForETH

**Frontend helper** - Estimate tokens for a given ETH amount.

```solidity
function estimateTokensForETH(address _token, uint256 _ethAmount) external view 
    returns (uint256 tokensOut)
```

### getTokenStats

```solidity
function getTokenStats(address _token) external view returns (
    uint256 totalSold,
    uint256 totalRaised,
    uint256 currentPrice,
    bool isOpen
)
```

### getTokenTradingData

```solidity
function getTokenTradingData(address _token) external view returns (
    uint256 totalSold,
    uint256 totalRaised,
    bool isOpen,
    uint256 createdAt,
    uint256 currentPrice,
    uint256 buyPrice,
    uint256 sellPrice
)
```

## Sell Price Discount

Selling price is **5% lower** than buying price to prevent arbitrage:

```solidity
sellPrice = buyPrice * 95 / 100
```
