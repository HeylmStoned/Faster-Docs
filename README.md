# Memecoin Launchpad

Welcome to the **Memecoin Launchpad** documentation - a pump.fun-style token launchpad built on MegaETH.

## What is Memecoin Launchpad?

A decentralized platform where anyone can create and trade memecoins with:

- **Bonding Curve Trading** - Tokens start on an exponential bonding curve
- **Auto-Graduation** - Tokens automatically graduate to Uniswap V3 when targets are met
- **Fair Launch Options** - Anti-sniper and per-wallet limits
- **Creator Rewards** - Earn fees from your token's trading activity

## How It Works

```
1. CREATE → Token minted, trading opens on bonding curve
2. TRADE  → Users buy/sell, price increases exponentially  
3. GRADUATE → At 30 ETH raised or 400k tokens sold, auto-lists on Uniswap V3
4. DEX → Token trades freely, LP fees distributed
```

## Quick Links

- [Architecture Overview](architecture.md)
- [Getting Started](integration/getting-started.md)
- [Contract Reference](contracts/factory.md)
- [Deployed Addresses](reference/addresses.md)

## Network

**MegaETH Testnet v2** (Chain ID: 6343)

| Contract | Address |
|----------|---------|
| MemecoinFactory | `0x855EF543a8d611Ee76724C2E4051E7db4947158F` |
| BondingCurve | `0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42` |
| FeeManager | `0x194c622e62FA1C91A8A678A898F4b10E138124cD` |
| DEXManager | `0x98e57d776226cE97C0c850E2Af33669aa9F99416` |
| SecurityManager | `0xF9e0094Ac5379071970533897Af8f274F8ce2E1c` |

**Block Explorer**: [megaeth-testnet-v2.blockscout.com](https://megaeth-testnet-v2.blockscout.com)
