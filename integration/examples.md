# Code Examples

Complete integration examples for the Faster Launchpad Diamond contract.

## Setup

```javascript
import { ethers } from "ethers";

const DIAMOND_ADDRESS = "0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e";

// Diamond ABI combines all facet ABIs
import DIAMOND_ABI from "./abis/Diamond.json";
import ERC20_ABI from "./abis/ERC20.json"; // Standard ERC-20 for wrappers

const provider = new ethers.JsonRpcProvider("https://timothy.megaeth.com/rpc");
const signer = new ethers.Wallet(PRIVATE_KEY, provider);

const diamond = new ethers.Contract(DIAMOND_ADDRESS, DIAMOND_ABI.abi, signer);
```

---

## Create Token

### Basic Token

Creates an ERC-6909 token + ERC-20 wrapper (~50k gas vs 1-2M traditional).

All tokens have a **fixed supply of 1 million tokens** (684k for bonding curve, 316k for DEX).

```javascript
async function createBasicToken() {
    const tx = await diamond.createToken(
        "My Memecoin",                       // _name
        "MEME",                              // _symbol
        "The best memecoin!",                // _description
        "https://example.com/image.png",     // _imageUrl
        "https://example.com",               // _website
        "@mymemecoin",                       // _twitter
        "https://t.me/mymemecoin",           // _telegram
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
    const tokenId = parsed.args[0];
    const wrapper = parsed.args[5];
    
    console.log("Token ID:", tokenId.toString());
    console.log("ERC-20 Wrapper:", wrapper);
    
    // IMPORTANT: Admin must initialize trading
    await diamond.initializeToken(wrapper);
    
    return { tokenId, wrapper };
}
```

### Token with Fair Launch

Fair launch can be enabled at token creation:

```javascript
async function createFairLaunchToken() {
    const tx = await diamond.createToken(
        "Fair Launch Token",
        "FAIR",
        "A fair launch memecoin",
        "https://example.com/image.png",
        "", "", "",
        50, 25, 25,      // BC fees (creator, badBunnz, buyback)
        50, 25, 25,      // DEX fees (creator, badBunnz, buyback) - platform gets 20% fixed
        // Fair launch ENABLED
        true,                            // _enableFairLaunch
        3600,                            // 1 hour duration
        ethers.parseEther("10000"),      // max 10k tokens per wallet
        ethers.parseEther("0.00005")     // fixed price 0.00005 ETH
    );

    const receipt = await tx.wait();
    console.log("Fair launch token created!");
}
```

---

## Buy Tokens

### Buy with ETH

```javascript
async function buyTokens(wrapperAddress, ethAmount) {
    // Get estimated tokens
    const estimate = await diamond.estimateTokensForETH(
        wrapperAddress,
        ethers.parseEther(ethAmount)
    );
    
    console.log("Estimated tokens:", ethers.formatEther(estimate));
    
    // Calculate minimum with 10% slippage
    const minTokensOut = estimate * 90n / 100n;
    
    // Execute buy
    const tx = await diamond.buyWithETH(
        wrapperAddress,
        signer.address,  // buyer
        minTokensOut,
        { value: ethers.parseEther(ethAmount) }
    );
    
    const receipt = await tx.wait();
    
    // Parse event for actual amounts
    const event = receipt.logs.find(log => {
        try {
            return diamond.interface.parseLog(log)?.name === "TokenBought";
        } catch { return false; }
    });
    
    if (event) {
        const parsed = diamond.interface.parseLog(event);
        console.log("Bought:", ethers.formatEther(parsed.args.amount), "tokens");
        console.log("Spent:", ethers.formatEther(parsed.args.price), "ETH");
    }
}

// Usage
await buyTokens("0x...", "0.1"); // Buy with 0.1 ETH
```

### Get Buy Quote

```javascript
async function getBuyQuote(wrapperAddress, tokenAmount) {
    const [price, fee] = await diamond.getBuyPrice(
        wrapperAddress,
        ethers.parseEther(tokenAmount)
    );
    
    console.log("Price:", ethers.formatEther(price), "ETH");
    console.log("Fee:", ethers.formatEther(fee), "ETH");
    console.log("Total:", ethers.formatEther(price + fee), "ETH");
    
    return { price, fee, total: price + fee };
}
```

---

## Sell Tokens

**Note:** Sells are disabled by default. Admin must enable via `setSellsEnabled(wrapper, true)`.

### Sell Tokens for ETH

```javascript
async function sellTokens(wrapperAddress, tokenAmount) {
    // Check if sells are enabled
    const sellsEnabled = await diamond.areSellsEnabled(wrapperAddress);
    if (!sellsEnabled) {
        console.log("Sells are disabled for this token");
        return;
    }
    
    const wrapper = new ethers.Contract(wrapperAddress, ERC20_ABI, signer);
    const amount = ethers.parseEther(tokenAmount);
    
    // Approve Diamond to spend tokens
    const approveTx = await wrapper.approve(DIAMOND_ADDRESS, amount);
    await approveTx.wait();
    console.log("Approved");
    
    // Get sell quote
    const [price, fee] = await diamond.getSellPrice(wrapperAddress, amount);
    const netProceeds = price - fee;
    
    console.log("Expected:", ethers.formatEther(netProceeds), "ETH");
    
    // Calculate minimum with 10% slippage
    const minEthOut = netProceeds * 90n / 100n;
    
    // Execute sell
    const tx = await diamond.sellToken(
        wrapperAddress,
        amount,
        signer.address,  // seller
        minEthOut
    );
    await tx.wait();
    
    console.log("Sold successfully!");
}

// Usage
await sellTokens("0x...", "1000"); // Sell 1000 tokens
```

---

## Check Token Status

### Get Trading Stats

```javascript
async function getTokenStatus(wrapperAddress) {
    const stats = await diamond.getTokenStats(wrapperAddress);
    
    console.log("=== Token Status ===");
    console.log("Sold:", ethers.formatEther(stats.totalSold), "tokens");
    console.log("Raised:", ethers.formatEther(stats.totalRaised), "ETH");
    console.log("Current Price:", ethers.formatEther(stats.currentPrice), "ETH");
    console.log("Trading Open:", stats.isOpen);
    
    // Progress to graduation (30 ETH target)
    const raisedProgress = (Number(stats.totalRaised) / Number(ethers.parseEther("30"))) * 100;
    console.log("Progress to graduation:", raisedProgress.toFixed(2), "%");
    
    return stats;
}
```

### Get Full Token Info

```javascript
async function getFullTokenInfo(wrapperAddress) {
    const [metadata, stats, graduated, sellsEnabled] = await Promise.all([
        diamond.getTokenMetadata(wrapperAddress),
        diamond.getTokenStats(wrapperAddress),
        diamond.isTokenGraduated(wrapperAddress),
        diamond.areSellsEnabled(wrapperAddress)
    ]);
    
    return {
        name: metadata.name,
        symbol: metadata.symbol,
        description: metadata.description,
        creator: metadata.creator,
        createdAt: new Date(Number(metadata.createdAt) * 1000),
        sold: ethers.formatEther(stats.totalSold),
        raised: ethers.formatEther(stats.totalRaised),
        currentPrice: ethers.formatEther(stats.currentPrice),
        isOpen: stats.isOpen,
        graduated,
        sellsEnabled
    };
}
```

### Check Graduation Status

```javascript
async function checkGraduation(wrapperAddress) {
    const graduated = await diamond.isTokenGraduated(wrapperAddress);
    
    if (graduated) {
        const status = await diamond.getGraduationStatus(wrapperAddress);
        console.log("=== Graduated to Prism-Dex(UniV3) ===");
        console.log("Pool:", status.pool);
        console.log("Position ID:", status.positionId.toString());
        console.log("Liquidity:", status.liquidity.toString());
    } else {
        console.log("Token not yet graduated");
    }
    
    return graduated;
}
```

---

## Creator Rewards

### Check Rewards

```javascript
async function checkRewards(creatorAddress) {
    const rewards = await diamond.getCreatorRewards(creatorAddress);
    console.log("Pending rewards:", ethers.formatEther(rewards), "ETH");
    return rewards;
}
```

### Claim Rewards

```javascript
async function claimRewards() {
    const rewards = await diamond.getCreatorRewards(signer.address);
    
    if (rewards === 0n) {
        console.log("No rewards to claim");
        return;
    }
    
    console.log("Claiming:", ethers.formatEther(rewards), "ETH");
    
    const tx = await diamond.claimCreatorRewards();
    await tx.wait();
    
    console.log("Rewards claimed!");
}
```

### Collect LP Fees (Post-Graduation)

```javascript
async function collectLPFees(wrapperAddress) {
    const graduated = await diamond.isTokenGraduated(wrapperAddress);
    if (!graduated) {
        console.log("Token not graduated yet");
        return;
    }
    
    // Anyone can call this - fees auto-distribute
    const tx = await diamond.collectFees(wrapperAddress);
    const receipt = await tx.wait();
    
    const event = receipt.logs.find(log => {
        try {
            return diamond.interface.parseLog(log)?.name === "FeesCollected";
        } catch { return false; }
    });
    
    if (event) {
        const { amount0, amount1 } = diamond.interface.parseLog(event).args;
        console.log("Collected:", ethers.formatEther(amount0), "tokens");
        console.log("Collected:", ethers.formatEther(amount1), "ETH");
    }
}
```

---

## List All Tokens

```javascript
async function getAllTokens() {
    const tokens = await diamond.getAllTokens();
    
    console.log("Total tokens:", tokens.length);
    
    for (const wrapperAddress of tokens) {
        const metadata = await diamond.getTokenMetadata(wrapperAddress);
        console.log(`${metadata.name} (${metadata.symbol}): ${wrapperAddress}`);
    }
    
    return tokens;
}
```

---

## Monitor Events

```javascript
// Listen for new tokens
diamond.on("TokenCreated", (tokenId, wrapper, creator, name, symbol) => {
    console.log(`New token: ${name} (${symbol})`);
    console.log(`  Wrapper: ${wrapper}`);
    console.log(`  Creator: ${creator}`);
});

// Listen for trades
diamond.on("TokenBought", (token, buyer, amount, price, fee) => {
    console.log(`Buy: ${ethers.formatEther(amount)} tokens for ${ethers.formatEther(price)} ETH`);
});

diamond.on("TokenSold", (token, seller, amount, price, fee) => {
    console.log(`Sell: ${ethers.formatEther(amount)} tokens for ${ethers.formatEther(price)} ETH`);
});

// Listen for graduations
diamond.on("TokenGraduated", (token, pool, positionId, tokenAmount, ethAmount) => {
    console.log(`Token graduated to Prism-Dex(UniV3)!`);
    console.log(`  Token: ${token}`);
    console.log(`  Pool: ${pool}`);
});

// Listen for reward claims
diamond.on("CreatorRewardsClaimed", (creator, amount) => {
    console.log(`Creator ${creator} claimed ${ethers.formatEther(amount)} ETH`);
});
```

---

## Error Handling

```javascript
async function safeBuy(wrapperAddress, ethAmount) {
    try {
        const tx = await diamond.buyWithETH(
            wrapperAddress,
            signer.address,
            0n, // No slippage protection for example
            { value: ethers.parseEther(ethAmount) }
        );
        await tx.wait();
    } catch (error) {
        if (error.message.includes("Trading is closed")) {
            console.log("Token has graduated to Prism-Dex(UniV3)");
        } else if (error.message.includes("Output below minimum")) {
            console.log("Slippage too high, try again");
        } else if (error.message.includes("Exceeds max buy amount")) {
            console.log("Max 1 ETH per transaction");
        } else if (error.message.includes("Token is paused")) {
            console.log("Token trading is paused");
        } else {
            throw error;
        }
    }
}
```

---

## ERC-6909 Direct Access

For advanced use cases, you can interact with ERC-6909 directly:

```javascript
// Get ERC-6909 token ID from wrapper
const tokenId = await diamond.getTokenId(wrapperAddress);

// Check ERC-6909 balance
const balance = await diamond.balanceOf(signer.address, tokenId);
console.log("ERC-6909 Balance:", ethers.formatEther(balance));

// Transfer via ERC-6909
const tx = await diamond.transfer(recipientAddress, tokenId, amount);
await tx.wait();

// Or use the ERC-20 wrapper (same result)
const wrapper = new ethers.Contract(wrapperAddress, ERC20_ABI, signer);
const wrapperBalance = await wrapper.balanceOf(signer.address);
console.log("ERC-20 Balance:", ethers.formatEther(wrapperBalance));
```
