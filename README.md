# From Directing to Dialogue

> Ten Weeks of AI Agent Coding for Performance Engineering — a first-person retrospective from two ALCF-adjacent projects (CCS and rust-gpu + claspr).

Source materials for an ALCF talk evaluating my use of Claude Code on two real projects, one I owned and one I didn't.

## Contents

| File | What it is |
|---|---|
| [`ai-agent-coding-alcf.pptx`](./ai-agent-coding-alcf.pptx) | The deck (32 slides, ALCF template, ~30 min talk) |
| [`ai-agent-coding-alcf.pdf`](./ai-agent-coding-alcf.pdf) | PDF render for quick viewing |
| [`build_deck.py`](./build_deck.py) | `python-pptx` generator — re-run to rebuild from edits |
| [`deck_outline.md`](./deck_outline.md) | Slide-by-slide outline plus the numbers used |
| [`session_stats.json`](./session_stats.json) | Mined metrics from the local Claude Code session JSONLs |
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
  - **CCS** (`argonne-lcf/CCS`) — C99 autotuning configuration-space library; I wrote and maintain it. 3 sessions, 530 user prompts, 96 PRs merged in 6 weeks.
  - **rust-gpu + claspr** — OpenCL Kernel execution model added to `Rust-GPU/rust-gpu` (PR #3 on the fork, +24,540 / −595 across 761 files) plus the new `claspr` single-source OpenCL host layer (~81K LOC). 4 sessions, 964 user prompts.
- Metrics in `session_stats.json` are mined from the **locally-available** session JSONLs only (the two largest rust-gpu sessions). Totals across all 7 sessions are larger; numbers in the deck are conservative.
- The deck is built from the official ALCF PowerPoint template, which is **committed to this repo** so the build is self-contained.

## Authoring notes

The talk itself was prepared with Claude Code — Opus 4.7 on a Linux laptop — which is mildly on-the-nose given the subject matter. The build script, outline, and the session-mining all sit in this repo so the whole thing is reproducible end-to-end.
