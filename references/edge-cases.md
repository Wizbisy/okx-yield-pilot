# OKX Yield Pilot — Edge Cases

### V3 NFT Positions
V3 LP positions use NFT token IDs. When compounding V3 positions:
- `onchainos defi position-detail` returns `tokenId` — pass it to `onchainos defi collect` as `--token-id`
- For V3 fee collection, use `--reward-type V3_FEE` instead of `REWARD_INVESTMENT`
- V3 reinvestment may require specifying `--tick-lower` and `--tick-upper`

### Multi-Token Rewards
Some positions emit multiple reward tokens (e.g., RAY + MNDE on Raydium):
- Iterate through `rewardTokens[]` from `onchainos defi position-detail`
- Swap each reward token separately
- Sum all swapped amounts before reinvesting

### Zero or Dust Rewards
If `pendingReward × tokenPrice < $0.50`:
- SKIP — the gas cost of claiming is not worth it
- Log: "Skipping {pool}: rewards ($X.XX) below dust threshold"
- Do NOT alert user unless they explicitly ask for a scan

### Position Fully Exited
If `onchainos defi position-detail` returns empty or `coinAmount: "0"`:
- Remove from tracking
- Inform user: "{pool} position has been fully exited"

### Cross-Chain Reward Optimization
When compounding on high-gas chains (Ethereum), consider:
1. Claim rewards on Ethereum
2. Bridge reward tokens to X Layer (zero gas) or Solana (low gas)
3. Swap and reinvest on the cheaper chain

This flow uses the bridge commands if available, or suggests the user bridge manually:
```
# After claiming rewards on Ethereum:
onchainos swap execute \
  --from <reward_token> --to usdc \
  --readable-amount <amount> --chain ethereum --wallet <addr>

# User bridges USDC to X Layer manually via https://web3.okx.com/xlayer/bridge
# Then reinvest on X Layer with zero gas:
onchainos defi invest \
  --investment-id <xlayer_pool_id> --address <addr> \
  --token USDC --amount <minimal_units> --chain xlayer
```

### Stale Position Data
If compound or rebalance fails with error 84021 (asset syncing):
- Wait 30 seconds
- Re-fetch `onchainos defi position-detail` — NEVER use cached data
- Retry once; if still failing, skip and alert user

### Solana Transaction Expiry
Solana calldata expires in ~60 seconds. If the user is slow to confirm:
- Re-generate calldata by re-running `onchainos defi collect` / `onchainos defi invest`
- Warn: "Previous transaction expired. Generating fresh calldata."
- Never attempt to broadcast expired calldata