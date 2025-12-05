# Events

All events emitted by the Diamond contract facets.

## TokenFacet

### TokenCreated

Emitted when a new ERC-6909 token is created with its ERC-20 wrapper.

```solidity
event TokenCreated(
    uint256 indexed id,
    address indexed creator,
    string name,
    string symbol,
    uint256 totalSupply,
    address wrapper
);
```

### Transfer (ERC-6909)

Emitted on ERC-6909 token transfers.

```solidity
event Transfer(
    address caller,
    address indexed sender,
    address indexed receiver,
    uint256 indexed id,
    uint256 amount
);
```

### Approval (ERC-6909)

```solidity
event Approval(
    address indexed owner,
    address indexed spender,
    uint256 indexed id,
    uint256 amount
);
```

### OperatorSet (ERC-6909)

```solidity
event OperatorSet(
    address indexed owner,
    address indexed operator,
    bool approved
);
```

---

## TradingFacet

### TokenBought

Emitted when tokens are purchased on the bonding curve.

```solidity
event TokenBought(
    address indexed token,
    address indexed buyer,
    uint256 amount,
    uint256 price,
    uint256 fee
);
```

### TokenSold

Emitted when tokens are sold back to the bonding curve.

```solidity
event TokenSold(
    address indexed token,
    address indexed seller,
    uint256 amount,
    uint256 price,
    uint256 fee
);
```

### SellsEnabledChanged

Emitted when sells are enabled/disabled for a token.

```solidity
event SellsEnabledChanged(
    address indexed token,
    bool enabled
);
```

### TradingClosed

Emitted when bonding curve trading closes (graduation triggered).

```solidity
event TradingClosed(
    address indexed token,
    uint256 totalSold,
    uint256 totalRaised
);
```

---

## GraduationFacet

### TokenGraduated

Emitted when a token graduates to Prism-Dex(UniV3).

```solidity
event TokenGraduated(
    address indexed token,
    address indexed pool,
    uint256 positionId,
    uint256 tokenAmount,
    uint256 ethAmount
);
```

### PoolCreated

Emitted when a Prism-Dex(UniV3) pool is created.

```solidity
event PoolCreated(
    address indexed token,
    address indexed pool
);
```

### PositionCreated

Emitted when an LP position is minted.

```solidity
event PositionCreated(
    address indexed token,
    uint256 positionId,
    uint128 liquidity
);
```

### FeesCollected

Emitted when LP fees are collected from Prism-Dex(UniV3).

```solidity
event FeesCollected(
    address indexed token,
    uint256 amount0,
    uint256 amount1
);
```

---

## FeeFacet

### FeesDistributed

Emitted when trading fees are distributed.

```solidity
event FeesDistributed(
    address indexed token,
    address indexed creator,
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee
);
```

### CreatorRewardsClaimed

Emitted when a creator claims their accumulated rewards.

```solidity
event CreatorRewardsClaimed(
    address indexed creator,
    uint256 amount
);
```

### PlatformFeesWithdrawn

```solidity
event PlatformFeesWithdrawn(uint256 amount);
```

### BadBunnzFeesWithdrawn

```solidity
event BadBunnzFeesWithdrawn(uint256 amount);
```

### BuybackFeesWithdrawn

```solidity
event BuybackFeesWithdrawn(uint256 amount);
```

### GraduationFeePaid

```solidity
event GraduationFeePaid(
    address indexed token,
    uint256 amount
);
```

---

## SecurityFacet

### SniperProtectionEnabled

```solidity
event SniperProtectionEnabled(
    address indexed token,
    uint256 duration
);
```

### FairLaunchEnabled

```solidity
event FairLaunchEnabled(
    address indexed token,
    uint256 duration,
    uint256 maxPerWallet,
    uint256 fixedPrice
);
```

### TokenPaused

```solidity
event TokenPaused(
    address indexed token,
    bool paused
);
```

### SecurityDisabled

```solidity
event SecurityDisabled(
    address indexed token
);
```

---

## AdminFacet

### OwnershipTransferred

```solidity
event OwnershipTransferred(
    address indexed previousOwner,
    address indexed newOwner
);
```

### FeeWalletsUpdated

```solidity
event FeeWalletsUpdated(
    address platform,
    address buyback
);
```

### EmergencyWithdraw

```solidity
event EmergencyWithdraw(
    address indexed token,
    uint256 amount
);
```

---

## ERC-20 Wrapper Events

Each ERC-20 wrapper emits standard ERC-20 events:

### Transfer

```solidity
event Transfer(
    address indexed from,
    address indexed to,
    uint256 value
);
```

### Approval

```solidity
event Approval(
    address indexed owner,
    address indexed spender,
    uint256 value
);
```

---

## Listening to Events (JavaScript)

```javascript
// Listen for new tokens
diamond.on("TokenCreated", (tokenId, wrapper, creator, name, symbol) => {
    console.log(`New token: ${name} (${symbol})`);
    console.log(`  Wrapper: ${wrapper}`);
    console.log(`  Creator: ${creator}`);
});

// Listen for trades
diamond.on("TokenBought", (token, buyer, amount, price, fee) => {
    console.log(`Buy: ${ethers.formatEther(amount)} tokens`);
    console.log(`  Price: ${ethers.formatEther(price)} ETH`);
});

diamond.on("TokenSold", (token, seller, amount, price, fee) => {
    console.log(`Sell: ${ethers.formatEther(amount)} tokens`);
});

// Listen for graduations
diamond.on("TokenGraduated", (token, pool, positionId, tokenAmount, ethAmount) => {
    console.log(`Token graduated to Prism-Dex(UniV3)!`);
    console.log(`  Pool: ${pool}`);
    console.log(`  Liquidity: ${ethers.formatEther(tokenAmount)} tokens + ${ethers.formatEther(ethAmount)} ETH`);
});

// Listen for reward claims
diamond.on("CreatorRewardsClaimed", (creator, amount) => {
    console.log(`Creator ${creator} claimed ${ethers.formatEther(amount)} ETH`);
});

// Historical events
const events = await diamond.queryFilter(
    diamond.filters.TokenCreated(),
    fromBlock,
    toBlock
);
```
