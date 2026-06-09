# ccusage snapshots — for cross-laptop merge

Raw output from `ccusage@latest` run on the Arm-Mac laptop on **2026-06-08**. These are the **canonical cost numbers** for the talk — they supersede the naive-Opus estimate I wrote earlier into `session_stats.json`.

## Files

| File | Command |
|---|---|
| [`session.json`](./session.json) | `npx ccusage@latest claude session --json` |
| [`daily.json`](./daily.json) | `npx ccusage@latest claude daily --json` |
| [`blocks.json`](./blocks.json) | `npx ccusage@latest claude blocks --json` |

## Scope

Only the sessions whose JSONLs survived 30-day eviction. On this laptop that means:

| Session | First → last activity | Cost | Tokens (in/out/CW/CR) |
|---|---|---|---|
| `a910ecaa` (rust-gpu s3) | May 4 → May 13 | **$234.06** | 9.9K / 648K / 5.8M / 363M |
| `11e6e374` (rust-gpu s4) | May 13 → Jun 8 | **$2,270.59** | 110K / 5.3M / 93.4M / 3.12B |
| `18f718da` (this talk authoring) | Jun 8 | $36.04 | 21K / 161K / 2.2M / 36.1M |
| `d981c9a9` (tiny smoke session) | Jun 8 | $1.31 | ~0 / 1K / 201K / 52K |
| **Total** | | **$2,541.99** | |

The first two are the "real" deck data; the last two are housekeeping. Slide 28 quotes the **rust-gpu surviving subset** ($2,505).

## Cache cost on May–Jun days (NOT a bug, my first read was wrong)

I initially flagged the days with high `cache-write-cost / total-cost` as "the bug" Brice mentioned, but Brice corrected me: **the bug was in April, not May**. The April-era sessions (`dd63080c`, `e493c2fd`, and CCS `ca6a2dde`) are exactly the ones whose JSONLs are evicted — so the bug is invisible to us here.

What the May–Jun days actually show is normal behaviour for high-context-churn work:

| Pattern | CW $ ratio | CR/CW ratio | Days |
|---|---|---|---|
| Cache reused well | <40% | 50–150× | May 12, 16, 26, 27 |
| Lots of context churn | >70% | 8–35× | May 4, 11, 13, 14, 15, 19, 20, 28, 29, Jun 5, Jun 8 |

The "churn" days line up with compaction events, sub-agent fan-out (each `Explore` agent reads a chunk of fresh code), or quickly-ended sessions where the cache had no time to amortise. Not a bug, just a real cost of how the work was structured.

## Projecting to the full timeline (slide 28)

```python
import json
with open("ccusage/session.json") as f: cc = json.load(f)
# Use the 2 surviving rust-gpu sessions to derive a per-prompt rate.
visible_cost = sum(s["totalCost"] for s in cc["sessions"]
                   if s["sessionId"] in {"a910ecaa-c981-40a6-9c37-62be9c7688c4",
                                         "11e6e374-f5e8-4c37-8bd1-2c629b4b6d0b"})
visible_prompts = 87 + 620  # from session_stats.json
rate_per_prompt = visible_cost / visible_prompts  # ≈ $3.54
total_prompts = 1494
# Upper bound: uniform per-prompt rate
upper = rate_per_prompt * total_prompts          # ≈ $5.3K
# Lower bound: CCS prompts assumed cheaper (terser PRs, less context buildup)
lower = rate_per_prompt * 964 + 0.55*rate_per_prompt * 530  # ≈ $4.4K
print(f"${lower:.0f}–${upper:.0f}")
```

The April cache bug (now invisible) would have pushed the early-rust-gpu portion above this projection — so treat the $4.5–5.5K band as a **floor** for the real full-project bill.

## How the other-laptop agent should merge

1. Run the same three commands on the native-Intel-Ubuntu laptop:
   ```bash
   npx ccusage@latest claude session --json > ccusage/session.linux.json
   npx ccusage@latest claude daily   --json > ccusage/daily.linux.json
   npx ccusage@latest claude blocks  --json > ccusage/blocks.linux.json
   ```
2. **Do not overwrite this laptop's snapshots.** Add the `.linux.json` suffix so both sets are preserved.
3. For aggregate totals, sum across both files but **dedup by session ID** — if a session shows up in both (shouldn't happen for survivor JSONLs, since each session is laptop-local, but be defensive), trust the laptop that originated it.
4. Update the numbers on slide 28 to reflect the union, and re-run the bug-tax estimator across the combined `daily.json`.
5. If the other laptop's data has additional surviving CCS sessions (within its 30-day window), this would also rewrite slide 28's framing from "2 rust-gpu sessions" to whatever the actual coverage becomes.
