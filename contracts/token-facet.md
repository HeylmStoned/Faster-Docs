# TokenFacet

Creates ERC-6909 multi-tokens with metadata and fee configuration. Each token gets a unique ID within the Diamond and an ERC-20 wrapper for DeFi compatibility.

**Facet Address**: [`0x6799c57642E9F5813dBE2B01c27ce024cb417f87`](https://megaeth-testnet-v2.blockscout.com/address/0x6799c57642E9F5813dBE2B01c27ce024cb417f87)

## Overview

TokenFacet handles:
- Creating new tokens (ERC-6909 + ERC-20 wrapper)
- Token metadata storage
- Fee configuration per token
- Fair launch settings
- ERC-6909 transfers and balances

## Constants

```solidity
uint256 public constant TOTAL_SUPPLY = 1_000_000 ether; // 1 million tokens (fixed)
```

All tokens are created with a fixed supply of **1 million tokens**:
- 684,000 tokens (68.4%) sold via bonding curve
- 316,000 tokens (31.6%) reserved for DEX liquidity

## Functions

### createToken

Creates a new memecoin with metadata, fee configuration, and optional fair launch.

```solidity
function createToken(
    string calldata _name,
    string calldata _symbol,
    string calldata _description,
    string calldata _imageUrl,
    string calldata _website,
    string calldata _twitter,
    string calldata _telegram,
    uint256 _creatorFeePercentage,
    uint256 _badBunnzFeePercentage,
    uint256 _buybackFeePercentage,
    uint256 _dexCreatorFeePercentage,
    uint256 _dexBadBunnzFeePercentage,
    uint256 _dexBuybackFeePercentage,
    bool _enableFairLaunch,
    uint256 _fairLaunchDuration,
    uint256 _maxPerWallet,
    uint256 _fixedPrice
) external returns (uint256 id, address wrapper)
```

**Parameters:**
- `_name` - Token name (e.g., "My Memecoin")
- `_symbol` - Token symbol (e.g., "MEME")
- `_description` - Token description
- `_imageUrl` - URL to token image
- `_website` - Project website
- `_twitter` - Twitter handle
- `_telegram` - Telegram group link
- `_creatorFeePercentage` - Creator's share of BC trading fees (must sum to 100 with others)
- `_badBunnzFeePercentage` - Bad Bunnz share of BC trading fees
- `_buybackFeePercentage` - Buyback share of BC trading fees
- `_dexCreatorFeePercentage` - Creator share of DEX LP fees (must sum to 100 with others)
- `_dexBadBunnzFeePercentage` - Bad Bunnz share of DEX LP fees
- `_dexBuybackFeePercentage` - Buyback share of DEX LP fees
- `_enableFairLaunch` - Whether to enable fair launch mode
- `_fairLaunchDuration` - Duration of fair launch in seconds (if enabled)
- `_maxPerWallet` - Max tokens per wallet during fair launch (if enabled)
- `_fixedPrice` - Fixed price during fair launch in wei (if enabled)

**Note:** Platform gets a fixed 20% on DEX LP fees automatically - not configurable.

**Returns:**
- `id` - ERC-6909 token ID within the Diamond
- `wrapper` - ERC-20 wrapper address for DeFi compatibility

**Gas Cost**: ~50,000 gas (vs ~1-2M for traditional ERC-20 deployment)

**Note:** After creation, admin must call `initializeToken(wrapper)` on TradingFacet to open trading.

---

### getTokenMetadata

Get token metadata by wrapper address.

```solidity
function getTokenMetadata(address wrapper) external view returns (
    string memory name,
    string memory symbol,
    string memory description,
    string memory imageUrl,
    string memory website,
    string memory twitter,
    string memory telegram,
    address creator,
    uint256 createdAt
)
```

---

### getTokenId

Get the ERC-6909 token ID for a wrapper address.

```solidity
function getTokenId(address wrapper) external view returns (uint256)
```

---

### getWrapper

Get the ERC-20 wrapper address for a token ID.

```solidity
function getWrapper(uint256 tokenId) external view returns (address)
```

---

### getAllTokens

Get all token wrapper addresses.

```solidity
function getAllTokens() external view returns (address[] memory)
```

---

### getTokensByCreator

Get all tokens created by an address.

```solidity
function getTokensByCreator(address creator) external view returns (address[] memory)
```

---

## ERC-6909 Functions

### balanceOf

```solidity
function balanceOf(address owner, uint256 id) external view returns (uint256)
```

### transfer

```solidity
function transfer(address receiver, uint256 id, uint256 amount) external returns (bool)
```

### transferFrom

```solidity
function transferFrom(
    address sender,
    address receiver,
    uint256 id,
    uint256 amount
) external returns (bool)
```

### approve

```solidity
function approve(address spender, uint256 id, uint256 amount) external returns (bool)
```

### setOperator

```solidity
function setOperator(address operator, bool approved) external returns (bool)
```

---

## Events

### TokenCreated

```solidity
event TokenCreated(
    uint256 indexed tokenId,
    address indexed wrapper,
    address indexed creator,
    string name,
    string symbol
);
```

### Transfer (ERC-6909)

```solidity
event Transfer(
    address indexed sender,
    address indexed receiver,
    uint256 indexed id,
    uint256 amount
);
```
