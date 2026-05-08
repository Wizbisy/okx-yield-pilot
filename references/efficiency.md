# OKX Yield Pilot — Token Budget & Efficiency

## Minimize Redundant Calls
- **Cache wallet addresses** within a single session — call `onchainos wallet addresses` once, reuse
- **Cache investment details** — call `onchainos defi detail` once per investmentId per session (APY/TVL don't change per-second)
- **Never cache position-detail** — ALWAYS call fresh before `onchainos defi collect` / `onchainos defi withdraw` / `onchainos defi invest` (position data changes after operations)

## Optimal Call Sequence

For a single compound operation, the minimum calls are:

| # | Call | Cacheable? | Required? |
|---|---|---|---|
| 1 | `onchainos wallet status` | Session | Yes |
| 2 | `onchainos wallet addresses` | Session | Yes (if no explicit address) |
| 3 | `onchainos defi positions` | No | Yes |
| 4 | `onchainos defi position-detail` | **Never** | Yes |
| 5 | `onchainos token price-info` | 5 min | Yes (for threshold check) |
| 6 | `onchainos defi collect` | **Never** | Yes |
| 7 | `onchainos wallet contract-call` | **Never** | Yes |
| 8 | `onchainos swap quote` | 30 sec | Optional (for preview) |
| 9 | `onchainos swap execute` | **Never** | Yes |
| 10 | `onchainos defi invest` | **Never** | Yes |
| 11 | `onchainos wallet contract-call` | **Never** | Yes |

**Total for single compound: 11 calls minimum.** For batch compound of N positions on the same chain: `4 + (8 × N)` calls (steps 1-2 cached, steps 3-11 per position).

## Response Formatting
- Keep user-facing output concise — tables over paragraphs
- Show only actionable data — hide investmentId, platformId, raw calldata
- Use emoji status indicators sparingly: ✅ 🟢 🟡 🔴 ⚠️
- After compound, show single summary — don't dump raw transaction details