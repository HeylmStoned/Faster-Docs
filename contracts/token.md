# MemecoinToken

Individual ERC20 token contract created for each memecoin.

## Overview

Each token created through the launchpad deploys a new MemecoinToken contract with:
- Standard ERC20 functionality
- Burnable tokens
- Pausable transfers
- Metadata storage
- Trading statistics

## Token Properties

| Property | Value |
|----------|-------|
| Total Supply | 1,000,000 tokens |
| Decimals | 18 |
| Standard | ERC20 + Burnable + Pausable |

## Immutable Properties

Set at creation, cannot be changed:
- `creator` - Address that created the token
- `createdAt` - Timestamp of creation

## Functions

### updateMetadata

Update token metadata. Only creator or factory can call.

```solidity
function updateMetadata(
    string memory _description,
    string memory _imageUrl,
    string memory _website,
    string memory _twitter,
    string memory _telegram
) external
```

---

### pauseToken

Pause or unpause token transfers.

```solidity
function pauseToken(bool _paused) external onlyFactoryOrOwner
```

When paused, all transfers (including buy/sell) are blocked.

---

### verifyToken

Mark token as verified. Only owner can call.

```solidity
function verifyToken(bool _verified) external onlyOwner
```

---

### recordTrade

Record trading activity. Called by Factory/BondingCurve.

```solidity
function recordTrade(
    uint256 _amount, 
    uint256 _price, 
    address _buyer
) external
```

Updates:
- `totalTrades`
- `totalVolume`
- `lastTradeTime`
- `uniqueBuyers`

---

## View Functions

### getMetadata

```solidity
function getMetadata() external view returns (
    string memory description,
    string memory imageUrl,
    string memory website,
    string memory twitter,
    string memory telegram
)
```

### getTokenInfo

```solidity
function getTokenInfo() external view returns (
    string memory tokenName,
    string memory tokenSymbol,
    address tokenCreator,
    uint256 creationTime,
    bool paused,
    bool verified,
    uint256 trades,
    uint256 volume
)
```

### getTradingStats

```solidity
function getTradingStats() external view returns (
    uint256 trades,
    uint256 volume,
    uint256 lastTrade,
    uint256 averagePrice
)
```

## State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `creator` | address | Token creator (immutable) |
| `createdAt` | uint256 | Creation timestamp (immutable) |
| `isPaused` | bool | Transfer pause state |
| `isVerified` | bool | Verification status |
| `tokenDescription` | string | Description |
| `tokenImageUrl` | string | Image URL |
| `tokenWebsite` | string | Website URL |
| `tokenTwitter` | string | Twitter handle |
| `tokenTelegram` | string | Telegram link |
| `totalTrades` | uint256 | Total trade count |
| `totalVolume` | uint256 | Total ETH volume |
| `lastTradeTime` | uint256 | Last trade timestamp |
| `uniqueBuyers` | uint256 | Unique buyer count |

## ERC20 Functions

Standard ERC20 functions with pause check:

```solidity
function transfer(address to, uint256 amount) public whenNotPaused returns (bool)
function transferFrom(address from, address to, uint256 amount) public whenNotPaused returns (bool)
```

Plus standard:
- `balanceOf(address)`
- `allowance(address, address)`
- `approve(address, uint256)`
- `totalSupply()`
- `name()`
- `symbol()`
- `decimals()`
