# Diamond Contract

The main entry point for all launchpad operations. Built on EIP-2535 Diamond Standard.

**Address**: [`0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B`](https://mega.etherscan.io/address/0xabFf1341b5aF1D71394D44ad84E07d02ab3fbd4B)

**Network**: MegaETH Mainnet (Chain ID: 4326)

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
