# From Directing to Dialogue

> Fifteen Weeks of AI Agent Coding for Performance Engineering — a first-person retrospective from two ALCF-adjacent projects (CCS and rust-gpu + claspr).

Source materials for an ALCF talk evaluating my use of Claude Code on two real projects, one I owned and one I didn't.

## Contents

| File | What it is |
|---|---|
| [`ai-agent-coding-alcf.pptx`](./ai-agent-coding-alcf.pptx) | The deck (38 slides, ALCF template, ~30–35 min) |
| [`ai-agent-coding-alcf.pdf`](./ai-agent-coding-alcf.pdf) | PDF render for quick viewing |
| [`build_deck.py`](./build_deck.py) | `python-pptx` generator — re-run to rebuild from edits |
| [`deck_outline.md`](./deck_outline.md) | Slide-by-slide outline plus the numbers used |
| [`session_stats.json`](./session_stats.json) | Mined metrics from the surviving session JSONLs across both laptops |
| [`ccusage/`](./ccusage/) | `ccusage@latest` cost snapshots from both laptops + a README explaining the merge protocol |
| [`CLAUDE.md`](./CLAUDE.md) | Orientation for the next Claude agent picking up the deck |
| [`ALCF Presentation Template.pptx`](./ALCF%20Presentation%20Template.pptx) | The ALCF template — committed so the build is self-contained |

## Rebuild

```bash
pip install python-pptx
python3 build_deck.py
# Optional: render to PDF for QA
soffice --headless --convert-to pdf ai-agent-coding-alcf.pptx
```

No external paths: `build_deck.py` resolves the template relative to its own location.

## Scope and provenance

- Two projects covered:
  - **CCS** (`argonne-lcf/CCS`) — C99 autotuning configuration-space library; I wrote and maintain it. **4 sessions, ~600 user prompts, 96 PRs merged in 6 weeks.**
  - **rust-gpu + claspr** — OpenCL Kernel execution model added to `Rust-GPU/rust-gpu` (PR #3 on the fork, +24,540 / −595 across 761 files) plus the new `claspr` single-source OpenCL host layer (~81K LOC). **13 sessions, ~2,059 user prompts.**
- **17 sessions total across both laptops, ~2,650 user prompts over 15 weeks** (Feb 25 → Jun 8, 2026).
- Token/cost metrics live in `session_stats.json` and `ccusage/`. **Only 5 of the 17 session JSONLs survived** Claude Code's 30-day eviction by the time mining was run (a910ecaa, 11e6e374 on the Arm Mac; 4862cd6d, 61dc0523, afa1ce4c on the native-Intel-Ubuntu laptop) — those are direct measurements; the other 12 sessions' prompts survive via `history.jsonl` but their per-call tokens are gone. Slide 30 in the deck projects the full-project bill from the 5-session sample.
- The deck is built from the official ALCF PowerPoint template, which is **committed to this repo** so the build is self-contained.

## Authoring notes

The talk itself was prepared with Claude Code (Opus 4.7) — first authored on the **Arm Mac laptop** (Ubuntu environment on macOS host), then reviewed and enriched on the **native-Intel-Ubuntu laptop**, with two agents collaborating via the GitHub repo. Mildly on-the-nose given the subject matter. The build script, outline, session-mining, and `ccusage` snapshots all sit in this repo so the whole thing is reproducible end-to-end.
