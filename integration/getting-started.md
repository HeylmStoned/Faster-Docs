# Getting Started

This guide covers how to integrate with the Memecoin Launchpad.

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
    factory: "0x855EF543a8d611Ee76724C2E4051E7db4947158F",
    bondingCurve: "0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42",
    feeManager: "0x194c622e62FA1C91A8A678A898F4b10E138124cD",
    dexManager: "0x98e57d776226cE97C0c850E2Af33669aa9F99416",
    securityManager: "0xF9e0094Ac5379071970533897Af8f274F8ce2E1c"
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

// Contract instances
const factory = new ethers.Contract(addresses.factory, FACTORY_ABI, wallet);
const bondingCurve = new ethers.Contract(addresses.bondingCurve, BONDING_CURVE_ABI, provider);
const feeManager = new ethers.Contract(addresses.feeManager, FEE_MANAGER_ABI, provider);
```

## ABIs

ABIs are available in the repository at `/app/abis/`:
- `MemecoinFactory.json`
- `BondingCurve.json`
- `FeeManager.json`
- `DEXManager.json`
- `SecurityManager.json`

## Quick Start

### 1. Create a Token

```javascript
const tx = await factory.createToken(
    "My Token",      // name
    "MTK",           // symbol
    "Description",   // description
    "https://...",   // imageUrl
    "",              // website
    "",              // twitter
    "",              // telegram
    50, 25, 25,      // bonding curve fees (creator, badBunnz, buyback)
    30, 50, 10, 10,  // DEX fees (platform, creator, badBunnz, buyback)
    false, 0, 0, 0   // fair launch (disabled)
);

const receipt = await tx.wait();
console.log("Token created!");
```

### 2. Buy Tokens

```javascript
// Estimate tokens for 0.1 ETH
const estimate = await bondingCurve.estimateTokensForETH(
    tokenAddress, 
    ethers.parseEther("0.1")
);

// Buy with 10% slippage tolerance
const minTokens = estimate * 90n / 100n;

const tx = await factory.buyWithETH(tokenAddress, minTokens, {
    value: ethers.parseEther("0.1")
});
await tx.wait();
```

### 3. Check Token Status

```javascript
const stats = await bondingCurve.getTokenStats(tokenAddress);
console.log("Sold:", ethers.formatEther(stats.totalSold));
console.log("Raised:", ethers.formatEther(stats.totalRaised), "ETH");
console.log("Price:", ethers.formatEther(stats.currentPrice), "ETH");
console.log("Trading open:", stats.isOpen);
```

## Next Steps

- [Code Examples](examples.md) - Complete integration examples
- [Contract Reference](../contracts/factory.md) - Full API documentation
- [Events](../reference/events.md) - Event monitoring
