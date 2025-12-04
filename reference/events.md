# Events

All events emitted by the launchpad contracts.

## MemecoinFactory

### TokenCreated

Emitted when a new token is created.

```solidity
event TokenCreated(
    address indexed token,
    address indexed creator,
    string name
);
```

### TokenBought

Emitted when tokens are purchased.

```solidity
event TokenBought(
    address indexed token,
    address indexed buyer,
    uint256 amount,
    uint256 price
);
```

### TokenSold

Emitted when tokens are sold.

```solidity
event TokenSold(
    address indexed token,
    address indexed seller,
    uint256 amount,
    uint256 price
);
```

### TokenGraduated

Emitted when a token graduates to Uniswap V3.

```solidity
event TokenGraduated(
    address indexed token,
    address indexed pool
);
```

### TokenPaused

Emitted when a token is paused/unpaused.

```solidity
event TokenPaused(
    address indexed token,
    bool paused
);
```

### TokenVerified

Emitted when a token is verified/unverified.

```solidity
event TokenVerified(
    address indexed token,
    bool verified
);
```

### TargetUpdated

Emitted when graduation target is changed.

```solidity
event TargetUpdated(
    uint256 oldTarget,
    uint256 newTarget
);
```

### GraduationFailed

Emitted when auto-graduation fails.

```solidity
event GraduationFailed(
    address indexed token,
    string reason
);
```

---

## BondingCurve

### TokenBought

```solidity
event TokenBought(
    address indexed token,
    address indexed buyer,
    uint256 amount,
    uint256 price
);
```

### TokenSold

```solidity
event TokenSold(
    address indexed token,
    address indexed seller,
    uint256 amount,
    uint256 price
);
```

### TokenTradingDataUpdated

Emitted after each trade with updated stats.

```solidity
event TokenTradingDataUpdated(
    address indexed token,
    uint256 totalSold,
    uint256 totalRaised,
    bool isOpen
);
```

---

## FeeManager

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

Emitted when a creator claims rewards.

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

### FeeConfigSet

```solidity
event FeeConfigSet(
    address indexed token,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee
);
```

### DEXFeeConfigSet

```solidity
event DEXFeeConfigSet(
    address indexed token,
    uint256 platformFee,
    uint256 creatorFee,
    uint256 badBunnzFee,
    uint256 buybackFee
);
```

---

## DEXManager

### PoolCreated

```solidity
event PoolCreated(
    address indexed token,
    address indexed pool
);
```

### PositionCreated

```solidity
event PositionCreated(
    address indexed token,
    uint256 positionId
);
```

### TokenGraduated

```solidity
event TokenGraduated(
    address indexed token,
    address indexed pool,
    uint256 positionId
);
```

### FeesCollected

```solidity
event FeesCollected(
    address indexed token,
    uint256 amount0,
    uint256 amount1
);
```

---

## SecurityManager

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
    uint256 maxPerWallet
);
```

### TokenPaused

```solidity
event TokenPaused(
    address indexed token,
    bool paused
);
```

### SecurityConfigUpdated

```solidity
event SecurityConfigUpdated(
    address indexed token,
    bool sniperProtection,
    bool fairLaunch
);
```

---

## MemecoinToken

### MetadataUpdated

```solidity
event MetadataUpdated(
    string description,
    string imageUrl
);
```

### TokenPaused

```solidity
event TokenPaused(bool paused);
```

### TokenVerified

```solidity
event TokenVerified(bool verified);
```

### TradeExecuted

```solidity
event TradeExecuted(
    address indexed buyer,
    uint256 amount,
    uint256 price
);
```

### FactoryUpdated

```solidity
event FactoryUpdated(
    address indexed oldFactory,
    address indexed newFactory
);
```

---

## Listening to Events (JavaScript)

```javascript
// Single event
factory.on("TokenCreated", (token, creator, name) => {
    console.log(`New token: ${name}`);
});

// Multiple events
const filter = factory.filters.TokenBought();
factory.on(filter, (token, buyer, amount, price) => {
    console.log(`Trade on ${token}`);
});

// Historical events
const events = await factory.queryFilter(
    factory.filters.TokenCreated(),
    fromBlock,
    toBlock
);
```
