# Getting Started

This guide covers how to integrate with the Faster Launchpad Diamond contract.

## Prerequisites

- Node.js 16+
- ethers.js v6
- Access to MegaETH Mainnet

## Network Configuration

```javascript
const networkConfig = {
    chainId: 4326,
    rpcUrl: "https://mainnet.megaeth.com/rpc",
    blockExplorer: "https://mega.etherscan.io"
};
```

## Contract Addresses

```javascript
const addresses = {
    // Main entry point - all calls go through Diamond
    diamond: "0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B",
    
    // External dependencies (Uniswap V3)
    uniswapV3Factory: "0xef349aa6cc5e87559e716ac293845a48cadf30d5",
    positionManager: "0x9feaf944c518164d5d0c45f28255758acff8e987",
    weth: "0x4200000000000000000000000000000000000006"
};
```

## Installation

```bash
npm install ethers
```

## Basic Setup

```javascript
import { ethers } from "ethers";

// Connect to MegaETH Mainnet
const provider = new ethers.JsonRpcProvider("https://mainnet.megaeth.com/rpc");

// With wallet
const wallet = new ethers.Wallet(PRIVATE_KEY, provider);

// Diamond contract instance (single entry point for all operations)
const diamond = new ethers.Contract(addresses.diamond, DIAMOND_ABI, wallet);
```

## ABIs

ABIs are in this repo under **`abis/`**: use `abis/Diamond.json` for the Diamond (all launchpad functions) and `abis/ERC20.json` for wrapper tokens (balance, transfer, approve).

## Wrapper address = your token address

Each launchpad token has a **wrapper** address (returned in `TokenCreated`). **Use this address exactly like a standard ERC-20 token:**

| Use case | What to do |
|----------|------------|
| Add to wallet | Use the wrapper address as the token contract address |
| Check balance | `wrapper.balanceOf(user)` (standard ERC-20) |
| Transfer | `wrapper.transfer(to, amount)` or `wrapper.transferFrom(from, to, amount)` |
| Approve (e.g. for DEX) | `wrapper.approve(spender, amount)` |
| Trade on Uniswap V3 | Use wrapper address as the token in the pair |

For **launchpad-specific** actions (buy/sell on bonding curve, graduation, fees, rewards), call the **Diamond** with the wrapper address as the `token` parameter. See [ERC-20 Wrappers](../contracts/wrappers.md) for the full picture.

## Quick Start

### 1. Create a Token

Creates an ERC-6909 token + ERC-20 wrapper (~50k gas vs 1-2M traditional).

All tokens have a fixed supply of **1 million tokens** (684k for bonding curve, 316k for DEX).

```javascript
const tx = await diamond.createToken(
    "My Token",                          // _name
    "MTK",                               // _symbol
    "The best memecoin!",                // _description
    "https://example.com/image.png",     // _imageUrl
    "https://example.com",               // _website
    "@mytoken",                          // _twitter
    "https://t.me/mytoken",              // _telegram
    // Bonding curve fee split (must sum to 100)
    50,                                  // _creatorFeePercentage
    25,                                  // _badBunnzFeePercentage
    25,                                  // _buybackFeePercentage
    // DEX LP fee split (must sum to 100) - platform gets fixed 20% separately
    50,                                  // _dexCreatorFeePercentage
    25,                                  // _dexBadBunnzFeePercentage
    25,                                  // _dexBuybackFeePercentage
    // Fair launch (disabled)
    false,                               // _enableFairLaunch
    0,                                   // _fairLaunchDuration
    0,                                   // _maxPerWallet
    0                                    // _fixedPrice
);

const receipt = await tx.wait();

// Get token ID and wrapper address from event
const event = receipt.logs.find(log => {
    try {
        return diamond.interface.parseLog(log)?.name === "TokenCreated";
    } catch { return false; }
});
// TokenCreated(id, creator, name, symbol, totalSupply, wrapper)
const parsed = diamond.interface.parseLog(event);
console.log("Token ID:", parsed.args.id ?? parsed.args[0]);
console.log("ERC-20 Wrapper:", parsed.args.wrapper ?? parsed.args[5]);

// IMPORTANT: Admin must initialize trading
await diamond.initializeToken(parsed.args.wrapper ?? parsed.args[5]);
```

### 2. Buy Tokens

```javascript
// Estimate tokens for 0.1 ETH
const estimate = await diamond.estimateTokensForETH(
    wrapperAddress, 
    ethers.parseEther("0.1")
);

// Buy with 10% slippage tolerance (buyWithETH(token, buyer, minTokensOut))
const minTokensOut = estimate * 90n / 100n;

const tx = await diamond.buyWithETH(wrapperAddress, wallet.address, minTokensOut, {
    value: ethers.parseEther("0.1")
});
await tx.wait();
```

### 3. Check Token Status

Trading functions use the **wrapper address** (from TokenCreated). For full metadata use `getTokenData(tokenId)` with the id from creation.

```javascript
const stats = await diamond.getTokenStats(wrapperAddress);
console.log("Sold:", ethers.formatEther(stats.totalSold));
console.log("Raised:", ethers.formatEther(stats.totalRaised), "ETH");
console.log("Price:", ethers.formatEther(stats.currentPrice), "ETH");
console.log("Trading open:", stats.isOpen);

// Check graduation status
const graduated = await diamond.isTokenGraduated(wrapperAddress);
console.log("Graduated to Uniswap V3:", graduated);
```

### 4. Claim Creator Rewards

```javascript
const rewards = await diamond.getCreatorRewards(wallet.address);
console.log("Pending rewards:", ethers.formatEther(rewards), "ETH");

if (rewards > 0n) {
    const tx = await diamond.claimCreatorRewards();
    await tx.wait();
    console.log("Rewards claimed!");
}
```

## Next Steps

- [Code Examples](examples.md) - Complete integration examples
- [Contract Reference](../contracts/diamond.md) - Full API documentation
- [Events](../reference/events.md) - Event monitoring
