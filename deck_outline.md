# Deck outline — Evaluating AI Agent Coding for Performance Engineering at ALCF

**Target:** 30-min talk, ~30 content slides + cover + closing (32 total). Audience: ALCF perf-eng peers (deep technical). Style: conservative ALCF template.

## Numbers I'll use (sourced)

- **CCS**: 3 sessions, 530 user prompts in history, Feb 25 → Apr 7 2026 (~6 weeks), 99 PRs by bricevideau-ai (96 merged, 3 closed). C/C++ codebase ~52K LOC.
- **rust-gpu**: 4 sessions, 964 user prompts in history, Mar 27 → Jun 8 2026 (~10 weeks). PR #3 = +24,540/-595 across 761 files. Rust codebase ~142K LOC (full rust-gpu).
- **claspr**: green-field repo, 182 commits, ~81K LOC.
- **Local JSONLs**: only sessions `a910ecaa` (7.4 MB, 209 user / 1747 asst msgs) and `11e6e374` (75 MB, 682 user / 12,222 asst msgs). Other sessions only have prompts via `history.jsonl`.
- **Token usage (available rust-gpu sessions only)**: 273K raw input, 15.5M output, 254M cache writes, **7.18B cache reads**. Cache-read dominates by ~30×; effective bill is much lower than naïve in+out reads.
- **Cost ceiling** at Opus 4 pricing for the 2 available sessions: ~$16.7K. Actual was much lower — most was on Sonnet 4.x, and the cache-read price is 1/10 the cache-write.
- **Tool distribution (11e6e374)**: Bash 3791, Edit 1687, Read 831, TaskUpdate 407, Write 313.
- **Sub-agents**: 24 total across rust-gpu sessions. Mix: 16 Explore, 7 general-purpose, 1 Plan. **Zero in first rust-gpu session**, all 24 in the last one — clear adoption curve.
- **Skills**: 0 invoked via the Skill tool in available transcripts (skills came later). User mentioned `/skills` mid-CCS in March.
- **Slash commands** (external prompts): `/install-github-app`, `/skills`, `/model`, `/context`, `/compact`. Very low total — most session control was natural language.
- **Auto-context-compaction events** (the "this session is being continued..." marker) in 11e6e374: **8 distinct days** = ~8 hits on the 200K context window in one rolling session.
- **Memory entries**: 30+ files in `~/.claude/projects/-home-claudecode-projects/memory/` covering feedback, project state, reference. Built incrementally over rust-gpu sessions.

## Slide-by-slide

1. **Title** — Evaluating AI Agent Coding for Performance Engineering / Brice Videau / ALCF / Date. Use **Title Slide** layout.
2. **Agenda** — 5 bullets: Setup → CCS → rust-gpu/claspr → cross-cutting analysis → recommendations. (Layout: *Title, Subtitle and Bullets*)
3. **Why this evaluation** — Three boxes: (a) what perf-eng work looks like at ALCF, (b) the hypothesis (AI agents can carry implementation while expert directs), (c) what success means here. (Layout: *4 block big idea* or 3-column)
4. **Setup: how Claude Code was used** — env (Ubuntu laptop + macOS laptop), tooling (`gh` CLI, autotools, cargo, MCPs), persona (`bricevideau-ai` GitHub identity, gmail), `.claude/` directory model: per-project sessions, memory, todos, history. (3-column or bullets)
5. **The two projects, side by side** — Comparison table: dimension / CCS / rust-gpu+claspr. (Layout: 2-column or *Title, Subtitle and Bullets* + table)
6. **Section break: Project 1 — CCS** — autotuning configuration-space library, C with Python/Ruby bindings, mature codebase I wrote and maintain. (Section break layout)
7. **CCS in 60 seconds** — what is CCS, why does it exist, where does perf-eng use it. (Layout: *Title and Subtitle Only* with prose)
8. **CCS workflow with the agent** — PR-per-task workflow, devel branch, rebase discipline, agent identity `bricevideau-ai`, code review of agent PRs. (Bullets + diagram if I can)
9. **CCS: what landed** — **96 PRs merged in 6 weeks** chart. Topic histogram: JSON serialization helpers, deserialize refactor, code-coverage push (gcov), bug fixes from coverage analysis, doc rewrites, binding bug fixes, sanitizer CI. (Layout: 2-column with stat callouts)
10. **CCS: how I interacted** — Annotated quote board: "No, check again" / "I noticed you started using clang-format instead of clang-format-17" / "I don't think computer scientists should be lazy". Pattern: short directive prompts, frequent corrections, building style discipline. (4-block or callouts)
11. **CCS: wins** — Coverage went up materially, dormant bug classes surfaced (uninit-after-goto, realloc cleanup paths, deserialize bounds), JSON format documented, binding parity restored. (Layout: 3-column)
12. **CCS: pitfalls observed** — Agent will silently swap toolchain versions (clang-format vs -17); will reach for "lazy" patterns the senior author rejects; needs explicit reminders about project conventions (CCS_REFUTE_ERR_GOTO macro family). Mitigated by: short PRs + active review + CLAUDE.md-style memory. (Bullets)
13. **Section break: Project 2 — rust-gpu + claspr** — adding OpenCL Kernel execution model to rust-gpu, then building claspr (single-source OpenCL on top). (Section break)
14. **rust-gpu + claspr in 60 seconds** — what rust-gpu is, what's missing for OpenCL, what the OpenCL Kernel target adds (Physical64, OpenCL.std intrinsics, native CL vector types, printf, samplers, atomics, groups). claspr = the host-side single-source proc-macro layer. (Layout: *Title and Subtitle Only*)
15. **The knowledge-gap shift** — I am a perf-eng expert and OpenCL expert. **I am not a Rust developer. I am not a SPIR-V expert.** This changes the agent collaboration mode from "direct" to "explore-then-decide". (Big idea / 2-column)
16. **rust-gpu: what landed** — Numbers callout: **PR #3 = 24,540 lines added, 595 deleted, 761 files**. claspr: greenfield, 182 commits, ~81K LOC. Subagent fan-out used heavily to explore Rust patterns (cuda-oxide, SYCL, cust). (Layout: 2-column or stat callouts)
17. **rust-gpu workflow** — Issue reproduction → fork → branch (`opencl-kernel-support` stable, `-v2` dev) → repeated rebase → eventual upstream. Cross-laptop work (Intel macOS + Ubuntu Arm) — sessions explicitly handed off ("lets stop here, we'll continue on the linux laptop"). (Bullets + diagram)
18. **rust-gpu: how I interacted (knowledge-gap mode)** — Quote board: "any reason to use the same licenses as rust-gpu?" / "let me refocus" (scope-cutting) / "I remember you telling me that the runner did put the arguments in different order" (memory-anchored) / "I want to have a conversation around the execution model Async/Sync and InOrder/OutOfOrder. But first I have a question, can rust polymorphism (method selection) depend on the return value?" (collaborative design). (4-block)
19. **rust-gpu/claspr: design dialogue examples** — Concrete cases where the agent surfaced design options I had to pick from (and_then_host async chain → returning values → with_context removal; access markers; alloc_uninit safety story; KernelOp decoupling so proc-macro doesn't depend on async). I drove decisions, agent enumerated trade-offs. (2-column)
20. **rust-gpu: pitfalls observed** — (a) Context-window exhaustion: **~8 auto-compactions in one rolling session**; (b) Drift after compaction (had to add "remember difftests run command" memory); (c) Cleaning up legitimate spike scenarios as if they were bugs (resulted in `feedback_respect_spike_intent.md`); (d) Tool/version regressions across sessions (pocl rebuild + LLVM drift); (e) Quietly diverging from sample patterns. (Bullets + stat)
21. **Section break: Cross-cutting analysis** — (Section break)
22. **Tool-call distribution** — Bar/donut of: Bash 47% / Edit 18% / Read 9% / TaskUpdate 5% / Write 4% / Grep, MultiEdit, … (Layout: 2-column with text + chart-as-image; or table)
23. **Sub-agent adoption curve** — Timeline showing Explore/Plan/general usage over the 4 sessions. **0, 0, 0, 24** — last session adopted parallel exploration heavily. Quote example agent labels: "Map current claspr public API", "Audit spike scenarios vs original intent", "Tier1/Tier2 abstraction-parity audit". (2-column or timeline)
24. **Token economics** — Headline: 7.18 **billion** cache-read tokens vs 273K raw input. Cache hit rate ≈ 96% of effective input. Output 15.5M tokens (~12K assistant messages × ~1.3K avg). Cost dominated by cache-read at $1.50/MTok (Anthropic Opus pricing) and output at $75/MTok. (Stat callouts; 4-block)
25. **Memory system in practice** — 30 entries built up; categories (feedback / project / reference); examples (clang-format-17 must be 17, difftest run command, opencl branch workflow, pocl quirks, …). The memory was **load-bearing** for surviving compactions. (Layout: 3-column)
26. **Context-window management** — 200K window hit repeatedly; 8 auto-compactions in the longest rolling session; manual `/compact` and `/clear` used preemptively ("be careful context is almost full, please anticipate"). CLAUDE.md + memory + WIP design docs the user told the agent to write *before* compaction were essential. (2-column)
27. **Evolution: CCS → claspr** — Side-by-side: CCS-era usage (short directive prompts, PR-per-task, no subagents, no skills) → claspr-era (longer prompts, design dialogues, parallel Explore agents, memory-anchored). The tooling matured; the user's prompting style matured with it. (2-column)
28. **What worked best** — (a) Senior author + agent on familiar code = high throughput, low risk. (b) Knowledge-gap collaboration with explicit design dialogues. (c) Short, focused PRs with strong review. (d) Memory + CLAUDE.md as cross-session ground truth. (4-block)
29. **Anti-patterns observed** — (a) Letting one session run forever (compaction tax). (b) Trusting CI-green as a quality signal without inspecting the diff. (c) Letting the agent pick the toolchain. (d) Removing "weird" patterns that were actually intentional. (e) Skipping the agent's enumeration step when in unfamiliar territory. (4-block or bullets)
30. **Implications for perf-eng at ALCF** — (a) Agents are net-positive on mature codebases the perf-eng owns. (b) On unfamiliar code, the agent should be in *explore-then-decide* mode, never pure-implement. (c) Cost is real but cache-dominated; budget per session, not per token. (d) Identity, memory, and PR discipline are not optional. (3-column)
31. **Recommendations** — checklist: per-project CLAUDE.md from day 1 / memory hygiene / agent github identity / short PRs + active review / preempt compaction / `Explore` subagents for unfamiliar territory / `Plan` subagents before non-trivial work / `/code-review` before push / never let the agent skip a hook. (Bullets)
32. **Closing** — ALCF closing-template slide.

## Layouts I will use

- Layout 0 (Title Slide) — slides 1
- Layout 2 (*Section Break) — slides 6, 13, 21
- Layout 3 (*Title, Subtitle and Bullets) — most content slides
- Layout 4 (*Title and Subtitle Only) — narrative/prose slides 7, 14
- Layout 5/6/7 (2/3/4 column) — comparison slides 5, 10–12, 16, 19, 22–26, 28
- Layout 9 (*4 block big idea) — 3, 18, 28, 29
- Layout 13 or 14 or 15 (Closing) — slide 32

## What stays as TBD/handoff for the other laptop
- Real prompts / Claude outputs from the **CCS** sessions (their JSONLs aren't on this laptop; I only have prompt-side via `history.jsonl`).
- Real prompts from rust-gpu sessions `dd63080c` and `e493c2fd` (only prompt-side here).
- Any photos/screenshots the user wants.
