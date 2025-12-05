# ERC-20 Wrappers

Minimal proxy wrappers that provide ERC-20 compatibility for ERC-6909 tokens.

## Overview

ERC-6909 tokens aren't directly compatible with Prism-Dex(UniV3) and other DeFi protocols. We solve this with **minimal proxy wrappers** (EIP-1167).

```
User wants to trade on Prism-Dex(UniV3)
            │
            ▼
┌─────────────────────────┐
│   ERC-20 Wrapper        │  ◄── Thin proxy (~45 bytes)
│   (Minimal Proxy)       │
└───────────┬─────────────┘
            │ delegates to
            ▼
┌─────────────────────────┐
│   Diamond Contract      │  ◄── Actual token logic
│   (ERC-6909 storage)    │
└─────────────────────────┘
```

## Why Wrappers?

| Aspect | Traditional ERC-20 | ERC-20 Wrapper |
|--------|-------------------|----------------|
| Bytecode Size | ~5KB | ~45 bytes |
| Deployment Gas | ~1-2M | ~50k |
| Storage | Own contract | Diamond (shared) |
| Compatibility | Native | Full ERC-20 |

**For users and token holders, nothing changes.** Every token has a standard ERC-20 address that works everywhere - wallets, DEXs, block explorers, portfolio trackers.

## How It Works

1. **Token Creation** - When `createToken()` is called, a wrapper is deployed automatically
2. **Delegation** - All ERC-20 calls to the wrapper are delegated to the Diamond
3. **Storage** - Balances and allowances stored in Diamond's ERC-6909 storage
4. **Transfers** - Standard ERC-20 `transfer()` and `transferFrom()` work normally

## Wrapper Functions

Each wrapper implements the full ERC-20 interface:

```solidity
// Standard ERC-20
function name() external view returns (string memory)
function symbol() external view returns (string memory)
function decimals() external view returns (uint8)
function totalSupply() external view returns (uint256)
function balanceOf(address account) external view returns (uint256)
function transfer(address to, uint256 amount) external returns (bool)
function allowance(address owner, address spender) external view returns (uint256)
function approve(address spender, uint256 amount) external returns (bool)
function transferFrom(address from, address to, uint256 amount) external returns (bool)
```

## Getting Wrapper Address

### From Token Creation

```javascript
const tx = await diamond.createToken(
    "My Token", "MTK", "Description", "https://...",
    "", "", "", ethers.parseEther("1000000000"), diamond.address
);
const receipt = await tx.wait();

// Parse TokenCreated event
const event = receipt.logs.find(log => {
    try {
        return diamond.interface.parseLog(log)?.name === "TokenCreated";
    } catch { return false; }
});
const { wrapper } = diamond.interface.parseLog(event).args;
```

### From Token ID

```javascript
const wrapper = await diamond.getWrapper(tokenId);
```

### From Wrapper to Token ID

```javascript
const tokenId = await diamond.getTokenId(wrapperAddress);
```

## Using Wrappers

### In Wallets

Add the wrapper address as a custom token in MetaMask or any ERC-20 compatible wallet.

### On Prism-Dex(UniV3)

After graduation, the wrapper address is used for the trading pair:
- Pool: `WRAPPER_ADDRESS / WETH`
- Trade normally via Prism-Dex(UniV3) interface

### In DeFi

The wrapper works with any protocol expecting ERC-20:
- Lending protocols
- Yield aggregators
- Portfolio trackers
- Block explorers

## Events

Wrappers emit standard ERC-20 events:

```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
event Approval(address indexed owner, address indexed spender, uint256 value);
```

## WrapperFacet

**Facet Address**: `0x8216a7e5BE75bA5FA5c28b9f3C53f2a6d6942278`

Manages wrapper deployment and ERC-20 interface implementation.

### deployWrapper

Deploy a wrapper for an existing ERC-6909 token ID.

```solidity
function deployWrapper(uint256 tokenId) external returns (address wrapper)
```

**Note:** Usually called automatically during `createToken()`.

### getWrapperImplementation

Get the implementation address for wrappers.

```solidity
function getWrapperImplementation() external view returns (address)
```
