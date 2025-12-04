# Code Examples

Complete integration examples for common operations.

## Setup

```javascript
import { ethers } from "ethers";

const FACTORY_ADDRESS = "0x855EF543a8d611Ee76724C2E4051E7db4947158F";
const BONDING_CURVE_ADDRESS = "0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42";
const FEE_MANAGER_ADDRESS = "0x194c622e62FA1C91A8A678A898F4b10E138124cD";

// ABIs (import from /app/abis/)
import FACTORY_ABI from "./abis/MemecoinFactory.json";
import BONDING_CURVE_ABI from "./abis/BondingCurve.json";
import FEE_MANAGER_ABI from "./abis/FeeManager.json";
import TOKEN_ABI from "./abis/MemecoinToken.json";

const provider = new ethers.JsonRpcProvider("https://carrot.megaeth.com/rpc");
const signer = new ethers.Wallet(PRIVATE_KEY, provider);

const factory = new ethers.Contract(FACTORY_ADDRESS, FACTORY_ABI.abi, signer);
const bondingCurve = new ethers.Contract(BONDING_CURVE_ADDRESS, BONDING_CURVE_ABI.abi, provider);
const feeManager = new ethers.Contract(FEE_MANAGER_ADDRESS, FEE_MANAGER_ABI.abi, signer);
```

---

## Create Token

### Basic Token

```javascript
async function createBasicToken() {
    const tx = await factory.createToken(
        "My Memecoin",           // name
        "MEME",                  // symbol
        "The best memecoin!",    // description
        "https://example.com/image.png",  // imageUrl
        "https://example.com",   // website
        "@mymemecoin",           // twitter
        "https://t.me/mymemecoin", // telegram
        // Bonding curve fee split (must = 100)
        50,  // creator gets 50%
        25,  // badBunnz gets 25%
        25,  // buyback gets 25%
        // DEX fee split (must = 100)
        30,  // platform gets 30%
        50,  // creator gets 50%
        10,  // badBunnz gets 10%
        10,  // buyback gets 10%
        // Fair launch (disabled)
        false,
        0,
        0,
        0
    );

    const receipt = await tx.wait();
    
    // Get token address from event
    const event = receipt.logs.find(log => {
        try {
            return factory.interface.parseLog(log)?.name === "TokenCreated";
        } catch { return false; }
    });
    
    const parsed = factory.interface.parseLog(event);
    const tokenAddress = parsed.args.token;
    
    console.log("Token created:", tokenAddress);
    return tokenAddress;
}
```

### Token with Fair Launch

```javascript
async function createFairLaunchToken() {
    const tx = await factory.createToken(
        "Fair Launch Token",
        "FAIR",
        "A fair launch memecoin",
        "https://example.com/image.png",
        "", "", "",
        50, 25, 25,      // bonding curve fees
        30, 50, 10, 10,  // DEX fees
        // Fair launch enabled
        true,
        3600,                        // 1 hour duration
        ethers.parseEther("10000"),  // max 10k tokens per wallet
        ethers.parseEther("0.00005") // fixed price 0.00005 ETH
    );

    const receipt = await tx.wait();
    console.log("Fair launch token created!");
}
```

---

## Buy Tokens

### Buy with ETH

```javascript
async function buyTokens(tokenAddress, ethAmount) {
    // Get estimated tokens
    const estimate = await bondingCurve.estimateTokensForETH(
        tokenAddress,
        ethers.parseEther(ethAmount)
    );
    
    console.log("Estimated tokens:", ethers.formatEther(estimate));
    
    // Calculate minimum with 10% slippage
    const minTokensOut = estimate * 90n / 100n;
    
    // Execute buy
    const tx = await factory.buyWithETH(tokenAddress, minTokensOut, {
        value: ethers.parseEther(ethAmount)
    });
    
    const receipt = await tx.wait();
    
    // Parse event for actual amounts
    const event = receipt.logs.find(log => {
        try {
            return factory.interface.parseLog(log)?.name === "TokenBought";
        } catch { return false; }
    });
    
    if (event) {
        const parsed = factory.interface.parseLog(event);
        console.log("Bought:", ethers.formatEther(parsed.args.amount), "tokens");
        console.log("Spent:", ethers.formatEther(parsed.args.price), "ETH");
    }
}

// Usage
await buyTokens("0x...", "0.1"); // Buy with 0.1 ETH
```

### Get Buy Quote

```javascript
async function getBuyQuote(tokenAddress, tokenAmount) {
    const [price, fee] = await bondingCurve.getBuyPrice(
        tokenAddress,
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

### Sell Tokens for ETH

```javascript
async function sellTokens(tokenAddress, tokenAmount) {
    const token = new ethers.Contract(tokenAddress, TOKEN_ABI.abi, signer);
    const amount = ethers.parseEther(tokenAmount);
    
    // Approve bonding curve to spend tokens
    const approveTx = await token.approve(BONDING_CURVE_ADDRESS, amount);
    await approveTx.wait();
    console.log("Approved");
    
    // Get sell quote
    const [price, fee] = await bondingCurve.getSellPrice(tokenAddress, amount);
    const netProceeds = price - fee;
    
    console.log("Expected:", ethers.formatEther(netProceeds), "ETH");
    
    // Calculate minimum with 10% slippage
    const minEthOut = netProceeds * 90n / 100n;
    
    // Execute sell
    const tx = await factory.sellToken(tokenAddress, amount, minEthOut);
    const receipt = await tx.wait();
    
    console.log("Sold successfully!");
}

// Usage
await sellTokens("0x...", "1000"); // Sell 1000 tokens
```

---

## Check Token Status

### Get Trading Stats

```javascript
async function getTokenStatus(tokenAddress) {
    const stats = await bondingCurve.getTokenStats(tokenAddress);
    
    console.log("=== Token Status ===");
    console.log("Sold:", ethers.formatEther(stats.totalSold), "tokens");
    console.log("Raised:", ethers.formatEther(stats.totalRaised), "ETH");
    console.log("Current Price:", ethers.formatEther(stats.currentPrice), "ETH");
    console.log("Trading Open:", stats.isOpen);
    
    // Progress to graduation
    const soldProgress = (Number(stats.totalSold) / Number(ethers.parseEther("400000"))) * 100;
    const raisedProgress = (Number(stats.totalRaised) / Number(ethers.parseEther("30"))) * 100;
    
    console.log("Progress (tokens):", soldProgress.toFixed(2), "%");
    console.log("Progress (ETH):", raisedProgress.toFixed(2), "%");
    
    return stats;
}
```

### Get Full Token Info

```javascript
async function getFullTokenInfo(tokenAddress) {
    const [factoryInfo, tradingData] = await Promise.all([
        factory.getTokenInfo(tokenAddress),
        bondingCurve.getTokenTradingData(tokenAddress)
    ]);
    
    return {
        name: factoryInfo.name,
        symbol: factoryInfo.symbol,
        creator: factoryInfo.creator,
        createdAt: new Date(Number(factoryInfo.createdAt) * 1000),
        isPaused: factoryInfo.isPaused,
        isVerified: factoryInfo.isVerified,
        totalSupply: ethers.formatEther(factoryInfo.totalSupply),
        sold: ethers.formatEther(tradingData.totalSold),
        raised: ethers.formatEther(tradingData.totalRaised),
        isOpen: tradingData.isOpen,
        currentPrice: ethers.formatEther(tradingData.currentPrice)
    };
}
```

---

## Creator Rewards

### Check Rewards

```javascript
async function checkRewards(creatorAddress) {
    const rewards = await feeManager.getCreatorRewards(creatorAddress);
    console.log("Pending rewards:", ethers.formatEther(rewards), "ETH");
    return rewards;
}
```

### Claim Rewards

```javascript
async function claimRewards() {
    const rewards = await feeManager.getCreatorRewards(signer.address);
    
    if (rewards === 0n) {
        console.log("No rewards to claim");
        return;
    }
    
    console.log("Claiming:", ethers.formatEther(rewards), "ETH");
    
    const tx = await feeManager.claimCreatorRewards();
    await tx.wait();
    
    console.log("Rewards claimed!");
}
```

---

## List All Tokens

```javascript
async function getAllTokens() {
    const tokens = await factory.getAllTokens();
    
    console.log("Total tokens:", tokens.length);
    
    for (const tokenAddress of tokens) {
        const info = await factory.getTokenInfo(tokenAddress);
        console.log(`${info.name} (${info.symbol}): ${tokenAddress}`);
    }
    
    return tokens;
}
```

---

## Monitor Events

```javascript
// Listen for new tokens
factory.on("TokenCreated", (token, creator, name) => {
    console.log(`New token: ${name} (${token}) by ${creator}`);
});

// Listen for trades
factory.on("TokenBought", (token, buyer, amount, price) => {
    console.log(`Buy: ${ethers.formatEther(amount)} tokens for ${ethers.formatEther(price)} ETH`);
});

factory.on("TokenSold", (token, seller, amount, price) => {
    console.log(`Sell: ${ethers.formatEther(amount)} tokens for ${ethers.formatEther(price)} ETH`);
});

// Listen for graduations
factory.on("TokenGraduated", (token, pool) => {
    console.log(`Token graduated! ${token} → Pool: ${pool}`);
});
```

---

## Error Handling

```javascript
async function safeBuy(tokenAddress, ethAmount) {
    try {
        const tx = await factory.buyWithETH(
            tokenAddress,
            0n, // No slippage protection for example
            { value: ethers.parseEther(ethAmount) }
        );
        await tx.wait();
    } catch (error) {
        if (error.message.includes("Trading is closed")) {
            console.log("Token has graduated to DEX");
        } else if (error.message.includes("Output below minimum")) {
            console.log("Slippage too high, try again");
        } else if (error.message.includes("Exceeds max buy amount")) {
            console.log("Max 1 ETH per transaction");
        } else {
            throw error;
        }
    }
}
```
