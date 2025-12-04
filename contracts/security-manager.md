# SecurityManager

Provides security features for token launches.

**Address**: `0xF9e0094Ac5379071970533897Af8f274F8ce2E1c`

## Overview

SecurityManager provides:
- **Fair Launch** - Fixed price period with per-wallet limits
- **Sniper Protection** - Delays for anti-bot measures
- **Pause Control** - Emergency token pause

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

### Pause Control

Emergency pause functionality to:
- Stop trading on compromised tokens
- Prevent exploits
- Allow investigation time

## Functions

### enableFairLaunch

Enable fair launch for a token.

```solidity
function enableFairLaunch(
    address _token,
    uint256 _duration,
    uint256 _maxPerWallet,
    uint256 _fixedPrice
) external
```

**Parameters:**
- `_duration` - Duration in seconds
- `_maxPerWallet` - Maximum tokens per wallet
- `_fixedPrice` - Fixed price per token (in wei)

**Requirements:**
- Caller must be factory
- Duration > 0
- Max per wallet > 0
- Fixed price > 0

---

### enableSniperProtection

Enable sniper protection for a token.

```solidity
function enableSniperProtection(
    address _token,
    uint256 _duration
) external onlyFactory
```

**Parameters:**
- `_duration` - Protection duration in seconds

---

### pauseToken

Pause or unpause a token.

```solidity
function pauseToken(address _token, bool _paused) external onlyFactoryOrOwner
```

---

### validateBuy

Validate a buy transaction. Called by Factory before executing buys.

```solidity
function validateBuy(
    address _token,
    address _buyer,
    uint256 _amount
) external view returns (bool)
```

**Checks:**
1. Token not paused
2. Fair launch limits (if active)

---

### validateSell

Validate a sell transaction.

```solidity
function validateSell(
    address _token,
    address _seller,
    uint256 _amount
) external view returns (bool)
```

**Checks:**
1. Token not paused

---

### recordWalletPurchase

Record a wallet's purchase for fair launch tracking.

```solidity
function recordWalletPurchase(
    address _token, 
    address _wallet, 
    uint256 _amount
) external onlyFactory
```

---

## View Functions

### isSniperProtectionActive

```solidity
function isSniperProtectionActive(address _token) external view returns (bool)
```

### isFairLaunchActive

```solidity
function isFairLaunchActive(address _token) external view returns (bool)
```

### getWalletPurchaseAmount

```solidity
function getWalletPurchaseAmount(address _token, address _wallet) external view returns (uint256)
```

### getFairLaunchConfig

```solidity
function getFairLaunchConfig(address _token) external view returns (
    bool enabled,
    uint256 duration,
    uint256 maxPerWallet,
    uint256 fixedPrice,
    uint256 startTime
)
```

### getSniperProtectionConfig

```solidity
function getSniperProtectionConfig(address _token) external view returns (
    bool enabled,
    uint256 duration,
    uint256 startTime
)
```

### getSecurityStatus

```solidity
function getSecurityStatus(address _token) external view returns (
    bool isPaused,
    bool sniperProtectionActive,
    bool fairLaunchActive,
    uint256 sniperProtectionRemaining,
    uint256 fairLaunchRemaining
)
```

### getSniperProtectionRemainingTime

```solidity
function getSniperProtectionRemainingTime(address _token) external view returns (uint256)
```

### getFairLaunchRemainingTime

```solidity
function getFairLaunchRemainingTime(address _token) external view returns (uint256)
```

---

## Admin Functions

### emergencyDisableSecurity

Disable all security features for a token.

```solidity
function emergencyDisableSecurity(address _token) external onlyOwner
```

Disables:
- Sniper protection
- Fair launch
- Pause state
