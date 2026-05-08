# Pre-flight Checks

Run these checks at the start of every workflow. Do NOT skip.

## 1. Wallet Login

```
onchainos wallet status
```

- If `loggedIn: false` → tell user: "You need to log in first. Run `onchainos wallet login` or say 'log me in'."
- If `loggedIn: true` → continue

## 2. Address Resolution

```
onchainos wallet addresses
```

- Store EVM address (`0x…`) for: Ethereum, BSC, Base, X Layer
- Store Solana address (base58) for: Solana
- **NEVER** pass an EVM address to `--chains solana` or vice versa

## 3. Connectivity Check

Try a lightweight query to verify API access:

```
onchainos token price-info --address 0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee --chain ethereum
```

- If returns price data → API is healthy ✅
- If error 50011 (rate limit) → wait 5 seconds and retry
- If network error → inform user: "Unable to reach onchainOS API. Check your connection."

## 4. Active Positions

```
onchainos defi positions --address <evm_addr> --chains "xlayer,ethereum,bsc,base"
onchainos defi positions --address <sol_addr> --chains "solana"
```

- If zero positions found → inform user: "No active yield farming positions found. Would you like to explore DeFi products?" (→ route to `okx-defi-invest`)
- If positions found → continue to requested workflow
