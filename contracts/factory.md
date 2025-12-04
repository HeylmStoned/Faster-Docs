# MemecoinFactory

The main entry point for token creation and trading.

**Address**: `0x855EF543a8d611Ee76724C2E4051E7db4947158F`

## Overview

MemecoinFactory handles:
- Creating new memecoin tokens
- Routing buy/sell transactions to BondingCurve
- Auto-graduating tokens to Uniswap V3

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `target` | 30 ETH | ETH target for graduation |
| `TOKEN_LIMIT` | 400,000 tokens | Token sales target for graduation |
| `TOTAL_SUPPLY` | 1,000,000 tokens | Total supply per token |
| `BONDING_CURVE_SUPPLY` | 800,000 tokens | Tokens allocated to bonding curve |

## Functions

### createToken

Creates a new memecoin with customizable fee structure.

```solidity
function createToken(
    string memory _name,
    string memory _symbol,
    string memory _description,
    string memory _imageUrl,
    string memory _website,
    string memory _twitter,
    string memory _telegram,
    // Bonding curve fee split (must sum to 100)
    uint256 _creatorFeePercentage,
    uint256 _badBunnzFeePercentage,
    uint256 _buybackFeePercentage,
    // DEX fee split (must sum to 100)
    uint256 _dexPlatformFeePercentage,
    uint256 _dexCreatorFeePercentage,
    uint256 _dexBadBunnzFeePercentage,
    uint256 _dexBuybackFeePercentage,
    // Fair launch (optional)
    bool _enableFairLaunch,
    uint256 _fairLaunchDuration,
    uint256 _maxPerWallet,
    uint256 _fixedPrice
) external returns (address token)
```

**Parameters:**
- `_name` - Token name (e.g., "My Memecoin")
- `_symbol` - Token symbol (e.g., "MEME")
- `_description` - Token description
- `_imageUrl` - URL to token image
- `_website` - Project website
- `_twitter` - Twitter handle
- `_telegram` - Telegram group link
- Fee percentages must sum to 100 for each category
- Fair launch params only used if `_enableFairLaunch` is true

**Returns:** Address of the newly created token

---

### buyWithETH

Buy tokens by sending ETH. Pump.fun style - you specify ETH amount, receive tokens based on current curve price.

```solidity
function buyWithETH(
    address _token,
    uint256 _minTokensOut
) external payable
```

**Parameters:**
- `_token` - Token address to buy
- `_minTokensOut` - Minimum tokens to receive (slippage protection)

**Note:** If this purchase triggers graduation (400k sold or 30 ETH raised), the token automatically graduates to Uniswap V3.

---

### sellToken

Sell tokens back to the bonding curve for ETH.

```solidity
function sellToken(
    address _token,
    uint256 _amount,
    uint256 _minEthOut
) external
```

**Parameters:**
- `_token` - Token address to sell
- `_amount` - Amount of tokens to sell
- `_minEthOut` - Minimum ETH to receive (slippage protection)

**Note:** User must approve BondingCurve to spend their tokens first.

---

### graduateToDEX

Manual graduation fallback. Normally tokens auto-graduate, but this can be called if auto-graduation fails.

```solidity
function graduateToDEX(address _token) external
```

**Requirements:**
- Token must have reached target (trading closed)
- Token must not already be graduated

---

## View Functions

### getTokenInfo

```solidity
function getTokenInfo(address _token) external view returns (
    string memory name,
    string memory symbol,
    address creator,
    uint256 createdAt,
    bool isPaused,
    bool isVerified,
    uint256 totalSupply,
    uint256 sold,
    uint256 raised,
    bool isOpen
)
```

### getAllTokens

```solidity
function getAllTokens() external view returns (address[] memory)
```

### getTokensByCreator

```solidity
function getTokensByCreator(address _creator) external view returns (address[] memory)
```

### getTarget

```solidity
function getTarget() external view returns (uint256)
```

Returns the ETH target for graduation (30 ETH).

---

## Admin Functions

### setTarget

```solidity
function setTarget(uint256 _newTarget) external onlyOwner
```

Update graduation target. Max 1000 ETH.

### pauseToken

```solidity
function pauseToken(address _token, bool _paused) external onlyOwner
```

### verifyToken

```solidity
function verifyToken(address _token, bool _verified) external onlyOwner
```

### pause / unpause

```solidity
function pause() external onlyOwner
function unpause() external onlyOwner
```

Emergency pause all factory operations.
