# Marketing assets — Markdown Review

Dark-mode, **true retina 3200×1800** (1600×900 viewport at devicePixelRatio 2),
captured from **production** (`markdown-review.kanzen.sh`) reviewing the two
demo PRs in this repo. All visible references to `erik-slovak/relay-docs` were
rewritten in the DOM to the fictional **`relayhq/relay`** before capture.

## Stills → suggested thread mapping

| File | Shows | Tweet |
| --- | --- | --- |
| `01-hero.png` | Full two-pane layout: diff + rendered, sectioned, orange-marked modified code block, table | 1 (opener) |
| `02-highlight.png` | **Best single shot** — word-level diff, per-bullet green marks, red deletions rendered in place, inline comment thread | 2 (highlighting) |
| `03-mermaid.png` | Mermaid flowchart rendered in dark beside its source diff; red/green prose blocks below | 3 (mermaid) |
| `06-pr2-mermaid-seq.png` | Mermaid sequence diagram, all-green new doc (PR 2) | 3 alternative |
| `07-range-picker.png` | Commit-range picker open: v0 BASE → v4 HEAD version labels | 4 (range scope) |
| `05-range-scope.png` | Pinned scope v1 → v3 shown in both pickers | 4 alternative |
| `04-viewed.png` | Viewed tracking: 1/2 viewed progress, checked section collapsed | spare |
| `demo-slideshow.mp4` | 16.7 s crossfade reel of the six stills, 1920×1080 | opener alternative |

## Motion — `commit-picker/`

`commit-picker.mp4` (1600-wide, web-friendly), `commit-picker-retina.mp4`
(3200×1800), and `commit-picker.gif` — a **scripted webreel demo** of the
commit-range picker: the cursor opens the base dropdown, glides down the
version list (v0…v4 with SHAs + commit messages), picks **v1**, and the diff
count updates (`+28 −7` → `+25 −4`) with the scope label.

⚠️ **Honesty note:** this clip is *not* a screen recording of the live app.
The commit picker is a native `<select>`, whose OS-drawn popup no headless
tool can capture or animate — so `commit-picker/mock.html` is a self-contained
HTML mock that reproduces the real UI, styling, SHAs and commit messages
exactly, driven by `webreel.config.json`. It's a faithful UX demo of a real
feature, but a mock. (The static `07-range-picker.png` uses the same
reasoning: a styled DOM overlay standing in for the native popup.)

## Capture notes (updated)

- **Retina, correctly this time.** agent-browser over CDP; the crispness
  lever is `agent-browser set viewport 1600 900 2` (device-metrics DPR = 2),
  **not** the `--force-device-scale-factor` launch flag, which the reused
  daemon silently ignored — that's why the first batch came out 1600×900 and
  looked soft. Fully restart the daemon (`close --all`) so context options
  actually apply.
- **webreel is usable after all — for mocks, not the live app.** It renders
  over CDP and, via the `viewport 3200×1800 + zoom:2` trick, outputs at 2×.
  But it launches a throwaway Chrome profile every run and exposes no
  cookie / storageState / JS-eval step, so it **cannot** pass the Clerk gate,
  inject the private-repo PAT, or rewrite the repo name — it can only drive
  self-contained pages like the picker mock. Gotcha found the hard way:
  webreel drives frames deterministically (`--enable-begin-frame-control`),
  which **freezes CSS `animation`/`transition` timelines** — reveal the
  dropdown with a plain `display` toggle, never an animated fade, or it stays
  stuck on its first (invisible) keyframe.
- **Native macOS `screencapture` still blocked**: the terminal lacks the
  *Screen Recording* permission (System Settings → Privacy & Security).
