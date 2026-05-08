# OKX Yield Pilot — Batch Compound Optimization

When the user says "compound all" or "compound everything", optimize execution order:

```
1. Group positions by chain:
   positions_by_chain = {
     "xlayer": [pos1, pos2],       # Zero gas — always process first
     "solana": [pos3],             # Low gas — process second
     "base": [pos4],              # Low gas — process third
     "bsc": [pos5],               # Low gas — process fourth
     "ethereum": [pos6, pos7]     # High gas — process last, may skip
   }

2. Sort chains by gas cost (ascending):
   processing_order = ["xlayer", "solana", "base", "bsc", "ethereum"]

3. Pre-filter within each chain:
   For each position in chain:
     reward_usd = pendingReward × tokenPrice
     IF reward_usd < chain_min_threshold:
       SKIP position, log: "Skipping {pool}: ${reward_usd} below ${threshold} min"

   Chain minimum thresholds:
     xlayer:   $0.01  (zero gas — compound everything)
     solana:   $2.00
     base:     $5.00
     bsc:      $5.00
     ethereum: $50.00

4. Execute per chain (sequential within chain, chains processed in order):
   For each chain in processing_order:
     For each eligible position:
       a. Claim rewards (Flow 2, Phase A)
       b. Swap to base token (Flow 2, Phase B)
       c. Reinvest (Flow 2, Phase C)
     Show per-chain subtotal when done

5. After all chains processed, show batch summary:
   ═══════════════════════════════════════════
     BATCH COMPOUND — Summary
   ═══════════════════════════════════════════
     Positions compounded:  {count}
     Positions skipped:     {skip_count} (below threshold)
     Total rewards claimed: ${total_rewards}
     Total gas spent:       ${total_gas}
     Net reinvested:        ${total_reinvested}
   ═══════════════════════════════════════════
```

## Batch Savings Report

After batch compound, show the user how much they saved:

```
💡 Batch savings:
   - Wallet auth: 1 login instead of {position_count} logins
   - X Layer positions: $0 gas (vs ~$5-25 per compound on Ethereum)
   - Processing time: ~{minutes} min (vs ~{manual_minutes} min manually)
```
