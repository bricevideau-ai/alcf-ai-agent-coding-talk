# Repo orientation for Claude — alcf-ai-agent-coding-talk

This is a 32-slide ALCF talk evaluating Brice's use of Claude Code on two projects: CCS and rust-gpu + claspr. The deck was authored on the **Arm Mac laptop** (Ubuntu environment on macOS host — `aarch64`, hostname `bvideau-VMware20-1`). It will be reviewed and finalized on the **other laptop**, which is a **native Intel Ubuntu install** — where you, the next agent, are reading this. Read this before editing.

If a user prompt anywhere in the surviving session history says "the linux laptop", they mean the native-Intel-Ubuntu laptop (i.e. you). If they say "the mac" or "the other laptop", they mean the Arm one where this deck was authored.

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
- Title: *From Directing to Dialogue: Fifteen Weeks of AI Agent Coding for Performance Engineering*. Subtitle: *Two projects · 7 sessions · ~1,500 prompts*. The "directing → dialogue" framing is the thesis the rest of the deck builds on; CCS slides set up "directing", rust-gpu slides set up "dialogue". Fifteen weeks = Feb 25 → Jun 8 (CCS start to last claspr commit), not just the rust-gpu portion.
- Slide order (Title · Agenda · Why · Setup · Side-by-side · Chronology · §CCS×7 · §rust-gpu/claspr×9 · §Cross-cutting×9 · Implications · Recommendations · Closing). Each §-block includes a "what user code looks like" showcase slide. Slide 6 is the session-chronology text-timeline that surfaces the one 11-day cross-project overlap (CCS s3 + rust-gpu s1, Mar 27 → Apr 7). Rearranging requires updating the agenda slide (#2) and §-break slides.
- ALCF template layouts only. No custom layouts, no images injected. The template's master controls fonts/colors; we do not override them — except inside the `set_block` and `set_code` helpers (see *Systemic QA issues* below).
- 36 slides total. Sized for ~30–35 min of presenting. Slot is up to an hour including Q&A and Brice plans to add his own slides too — so don't pad this deck further; trim if you want to grow elsewhere.

**Open for the other laptop to enrich:**
- §CCS slides (#7–14). Built from the CCS repo + the user prompts surviving in `~/.claude/history.jsonl` only — the full session JSONLs were already evicted by Claude Code's 30-day session retention before this deck was built (see *Data availability* below). If the other laptop has surviving artifacts (e.g. local notes, screenshots, anything else), those are net-new and worth adding.
- Slides #25–28 (tool-call distribution / sub-agent adoption / model progression / token economics): currently aggregated from **only the two locally-surviving rust-gpu sessions** (a910ecaa, 11e6e374). For tool/sub-agent counts use the merge recipe at the bottom; **for cost, use ccusage** (see `ccusage/README.md`).
- Quotes on slides #12 and #21 are paraphrased lightly from real user prompts; verbatim alternatives are fine.
- Code showcases (#8 CCS, #16 claspr): trimmed examples; longer or different illustrative snippets are fine if they fit at 9pt Consolas in the left column.
- Chronology slide (#6) is text-based monospace ASCII. If the other-laptop session list (within its 30-day window) reveals any I missed, add a row and update the parallelism caption.

## Critical context: data availability

> The new Claude Code 30-day session-eviction policy is what bit us. Sessions older than 30 days at the time of authoring (2026-06-08) are gone from disk. They are not on the other laptop either — they're just lost.

Available on the Linux laptop at authoring time:
- `~/.claude/projects/-home-claudecode-projects/a910ecaa-*.jsonl` (rust-gpu, May 4–13)
- `~/.claude/projects/-home-claudecode-projects/11e6e374-*.jsonl` (rust-gpu, May 13–Jun 8)
- `~/.claude/projects/-home-claudecode-projects/memory/` (30+ entries — these survive)
- `~/.claude/history.jsonl` (prompts only — does survive eviction; ~1,500 prompts across all 7 sessions)

Lost (do not bother searching for):
- CCS sessions `3f086a62`, `add0d69f`, `ca6a2dde` (Feb–Apr 2026)
- rust-gpu sessions `dd63080c`, `e493c2fd` (Mar–Apr 2026)

What survives for those lost sessions:
- User prompts in `~/.claude/history.jsonl` (the deck pulls extensively from these — see `deck_outline.md`)
- Git history of the actual codebases (CCS, rust-gpu, claspr) and the PRs in those repos
- Memory entries built up over time

**Optional add for the other agent:** if you think the 30-day eviction caveat itself is worth a bullet on slide #22 or #26, it's a fair methodological note. I left it out to keep the deck focused, but it's a real signal about how this kind of retrospective gets harder over time.

## Systemic QA issues already fixed — do not regress

The first build broke in predictable ways. The current `build_deck.py` has fixes in place; if you re-edit, keep them:

1. **4-block "big idea" layout (Layout 9)**. The placeholder default is 32pt bold white with `<a:noAutofit/>`. Putting `"Title\nbody"` as one paragraph blew out the box. Use the `set_block(ph, header, body)` helper, which writes the header at level-0 default (inherits 32pt bold) plus a second paragraph with `Pt(14)` and `RGBColor(0xFF, 0xFF, 0xFF)` forced (color inheritance is unreliable across renderers). Keep block bodies to ~25 words max; keep block headers to one line (two-line headers eat body height and trigger clipping at the box bottom — caught on the original slides 3 and 11 before fixing).
2. **Subtitle placeholder (idx=13)** is 26pt bold accent-2 color, 0.64" tall, `noAutofit`. Anything wrapping to two lines collides with the body below or the title above. **Cap subtitles at ~80 chars / single line.**
3. **Bullet body (idx=14)** ends at y≈6.86"; the master's footer starts soon after. **Cap content lists so the rendered text doesn't approach the footer**. Trim 1–2 bullets if a slide gets close.
4. **Title slide picture placeholder (idx=10)** has a "click to insert image" prompt that renders as a gray box if left empty. The current code removes it explicitly along with unused presenter slots (idx 19/20/21/22). If you ever re-use Layout 0, do the same.
5. **Typo audit**: real user prompts get cleaned (e.g. `"Allright"` → `"Alright"`). If a typo is the point of the quote, mark it `[sic]`.

The `build_deck.py` helpers (`set_text`, `set_block`, `set_bullets`, `set_code`, `get_ph`, `remove_placeholder`) encode these fixes. Use them, don't bypass them.

`set_code(ph, code, *, font="Consolas", size=9)` is for code-showcase slides. It explicitly overrides paragraph spacing (`spcBef`/`spcAft` to 0 and `lnSpc` to 100%) because the master adds bullet-style paragraph spacing that ruins code density. Keep code under ~20 lines at the default 9pt — anything longer overflows the 5.96" wide × 3.97" tall column.

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
| 530 / 964 user prompts | `~/.claude/history.jsonl` filtered by sessionId |
| rust-gpu PR #3 +24,540/-595/761 files | `gh pr view 3 -R bricevideau-ai/rust-gpu --json additions,deletions,changedFiles` |
| claspr 182 commits / ~81K LOC | `git log --oneline` and `find … -name '*.rs' \| xargs wc -l` in `~/projects/claspr` |
| Cost figures on slide 28 ($2,505 measured, $4.5–5.5K projected full project) | **`ccusage/` directory** — `ccusage@latest claude {session,daily,blocks} --json` snapshots. **This is canonical for cost** — do not use the per-token totals I computed in `session_stats.json`. The README there has the projection method. |
| 3.5B cache reads / 141K input / 6.1M output | `ccusage/session.json` (rust-gpu sessions, summed); supersedes my earlier `session_stats.json` numbers which were double-counted. Visible-portion-only. |
| 24 sub-agents (16 Explore, 7 general, 1 Plan) | `session_stats.json` (this metric was correct) |
| 8 auto-compaction events | grep `"This session is being continued"` in `11e6e374-*.jsonl` |

If you re-mine on the other laptop, **merge into** `session_stats.json` (don't overwrite — read its `merge_instructions` field for the rules: same-session data union, prefer newer/larger transcripts). Then update slide-level numbers in `build_deck.py` and rebuild.

### Cross-laptop merge recipe (paste-ready)

```python
import json
# 1. Load the existing aggregated file (from this laptop) — committed in the repo.
with open("session_stats.json") as f:
    existing = json.load(f)

# 2. Mine your own surviving session JSONLs the same way build_deck-side mining
#    works (see project_stats.json's structure — by_session keys are session-id
#    strings, each with first_date/last_date/tokens/tool_uses/etc.).
#    For each session id in your scan, if existing["by_session"][sid]
#    .available_locally is False (or sid is absent), replace/insert with yours.

# 3. Recompute by_project totals by summing across by_session entries.
# 4. Re-derive parallelism_note if any new session spans overlap.
# 5. Write back to session_stats.json, then run build_deck.py with updated
#    slide-text numbers.
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
