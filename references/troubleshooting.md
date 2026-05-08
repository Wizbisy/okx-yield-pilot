# OKX Yield Pilot — Troubleshooting Guide

Common errors and recovery steps for yield farming operations.

## DeFi Errors

### Error 84400 — Parameter null
**Cause**: Missing required parameter (e.g., `--amount` or `--ratio` for withdraw).
**Fix**: Check `defi position-detail` output for required fields. Partial exit needs `--amount` or `--ratio`.

### Error 84021 — Asset syncing
**Cause**: Position data is being indexed after a recent on-chain operation.
**Fix**: Wait 30 seconds, then retry. If persistent after 2 minutes, check the chain explorer for tx confirmation.

### Error 84014 — Balance check failed
**Cause**: Insufficient token balance for the requested operation.
**Fix**: Run `portfolio all-balances` to verify. If compounding, skip this position and alert user.

### Error 84018 — Balancing failed
**Cause**: V3 pool balancing calculation failed (usually price moved during calculation).
**Fix**: Increase `--slippage` to `"0.05"` (5%) and retry. If still failing, skip V3 rebalance for this cycle.

### Error 84010 — Token not supported
**Cause**: The deposit/withdraw token is not supported by the DeFi aggregator.
**Fix**: Check `defi detail` for supported tokens. Use an alternative token if available.

### Error 84019 — Address format mismatch
**Cause**: EVM address passed for Solana chain (or vice versa).
**Fix**: Use separate calls — EVM address for EVM chains, Solana address for Solana.

## Swap Errors

### Error 82000 — Token dead/no liquidity
**Cause**: Reward token has been rugged, delisted, or has zero liquidity.
**Fix**: Do NOT retry. Alert user: "Reward token may be rugged or have no liquidity. Manual review required." Check `token advanced-info` for `devRugPullTokenCount` and `tokenTags`.

### Error 81362 — Risk warning (broadcast risk)
**Cause**: Backend risk system flagged the transaction as potentially dangerous.
**Fix**: Do NOT auto-retry or auto-force. Show user the exact risk warning. Only re-run with `--force` after explicit user confirmation that they accept the risk.

### Error 51006 — Swap not available
**Cause**: No swap route found between the token pair.
**Fix**: Try alternative base token (USDC → USDT → native). If all fail, hold rewards without swapping.

### Price impact > 5%
**Cause**: Low liquidity for the reward token.
**Fix**: Warn user prominently. Consider smaller swap amounts or waiting for better liquidity.

## Wallet Errors

### Not logged in
**Cause**: Wallet session expired or user never logged in.
**Fix**: Run `onchainos wallet status`. If not logged in, guide user to `onchainos wallet login`.

### Transaction simulation failed
**Cause**: `executeResult: false` from pre-execution simulation.
**Fix**: Show `executeErrorMsg` to user. Common causes: insufficient gas, contract reverted, wrong parameters. Do NOT broadcast.

### Gas Station required (exit code 3)
**Cause**: Native gas token insufficient, Gas Station not set up.
**Fix**: Follow Gas Station setup flow in `okx-agentic-wallet` skill.

## Network Errors

### Error 50011 — Rate limit
**Cause**: Too many API requests in short period.
**Fix**: Wait 5 seconds, then retry once. If persistent, suggest user create a personal API key at the OKX Developer Portal.

### Timeout / Network error
**Cause**: Network connectivity issue.
**Fix**: Retry once after 3 seconds. If retry fails, inform user and suggest trying again later.

## Compound-Specific Recovery

| Failure point | Funds state | Recovery action |
|---|---|---|
| Collect fails | Rewards still in pool | Skip to next position, retry next cycle |
| Swap fails | Rewards in wallet (uncollected form) | Hold in wallet, alert user for manual swap |
| Reinvest fails | Swapped tokens in wallet | Hold in wallet, alert user — tokens are safe |
| Multiple failures | Mixed state | Generate detailed status report showing what succeeded/failed |

**Rule**: Never retry more than once per position per cycle. Failing twice indicates a systemic issue.

## V3 NFT Position Issues

### V3 fee collection returns empty dataList
**Cause**: No fees have accrued since last collection, or position is out of range.
**Fix**: Check if position `tokenId` exists and pool has volume. Out-of-range V3 positions don't earn fees — alert user: "Position is out of range. No fees accruing."

### Error when using `--reward-type REWARD_INVESTMENT` on V3
**Cause**: V3 positions use `V3_FEE` for trading fee collection, not `REWARD_INVESTMENT`.
**Fix**: Switch to `--reward-type V3_FEE` and include `--token-id <tokenId>` from `position-detail`.

### V3 reinvestment fails
**Cause**: V3 concentrated liquidity requires tick range specification.
**Fix**: If `defi invest` fails for V3, try adding `--tick-lower` and `--tick-upper` from the original position data. If unavailable, suggest user reinvest via the DApp UI directly.

## Batch Compound Issues

### Batch compound partially fails
**Cause**: One or more positions failed during a batch compound run.
**Fix**: 
1. Continue processing remaining positions — don't abort the entire batch
2. Track which positions succeeded and which failed
3. After batch completes, show summary:
   ```
   ✅ Compounded: pos1, pos2, pos4
   ❌ Failed: pos3 (error 84021 — asset syncing), pos5 (swap error — no liquidity)
   💰 Total reinvested: $X.XX | Total gas: $Y.YY
   ```
4. Suggest: "Retry failed positions?" or "Skip and check health?"

### All positions below threshold
**Cause**: No position has rewards above the chain's minimum compound threshold.
**Fix**: Inform user: "No positions have enough rewards to compound cost-effectively. Smallest gap: {pool} has ${X} in rewards vs ${Y} minimum. Try again in a few days."

## Solana-Specific Issues

### Transaction expired before signing
**Cause**: User took more than 60 seconds to approve the Solana transaction.
**Fix**:
1. Do NOT attempt to broadcast expired calldata
2. Re-generate fresh calldata: re-run `defi collect` or `defi invest`
3. Warn: "Previous transaction expired. I've generated fresh calldata — please sign within 60 seconds."

### Solana RPC timeout
**Cause**: Solana RPC nodes can be unreliable during high-traffic periods.
**Fix**: Retry once after 3 seconds. If still failing, suggest trying again in 5 minutes.

## Cross-Chain Issues

### Bridge not available via onchainos
**Cause**: Not all bridge routes are supported through the CLI.
**Fix**: After claiming and swapping rewards on the source chain, instruct user: "Bridge your {amount} USDC to X Layer using https://web3.okx.com/xlayer/bridge for zero-gas reinvestment." Do NOT attempt unsupported bridge commands.

### Address mismatch across chains
**Cause**: Using an EVM address for Solana operations or vice versa.
**Fix**: Always resolve addresses per chain category:
- EVM chains (Ethereum, BSC, Base, X Layer): use `0x…` address
- Solana: use base58 address
- Make separate API calls — never mix in a single `--chains` parameter

## Dust & Edge Cases

### Position shows $0 value but exists
**Cause**: Position has been fully exited but not cleaned up in the aggregator index.
**Fix**: Skip position. Log: "Position {pool} has zero value — likely fully exited." Remove from tracking.

### Reward token not found via token search
**Cause**: Obscure or newly launched reward token not indexed yet.
**Fix**: 
1. Try `token search` with the full token name instead of symbol
2. If still not found, use the token contract address directly from `position-detail.rewardTokens[].tokenAddress`
3. If address also fails, hold rewards unclaimed and alert user

### Multiple reward tokens on single position
**Cause**: Some protocols (e.g., Raydium, Convex) emit 2+ reward tokens.
**Fix**: Process each reward token sequentially:
1. Swap token A → base token
2. Swap token B → base token
3. Sum all swapped amounts
4. Reinvest total in one `defi invest` call
