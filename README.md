# OKX Yield Pilot

> Autonomous yield farming autopilot for onchainOS — auto-compound, rebalance, and risk-guard DeFi positions across Solana and X Layer.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![onchainOS](https://img.shields.io/badge/onchainOS-Skill-purple.svg)](https://web3.okx.com/onchainos)
[![skills.sh](https://img.shields.io/badge/skills.sh-Directory-orange.svg)](https://skills.sh)

---

## What It Does

Yield Pilot is an **onchainOS Agent Skill** that automates the tedious, gas-intensive work of managing yield farming positions. Instead of manually claiming rewards, swapping tokens, and reinvesting every day — Yield Pilot does it in a single conversational command.

### Core Capabilities

| Capability | Description |
|---|---|
| **Auto-Compound** | Claim → swap → reinvest rewards across all positions |
| **Batch Compound** | Compound all positions in one command — chains sorted by gas cost |
| **Portfolio Rebalance** | Maintain target allocations across multiple farms |
| **Health Monitoring** | APY anomaly detection, IL alerts, TVL drift warnings |
| **Gas Optimization** | Chain-aware thresholds — skip compounds when gas > reward |
| **Performance Reporting** | Effective APY, gas efficiency, net profit tracking |
| **Intelligent Routing** | Decision tree + reasoning chain for optimal workflow selection |

### Supported Chains

| Chain | Gas Profile | Best For |
|---|---|---|
| X Layer | **Zero gas** | Rebalancing, intermediate swaps |
| Solana | ~$0.001/tx | Frequent compounding |
| Ethereum | Variable | Large positions only |
| BSC | ~$0.05/tx | Mid-tier compounding |
| Base | ~$0.01/tx | Frequent compounding |

---

## Installation

This is an **onchainOS skill** — it runs inside your AI agent (Claude, Cursor, etc.) with the OKX Agentic Wallet.

### Prerequisites

1. **OKX Agentic Wallet** — [Set up here](https://web3.okx.com/onchainos)
2. **onchainOS CLI** — Installed automatically via preflight checks
3. **Active DeFi positions** — At least one LP or staking position on a supported chain

### Setup

Install via the [Skills Directory](https://skills.sh):

```bash
npx skills add Wizbisy/okx-yield-pilot
```

Or clone manually into your agent's skills directory:

```bash
# Claude Code / Cursor
git clone https://github.com/Wizbisy/okx-yield-pilot.git ~/.claude/skills/okx-yield-pilot
```

The skill will be automatically detected by your AI agent.

---

## Usage

Talk to your agent naturally in **English or Chinese**:

### Scan Positions
```
"Show me all my yield farming positions"
"What rewards do I have unclaimed?"
"Scan my DeFi positions on Solana and X Layer"
```

### Auto-Compound
```
"Compound all my farming rewards"
"Harvest and reinvest my Solana yields"
"Auto-compound my X Layer positions"
"自动复投收益" / "复投我的DeFi奖励"
```

### Rebalance
```
"Rebalance my yield farms — 50% Solana, 50% X Layer"
"My SOL-USDC pool is overweight, rebalance to target"
"重新平衡我的农场"
```

### Health Check
```
"Check the health of my farming positions"
"Is my impermanent loss getting too high?"
"Any APY drops I should worry about?"
"查看农场健康" / "检查无常损失"
```

### Performance Report
```
"How much have I earned from farming this month?"
"Show me my yield farming performance report"
"What's my effective APY after gas costs?"
"收益报告"
```

### Automation
```
"Set up daily auto-compounding for my positions"
"Enable yield autopilot"
"Pause auto-compound"
"启动收益自动化"
```

---

## How It Works

Yield Pilot orchestrates multi-step onchainOS CLI operations that would otherwise require manual sequencing:

```
┌─────────────────────────────────────────────────────────┐
│                    YIELD PILOT                          │
│                                                         │
│   User: "Compound my rewards"                           │
│                                                         │
│   ┌─── Scan ────────────────────────────────────────┐   │
│   │ onchainos defi positions                        │   │
│   │ onchainos defi position-detail (for each)       │   │
│   │ → Identify positions with claimable rewards     │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                 │
│   ┌─── Collect ─────────────────────────────────────┐   │
│   │ onchainos defi collect                          │   │
│   │ onchainos wallet contract-call (sign & send)    │   │
│   │ → Rewards claimed to wallet                     │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                 │
│   ┌─── Swap ────────────────────────────────────────┐   │
│   │ onchainos swap execute                          │   │
│   │ → Rewards converted to pool base tokens         │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                 │
│   ┌─── Reinvest ────────────────────────────────────┐   │
│   │ onchainos defi invest                           │   │
│   │ onchainos wallet contract-call (sign & send)    │   │
│   │ → Capital back in the pool, compounding         │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                 │
│   ✅ Report: Rewards, gas cost, effective APY           │
└─────────────────────────────────────────────────────────┘
```

---

## Safety Features

| Feature | Description |
|---|---|
| **Gas Guard** | Skips compound if gas cost > 50% of reward value |
| **IL Circuit Breaker** | Pauses auto-compound if IL exceeds threshold |
| **APY Floor** | Flags positions where APY drops below minimum |
| **TVL Guard** | Alerts on TVL drops > 60% (liquidity risk) |
| **Slippage Guard** | Blocks swaps with > 3% slippage |
| **Honeypot Check** | Blocks swaps for flagged reward tokens |
| **User Confirmation** | Every fund-moving operation requires explicit approval |
| **Solana Expiry Warning** | Warns about 60-second signing window for Solana txs |

---

## File Structure

```
okx-yield-pilot/
├── SKILL.md                      # Main skill — 6 flows, routing, safety gates
├── _shared/
│   └── preflight.md              # Pre-flight checks (wallet login, address resolution, positions)
├── references/
│   ├── cli-reference.md          # Full onchainos CLI parameter tables and return schemas
│   ├── troubleshooting.md        # Error codes and recovery procedures
│   ├── examples.md               # Worked examples: Solana SOL-USDC, X Layer OKB-USDT
│   ├── edge-cases.md             # V3 NFT, multi-token rewards, cross-chain optimization
│   ├── batch-compound.md         # Batch execution order, thresholds, savings report
│   ├── chains.md                 # Chain index, gas profiles, native token addresses
│   └── efficiency.md             # Optimal call sequence, caching rules
├── evals.json                    # 18 trigger/non-trigger test cases
├── README.md                     # This file
└── LICENSE                       # MIT License
```

---

## Why This Skill?

Manual yield farming is tedious and gas-wasteful:

| Task | Manual | With Yield Pilot |
|---|---|---|
| Claim + swap + reinvest 5 positions | ~30 min, $15+ gas | One command, gas-optimized |
| Check all positions for APY drops | Open 5 DApps, compare manually | "Check health" → instant report |
| Rebalance across chains | Multiple bridges + swaps | "Rebalance" → auto-route via X Layer |
| Know effective APY after gas | Spreadsheet math | "Show report" → net APY calculated |

The key differentiator: **X Layer zero-gas routing**. Yield Pilot routes intermediate operations through X Layer whenever possible, saving $5-25 per compound cycle on Ethereum.

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.1.0 | 2026-05-08 | Extracted examples, edge cases, batch, chains, efficiency to references/; fixed Flow 6 persistence; fixed native token addresses per chain |
| 1.0.0 | 2026-05-07 | Initial release |

### Check for Updates

```bash
# Re-install to get the latest version
npx skills add Wizbisy/okx-yield-pilot
```

Or check the version in your installed `SKILL.md` metadata against the [latest on GitHub](https://github.com/Wizbisy/okx-yield-pilot).

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built for the [OKX onchainOS](https://web3.okx.com/onchainos) ecosystem by [Wizbisy](https://github.com/Wizbisy)*