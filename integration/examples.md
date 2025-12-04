# Code Examples

Integration examples for the Memecoin Launchpad.

## Setup

```js
import { ethers } from "ethers";

const FACTORY = "0x855EF543a8d611Ee76724C2E4051E7db4947158F";
const BONDING_CURVE = "0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42";
const FEE_MANAGER = "0x194c622e62FA1C91A8a678A898F4b10E138124cD";

const provider = new ethers.JsonRpcProvider("https://carrot.megaeth.com/rpc");
const signer = new ethers.Wallet(PRIVATE_KEY, provider);
```

## Create Token

```js
const tx = await factory.createToken(
    "My Memecoin", "MEME", "Description", "https://img.url",
    "https://website", "@twitter", "https://t.me/group",
    50, 25, 25,      // bonding fees (creator, badBunnz, buyback)
    30, 50, 10, 10,  // DEX fees (platform, creator, badBunnz, buyback)
    false, 0, 0, 0   // fair launch disabled
);
const receipt = await tx.wait();
```

## Buy Tokens

```js
// Get estimate
const estimate = await bondingCurve.estimateTokensForETH(tokenAddress, ethers.parseEther("0.1"));
const minTokens = estimate * 90n / 100n; // 10% slippage

// Execute buy
await factory.buyWithETH(tokenAddress, minTokens, { value: ethers.parseEther("0.1") });
```

## Sell Tokens

```js
// Approve first
await token.approve(BONDING_CURVE, amount);

// Get quote and sell
const [price, fee] = await bondingCurve.getSellPrice(tokenAddress, amount);
const minEth = (price - fee) * 90n / 100n;
await factory.sellToken(tokenAddress, amount, minEth);
```

## Check Status

```js
const stats = await bondingCurve.getTokenStats(tokenAddress);
// Returns: totalSold, totalRaised, currentPrice, isOpen
```

## Claim Creator Rewards

```js
const rewards = await feeManager.getCreatorRewards(address);
if (rewards > 0n) {
    await feeManager.claimCreatorRewards();
}
```

## Listen to Events

```js
factory.on("TokenCreated", (token, creator, name) => console.log("New:", name));
factory.on("TokenBought", (token, buyer, amount, price) => console.log("Buy:", amount));
factory.on("TokenGraduated", (token, pool) => console.log("Graduated:", token));
```
