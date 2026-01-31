# ERC-20 Wrappers

Minimal proxy wrappers that provide ERC-20 compatibility for ERC-6909 tokens. Managed by WrapperFacet.

## TL;DR for integrators

**You can interact with launchpad tokens exactly like standard ERC-20 tokens.** Each token has a **wrapper contract address** (from `TokenCreated`). Use that address everywhere you would use an ERC-20 contract:

- **Wallets** – Add wrapper address as custom token; balances and history work as usual.
- **Transfers** – `transfer(to, amount)`, `transferFrom(from, to, amount)` on the wrapper.
- **Approvals** – `approve(spender, amount)` for DEXs, routers, or other contracts.
- **DEXs / DeFi** – Use the wrapper address as the token in Uniswap V3 on Prism, lending protocols, etc.

No special handling or Diamond ABI is needed for normal token usage. The wrapper implements the full ERC-20 interface and delegates to the Diamond internally.

## Overview

ERC-6909 tokens aren't directly compatible with Uniswap V3 on Prism and other DeFi protocols. We solve this with **minimal proxy wrappers** (EIP-1167).

```
User wants to trade on Uniswap V3 on Prism
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

1. **Token Creation** - When `createToken()` is called, a wrapper is deployed via `wrap6909()` (token = Diamond, tokenId, initialDeposit)
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
const tx = await diamond.createToken(/* ... */);
const receipt = await tx.wait();

// TokenCreated(id, creator, name, symbol, totalSupply, wrapper)
const event = receipt.logs.find(log => {
    try {
        return diamond.interface.parseLog(log)?.name === "TokenCreated";
    } catch { return false; }
});
const { id, wrapper } = diamond.interface.parseLog(event).args;
```

### From Token ID (TokenFacet / ERC6909Facet)

```javascript
const wrapper = await diamond.getTokenWrapper(tokenId);
```

### From Wrapper to Token ID

There is no on-chain `getTokenId(wrapper)`. Store the token `id` from the `TokenCreated` event at creation time. Alternatively, iterate `tokenCount()` and compare `getTokenWrapper(i)` to the wrapper address to find the id.

## WrapperFacet

**Facet Address**: [`0x13A3fe336c93662D4902183F3D5D11af4483C09E`](https://mega.etherscan.io/address/0x13A3fe336c93662D4902183F3D5D11af4483C09E)

Manages wrapper deployment and ERC-20 interface. Wrappers are keyed by **(token, tokenId)** where `token` is the Diamond address for launchpad tokens.

### wrap6909

Deploy a wrapper for an ERC-6909 token and optionally deposit initial supply.

```solidity
function wrap6909(
    address token,
    uint256 tokenId,
    uint256 initialDeposit
) external returns (address wrapper)
```

Usually called automatically during `createToken()`.

### getWrapper

Get the wrapper address for a (token, tokenId) pair.

```solidity
function getWrapper(address token, uint256 tokenId) external view returns (address wrapper)
```

### predictWrapper

Predict the wrapper address (CREATE2) before deployment.

```solidity
function predictWrapper(address token, uint256 tokenId) external view returns (address predicted)
```

### getImplementation

Get the implementation address for wrappers.

```solidity
function getImplementation() external view returns (address implementation)
```

### setWrapperImplementation

Set the wrapper implementation (owner only).

```solidity
function setWrapperImplementation(address _implementation) external
```

## Events

### WrapperCreated

```solidity
event WrapperCreated(
    address indexed token,
    uint256 indexed tokenId,
    address wrapper
);
```

### ImplementationSet

```solidity
event ImplementationSet(address implementation);
```

## Using Wrappers

### In Wallets

Add the wrapper address as a custom token in MetaMask or any ERC-20 compatible wallet.

### On Uniswap V3 on Prism

After graduation, the wrapper address is used for the trading pair:
- Pool: `WRAPPER_ADDRESS / WETH`
- Trade normally via Uniswap V3 on Prism interface

### In DeFi

The wrapper works with any protocol expecting ERC-20:
- Lending protocols
- Yield aggregators
- Portfolio trackers
- Block explorers

## ERC-20 Wrapper Events

Wrappers emit standard ERC-20 events:

```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
event Approval(address indexed owner, address indexed spender, uint256 value);
```
