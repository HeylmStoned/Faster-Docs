# SecurityFacet

Provides security features for token launches.

**Facet Address**: [`0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17`](https://megaeth-testnet-v2.blockscout.com/address/0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17)

## Overview

SecurityFacet provides:
- **Fair Launch** - Fixed price period with per-wallet limits
- **Sniper Protection** - Delays for anti-bot measures
- **Pause Control** - Emergency token pause
- **Sell Protection** - Sells disabled by default

## Features

### Fair Launch

Allows creators to set a fixed-price period where:
- Token price is fixed (not bonding curve)
- Each wallet has a maximum purchase limit
- Prevents whale accumulation at launch

### Sniper Protection

Adds a delay period at launch to prevent:
- Bot frontrunning
- MEV exploitation
- Unfair early accumulation

### Sell Protection

**Sells are disabled by default** to prevent pump-and-dump schemes.

Admin must explicitly enable sells per token:
```solidity
diamond.setSellsEnabled(tokenAddress, true);
```

This does NOT affect DEX trading after graduation - only bonding curve sells.

## Functions

### enableFairLaunch

Enable fair launch for a token.

```solidity
function enableFairLaunch(
    address token,
    uint256 duration,
    uint256 maxPerWallet,
    uint256 fixedPrice
) external
```

**Parameters:**
- `duration` - Duration in seconds
- `maxPerWallet` - Maximum tokens per wallet
- `fixedPrice` - Fixed price per token (in wei)

**Requirements:**
- Caller must be token creator or owner
- Duration > 0
- Max per wallet > 0
- Fixed price > 0

---

### enableSniperProtection

Enable sniper protection for a token.

```solidity
function enableSniperProtection(
    address token,
    uint256 duration
) external
```

**Parameters:**
- `duration` - Protection duration in seconds

---

### pauseToken

Pause or unpause a token.

```solidity
function pauseToken(address token, bool paused) external onlyOwner
```

When paused, all transfers (including buy/sell) are blocked.

---

## View Functions

### isSniperProtectionActive

```solidity
function isSniperProtectionActive(address token) external view returns (bool)
```

### isFairLaunchActive

```solidity
function isFairLaunchActive(address token) external view returns (bool)
```

### getWalletPurchaseAmount

```solidity
function getWalletPurchaseAmount(address token, address wallet) external view returns (uint256)
```

### getFairLaunchConfig

```solidity
function getFairLaunchConfig(address token) external view returns (
    bool enabled,
    uint256 duration,
    uint256 maxPerWallet,
    uint256 fixedPrice,
    uint256 startTime
)
```

### getSniperProtectionConfig

```solidity
function getSniperProtectionConfig(address token) external view returns (
    bool enabled,
    uint256 duration,
    uint256 startTime
)
```

### getSecurityStatus

```solidity
function getSecurityStatus(address token) external view returns (
    bool isPaused,
    bool sniperProtectionActive,
    bool fairLaunchActive,
    uint256 sniperProtectionRemaining,
    uint256 fairLaunchRemaining
)
```

---

## Admin Functions

### emergencyDisableSecurity

Disable all security features for a token.

```solidity
function emergencyDisableSecurity(address token) external onlyOwner
```

Disables:
- Sniper protection
- Fair launch
- Pause state

---

## Events

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
