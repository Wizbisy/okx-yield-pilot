---
name: okx-yield-pilot
description: "Autonomous multi-chain yield farming autopilot — auto-compound, rebalance, and risk-guard DeFi positions across Solana and X Layer using onchainOS. Triggers (EN): 'auto-compound my yields', 'compound my DeFi rewards', 'rebalance my yield farms', 'optimize my DeFi portfolio', 'auto-reinvest farming rewards', 'manage my LP positions', 'yield autopilot', 'set up auto-compounding', 'harvest and reinvest', 'compound rewards on Solana', 'compound rewards on X Layer', 'rebalance DeFi allocations', 'check yield farm health', 'monitor impermanent loss', 'show farming performance', 'stop auto-compound', 'pause yield strategy', 'resume yield farming', 'yield farm status', 'farming APY report', 'how much have I earned from farming', 'gas-efficient compounding', 'batch compound my positions', 'auto-harvest DeFi', 'optimize LP gas costs', 'schedule compound cycle', 'detect APY drop', 'farming circuit breaker', 'IL alert', 'impermanent loss check', 'exit underperforming farms', 'rotate to higher APY', 'yield comparison across chains'. Triggers (ZH): '自动复投收益', '复投我的DeFi奖励', '重新平衡我的农场', '优化DeFi组合', '查看农场健康', '检查无常损失', '收益报告', '暂停自动复投', '启动收益自动化'. Do NOT use for: manual one-off swaps (okx-dex-swap), DeFi product discovery or deposit (okx-defi-invest), DeFi position viewing only (okx-defi-portfolio), wallet balance checks (okx-wallet-portfolio), token prices (okx-dex-market)."
license: MIT
metadata:
  author: Wizbisy 
  version: "1.0.0"
  last_updated: "2026-05-08"
  min_onchainos_version: "1.0.0"
  homepage: "https://github.com/Wizbisy/okx-yield-pilot"
---

# OKX Yield Pilot

Autonomous yield farming autopilot — auto-compound, portfolio rebalance, and risk-guard DeFi positions across Solana and X Layer via onchainOS CLI.

For CLI parameter details, see [references/cli-reference.md](references/cli-reference.md).

## Step 0 — Skill Routing (run before every other step)

Before running any workflow in this skill, classify the user's intent:

- **One-off DeFi deposit/withdraw** (no automation) → use `okx-defi-invest`
- **View DeFi positions only** → use `okx-defi-portfolio`
- **One-off token swap** → use `okx-dex-swap`
- **Wallet balance check** → use `okx-wallet-portfolio` or `okx-agentic-wallet`
- **Token price / chart** → use `okx-dex-market`
- **Named DApp interaction** (Aave, Raydium, Orca, etc.) → use `okx-dapp-discovery`

Stay in this skill when the user wants **ongoing automated management**: auto-compounding, scheduled rebalancing, portfolio optimization, IL monitoring, or performance reporting across yield farm positions.

## Skill Routing Table

| User intent | Target skill |
|---|---|
| Auto-compound / harvest & reinvest / yield autopilot | **This skill** |
| Rebalance DeFi allocations automatically | **This skill** |
| Monitor IL / APY anomalies / farming health | **This skill** |
| View farming performance report | **This skill** |
| One-off deposit into a DeFi product | `okx-defi-invest` |
| View current DeFi positions (no action) | `okx-defi-portfolio` |
| Swap tokens | `okx-dex-swap` |
| Check wallet balance | `okx-wallet-portfolio` |
| Token price or chart | `okx-dex-market` |
| Named DApp (Aave, Orca, Raydium…) | `okx-dapp-discovery` |

## Pre-flight Checks

> Read the preflight checks before any workflow. Look for the file in this order:
> 1. `_shared/preflight.md` (relative to this SKILL.md file)
> 2. `../_shared/preflight.md` (if skill is nested inside a parent skills folder)
> 3. `../okx-agentic-wallet/_shared/preflight.md` (shared OKX preflight)
>
> If none exist, run these checks inline: (a) `onchainos wallet status` — verify logged in, (b) `onchainos wallet addresses` — resolve EVM + Solana addresses, (c) `onchainos defi positions` — verify at least one active position exists.

## Decision Tree

When the user's request reaches this skill, follow this decision tree to select the correct workflow:

```
User request received
│
├─ Mentions "scan" / "show" / "list" / "what positions" / "what rewards"?
│  → Flow 1: Scan Positions
│
├─ Mentions "compound" / "harvest" / "reinvest" / "claim and reinvest" / "复投"?
│  ├─ Specifies a single chain or pool?
│  │  → Flow 2: Auto-Compound (single position)
│  └─ Says "all" or no filter?
│     → Flow 2: Auto-Compound (batch — all eligible positions)
│
├─ Mentions "rebalance" / "reallocate" / "drift" / "adjust weights" / "平衡"?
│  → Flow 3: Rebalance
│
├─ Mentions "health" / "check" / "IL" / "impermanent loss" / "risk" / "APY drop" / "safe" / "健康"?
│  → Flow 4: Health Check
│
├─ Mentions "report" / "performance" / "how much earned" / "effective APY" / "summary" / "报告"?
│  → Flow 5: Performance Report
│
├─ Mentions "schedule" / "autopilot" / "automate" / "daily" / "weekly" / "自动化"?
│  → Flow 6: Schedule Automation
│
└─ Unclear?
   → Run Flow 1 (Scan) first, then suggest next action based on results
```

## Reasoning Chain

Before executing any workflow, the agent MUST reason through these questions internally (do NOT show to user):

1. **Which chains does the user care about?** → If unspecified, scan ALL supported chains
2. **Are the positions worth acting on?** → Check reward value vs gas cost BEFORE executing
3. **Is there a safety concern?** → Run health gates mentally before suggesting actions
4. **What's the optimal execution order?** → Group operations by chain to minimize wallet auth overhead
5. **Can I save the user gas?** → Always prefer X Layer (zero gas) for intermediate operations

## onchainos Commands Used

This skill orchestrates multi-step workflows by combining these real `onchainos` CLI commands:

| # | Command | Used in |
|---|---|---|
| 1 | `onchainos defi positions` | Scan — discover active yield positions |
| 2 | `onchainos defi position-detail` | Scan / Compound / Rebalance — get position details + pending rewards |
| 3 | `onchainos defi detail` | Scan / Compound — get APY, TVL, underlying tokens |
| 4 | `onchainos defi collect` | Compound — claim pending rewards |
| 5 | `onchainos defi invest` | Compound / Rebalance — deposit tokens back into pool |
| 6 | `onchainos defi withdraw` | Rebalance — partially exit overweight positions |
| 7 | `onchainos defi rate-chart` | Health / Report — fetch historical APY data |
| 8 | `onchainos defi tvl-chart` | Health — detect TVL drift |
| 9 | `onchainos swap quote` | Compound — price-check reward token swap |
| 10 | `onchainos swap execute` | Compound / Rebalance — convert tokens (quote → approve → swap → broadcast) |
| 11 | `onchainos wallet status` | All flows — check login state |
| 12 | `onchainos wallet addresses` | All flows — resolve EVM + Solana addresses |
| 13 | `onchainos wallet contract-call` | Compound / Rebalance — sign and broadcast DeFi calldata via TEE |
| 14 | `onchainos portfolio all-balances` | Compound — verify sufficient balance before reinvesting |
| 15 | `onchainos token search` | Compound — resolve reward token contract address |
| 16 | `onchainos token price-info` | Health — fetch token prices for IL calculation |

## Chain Support

| Chain | Name | chainIndex | Gas profile |
|---|---|---|---|
| X Layer | `xlayer` | `196` | **Zero gas** — preferred for rebalancing |
| Solana | `solana` | `501` | Low gas (~$0.001/tx) |
| Ethereum | `ethereum` | `1` | Variable (batch to save gas) |
| BSC | `bsc` | `56` | Low gas (~$0.05/tx) |
| Base | `base` | `8453` | Low gas (~$0.01/tx) |

> **X Layer priority**: Always recommend X Layer for rebalancing operations due to zero gas fees. When compounding on high-gas chains (Ethereum), batch operations or suggest bridging rewards to X Layer first.

## Operation Flows

### Flow 1: Scan Positions

Discover all active yield farming positions and unclaimed rewards across chains.

```
1. Resolve wallet address:
   onchainos wallet status          → get active account
   onchainos wallet addresses       → get EVM + Solana addresses

2. Fetch DeFi positions per chain:
   onchainos defi positions --address <evm_addr> --chains "xlayer,ethereum,bsc,base"
   onchainos defi positions --address <sol_addr> --chains "solana"

3. For each position with investmentId:
   onchainos defi position-detail --address <addr> --chain <chain> --platform-id <pid>
   → Extract: investmentId, coinAmount, tokenPrecision, rewardTokens, pendingRewards

4. Enrich with current APY:
   onchainos defi detail --investment-id <id>
   → Extract: rate (APY), tvl, underlyingToken info

5. Display summary table (see Display Rules)
```

**Output format**:

| # | Chain | Platform | Pool | Position Value | Unclaimed Rewards | Current APY | TVL |
|---|---|---|---|---|---|---|---|
| 1 | X Layer | UniswapV3 | OKB-USDT | $2,450 | $12.30 (UNI) | 18.5% | $5.2M |
| 2 | Solana | Raydium | SOL-USDC | $1,800 | $8.70 (RAY) | 22.1% | $45M |

### Flow 2: Auto-Compound

Claim rewards from positions and reinvest them back into the same pool.

> **CRITICAL — position-detail is MANDATORY before every collect.** Position data changes after each on-chain operation. Do NOT reuse stale data.

```
Phase A — Reward Collection:
1. onchainos defi position-detail --address <addr> --chain <chain> --platform-id <pid>
   → MUST be called fresh — do NOT reuse prior results
   → Extract: investmentId, platformId, tokenId (if V3), rewardTokens

2. Check if rewards exceed minimum threshold:
   reward_usd = pendingReward × rewardTokenPrice
   IF reward_usd < gas_cost_estimate × 2:
     → SKIP: "Rewards ($X.XX) too small vs gas cost ($Y.YY). Skipping."
   ELSE:
     → PROCEED to collect

3. onchainos defi collect \
     --address <addr> --chain <chain> \
     --reward-type REWARD_INVESTMENT \
     --investment-id <id> --platform-id <pid>
   → Returns calldata (dataList)

4. Execute calldata via Agentic Wallet:
   EVM:
     onchainos wallet contract-call \
       --to <dataList[N].to> --chain <chainIndex> \
       --input-data <dataList[N].serializedData> \
       --value <value_in_UI_units> --biz-type defi
   Solana:
     onchainos wallet contract-call \
       --to <dataList[N].to> --chain 501 \
       --unsigned-tx <dataList[N].serializedData> \
       --biz-type defi

Phase B — Reward Swap:
5. Identify reward token address:
   onchainos token search --query <reward_symbol> --chains <chain>
   → Get tokenContractAddress

6. Get quote for swap to base token (e.g., USDC):
   onchainos swap quote \
     --from <reward_token_addr> --to <base_token_addr> \
     --readable-amount <claimed_amount> --chain <chain>
   → Check priceImpact < 5%, check isHoneyPot

7. Execute swap:
   onchainos swap execute \
     --from <reward_token_addr> --to <base_token_addr> \
     --readable-amount <claimed_amount> --chain <chain> \
     --wallet <addr>

Phase C — Reinvest:
8. Get target pool details:
   onchainos defi detail --investment-id <id>
   → Get underlyingToken[].tokenAddress, tokenPrecision

9. Convert swapped amount to minimal units:
   minimal_units = floor(amount × 10^tokenPrecision)

10. Check wallet balance before reinvesting:
    onchainos portfolio all-balances --address <addr> --chains <chain>
    → Verify sufficient balance

11. Reinvest:
    onchainos defi invest \
      --investment-id <id> --address <addr> \
      --token <underlying_token_symbol_or_addr> \
      --amount <minimal_units> --chain <chain>
    → Returns calldata → execute via wallet contract-call

12. Log compound event:
    Record: timestamp, rewards_claimed, gas_cost, reinvested_amount, effective_apy
```

**Gas optimization rules**:
- **X Layer**: Always compound — zero gas cost
- **Solana**: Compound if rewards > $2 (gas ~$0.001)
- **Ethereum**: Compound only if rewards > $50 (gas ~$5-25)
- **BSC/Base**: Compound if rewards > $5 (gas ~$0.05-0.10)
- **Batch rule**: If multiple positions on the same chain, process all in one session to amortize wallet login overhead

### Flow 3: Rebalance

Adjust position sizes to match target allocation weights.

```
1. Scan all positions (Flow 1) → get current USD values

2. Calculate current allocation:
   total_usd = sum(all position values)
   current_alloc[i] = position_value[i] / total_usd × 100

3. Compare against target allocation (from user config):
   drift[i] = current_alloc[i] - target_alloc[i]

4. Identify rebalancing moves:
   overweight = positions where drift > threshold (default: 10%)
   underweight = positions where drift < -threshold

5. For each overweight position:
   a. Calculate exit amount:
      exit_usd = drift[i] / 100 × total_usd
      exit_ratio = exit_usd / position_value[i]
   b. onchainos defi position-detail (FRESH — mandatory)
   c. onchainos defi withdraw \
        --investment-id <id> --address <addr> --chain <chain> \
        --ratio <exit_ratio> --platform-id <pid>
   d. Execute withdrawal calldata via wallet contract-call

6. Swap withdrawn tokens to target pool's base tokens:
   onchainos swap execute --from <withdrawn_token> --to <target_base_token> \
     --readable-amount <amount> --chain <chain> --wallet <addr>

7. For each underweight position:
   a. Calculate deposit amount from freed capital
   b. onchainos defi invest \
        --investment-id <id> --address <addr> \
        --token <token> --amount <minimal_units> --chain <chain>
   c. Execute deposit calldata via wallet contract-call

8. Verify new allocations match targets (re-run Flow 1)
```

**Rebalance rules**:
- Default drift threshold: 10% (configurable)
- Prefer X Layer for intermediate swaps (zero gas)
- Never rebalance if total gas cost > 1% of portfolio value
- Require user confirmation before executing any withdrawals

### Flow 4: Health Check

Monitor position health: APY anomalies, impermanent loss, TVL changes.

```
1. Scan all positions (Flow 1) → get current metrics

2. For each position, check:

   a. APY anomaly detection:
      onchainos defi rate-chart --investment-id <id> --time-range MONTH
      → Extract APY history
      → avg_apy = mean(rate values)
      → IF current_apy < avg_apy × 0.6:
           ALERT: "⚠️ {pool}: APY dropped {current}% vs {avg}% monthly average (-{drop}%)"

   b. TVL drift detection:
      onchainos defi tvl-chart --investment-id <id> --time-range WEEK
      → IF tvl_change_7d < -30%:
           ALERT: "⚠️ {pool}: TVL dropped {pct}% in 7 days — liquidity risk"

   c. Impermanent loss estimate (for LP positions):
      → Fetch current token prices via onchainos token price-info
      → Calculate IL using standard formula:
        price_ratio = current_price / entry_price
        il_pct = (2 × sqrt(price_ratio) / (1 + price_ratio) - 1) × 100
      → IF |il_pct| > max_il_threshold:
           ALERT: "⚠️ {pool}: Estimated IL {il_pct}% exceeds {threshold}% limit"

   d. High APY risk warning:
      → IF current_apy > 50%:
           WARN: "⚠️ {pool}: APY above 50% indicates elevated risk"

3. Display health report:
   Overall status: HEALTHY / WARNING / CRITICAL
   Per-position breakdown with alerts
```

**Health status levels**:

| Status | Condition | Action |
|---|---|---|
| 🟢 HEALTHY | All metrics normal | Continue autopilot |
| 🟡 WARNING | APY drop > 40% OR IL > threshold OR TVL drop > 30% | Alert user, suggest review |
| 🔴 CRITICAL | APY drop > 70% OR IL > 2× threshold OR TVL drop > 60% | Pause auto-compound, recommend exit |

### Flow 5: Performance Report

Generate comprehensive yield farming performance metrics.

```
1. Scan all positions (Flow 1)

2. For each position:
   a. Fetch historical APY:
      onchainos defi rate-chart --investment-id <id> --time-range MONTH
   b. Calculate effective APY:
      effective_apy = (total_rewards_claimed - total_gas_spent) / position_value × 365 / days_active × 100
   c. Calculate gas efficiency:
      gas_efficiency = total_gas / total_rewards × 100

3. Generate report:
   - Total portfolio value
   - Total rewards earned (lifetime)
   - Total gas spent
   - Net profit (rewards - gas)
   - Average effective APY
   - Best performing position
   - Worst performing position
   - Compound count and frequency
   - Recommendation: continue / adjust / exit
```

**Report output format**:

```
═══════════════════════════════════════════
  YIELD PILOT — Performance Report
  Period: {start_date} → {end_date}
═══════════════════════════════════════════

  Portfolio Value:      ${total_value}
  Total Rewards:        ${total_rewards}
  Total Gas Spent:      ${total_gas}
  Net Profit:           ${net_profit}
  Effective APY:        {eff_apy}%
  Gas Efficiency:       {gas_eff}% of rewards

  ┌────────────────────────────────────┐
  │ Position Breakdown                 │
  ├─────┬──────────┬────────┬──────────┤
  │ Pool│ Value    │ APY    │ Rewards  │
  ├─────┼──────────┼────────┼──────────┤
  │ ... │ ...      │ ...    │ ...      │
  └─────┴──────────┴────────┴──────────┘

  Recommendation: {recommendation}
═══════════════════════════════════════════
```

### Flow 6: Schedule Automation

Configure compounding frequency, rebalance triggers, and risk thresholds.

```
1. Collect user preferences:
   - Compound frequency: "daily" / "weekly" / "on-demand"
   - Rebalance threshold: drift % to trigger (default: 10%)
   - Min reward threshold: minimum USD to compound (default: auto by chain)
   - Max IL threshold: % before alerting (default: 5%)
   - APY floor: minimum APY before flagging (default: 5%)
   - Emergency exit: auto-exit if APY drops below X% (default: disabled)

2. Store configuration locally (agent memory)

3. When user says "run autopilot" or "start yield pilot":
   a. Run health check (Flow 4)
   b. If HEALTHY or WARNING: run compound (Flow 2) for all eligible positions
   c. Check if rebalance needed: run rebalance (Flow 3) if any drift > threshold
   d. Generate report (Flow 5)
   e. Display summary and next scheduled run
```

## Address Resolution

When the user does NOT provide a wallet address, resolve automatically:

```
1. onchainos wallet status          → check login, get active account
2. onchainos wallet addresses       → get addresses by chain category:
   - XLayer / EVM addresses
   - Solana addresses
3. Match address to target chain:
   - EVM chains → use EVM address
   - Solana     → use Solana address
```

Rules:
- If user provides explicit address, use directly — skip resolution
- If wallet not logged in → ask user to log in first (→ `okx-agentic-wallet`)
- **CRITICAL**: EVM addresses (`0x…`) can only query EVM chains; Solana addresses (base58) can only query `solana`. Never mix.

## Transaction Execution

After `defi invest`/`withdraw`/`collect` returns `dataList`, execute via Agentic Wallet:

**EVM chains (Ethereum, BSC, Base, X Layer)**:
```bash
onchainos wallet contract-call \
  --to <dataList[N].to> \
  --chain <chainIndex> \
  --input-data <dataList[N].serializedData> \
  --value <value_in_UI_units> \
  --biz-type defi
```

**Solana**:
```bash
onchainos wallet contract-call \
  --to <dataList[N].to> \
  --chain 501 \
  --unsigned-tx <dataList[N].serializedData> \
  --biz-type defi
```

**`--value` conversion**: `dataList[].value` is in minimal units (wei). `contract-call --value` expects UI units. Convert: `value_UI = value / 10^nativeToken.decimal` (18 for ETH/OKB, 9 for SOL). If value is `""`, `"0"`, or `"0x0"`, use `"0"`.

**Execution rules**:
- Execute `dataList[0]` first, then `[1]`, etc. Never in parallel.
- Wait for on-chain confirmation before next step.
- If any step fails, stop remaining steps and report which succeeded/failed.
- User confirmation required before every invest/withdraw/collect execution.

> **Solana transaction expiry**: Solana DeFi transactions expire in ~60 seconds. After receiving calldata, WARN the user: "This Solana transaction must be signed and broadcast within 60 seconds or it will expire."

## Displaying Results

### Position Table

| # | Chain | Platform | Pool | Value | Rewards | APY | Status |
|---|---|---|---|---|---|---|---|
| 1 | X Layer | Uniswap V3 | OKB-USDT | $2,450.00 | $12.30 | 18.5% | 🟢 |

- `investmentId` is internal — do NOT display to user
- `rate` from API is decimal → multiply by 100 and append `%`
- `tvl` → format as human-readable USD ($3.52B, $537M)
- Sort by position value descending
- **High APY warning**: If any position has APY > 50%, add: "⚠️ APY above 50% — elevated risk"

### Compound Summary

```
✅ Compound complete for {pool}
   Rewards claimed:    ${amount} ({token})
   Swapped to:         ${swap_amount} ({base_token})
   Reinvested:         ${reinvest_amount}
   Gas cost:           ${gas} ({chain})
   Effective APY:      {eff_apy}%
```

## Error Handling

| Error | Scenario | Recovery |
|---|---|---|
| 84400 | Parameter null | Check required params — position may have been fully exited |
| 84021 | Asset syncing | "Position data is syncing, please retry in 30 seconds" |
| 84014 | Balance check failed | Insufficient balance — skip this compound, alert user |
| 84018 | Balancing failed | Adjust slippage or skip V3 rebalance |
| 84010 | Token not supported | Skip position, log warning |
| 50011 | Rate limit | Wait 5 seconds and retry once |
| Swap 82000 | Token dead/rugged | **CRITICAL**: Do NOT retry. Alert: "Reward token may be rugged. Manual review required." |
| Swap 81362 | Risk warning | Do NOT auto-force. Alert user with risk details. |

> Load on error: [references/troubleshooting.md](references/troubleshooting.md)

**Compound failure recovery**:
1. If collect fails → skip to next position, log error
2. If swap fails → hold claimed rewards in wallet, alert user
3. If reinvest fails → rewards remain as swapped tokens in wallet, alert user
4. Never retry more than once per position per cycle

## Risk Controls

### Safety Gates

| Gate | Condition | Action |
|---|---|---|
| Gas guard | Gas cost > 50% of reward value | SKIP compound, log reason |
| IL circuit breaker | IL > 2× user threshold | PAUSE auto-compound, alert user |
| APY floor | APY drops below user minimum | FLAG position, suggest exit |
| TVL guard | TVL drops > 60% in 7 days | CRITICAL alert, recommend exit |
| Slippage guard | Swap slippage > 3% | SKIP swap, alert user |
| Honeypot check | isHoneyPot = true on reward token | BLOCK swap, alert: "Reward token flagged as honeypot" |

### Fund-action Confirmation

Every operation that moves funds requires explicit user confirmation:
- Withdrawing from a position → confirm
- Swapping tokens → confirm (unless in approved autopilot mode)
- Depositing into a position → confirm
- Emergency exit → always confirm, even in autopilot

### Silent / Automated Mode

Enabled ONLY when user explicitly authorizes. Rules:
1. **Explicit opt-in required**: User must say "enable autopilot" or equivalent
2. **Risk gates still apply**: CRITICAL alerts halt autopilot and notify user
3. **Execution log**: Log every automated action (timestamp, position, amount, txHash, status)
4. **Session limit**: Auto-mode expires after 24 hours — user must re-authorize

## Post-Action Suggestions

| Just completed | Suggest |
|---|---|
| Position scan | "Would you like me to compound the unclaimed rewards?" or "Want a health check on these positions?" |
| Auto-compound | "Want to see a performance report?" or "Scan for more positions to compound?" |
| Rebalance | "View updated allocations?" or "Generate a performance report?" |
| Health check | "Compound the healthy positions?" or "Exit the flagged positions?" |
| Performance report | "Adjust your strategy?" or "Set up a compounding schedule?" |

Present conversationally — never expose skill names, onchainos commands, or internal flow names to the user.

## Amount Display Rules

- Token amounts in UI units (`1.5 ETH`), never base units
- USD values with 2 decimal places
- Large amounts in shorthand (`$1.2M`)
- Gas fees in USD
- APY as percentage with 1 decimal
- IL as percentage with 1 decimal
- Sort positions by USD value descending

## Global Notes

- `--amount` must be in **minimal units** (integer). Convert: userAmount × 10^tokenPrecision
- The wallet address parameter for ALL defi commands is `--address`
- `--slippage` default `"0.01"` (1%); suggest `"0.03"`–`"0.05"` for volatile pools
- **Solana tx expiry**: Warn user — 60 second window for signing
- **X Layer zero gas**: Always highlight when suggesting chain for rebalancing
- **Address-chain compatibility**: EVM (`0x…`) → only EVM chains. base58 → only Solana. Make separate calls for each.
- User confirmation required before every invest/withdraw/collect
- Address used for calldata generation MUST match the signing address
- **Locale-aware output**: All user-facing content must be translated to match the user's language (English, Chinese, etc.)

## Worked Example: Compound SOL-USDC on Solana

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

## Edge Cases

### V3 NFT Positions
V3 LP positions use NFT token IDs. When compounding V3 positions:
- `defi position-detail` returns `tokenId` — pass it to `defi collect` as `--token-id`
- For V3 fee collection, use `--reward-type V3_FEE` instead of `REWARD_INVESTMENT`
- V3 reinvestment may require specifying `--tick-lower` and `--tick-upper`

### Multi-Token Rewards
Some positions emit multiple reward tokens (e.g., RAY + MNDE on Raydium):
- Iterate through `rewardTokens[]` from `position-detail`
- Swap each reward token separately
- Sum all swapped amounts before reinvesting

### Zero or Dust Rewards
If `pendingReward × tokenPrice < $0.50`:
- SKIP — the gas cost of claiming is not worth it
- Log: "Skipping {pool}: rewards ($X.XX) below dust threshold"
- Do NOT alert user unless they explicitly ask for a scan

### Position Fully Exited
If `defi position-detail` returns empty or `coinAmount: "0"`:
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
- Re-fetch `defi position-detail` — NEVER use cached data
- Retry once; if still failing, skip and alert user

### Solana Transaction Expiry
Solana calldata expires in ~60 seconds. If the user is slow to confirm:
- Re-generate calldata by re-running `defi collect` / `defi invest`
- Warn: "Previous transaction expired. Generating fresh calldata."
- Never attempt to broadcast expired calldata

## Worked Example: Compound OKB-USDT on X Layer (Zero Gas)

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

## Batch Compound Optimization

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

### Batch Savings Report

After batch compound, show the user how much they saved:

```
💡 Batch savings:
   - Wallet auth: 1 login instead of {position_count} logins
   - X Layer positions: $0 gas (vs ~$5-25 per compound on Ethereum)
   - Processing time: ~{minutes} min (vs ~{manual_minutes} min manually)
```

## Token Budget & Efficiency

To stay within token limits and maximize efficiency:

### Minimize Redundant Calls
- **Cache wallet addresses** within a single session — call `wallet addresses` once, reuse
- **Cache investment details** — call `defi detail` once per investmentId per session (APY/TVL don't change per-second)
- **Never cache position-detail** — ALWAYS call fresh before collect/withdraw/invest (position data changes after operations)

### Optimal Call Sequence
For a single compound operation, the minimum calls are:

| # | Call | Cacheable? | Required? |
|---|---|---|---|
| 1 | `wallet status` | Session | Yes |
| 2 | `wallet addresses` | Session | Yes (if no explicit address) |
| 3 | `defi positions` | No | Yes |
| 4 | `defi position-detail` | **Never** | Yes |
| 5 | `token price-info` | 5 min | Yes (for threshold check) |
| 6 | `defi collect` | **Never** | Yes |
| 7 | `wallet contract-call` | **Never** | Yes |
| 8 | `swap quote` | 30 sec | Optional (for preview) |
| 9 | `swap execute` | **Never** | Yes |
| 10 | `defi invest` | **Never** | Yes |
| 11 | `wallet contract-call` | **Never** | Yes |

**Total for single compound: 11 calls minimum.** For batch compound of N positions on the same chain: `4 + (8 × N)` calls (steps 1-2 cached, steps 3-11 per position).

### Response Formatting
- Keep user-facing output concise — tables over paragraphs
- Show only actionable data — hide investmentId, platformId, raw calldata
- Use emoji status indicators sparingly: ✅ 🟢 🟡 🔴 ⚠️
- After compound, show single summary — don't dump raw transaction details
