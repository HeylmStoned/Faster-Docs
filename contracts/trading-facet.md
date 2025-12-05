# TradingFacet

Handles bonding curve trading with an x^1.5 price curve.

**Facet Address**: [`0xA52978d657dE175fD4F537f5d3bfe166c40E47d8`](https://megaeth-testnet-v2.blockscout.com/address/0xA52978d657dE175fD4F537f5d3bfe166c40E47d8)

## Overview

TradingFacet handles:
- Buying tokens with ETH
- Selling tokens back to the curve (when enabled)
- Price calculations
- Sell enable/disable per token
- Fair launch fixed-price mode

## Bonding Curve Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `INITIAL_PRICE` | 0.00001 ETH | Starting price per token |
| `K` | 149585 | x^1.5 curve steepness (calibrated for price continuity) |
| `TOTAL_SUPPLY` | 1,000,000 tokens | Total supply per token |
| `TOKEN_LIMIT` | 684,000 tokens | Tokens sold via bonding curve |
| `MAX_BUY_AMOUNT` | 1 ETH | Maximum per transaction |
| `DEFAULT_ETH_TARGET` | 30 ETH | Default ETH target for graduation |

## Pricing Formula

```
price = INITIAL_PRICE + (K × sold^1.5) / 1e18
```

The K constant is calibrated so that selling 684k tokens raises ~30 ETH, with final price matching DEX opening price (~0.03% difference).

### Price Examples

| Tokens Sold | Price per Token |
|-------------|-----------------|
| 0 | 0.00001 ETH |
| 200,000 | ~0.00003 ETH |
| 400,000 | ~0.00005 ETH |
| 600,000 | ~0.00008 ETH |
| 684,000 (max) | ~0.0000946 ETH |

## Functions

### buyWithETH

Buy tokens by sending ETH.

```solidity
function buyWithETH(
    address token,
    address buyer,
    uint256 minTokensOut
) external payable returns (uint256 tokensOut, uint256 ethSpent)
```

**Parameters:**
- `token` - Token wrapper address
- `buyer` - Address receiving tokens
- `minTokensOut` - Minimum tokens (slippage protection)

**Returns:**
- `tokensOut` - Actual tokens purchased
- `ethSpent` - ETH spent (excluding fees)

**Note:** If this purchase triggers graduation (30 ETH raised), the token automatically graduates to Prism-Dex(UniV3).

---

### sellToken

Sell tokens back to the bonding curve for ETH.

```solidity
function sellToken(
    address token,
    uint256 amount,
    address seller,
    uint256 minEthOut
) external returns (uint256 ethOut)
```

**Parameters:**
- `token` - Token wrapper address
- `amount` - Amount of tokens to sell
- `seller` - Address selling
- `minEthOut` - Minimum ETH (slippage protection)

**Requirements:**
- Sells must be enabled for this token (`areSellsEnabled(token) == true`)
- User must have approved Diamond to spend their tokens

---

### setSellsEnabled

Admin function to enable/disable sells per token.

```solidity
function setSellsEnabled(address token, bool enabled) external onlyOwner
```

**Note:** Sells are **disabled by default** to prevent pump-and-dump schemes. This does NOT affect DEX trading after graduation.

---

### areSellsEnabled

Check if sells are enabled for a token.

```solidity
function areSellsEnabled(address token) external view returns (bool)
```

---

## View Functions

### getBuyPrice

Get price quote for buying tokens.

```solidity
function getBuyPrice(address token, uint256 amount) external view returns (uint256 price, uint256 fee)
```

### getSellPrice

Get price quote for selling tokens.

```solidity
function getSellPrice(address token, uint256 amount) external view returns (uint256 price, uint256 fee)
```

### getCurrentPrice

Get current spot price.

```solidity
function getCurrentPrice(address token) external view returns (uint256)
```

### estimateTokensForETH

**Frontend helper** - Estimate tokens for a given ETH amount.

```solidity
function estimateTokensForETH(address token, uint256 ethAmount) external view returns (uint256 tokensOut)
```

### getTokenStats

```solidity
function getTokenStats(address token) external view returns (
    uint256 totalSold,
    uint256 totalRaised,
    uint256 currentPrice,
    bool isOpen
)
```

---

## Events

### TokenBought

```solidity
event TokenBought(
    address indexed token,
    address indexed buyer,
    uint256 amount,
    uint256 price,
    uint256 fee
);
```

### TokenSold

```solidity
event TokenSold(
    address indexed token,
    address indexed seller,
    uint256 amount,
    uint256 price,
    uint256 fee
);
```

### SellsEnabledChanged

```solidity
event SellsEnabledChanged(
    address indexed token,
    bool enabled
);
```
