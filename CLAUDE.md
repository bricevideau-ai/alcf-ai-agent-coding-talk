# Repo orientation for Claude — alcf-ai-agent-coding-talk

This is a **38-slide** ALCF talk evaluating Brice's use of Claude Code on two projects: CCS and rust-gpu + claspr. The deck was authored on the **Arm Mac laptop** (Ubuntu environment on macOS host — `aarch64`, hostname `bvideau-VMware20-1`), then reviewed and enriched on the **native-Intel-Ubuntu laptop** on 2026-06-09 — see *Linux-laptop merge* below. On **2026-06-09 (afternoon)** a new claspr-depth slide was inserted after the code showcase (now slide 19; bumped what-landed → slide 21). On **2026-06-09 (evening)** a CCS interop story slide was inserted at slide 10, bumping every CCS slide after it (#11–15) and every rust-gpu/cross-cutting slide (#16–38) by one. Slide numbering is now rendered automatically on every content slide (skipped on Title, the three Section Breaks, and the Closing — those layouts have no slide-number placeholder).

If a user prompt anywhere in the surviving session history says "the linux laptop", they mean the native-Intel-Ubuntu laptop. If they say "the mac" or "the other laptop", they mean the Arm one where this deck was originally authored.

## Files

| File | Role |
|---|---|
| `ai-agent-coding-alcf.pptx` | The deck. Built by `build_deck.py`, not edited directly. |
| `ai-agent-coding-alcf.pdf` | Render for review. Regenerate with the workflow below. |
| `build_deck.py` | `python-pptx` generator. **Edit this**, not the pptx. |
| `deck_outline.md` | Slide-by-slide outline + the numbers used in the deck. |
| `session_stats.json` | Mined metrics from the Linux laptop's available session JSONLs. |
| `README.md` | Public-facing description of the repo. |

## How to rebuild

The template (`ALCF Presentation Template.pptx`) is committed at the repo root. `build_deck.py` resolves paths relative to its own location, so the build is self-contained — no external setup.

```bash
pip install --break-system-packages python-pptx
python3 build_deck.py
soffice --headless --convert-to pdf ai-agent-coding-alcf.pptx
pdftoppm -jpeg -r 90 ai-agent-coding-alcf.pdf slide   # per-slide JPGs for QA
```

After editing, `git add ai-agent-coding-alcf.pptx ai-agent-coding-alcf.pdf build_deck.py` and commit. Re-rendering the PDF on every change keeps the repo viewable on GitHub.

## What is locked in vs what is open

**Locked in (do not change without asking):**
- Title: *From Directing to Dialogue: 15 Weeks of AI Agent Coding for Performance Engineering*. Subtitle: *17 sessions · 2,650 prompts · 15 weeks*. The "directing → dialogue" framing is the thesis the rest of the deck builds on; CCS slides set up "directing", rust-gpu slides set up "dialogue". 15 weeks = Feb 25 → Jun 8 (CCS start to last claspr commit), not just the rust-gpu portion. Subtitle counts come from the Linux merge (4 CCS + 13 rust-gpu = 17 sessions; 600 + 2,059 = ~2,650 prompts) and must stay consistent with slide 5's side-by-side bullets if either is touched. (Aesthetic note: Layout 0's subtitle placeholder is narrow and a 3-segment subtitle wraps over the title block. Improving this would require putting a figure/image in the prescribed right-hand image block of Layout 0; deferred.)
- Slide order (Title · Agenda · Why · Setup · Side-by-side · Chronology · §CCS×8 · §rust-gpu/claspr×9 · §Cross-cutting×9 · Implications · Recommendations · Closing). Each §-block includes a "what user code looks like" showcase slide; CCS also has a 4-block interop story slide (slide 10) and rust-gpu has a 4-block claspr depth slide (slide 19, "more than a binding" — single-source / Tier 1+2 ops / type-state safety / three SPIR-V modes). Slide 6 is the session-chronology text-timeline. The headline parallelism it surfaces is the **26-day rust-gpu same-project parallel run** between `afa1ce4c` (Linux) and `11e6e374` (Mac), May 14 → Jun 8 — that's the figure in the slide's subtitle. The original 11-day Mar 27 → Apr 7 cross-project overlap (CCS `ca6a2dde` + rust-gpu `dd63080c`) is still visible in the timeline rows but no longer the headline. Rearranging requires updating the agenda slide (#2) and §-break slides.
- ALCF template layouts only. No custom layouts, no images injected. The template's master controls fonts/colors; we do not override them — except inside the `set_block` and `set_code` helpers (see *Systemic QA issues* below).
- 38 slides total. Sized for ~30–35 min of presenting. Slot is up to an hour including Q&A and Brice plans to add his own slides too — so don't pad this deck further; trim if you want to grow elsewhere.

**Open for the other laptop to enrich:**
- §CCS slides (#7–14). Built from the CCS repo + the user prompts surviving in `~/.claude/history.jsonl` only — the full session JSONLs were already evicted by Claude Code's 30-day session retention before this deck was built (see *Data availability* below). The Linux-merge pass on 2026-06-09 added one CCS session (`d1bcaa09`, 70 prompts) to the chronology but did not change §CCS slide content — its JSONL was also evicted.
- Slides #27–30 (tool-call distribution / sub-agent adoption / model progression / token economics): now aggregated from **5 surviving rust-gpu sessions** across both laptops (a910ecaa, 11e6e374, 4862cd6d, 61dc0523, afa1ce4c). For tool/sub-agent counts use the merge recipe at the bottom; **for cost, use ccusage** (see `ccusage/README.md`).
- Quotes on slides #12 and #21 are paraphrased lightly from real user prompts; verbatim alternatives are fine.
- Code showcases (#8 CCS, #16 claspr): trimmed examples; longer or different illustrative snippets are fine if they fit at 9pt Consolas in the left column.
- Chronology slide (#6) is text-based monospace ASCII, **now 17 rows** (Mac + Linux sessions interleaved by date) at 7pt. If a future merge adds rows, bump font down or trim the spacing comment to fit.

## Critical context: data availability

> The Claude Code 30-day session-eviction policy is what bit us. Sessions older than 30 days at the time of authoring/merging are gone from disk. The Linux merge on 2026-06-09 confirmed what the original Mac authoring suspected: the missing sessions are not on the other laptop either — they're just lost.

**Surviving full JSONLs (5 sessions, both laptops):**
- Mac    `a910ecaa-*.jsonl` (rust-gpu, May 4–13)
- Mac    `11e6e374-*.jsonl` (rust-gpu, May 13–Jun 8)
- Linux  `4862cd6d-*.jsonl` (rust-gpu, May 11)
- Linux  `61dc0523-*.jsonl` (rust-gpu, May 14)
- Linux  `afa1ce4c-*.jsonl` (rust-gpu, May 14–Jun 8)
- Plus `memory/` directories on both laptops (these survive eviction).

**Prompts-only via `~/.claude/history.jsonl` (the 12 evicted sessions):**
- CCS Mac:    `3f086a62`, `add0d69f`, `ca6a2dde`
- CCS Linux:  `d1bcaa09`
- rgpu Mac:   `dd63080c`, `e493c2fd`
- rgpu Linux: `daf9f59c`, `b74f64ed`, `e80de831`, `da4896fe`, `a9eacbbc`, `fd509bb7`

For those evicted sessions, what we still have:
- User prompts in `~/.claude/history.jsonl` on the originating laptop
- Git history of the actual codebases (CCS, rust-gpu, claspr) and PRs in those repos
- Memory entries built up over time

**Optional add for a future agent:** if you think the 30-day eviction caveat itself is worth a bullet on slide #22, it's a fair methodological note. Slide #26 already mentions it briefly as a caveat at the bottom of "What this means". If a third pass adds more context here, lean toward making the caveat more explicit on slide #26 (it's the most data-dependent slide).

## Systemic QA issues already fixed — do not regress

The first build broke in predictable ways. The current `build_deck.py` has fixes in place; if you re-edit, keep them:

1. **4-block "big idea" layout (Layout 9)**. The placeholder default is 32pt bold white with `<a:noAutofit/>`. Putting `"Title\nbody"` as one paragraph blew out the box. Use the `set_block(ph, header, body)` helper, which writes the header at level-0 default (inherits 32pt bold) plus a second paragraph with `Pt(14)` and `RGBColor(0xFF, 0xFF, 0xFF)` forced (color inheritance is unreliable across renderers). Keep block bodies to ~25 words max; keep block headers to one line (two-line headers eat body height and trigger clipping at the box bottom — caught on the original slides 3 and 11 before fixing).
2. **Subtitle placeholder (idx=13)** is 26pt bold accent-2 color, 0.64" tall, `noAutofit`. Anything wrapping to two lines collides with the body below or the title above. **Cap subtitles at ~80 chars / single line.**
3. **Bullet body (idx=14)** ends at y≈6.86"; the master's footer starts soon after. **Cap content lists so the rendered text doesn't approach the footer**. Trim 1–2 bullets if a slide gets close.
4. **Title slide picture placeholder (idx=10)** has a "click to insert image" prompt that renders as a gray box if left empty. The current code removes it explicitly along with unused presenter slots (idx 19/20/21/22). If you ever re-use Layout 0, do the same.
5. **Typo audit**: real user prompts get cleaned (e.g. `"Allright"` → `"Alright"`). If a typo is the point of the quote, mark it `[sic]`.

The `build_deck.py` helpers (`set_text`, `set_block`, `set_bullets`, `set_code`, `get_ph`, `remove_placeholder`, `enable_slide_number`) encode these fixes. Use them, don't bypass them.

`set_code(ph, code, *, font="Consolas", size=9)` is for code-showcase slides. It explicitly overrides paragraph spacing (`spcBef`/`spcAft` to 0 and `lnSpc` to 100%) because the master adds bullet-style paragraph spacing that ruins code density. Keep code under ~20 lines at the default 9pt — anything longer overflows the 5.96" wide × 3.97" tall column.

`enable_slide_number(slide)` is called automatically from the `add(layout_idx)` helper for every slide. It clones the layout's `<p:ph type="sldNum">` placeholder into the slide; python-pptx only auto-inherits *required* placeholders, and slide-number is optional. The cloned element carries `<a:fld type="slidenum">`, which both PowerPoint and LibreOffice resolve to the current page number. Layouts 0 (Title), 2 (Section Break), 13–15 (Closings) have no slide-number placeholder, so the helper is a no-op there — that's the intended behavior, those slides should not be numbered. Do not call `enable_slide_number` directly from a slide block; the `add()` helper already does it. If you ever want to *turn off* a number on one slide, `remove_placeholder(s, idx)` it after creation (the idx is 10 on most layouts, 12 on Layout 3).

## Visual QA workflow

```bash
# Render and convert
python3 build_deck.py
soffice --headless --convert-to pdf ai-agent-coding-alcf.pptx
rm -f slide-*.jpg
pdftoppm -jpeg -r 90 ai-agent-coding-alcf.pdf slide

# Inspect — recommend launching a fresh Agent subagent for visual QA
# (the pptx skill at ~/.claude/skills/pptx/SKILL.md has the canonical QA prompt).
```

LibreOffice renders close to PowerPoint but not identical. Final PowerPoint review will happen back on the Arm Mac (host macOS has the real Office). Don't worry about font substitution here — just make sure structurally nothing overflows.
- LibreOffice doesn't have the exact ALCF title font (substitutes). PowerPoint will use the real one.
- Multi-line title behavior in Layout 3/5/6 may differ by ~1 px. Watch the rust-gpu interaction title (#18) and Implications (#30) which both wrap to 2 lines.

## The source of truth for the numbers

| Number on slide | Source |
|---|---|
| 96/99 CCS PRs, ~16/wk, #24→#122 | `gh pr list -R argonne-lcf/CCS --author bricevideau-ai --state all --limit 200` |
| 600 / 2,059 user prompts (CCS / rust-gpu) | `~/.claude/history.jsonl` on **both** laptops, summed and deduped by sessionId. 17 sessions total. |
| rust-gpu PR #3 +24,540/-595/761 files | `gh pr view 3 -R bricevideau-ai/rust-gpu --json additions,deletions,changedFiles` |
| claspr 182 commits / ~81K LOC | `git log --oneline` and `find … -name '*.rs' \| xargs wc -l` in `~/projects/claspr` |
| Cost figures on slide 28 ($3,474 measured, $8.6–9.6K projected full project) | **`ccusage/` directory** — `ccusage@latest claude {session,daily,blocks} --json` snapshots from **both** laptops (`session.json` + `session.linux.json`). **Canonical for cost** — do not use the per-token totals from `session_stats.json`. The README there has the projection method. |
| 9.5B cache reads / 431K input / 18.7M output | `session_stats.json` `by_project.rust-gpu` (5 surviving rust-gpu sessions, both laptops, summed). Visible-portion-only. |
| 25 sub-agents (16 Explore, 7 general, 1 Plan, 1 claude-code-guide) — Mac 24 / Linux 1 across 26 parallel days | `session_stats.json` per-session counts (Mac `11e6e374` = 24; Linux `afa1ce4c` = 1; others 0) |
| 8 auto-compaction events | grep `"This session is being continued"` in `11e6e374-*.jsonl` |
| 26-day max overlap (afa1ce4c Linux / 11e6e374 Mac) | `session_stats.json` `overlaps_top[0]` — recompute from `session_chronology` if sessions change |

If you re-mine, **merge into** `session_stats.json` (don't overwrite — read its `merge_instructions` field for the rules: same-session data union, prefer newer/larger transcripts). Then update slide-level numbers in `build_deck.py` and rebuild.

**Schema is now v3** (after 2026-06-09 Linux merge):
- Each `by_session` entry has a `host` field (`arm-mac-ubuntu` or `linux-intel-ubuntu`) so you can attribute work cleanly.
- `merge_history` records each laptop that contributed, with date.
- `overlaps_top` (top-10 pairs by overlap days) replaced the single-string `parallelism_note`. The deck's chronology caption pulls from `overlaps_top[0]`.

### Re-merge recipe (paste-ready)

```python
import json, datetime
from collections import Counter, defaultdict
from pathlib import Path

REPO = Path('.')
existing = json.load(open(REPO/'session_stats.json'))

# 1. Mine your laptop's surviving session JSONLs.
#    Same schema as existing entries: project, session_id, host,
#    available_locally, first_date/last_date, first_ts_utc/last_ts_utc,
#    duration_hours_wallclock, user_msgs, assistant_msgs,
#    input/output/cache_creation/cache_read_tokens,
#    subagent_calls, subagent_types, tool_uses_top, model_uses,
#    slash_commands, skills_used.
# 2. Mine ~/.claude/history.jsonl on the same laptop for prompt counts by
#    sessionId; map each unfamiliar sessionId to its project by inspecting
#    the first prompt (look for "rust-gpu" / "CCS" / "claspr" mentions).
# 3. For each new sessionId: if existing[by_session][sid].available_locally
#    is False or absent, insert/replace with yours.
# 4. Recompute by_project totals (sum across by_session); recompute
#    overlaps_top by pairing all sessions whose date ranges intersect.
# 5. Update `merge_history`, bump `generated_at` and `generated_on_laptop`.
# 6. Update slide numbers in build_deck.py (subtitle, slide 5, 6, 25, 26, 28)
#    and the "source of truth" table above. Rebuild + visual QA.
# 7. Also rerun `ccusage` and save under ccusage/session.<host>.json etc.
#    Cost on slide 28 is sum of every session's totalCost across ALL
#    ccusage/session*.json files, deduped by sessionId.
```

## Public-repo etiquette

- Repo is **public** on `bricevideau-ai/alcf-ai-agent-coding-talk`. Do not commit anything that would embarrass the user, leak credentials, or quote third parties without consent. Quotes used so far are all the user's own prompts.
- Commit messages should be specific (what changed, why), one-topic. Co-author Claude.
- Pushing directly to `main` is fine here (it's a small artifact repo, not a code project with reviewers).

## Things I deliberately did not do

- Did not add the source `.jpg` slide renders — they're ephemeral, regenerate from the pptx.
- (Originally I did not commit the ALCF template. Brice corrected that — it's now in the repo so the build is self-contained.)
- Did not write a Makefile — the four-line workflow at the top is short enough.
- Did not propagate the QA pass into the build script as automated checks; visual QA via subagent is sufficient at this scale.
