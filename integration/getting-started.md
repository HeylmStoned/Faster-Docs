# Getting Started

This guide covers how to integrate with the Faster Launchpad Diamond contract.

## Prerequisites

- Node.js 16+
- ethers.js v6
- Access to MegaETH testnet

## Network Configuration

```javascript
const networkConfig = {
    chainId: 6343,
    rpcUrl: "https://carrot.megaeth.com/rpc",
    blockExplorer: "https://megaeth-testnet-v2.blockscout.com"
};
```

## Contract Addresses

```javascript
const addresses = {
    // Main entry point - all calls go through Diamond
    diamond: "0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e",
    
    // External dependencies
    prismDexFactory: "0x94996d371622304f2eb85df1eb7f328f7b317c3e",
    positionManager: "0x1279f3cbf01ad4f0cfa93f233464581f4051033a",
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

// Connect to MegaETH testnet
const provider = new ethers.JsonRpcProvider("https://carrot.megaeth.com/rpc");

// With wallet
const wallet = new ethers.Wallet(PRIVATE_KEY, provider);

// Diamond contract instance (single entry point for all operations)
const diamond = new ethers.Contract(addresses.diamond, DIAMOND_ABI, wallet);
```

## ABIs

The Diamond ABI combines all facet ABIs. Available in the repository at `/abis/`:
- `Diamond.json` - Combined ABI for all facets
- `TokenFacet.json`
- `TradingFacet.json`
- `GraduationFacet.json`
- `FeeFacet.json`
- `SecurityFacet.json`

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
const parsed = diamond.interface.parseLog(event);
console.log("Token ID:", parsed.args[0]);
console.log("ERC-20 Wrapper:", parsed.args[5]);

// IMPORTANT: Admin must initialize trading
await diamond.initializeToken(parsed.args[5]);
```

### 2. Buy Tokens

```javascript
// Estimate tokens for 0.1 ETH
const estimate = await diamond.estimateTokensForETH(
    wrapperAddress, 
    ethers.parseEther("0.1")
);

// Buy with 10% slippage tolerance
const minTokens = estimate * 90n / 100n;

const tx = await diamond.buyWithETH(wrapperAddress, wallet.address, minTokens, {
    value: ethers.parseEther("0.1")
});
await tx.wait();
```

### 3. Check Token Status

```javascript
const stats = await diamond.getTokenStats(wrapperAddress);
console.log("Sold:", ethers.formatEther(stats.totalSold));
console.log("Raised:", ethers.formatEther(stats.totalRaised), "ETH");
console.log("Price:", ethers.formatEther(stats.currentPrice), "ETH");
console.log("Trading open:", stats.isOpen);

// Check graduation status
const graduated = await diamond.isTokenGraduated(wrapperAddress);
console.log("Graduated to Prism-Dex(UniV3):", graduated);
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
