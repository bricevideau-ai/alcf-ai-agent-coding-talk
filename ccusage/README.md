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

## The cache-bug story (slide 28)

Across `daily.json`, days split cleanly into two regimes by `cache-write-cost / total-cost`:

| Pattern | CW $ ratio | CR/CW ratio | Days |
|---|---|---|---|
| Healthy caching | <40% | 50–150× | May 12, 16, 26, 27 |
| Cache bug | >70% | 8–35× | most other days (May 4, 11, 13, 14, 15, 19, 20, 28, 29, Jun 5, Jun 8) |

On "bug days" the cache is being created at the same volume as it's read — i.e. it's being invalidated and rebuilt almost every turn rather than amortised across many turns. Likely root causes are TodoWrite-class tool calls or other in-context events triggering cache invalidation, but I did not isolate the exact mechanism; if you want to file a Claude Code issue, the data here is what to attach.

**Estimating the "without the bug" cost.** I assumed each "bad day" could have hit CR/CW = 100× (a reasonable healthy baseline; the best days hit 150×+) and subtracted the excess cache-write cost. That comes to roughly **$1,250** vs the **$2,500** actually billed — about half the bill was bug tax. Quick python that produces this number:

```python
import json
PRICE_CW = 18.75 / 1_000_000
with open("ccusage/daily.json") as f: d = json.load(f)
total, ideal = 0, 0
for day in d["daily"]:
    cw, cr, cost = day["cacheCreationTokens"], day["cacheReadTokens"], day["totalCost"]
    cw_pct = 100 * cw * PRICE_CW / cost if cost else 0
    if cw_pct > 50:                                # bug-affected
        saving = max(0, (cw - cr/100) * PRICE_CW)  # if cache had hit CR/CW=100x
        ideal += cost - saving
    else:
        ideal += cost
    total += cost
print(f"actual ${total:.0f}, ideal ${ideal:.0f}, bug tax ${total-ideal:.0f}")
```

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
