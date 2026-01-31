# Deployed Addresses

## MegaETH Mainnet

**Network**: MegaETH Mainnet  
**Chain ID**: 4326  
**Block Explorer**: [mega.etherscan.io](https://mega.etherscan.io)

**Deployed**: 2026-01-31

### Diamond Contract (Main Entry Point)

| Contract | Address |
|----------|---------|
| **Diamond** | [`0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B`](https://mega.etherscan.io/address/0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B) |

All interactions go through the Diamond address above.

### Facets

| Facet | Address |
|-------|---------|
| DiamondCutFacet | [`0x8761b74ef07feC5a5c720cAa6e349fAFa25071b2`](https://mega.etherscan.io/address/0x8761b74ef07feC5a5c720cAa6e349fAFa25071b2) |
| DiamondLoupeFacet | [`0x43B639a153e8F3569151eACb9e9E9570d1b03c1D`](https://mega.etherscan.io/address/0x43B639a153e8F3569151eACb9e9E9570d1b03c1D) |
| TokenFacet | [`0xAA2b90D09db6a71c1022d9866E340B5E1481b8Fe`](https://mega.etherscan.io/address/0xAA2b90D09db6a71c1022d9866E340B5E1481b8Fe) |
| TradingFacet | [`0x505a94d251A9cA13b2d98C62121Ca886dfCE2DdB`](https://mega.etherscan.io/address/0x505a94d251A9cA13b2d98C62121Ca886dfCE2DdB) |
| GraduationFacet | [`0xe0955cB7Eb76aa7F5a070B8b2cE60fF93000821C`](https://mega.etherscan.io/address/0xe0955cB7Eb76aa7F5a070B8b2cE60fF93000821C) |
| FeeFacet | [`0x742c3b5E9201b18CbC370e2D069c25D68221D7A4`](https://mega.etherscan.io/address/0x742c3b5E9201b18CbC370e2D069c25D68221D7A4) |
| SecurityFacet | [`0x55681cfD658B4201C0079A242D3675deD847A0B9`](https://mega.etherscan.io/address/0x55681cfD658B4201C0079A242D3675deD847A0B9) |
| AdminFacet | [`0x287f524fBC260c67bDCC05bed0cad8ca88F032A0`](https://mega.etherscan.io/address/0x287f524fBC260c67bDCC05bed0cad8ca88F032A0) |
| ERC6909Facet | [`0x28939062EE09e8B64120E0ED9cC13236C1Fc69aD`](https://mega.etherscan.io/address/0x28939062EE09e8B64120E0ED9cC13236C1Fc69aD) |
| WrapperFacet | [`0x13A3fe336c93662D4902183F3D5D11af4483C09E`](https://mega.etherscan.io/address/0x13A3fe336c93662D4902183F3D5D11af4483C09E) |

### Supporting Contracts

| Contract | Address |
|----------|---------|
| DiamondInit | [`0xa5545233977626E15492267452f8e2d9868bF427`](https://mega.etherscan.io/address/0xa5545233977626E15492267452f8e2d9868bF427) |
| WrapperImplementation | [`0xE59DB204919454895f8815e7EF57BA6bC1b6EAf2`](https://mega.etherscan.io/address/0xE59DB204919454895f8815e7EF57BA6bC1b6EAf2) |

### External Dependencies (Uniswap V3 onm Prism)

| Contract | Address |
|----------|---------|
| UniswapV3Factory | `0xef349aa6cc5e87559e716ac293845a48cadf30d5` |
| PositionManager | `0x9feaf944c518164d5d0c45f28255758acff8e987` |
| WETH | `0x4200000000000000000000000000000000000006` |

---

## Network Configuration

### Add to Wallet (MetaMask)

```
Network Name: MegaETH Mainnet
RPC URL: https://mainnet.megaeth.com/rpc
Chain ID: 4326
Currency Symbol: ETH
Block Explorer: https://mega.etherscan.io
```

### ethers.js

```javascript
const provider = new ethers.JsonRpcProvider("https://mainnet.megaeth.com/rpc");
```

---

## Contract ABIs

ABIs are included in this repo under **`abis/`** (only what third parties need):

| File | Description |
|------|--------------|
| `abis/Diamond.json` | **Public/integration ABI** – token creation, trading (buy/sell), graduation, fee claims, token data, ERC-6909, wrapper lookup. Excludes admin, security, and upgrade functions. |
| `abis/ERC20.json` | Standard ERC-20 ABI for interacting with token wrappers (balance, transfer, approve). |

### Usage

```javascript
import DiamondABI from "./abis/Diamond.json";

const diamond = new ethers.Contract(
    "0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B",
    DiamondABI,  // abis/Diamond.json is a raw ABI array
    signer
);
```

---

## Gas Estimates

| Operation | Estimated Gas |
|-----------|---------------|
| Create Token | ~50,000 |
| Buy Tokens | ~150,000 |
| Sell Tokens | ~140,000 |
| Graduation | ~600,000 |
| Collect LP Fees | ~100,000 |
| Claim Rewards | ~50,000 |
