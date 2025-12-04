# Deployed Addresses

## MegaETH Testnet v2

**Chain ID**: 6343  
**Block Explorer**: [megaeth-testnet-v2.blockscout.com](https://megaeth-testnet-v2.blockscout.com)

### Core Contracts

| Contract | Address | Verified |
|----------|---------|----------|
| MemecoinFactory | [`0x855EF543a8d611Ee76724C2E4051E7db4947158F`](https://megaeth-testnet-v2.blockscout.com/address/0x855EF543a8d611Ee76724C2E4051E7db4947158F) | ✅ |
| BondingCurve | [`0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42`](https://megaeth-testnet-v2.blockscout.com/address/0xb60c5650DEDE1a0d6E558D2561Ba4df60CF29F42) | ✅ |
| FeeManager | [`0x194c622e62FA1C91A8A678A898F4b10E138124cD`](https://megaeth-testnet-v2.blockscout.com/address/0x194c622e62FA1C91A8A678A898F4b10E138124cD) | ✅ |
| DEXManager | [`0x98e57d776226cE97C0c850E2Af33669aa9F99416`](https://megaeth-testnet-v2.blockscout.com/address/0x98e57d776226cE97C0c850E2Af33669aa9F99416) | ✅ |
| SecurityManager | [`0xF9e0094Ac5379071970533897Af8f274F8ce2E1c`](https://megaeth-testnet-v2.blockscout.com/address/0xF9e0094Ac5379071970533897Af8f274F8ce2E1c) | ✅ |

### External Dependencies

| Contract | Address |
|----------|---------|
| Uniswap V3 Factory | `0x94996d371622304f2eb85df1eb7f328f7b317c3e` |
| Uniswap V3 Position Manager | `0x1279f3cbf01ad4f0cfa93f233464581f4051033a` |
| WETH | `0x4200000000000000000000000000000000000006` |

---

## Network Configuration

### Add to Wallet (MetaMask)

```
Network Name: MegaETH Testnet v2
RPC URL: https://carrot.megaeth.com/rpc
Chain ID: 6343
Currency Symbol: ETH
Block Explorer: https://megaeth-testnet-v2.blockscout.com
```

### Hardhat Config

```javascript
networks: {
    megaethTestnet: {
        url: "https://carrot.megaeth.com/rpc",
        chainId: 6343,
        accounts: [PRIVATE_KEY]
    }
}
```

### ethers.js

```javascript
const provider = new ethers.JsonRpcProvider("https://carrot.megaeth.com/rpc");
```

---

## Contract ABIs

ABIs are available in the repository:

```
/app/abis/
├── MemecoinFactory.json
├── BondingCurve.json
├── FeeManager.json
├── DEXManager.json
├── SecurityManager.json
└── MemecoinToken.json
```

### Usage

```javascript
import FactoryABI from "./abis/MemecoinFactory.json";

const factory = new ethers.Contract(
    "0x855EF543a8d611Ee76724C2E4051E7db4947158F",
    FactoryABI.abi,
    signer
);
```

---

## Gas Estimates

| Operation | Estimated Gas |
|-----------|---------------|
| Create Token | ~2,500,000 |
| Buy Tokens | ~200,000 |
| Sell Tokens | ~180,000 |
| Graduation | ~800,000 |
| Claim Rewards | ~50,000 |

---

## Testnet Faucet

Get testnet ETH from the MegaETH faucet:
- [testnet.megaeth.com](https://testnet.megaeth.com)
