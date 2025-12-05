# Diamond Contract

The main entry point for all launchpad operations. Built on EIP-2535 Diamond Standard.

**Address**: [`0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e`](https://megaeth-testnet-v2.blockscout.com/address/0x8Ee43427E435253Bdc94d9Ab81daeC441C03EB2e)

**Network**: MegaETH Testnet v2 (Chain ID: 6343)

## Overview

The Diamond contract is a proxy that delegates calls to specialized facets. All tokens, trading, and graduation logic is accessed through this single address.

```
┌─────────────────────────────────────────────────────────────┐
│                      Diamond Proxy                          │
│                 (Single Entry Point)                        │
├─────────────────────────────────────────────────────────────┤
│  TokenFacet    │  TradingFacet   │  GraduationFacet        │
│  FeeFacet      │  SecurityFacet  │  AdminFacet             │
│  ERC6909Facet  │  WrapperFacet   │  DiamondLoupe/Cut       │
├─────────────────────────────────────────────────────────────┤
│                   Shared Storage (Libraries)                │
│  LibToken │ LibTrading │ LibFee │ LibSecurity │ LibDEX     │
└─────────────────────────────────────────────────────────────┘
```

## Facet Addresses

| Facet | Address |
|-------|---------|
| DiamondCutFacet | [`0xD64C6adc320FF3D0B7d09F6770e3095Ed7bF1022`](https://megaeth-testnet-v2.blockscout.com/address/0xD64C6adc320FF3D0B7d09F6770e3095Ed7bF1022) |
| DiamondLoupeFacet | [`0x0edE8cCe0F0065AB48a6c8aa1108e6763a6D6453`](https://megaeth-testnet-v2.blockscout.com/address/0x0edE8cCe0F0065AB48a6c8aa1108e6763a6D6453) |
| TokenFacet | [`0x6799c57642E9F5813dBE2B01c27ce024cb417f87`](https://megaeth-testnet-v2.blockscout.com/address/0x6799c57642E9F5813dBE2B01c27ce024cb417f87) |
| TradingFacet | [`0xA52978d657dE175fD4F537f5d3bfe166c40E47d8`](https://megaeth-testnet-v2.blockscout.com/address/0xA52978d657dE175fD4F537f5d3bfe166c40E47d8) |
| GraduationFacet | [`0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE`](https://megaeth-testnet-v2.blockscout.com/address/0xc4a4771d91f6ce5732b287c21F60b2B193fdF0CE) |
| FeeFacet | [`0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb`](https://megaeth-testnet-v2.blockscout.com/address/0xE75A6cdDc836f4dB8aA1794B05e6545b89c774Bb) |
| SecurityFacet | [`0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17`](https://megaeth-testnet-v2.blockscout.com/address/0x73c1eCcc3eD0F14A720069e0B11fe9D6492e0A17) |
| AdminFacet | [`0xC1Eedf19d43D9e2EB462d982475Ac765c072d855`](https://megaeth-testnet-v2.blockscout.com/address/0xC1Eedf19d43D9e2EB462d982475Ac765c072d855) |
| ERC6909Facet | [`0x2976fD1C498070b6f7024Dc7807Ea788b8c3F67D`](https://megaeth-testnet-v2.blockscout.com/address/0x2976fD1C498070b6f7024Dc7807Ea788b8c3F67D) |
| WrapperFacet | [`0x99Ac8ccD4ed5F665330E7d5B49DB83408E8ea0Db`](https://megaeth-testnet-v2.blockscout.com/address/0x99Ac8ccD4ed5F665330E7d5B49DB83408E8ea0Db) |

## Key Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `TOTAL_SUPPLY` | 1,000,000 tokens | Fixed supply per token |
| `TOKEN_LIMIT` | 684,000 tokens | Tokens sold via bonding curve (68.4%) |
| `INITIAL_PRICE` | 0.00001 ETH | Starting price per token |
| `K` | 149585 | Bonding curve steepness constant |
| `DEFAULT_ETH_TARGET` | 30 ETH | Default ETH target for graduation |
| `MAX_BUY_AMOUNT` | 1 ETH | Maximum per transaction |
| `TOTAL_TRADING_FEE` | 1.2% (120 bps) | Total fee per trade |
| `GRADUATION_FEE_ETH` | 0.1 ETH | Flat fee on graduation |

## ERC-6909 Token Standard

All tokens are stored as ERC-6909 multi-tokens within the Diamond. Each token has:
- **Token ID** - Unique identifier within the Diamond
- **ERC-20 Wrapper** - Compatible address for DeFi integration

### ERC-6909 Functions

```solidity
// Balance of a specific token ID
function balanceOf(address owner, uint256 id) external view returns (uint256)

// Transfer tokens
function transfer(address receiver, uint256 id, uint256 amount) external returns (bool)

// Transfer from (with approval)
function transferFrom(address sender, address receiver, uint256 id, uint256 amount) external returns (bool)

// Approve operator for all tokens
function setOperator(address operator, bool approved) external returns (bool)

// Approve specific amount
function approve(address spender, uint256 id, uint256 amount) external returns (bool)
```

## Diamond Introspection

Query facets and selectors via DiamondLoupeFacet:

```solidity
// Get all facets
function facets() external view returns (Facet[] memory)

// Get selectors for a facet
function facetFunctionSelectors(address facet) external view returns (bytes4[] memory)

// Get facet for a selector
function facetAddress(bytes4 selector) external view returns (address)
```

## Upgradeability

The Diamond can be upgraded via DiamondCutFacet (owner only):

```solidity
function diamondCut(
    FacetCut[] calldata _diamondCut,
    address _init,
    bytes calldata _calldata
) external
```

This allows adding new features, fixing bugs, or optimizing gas without migrating tokens or liquidity.
