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
