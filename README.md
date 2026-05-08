# OKX Yield Pilot

> Autonomous yield farming autopilot for onchainOS — auto-compound, rebalance, and risk-guard DeFi positions across Solana and X Layer.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![onchainOS](https://img.shields.io/badge/onchainOS-Skill-purple.svg)](https://web3.okx.com/onchainos)
[![Contest](https://img.shields.io/badge/Agentic_Trading-Contest-green.svg)](https://web3.okx.com/boost/trading-competition/agentic-trading)

---

## What It Does

Yield Pilot is an **onchainOS Agent Skill** that automates the tedious, gas-intensive work of managing yield farming positions. Instead of manually claiming rewards, swapping tokens, and reinvesting every day — Yield Pilot does it in a single conversational command.

### Core Capabilities

| Capability | Description |
|---|---|
| **Auto-Compound** | Claim → swap → reinvest rewards across all positions |
| **Portfolio Rebalance** | Maintain target allocations across multiple farms |
| **Health Monitoring** | APY anomaly detection, IL alerts, TVL drift warnings |
| **Gas Optimization** | Chain-aware thresholds — skip compounds when gas > reward |
| **Performance Reporting** | Effective APY, gas efficiency, net profit tracking |

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

Clone this skill into your agent's skills directory:

```bash
# Claude Code / Cursor
git clone https://github.com/Wizbisy/okx-yield-pilot.git ~/.claude/skills/okx-yield-pilot
```

The skill will be automatically detected by your AI agent.

---

## Usage

Talk to your agent naturally. Here are example prompts:

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
```

### Rebalance
```
"Rebalance my yield farms — 50% Solana, 50% X Layer"
"My SOL-USDC pool is overweight, rebalance to target"
```

### Health Check
```
"Check the health of my farming positions"
"Is my impermanent loss getting too high?"
"Any APY drops I should worry about?"
```

### Performance Report
```
"How much have I earned from farming this month?"
"Show me my yield farming performance report"
"What's my effective APY after gas costs?"
```

### Automation
```
"Set up daily auto-compounding for my positions"
"Enable yield autopilot"
"Pause auto-compound"
```

---

## How It Works

Yield Pilot orchestrates multi-step onchainOS CLI operations that would otherwise require manual sequencing:

```
┌─────────────────────────────────────────────────────────┐
│                    YIELD PILOT                          │
│                                                         │
│   User: "Compound my rewards"                          │
│                                                         │
│   ┌─── Scan ────────────────────────────────────────┐   │
│   │ onchainos defi positions                        │   │
│   │ onchainos defi position-detail (for each)       │   │
│   │ → Identify positions with claimable rewards     │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                  │
│   ┌─── Collect ─────────────────────────────────────┐   │
│   │ onchainos defi collect                          │   │
│   │ onchainos wallet contract-call (sign & send)    │   │
│   │ → Rewards claimed to wallet                     │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                  │
│   ┌─── Swap ────────────────────────────────────────┐   │
│   │ onchainos swap execute                          │   │
│   │ → Rewards converted to pool base tokens         │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                  │
│   ┌─── Reinvest ────────────────────────────────────┐   │
│   │ onchainos defi invest                           │   │
│   │ onchainos wallet contract-call (sign & send)    │   │
│   │ → Capital back in the pool, compounding         │   │
│   └─────────────────────────────────────────────────┘   │
│                       ↓                                  │
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
├── SKILL.md                      # Main skill definition (YAML frontmatter + instructions)
├── references/
│   ├── cli-reference.md          # Full onchainos CLI parameter reference
│   └── troubleshooting.md        # Error codes and recovery procedures
├── README.md                     # This file
└── LICENSE                       # MIT License
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built for the onchainOS ecosystem by [Wizbisy](https://github.com/Wizbisy)*
