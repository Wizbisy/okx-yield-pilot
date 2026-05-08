# OKX Yield Pilot — Chain Reference

## Supported Chains

| Chain | Name string | chainIndex | Gas profile |
|---|---|---|---|
| X Layer | `xlayer` | `196` | **Zero gas** — preferred for rebalancing |
| Solana | `solana` | `501` | Low gas (~$0.001/tx) |
| Ethereum | `ethereum` | `1` | Variable (batch to save gas) |
| BSC | `bsc` | `56` | Low gas (~$0.05/tx) |
| Base | `base` | `8453` | Low gas (~$0.01/tx) |

> Use chain **name** (string) for `defi`/`swap`/`portfolio` commands.
> Use chain **index** (number) for `wallet contract-call --chain`.

**X Layer priority**: Always recommend X Layer for rebalancing and intermediate swaps due to zero gas. When compounding on Ethereum, consider batching or bridging rewards to X Layer first.

## Native Token Addresses

| Chain | Native Token | Address |
|---|---|---|
| Ethereum | ETH | `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` |
| BSC | BNB | `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` |
| Base | ETH | `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` |
| X Layer | OKB | `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` |
| Solana | SOL | `11111111111111111111111111111111` |

> `0xeeee...eeee` is the EVM convention for the native gas token — a sentinel address used by the OKX API, not a real contract. Do **not** use it on Solana.

## Well-Known Token Addresses

### Solana

| Token | Address | Decimals |
|---|---|---|
| USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | 6 |
| USDT | `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB` | 6 |
| RAY | `4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R` | 6 |

### EVM (Ethereum)

| Token | Address | Decimals |
|---|---|---|
| USDC | `0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` | 6 |
| USDT | `0xdac17f958d2ee523a2206206994597c13d831ec7` | 6 |

### BSC

| Token | Address | Decimals |
|---|---|---|
| USDC | `0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d` | 18 |
| USDT | `0x55d398326f99059ff775485246999027b3197955` | 18 |

> The CLI resolves `usdc`, `usdt`, `dai` automatically. Use `token search` for other tokens. Addresses differ per chain — always verify when unsure.