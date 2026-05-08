# OKX Yield Pilot — CLI Command Reference

Detailed parameter tables, return field schemas, and usage examples for all `onchainos` commands used by the Yield Pilot skill.

## DeFi Commands (Position Discovery & Execution)

### onchainos defi positions

List all DeFi positions for a wallet address across chains.

```bash
onchainos defi positions --address <addr> --chains <chains>
```

| Param | Required | Description |
|---|---|---|
| `--address` | Yes | Wallet address |
| `--chains` | Yes | Comma-separated chain names (e.g., `"xlayer,ethereum,bsc"`) |

**Return fields**:

| Field | Type | Description |
|---|---|---|
| `platformId` | String | Protocol platform identifier |
| `platformName` | String | Human-readable protocol name |
| `chainId` | String | Chain identifier |
| `investmentId` | String | Product identifier for follow-up queries |
| `tokenSymbol` | String | Position token symbol |
| `coinAmount` | String | Position size in human-readable units |
| `totalValue` | String | Position value in USD |

**Address format rules**:
- EVM address (`0x…`) → only pass EVM chains: `ethereum,bsc,polygon,arbitrum,base,xlayer`
- Solana address (base58) → only pass `solana`
- **Never mix** — API returns error 84019

---

### onchainos defi position-detail

Get detailed position info including rewards and precision data.

```bash
onchainos defi position-detail --address <addr> --chain <chain> --platform-id <pid>
```

| Param | Required | Description |
|---|---|---|
| `--address` | Yes | Wallet address |
| `--chain` | Yes | Single chain name |
| `--platform-id` | Yes | Platform ID from `defi positions` |

**Return fields**:

| Field | Type | Description |
|---|---|---|
| `investmentId` | String | Product ID for invest/withdraw/collect |
| `coinAmount` | String | Current position amount (human-readable) |
| `tokenPrecision` | Number | Token decimal precision |
| `rewardTokens[]` | Array | Pending reward tokens |
| `rewardTokens[].symbol` | String | Reward token symbol |
| `rewardTokens[].amount` | String | Pending reward amount |
| `tokenId` | String | NFT token ID (V3 positions only) |
| `platformId` | String | Platform identifier |

> **CRITICAL**: Must be called **fresh** before every `collect` or `withdraw`. Position data changes after each operation.

---

### onchainos defi detail

Get full product details including APY and TVL.

```bash
onchainos defi detail --investment-id <id>
```

| Param | Required | Description |
|---|---|---|
| `--investment-id` | Yes | Investment ID from position data |

**Return fields**:

| Field | Type | Description |
|---|---|---|
| `rate` | String | Current APY as decimal (multiply by 100 for %) |
| `tvl` | String | Total value locked in USD |
| `underlyingToken[]` | Array | Base tokens for the pool |
| `underlyingToken[].tokenAddress` | String | Token contract address |
| `underlyingToken[].tokenSymbol` | String | Token symbol |
| `underlyingToken[].coinAmount` | String | Amount in pool |
| `investmentName` | String | Human-readable product name |
| `platformName` | String | Protocol name |
| `chainId` | String | Chain identifier |

---

### onchainos defi collect

Claim rewards from a DeFi position. Returns unsigned calldata.

```bash
onchainos defi collect --address <addr> --chain <chain> --reward-type <type> --investment-id <id> --platform-id <pid>
```

| Param | Required | Description |
|---|---|---|
| `--address` | Yes | Wallet address |
| `--chain` | Yes | Chain name |
| `--reward-type` | Yes | Reward type (see table below) |
| `--investment-id` | Conditional | Required for most reward types |
| `--platform-id` | Conditional | Required for platform/investment rewards |
| `--token-id` | Conditional | Required for V3 fee collection |

**Reward types**:

| rewardType | When to use |
|---|---|
| `REWARD_INVESTMENT` | Product mining/staking rewards |
| `REWARD_PLATFORM` | Protocol-level rewards |
| `V3_FEE` | V3 trading fee collection |

**Return**: `dataList[]` — array of unsigned calldata steps to execute sequentially.

---

### onchainos defi invest

Deposit into a DeFi product. Returns unsigned calldata.

```bash
onchainos defi invest --investment-id <id> --address <addr> --token <symbol_or_addr> --amount <minimal_units> [--chain <chain>]
```

| Param | Required | Description |
|---|---|---|
| `--investment-id` | Yes | Target product ID |
| `--address` | Yes | Wallet address |
| `--token` | Yes | Deposit token symbol or contract address |
| `--amount` | Yes | Amount in **minimal units** (integer) |
| `--chain` | No | Chain name |
| `--slippage` | No | Slippage tolerance (default: `"0.01"`) |

**Amount conversion**: `amount = floor(userAmount × 10^tokenPrecision)`
Example: 100 USDC (precision=6) → `--amount 100000000`

**Return**: `dataList[]` — APPROVE + DEPOSIT steps.

---

### onchainos defi withdraw

Withdraw from a DeFi position. Returns unsigned calldata.

```bash
onchainos defi withdraw --investment-id <id> --address <addr> --chain <chain> [--ratio <0-1>] [--amount <minimal_units>]
```

| Param | Required | Description |
|---|---|---|
| `--investment-id` | Yes | Product ID |
| `--address` | Yes | Wallet address |
| `--chain` | Yes | Chain name |
| `--ratio` | One of | Exit ratio (e.g., `1` for full, `0.5` for half) |
| `--amount` | One of | Partial exit in minimal units |
| `--platform-id` | Conditional | Required for some protocols |
| `--token-id` | Conditional | V3 NFT token ID |

**Return**: `dataList[]` — withdrawal calldata steps.

---

### onchainos defi rate-chart

Historical APY chart data for trend analysis.

```bash
onchainos defi rate-chart --investment-id <id> [--time-range <range>]
```

| Param | Required | Default | Description |
|---|---|---|---|
| `--investment-id` | Yes | — | Product ID |
| `--time-range` | No | `WEEK` | `WEEK`, `MONTH`, `SEASON` (3mo), `YEAR` |

**Return fields**: `timestamp`, `rate` (APY), `bonusRate`, `limitValue` (1=peak, -1=trough)

---

### onchainos defi tvl-chart

Historical TVL chart data for liquidity analysis.

```bash
onchainos defi tvl-chart --investment-id <id> [--time-range <range>]
```

| Param | Required | Default | Description |
|---|---|---|---|
| `--investment-id` | Yes | — | Product ID |
| `--time-range` | No | `WEEK` | `WEEK`, `MONTH`, `SEASON`, `YEAR` |

**Return fields**: `chartVos[]` with `timestamp`, `tvl` (USD), `limitValue`

---

## Swap Commands (Reward Conversion)

### onchainos swap quote

Get swap quote for reward token conversion.

```bash
onchainos swap quote --from <addr> --to <addr> --readable-amount <amount> --chain <chain>
```

| Param | Required | Description |
|---|---|---|
| `--from` | Yes | Source token contract address |
| `--to` | Yes | Destination token contract address |
| `--readable-amount` | Yes | Human-readable amount (e.g., `"12.5"`) |
| `--chain` | Yes | Chain name |

**Return fields**: `toTokenAmount`, `fromTokenAmount`, `estimateGasFee`, `priceImpactPercent`, `fromToken.isHoneyPot`, `toToken.isHoneyPot`, `fromToken.taxRate`

---

### onchainos swap execute

One-shot swap: quote → approve → sign → broadcast.

```bash
onchainos swap execute --from <addr> --to <addr> --readable-amount <amount> --chain <chain> --wallet <addr> [--slippage <pct>] [--mev-protection]
```

| Param | Required | Default | Description |
|---|---|---|---|
| `--from` | Yes | — | Source token address |
| `--to` | Yes | — | Destination token address |
| `--readable-amount` | Yes | — | Human-readable sell amount |
| `--chain` | Yes | — | Chain name |
| `--wallet` | Yes | — | Wallet address |
| `--slippage` | No | autoSlippage | Slippage % |
| `--mev-protection` | No | — | Enable MEV protection (EVM) |
| `--tips` | No | — | Jito tips in SOL (Solana MEV) |

**Return**: `swapTxHash`, `fromAmount`, `toAmount`, `priceImpact`, `gasUsed`

---

## Wallet Commands (Execution & Auth)

### onchainos wallet status

Check login state and active account.

```bash
onchainos wallet status
```

**Return**: `loggedIn`, `email`, `accountId`, `accountName`

---

### onchainos wallet addresses

Get wallet addresses grouped by chain category.

```bash
onchainos wallet addresses [--chain <chain>]
```

**Return**: EVM address, Solana address, X Layer address

---

### onchainos wallet contract-call

Execute smart contract interaction via TEE signing.

```bash
# EVM
onchainos wallet contract-call --to <contract> --chain <chainIndex> --input-data <calldata> [--value <ui_units>] [--biz-type defi]

# Solana
onchainos wallet contract-call --to <program_id> --chain 501 --unsigned-tx <serializedData> [--biz-type defi]
```

| Param | Required | Description |
|---|---|---|
| `--to` | Yes | Target contract address |
| `--chain` | Yes | Chain index (1=ETH, 56=BSC, 196=XLayer, 501=SOL) |
| `--input-data` | EVM | Hex calldata for EVM chains |
| `--unsigned-tx` | Solana | Base58 serialized transaction for Solana |
| `--value` | No | Native token value in UI units (default: `"0"`) |
| `--biz-type` | No | Business type tag (use `defi` for DeFi operations) |

---

## Portfolio Commands (Balance Verification)

### onchainos portfolio all-balances

Check all token balances before reinvesting.

```bash
onchainos portfolio all-balances --address <addr> --chains <chains>
```

| Param | Required | Description |
|---|---|---|
| `--address` | Yes | Wallet address |
| `--chains` | Yes | Comma-separated chain names |

**Return**: Token list with `tokenSymbol`, `balance` (UI units), `tokenContractAddress`, `usdValue`

---

## Token Commands (Address Resolution)

### onchainos token search

Resolve token symbol to contract address.

```bash
onchainos token search --query <symbol> --chains <chain>
```

| Param | Required | Description |
|---|---|---|
| `--query` | Yes | Token symbol or name |
| `--chains` | Yes | Chain name |

**Return**: `tokenContractAddress`, `decimal`, `tokenSymbol`, `tokenName`

---

### onchainos token price-info

Get current token price for IL calculations.

```bash
onchainos token price-info --address <token_addr> --chain <chain>
```

**Return**: `price` (USD), `priceChange24H`, `marketCap`, `volume24H`

---

## Native Token Addresses

| Chain | Native Token Address |
|---|---|
| EVM (Ethereum, BSC, X Layer, etc.) | `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` |
| Solana | `11111111111111111111111111111111` |

## Common Stablecoin Shorthand

The CLI resolves these symbols automatically: `usdc`, `usdt`, `dai`
For other tokens, use `token search` to resolve addresses.
