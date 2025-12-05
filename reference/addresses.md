# Deployed Addresses

## MegaETH Testnet v2

**Chain ID**: 6343  
**Block Explorer**: [megaeth-testnet-v2.blockscout.com](https://megaeth-testnet-v2.blockscout.com)

**Deployed**: December 5, 2025 | All contracts verified ✅ | Fixed 20% DEX platform fee

### Diamond Contract (Main Entry Point)

| Contract | Address | Verified |
|----------|---------|----------|
| **Diamond** | [`0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e`](https://megaeth-testnet-v2.blockscout.com/address/0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e) | ✅ |

### Facets

| Facet | Address | Verified |
|-------|---------|----------|
| DiamondCutFacet | [`0xD64C6adc320FF3D0B7d09F6770e3095Ed7bF1022`](https://megaeth-testnet-v2.blockscout.com/address/0xD64C6adc320FF3D0B7d09F6770e3095Ed7bF1022) | ✅ |
| DiamondLoupeFacet | [`0x0edE8cCe0F0065AB48a6c8aa1108e6763a6D6453`](https://megaeth-testnet-v2.blockscout.com/address/0x0edE8cCe0F0065AB48a6c8aa1108e6763a6D6453) | ✅ |
| TokenFacet | [`0x6799c57642E9F5813dBE2B01c27ce024cb417f87`](https://megaeth-testnet-v2.blockscout.com/address/0x6799c57642E9F5813dBE2B01c27ce024cb417f87) | ✅ |
| TradingFacet | [`0xA52978d657dE175fD4F537f5d3bfe166c40E47d8`](https://megaeth-testnet-v2.blockscout.com/address/0xA52978d657dE175fD4F537f5d3bfe166c40E47d8) | ✅ |
| GraduationFacet | [`0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE`](https://megaeth-testnet-v2.blockscout.com/address/0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE) | ✅ |
| FeeFacet | [`0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb`](https://megaeth-testnet-v2.blockscout.com/address/0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb) | ✅ |
| SecurityFacet | [`0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17`](https://megaeth-testnet-v2.blockscout.com/address/0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17) | ✅ |
| AdminFacet | [`0xC1Eedf19d43D9e2EB462d982475Ac765c072d855`](https://megaeth-testnet-v2.blockscout.com/address/0xC1Eedf19d43D9e2EB462d982475Ac765c072d855) | ✅ |
| ERC6909Facet | [`0x2976fD1C498070b6f7024Dc7807Ea788b8c3F67D`](https://megaeth-testnet-v2.blockscout.com/address/0x2976fD1C498070b6f7024Dc7807Ea788b8c3F67D) | ✅ |
| WrapperFacet | [`0x99Ac8ccD4ed5F665330E7d5B49DB83408E8ea0Db`](https://megaeth-testnet-v2.blockscout.com/address/0x99Ac8ccD4ed5F665330E7d5B49DB83408E8ea0Db) | ✅ |

### Supporting Contracts

| Contract | Address | Verified |
|----------|---------|----------|
| DiamondInit | [`0x90dbE1608DcDdbD7407ccaC1005121b63201F7D0`](https://megaeth-testnet-v2.blockscout.com/address/0x90dbE1608DcDdbD7407ccaC1005121b63201F7D0) | ✅ |
| WrapperImpl | [`0x06aA00B602E7679cD25782aCe884Fb92f8F48b36`](https://megaeth-testnet-v2.blockscout.com/address/0x06aA00B602E7679cD25782aCe884Fb92f8F48b36) | ✅ |

### External Dependencies (Prism-Dex)

| Contract | Address |
|----------|---------|
| Prism-Dex(UniV3) Factory | `0x94996d371622304f2eb85df1eb7f328f7b317c3e` |
| Position Manager | `0x1279f3cbf01ad4f0cfa93f233464581f4051033a` |
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

The Diamond ABI combines all facet ABIs. Available in the repository:

```
/abis/
├── Diamond.json          # Combined ABI for all facets
├── TokenFacet.json
├── TradingFacet.json
├── GraduationFacet.json
├── FeeFacet.json
├── SecurityFacet.json
├── AdminFacet.json
├── ERC6909Facet.json
├── WrapperFacet.json
└── ERC20.json            # Standard ERC-20 for wrappers
```

### Usage

```javascript
import DiamondABI from "./abis/Diamond.json";

const diamond = new ethers.Contract(
    "0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e",
    DiamondABI.abi,
    signer
);
```

---

## Gas Estimates

| Operation | Estimated Gas | Notes |
|-----------|---------------|-------|
| Create Token | ~50,000 | 90%+ savings vs traditional ERC-20 |
| Buy Tokens | ~150,000 | |
| Sell Tokens | ~140,000 | Requires sells enabled |
| Graduation | ~600,000 | Auto-triggered at 30 ETH |
| Collect LP Fees | ~100,000 | Anyone can call |
| Claim Rewards | ~50,000 | |

---

## Testnet Faucet

Get testnet ETH from the MegaETH faucet:
- [testnet.megaeth.com](https://testnet.megaeth.com)
