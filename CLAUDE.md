# Repo orientation for Claude — alcf-ai-agent-coding-talk

This is a 32-slide ALCF talk evaluating Brice's use of Claude Code on two projects: CCS and rust-gpu + claspr. The deck was authored on the Linux laptop. It will be reviewed and finalized on the **other laptop** (Intel macOS), where you may be the next agent picking it up. Read this before editing.

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

The template (`ALCF Presentation Template.pptx`) is **not in the repo** — put it at `/tmp/ALCF Presentation Template.pptx` (or edit `TEMPLATE` in `build_deck.py`).

```bash
pip install --break-system-packages "markitdown[pptx]" python-pptx pillow
python3 build_deck.py
soffice --headless --convert-to pdf ai-agent-coding-alcf.pptx
pdftoppm -jpeg -r 90 ai-agent-coding-alcf.pdf slide   # per-slide JPGs for QA
```

After editing, `git add ai-agent-coding-alcf.pptx ai-agent-coding-alcf.pdf build_deck.py` and commit. Re-rendering the PDF on every change keeps the repo viewable on GitHub.

## What is locked in vs what is open

**Locked in (do not change without asking):**
- Title: *From Directing to Dialogue: Ten Weeks of AI Agent Coding for Performance Engineering*. Subtitle: *Two ALCF-adjacent projects · 7 sessions · ~1,500 prompts*. The "directing → dialogue" framing is the thesis the rest of the deck builds on; CCS slides set up "directing", rust-gpu slides set up "dialogue".
- Slide order (Title · Agenda · Why · Setup · Side-by-side · §CCS×6 · §rust-gpu/claspr×8 · §Cross-cutting×8 · Implications · Recommendations · Closing). Rearranging requires updating the agenda slide (#2) and §-break slides.
- ALCF template layouts only. No custom layouts, no images injected. The template's master controls fonts/colors; we do not override them.
- 32 slides total. Was sized for a 30-minute talk at ~1 slide/min.

**Open for the other laptop to enrich:**
- §CCS slides (#6–12). Built from the CCS repo + the user prompts surviving in `~/.claude/history.jsonl` only — the full session JSONLs were already evicted by Claude Code's 30-day session retention before this deck was built (see *Data availability* below). If the other laptop has surviving artifacts (e.g. local notes, screenshots, anything else), those are net-new and worth adding.
- Slides #22–24 (tool-call distribution / sub-agent adoption / token economics): currently aggregated from **only the two locally-surviving rust-gpu sessions**. If the other laptop has additional surviving sessions (within its own 30-day window), re-mine and recompute. The current numbers under-count.
- Quotes on slides #10 and #18 are paraphrased lightly from real user prompts; verbatim alternatives are fine.

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

1. **4-block "big idea" layout (Layout 9)**. The placeholder default is 32pt bold white with `<a:noAutofit/>`. Putting `"Title\nbody"` as one paragraph blew out the box. Use the `set_block(ph, header, body)` helper, which writes the header at level-0 default (inherits 32pt bold) plus a second paragraph with `Pt(14)` and `RGBColor(0xFF, 0xFF, 0xFF)` forced (color inheritance is unreliable across renderers). Keep block bodies to ~25 words max; keep block headers to one line (two-line headers eat body height and trigger clipping at the box bottom — caught on slides 3 and 10 before fixing).
2. **Subtitle placeholder (idx=13)** is 26pt bold accent-2 color, 0.64" tall, `noAutofit`. Anything wrapping to two lines collides with the body below or the title above. **Cap subtitles at ~80 chars / single line.**
3. **Bullet body (idx=14)** ends at y≈6.86"; the master's footer starts soon after. **Cap content lists so the rendered text doesn't approach the footer**. Trim 1–2 bullets if a slide gets close.
4. **Title slide picture placeholder (idx=10)** has a "click to insert image" prompt that renders as a gray box if left empty. The current code removes it explicitly along with unused presenter slots (idx 19/20/21/22). If you ever re-use Layout 0, do the same.
5. **Typo audit**: real user prompts get cleaned (e.g. `"Allright"` → `"Alright"`). If a typo is the point of the quote, mark it `[sic]`.

The `build_deck.py` helpers (`set_text`, `set_block`, `set_bullets`, `get_ph`, `remove_placeholder`) encode these fixes. Use them, don't bypass them.

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

LibreOffice renders close to PowerPoint but not identical. For final review, open the pptx in PowerPoint on macOS and skim. The known small differences:
- LibreOffice doesn't have the exact ALCF title font (substitutes). PowerPoint will use the real one.
- Multi-line title behavior in Layout 3/5/6 may differ by ~1 px. Watch the rust-gpu interaction title (#18) and Implications (#30) which both wrap to 2 lines.

## The source of truth for the numbers

| Number on slide | Source |
|---|---|
| 96/99 CCS PRs, ~16/wk, #24→#122 | `gh pr list -R argonne-lcf/CCS --author bricevideau-ai --state all --limit 200` |
| 530 / 964 user prompts | `~/.claude/history.jsonl` filtered by sessionId |
| rust-gpu PR #3 +24,540/-595/761 files | `gh pr view 3 -R bricevideau-ai/rust-gpu --json additions,deletions,changedFiles` |
| claspr 182 commits / ~81K LOC | `git log --oneline` and `find … -name '*.rs' \| xargs wc -l` in `~/projects/claspr` |
| 7.18B cache reads / 273K input / 15.5M output | aggregated from the 2 surviving rust-gpu JSONLs; see `session_stats.json` |
| 24 sub-agents (16 Explore, 7 general, 1 Plan) | same |
| 8 auto-compaction events | grep `"This session is being continued"` in `11e6e374-*.jsonl` |

If you re-mine on the other laptop, regenerate `session_stats.json` first, then update the affected slides.

## Public-repo etiquette

- Repo is **public** on `bricevideau-ai/alcf-ai-agent-coding-talk`. Do not commit anything that would embarrass the user, leak credentials, or quote third parties without consent. Quotes used so far are all the user's own prompts.
- Commit messages should be specific (what changed, why), one-topic. Co-author Claude.
- Pushing directly to `main` is fine here (it's a small artifact repo, not a code project with reviewers).

## Things I deliberately did not do

- Did not add the source `.jpg` slide renders — they're ephemeral, regenerate from the pptx.
- Did not commit the ALCF template — it's not ours to redistribute.
- Did not write a Makefile — the four-line workflow at the top is short enough.
- Did not propagate the QA pass into the build script as automated checks; visual QA via subagent is sufficient at this scale.
