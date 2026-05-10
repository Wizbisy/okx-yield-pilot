# OKX Yield Pilot — Worked Examples

## Example 1: Compound SOL-USDC on Solana

User says: *"Compound my Solana farming rewards"*

```
Step 1 — Resolve address:
  onchainos wallet status
  → loggedIn: true, accountName: "Main"
  onchainos wallet addresses
  → Solana: 5xYZ...abc

Step 2 — Fetch positions: 
  onchainos defi positions --address 5xYZ...abc --chains solana
  → platformId: "raydium_123", investmentId: "8502", tokenSymbol: "SOL-USDC"

Step 3 — Get position details (FRESH):
  onchainos defi position-detail --address 5xYZ...abc --chain solana --platform-id raydium_123
  → coinAmount: "18.5", tokenPrecision: 9
  → rewardTokens: [{ symbol: "RAY", amount: "12.3" }]
  → investmentId: "8502", platformId: "raydium_123"

Step 4 — Check if worth compounding:
  onchainos token price-info --address 4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R --chain solana
  → price: $2.15
  → reward_usd = 12.3 × $2.15 = $26.45
  → gas_estimate = $0.002 (Solana)
  → $26.45 >> $0.004 → PROCEED ✅

Step 5 — Claim rewards:
  onchainos defi collect \
    --address 5xYZ...abc --chain solana \
    --reward-type REWARD_INVESTMENT \
    --investment-id 8502 --platform-id raydium_123
  → Returns dataList[0]: { to: "...", serializedData: "..." }

Step 6 — Sign & broadcast claim:
  onchainos wallet contract-call \
    --to <dataList[0].to> --chain 501 \
    --unsigned-tx <dataList[0].serializedData> \
    --biz-type defi
  → txHash: "4xAbc..."
  ⚠️ "Sign within 60 seconds — Solana transactions expire."

Step 7 — Swap RAY → USDC:
  onchainos swap quote \
    --from 4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R \
    --to EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v \
    --readable-amount 12.3 --chain solana
  → toTokenAmount: "26.21" (USDC), priceImpact: "0.12%"
  → isHoneyPot: false ✅

  onchainos swap execute \
    --from 4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R \
    --to EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v \
    --readable-amount 12.3 --chain solana \
    --wallet 5xYZ...abc
  → swapTxHash: "5yDef...", toAmount: "26.21"

Step 8 — Reinvest USDC into SOL-USDC pool:
  onchainos defi detail --investment-id 8502
  → underlyingToken[0]: { tokenAddress: "EPjFWdd5..." (USDC), tokenSymbol: "USDC" }
  → tokenPrecision: 6
  → minimal_units = floor(26.21 × 10^6) = 26210000

  onchainos portfolio all-balances --address 5xYZ...abc --chains solana
  → USDC balance: 26.21 ✅

  onchainos defi invest \
    --investment-id 8502 --address 5xYZ...abc \
    --token USDC --amount 26210000 --chain solana
  → Returns dataList → execute via wallet contract-call

Step 9 — Report:
  ✅ Compound complete for SOL-USDC (Raydium)
     Rewards claimed:    $26.45 (12.3 RAY)
     Swapped to:         $26.21 (USDC)
     Reinvested:         $26.21
     Gas cost:           $0.003 (Solana)
     Effective APY:      22.1%
```

---

## Example 2: Compound OKB-USDT on X Layer (Zero Gas)

User says: *"Compound my X Layer positions"*

This example highlights X Layer's **zero gas** advantage — every step is free.

```
Step 1 — Resolve address:
  onchainos wallet status → loggedIn: true
  onchainos wallet addresses → EVM: 0xAbc...123

Step 2 — Fetch X Layer positions:
  onchainos defi positions --address 0xAbc...123 --chains xlayer
  → platformId: "uniswapV3_xlayer", investmentId: "19045"
  → tokenSymbol: "OKB-USDT", totalValue: "$3,200"

Step 3 — Get position details (FRESH):
  onchainos defi position-detail --address 0xAbc...123 --chain xlayer --platform-id uniswapV3_xlayer
  → coinAmount: "15.2", tokenPrecision: 18
  → tokenId: "58823" (V3 NFT position)
  → rewardTokens: [] (no mining rewards — V3 earns fees only)

Step 4 — Collect V3 trading fees:
  onchainos defi collect \
    --address 0xAbc...123 --chain xlayer \
    --reward-type V3_FEE \
    --investment-id 19045 --platform-id uniswapV3_xlayer \
    --token-id 58823
  → Returns dataList[0], dataList[1]

Step 5 — Execute fee collection (zero gas on X Layer):
  onchainos wallet contract-call \
    --to <dataList[0].to> --chain 196 \
    --input-data <dataList[0].serializedData> \
    --value "0" --biz-type defi
  → txHash: "0x7fa..."

  onchainos wallet contract-call \
    --to <dataList[1].to> --chain 196 \
    --input-data <dataList[1].serializedData> \
    --value "0" --biz-type defi
  → txHash: "0x8bc..."
  → Collected: 0.8 OKB + 45.2 USDT

Step 6 — Swap OKB → USDT for single-sided reinvest:
  onchainos swap execute \
    --from 0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee \
    --to <USDT_xlayer_address> \
    --readable-amount 0.8 --chain xlayer \
    --wallet 0xAbc...123
  → toAmount: "24.50" USDT (total available: 69.70 USDT)
  → Gas cost: $0.00 ✅ (X Layer zero gas)

Step 7 — Reinvest into OKB-USDT pool:
  onchainos defi detail --investment-id 19045
  → underlyingToken[0]: USDT, tokenPrecision: 6
  → minimal_units = floor(69.70 × 10^6) = 69700000

  onchainos defi invest \
    --investment-id 19045 --address 0xAbc...123 \
    --token USDT --amount 69700000 --chain xlayer
  → Returns dataList → execute via wallet contract-call (zero gas)

Step 8 — Report:
  ✅ Compound complete for OKB-USDT (Uniswap V3, X Layer)
     Fees collected:     $69.70 (0.8 OKB + 45.2 USDT)
     Swapped to:         $69.70 (USDT)
     Reinvested:         $69.70
     Gas cost:           $0.00 (X Layer zero gas) 🎉
     Effective APY:      18.5%
```

> **Key insight**: X Layer's zero-gas environment means compound frequency has zero cost. Even $1 in fees is worth compounding. This is why the skill always recommends X Layer for rebalancing operations.
