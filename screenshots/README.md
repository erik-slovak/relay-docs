# Marketing screenshots — Markdown Review

Dark-mode, 3200×1800 (2× retina, 1600×900 viewport), captured from
**production** (`markdown-review.kanzen.sh`) reviewing the two demo PRs in
this repo. All visible references to `erik-slovak/relay-docs` were rewritten
in the DOM to the fictional **`relayhq/relay`** before capture.

## Shots → suggested thread mapping

| File | Shows | Tweet |
| --- | --- | --- |
| `01-hero.png` | Full two-pane layout: diff + rendered, sectioned, orange-marked modified code block, table | 1 (opener) |
| `02-highlight.png` | **Best single shot** — word-level diff, per-bullet green marks, red deletions rendered in place, inline comment thread | 2 (highlighting) |
| `03-mermaid.png` | Mermaid flowchart rendered in dark beside its source diff; red/green prose blocks below | 3 (mermaid) |
| `06-pr2-mermaid-seq.png` | Mermaid sequence diagram, all-green new doc (PR 2) | 3 alternative |
| `07-range-picker.png` | Commit-range picker open: v0 BASE → v4 HEAD version labels (dropdown is a styled DOM overlay — native popups can't be captured headlessly) | 4 (range scope) |
| `05-range-scope.png` | Pinned scope v1 → v3 shown in both pickers | 4 alternative |
| `04-viewed.png` | Viewed tracking: 1/2 viewed progress, checked section collapsed | spare |
| `demo-slideshow.mp4` | 16.7 s crossfade reel of the six shots, 1920×1080 | opener alternative |

## Capture notes

- **agent-browser (CDP)** produced all images: `--color-scheme dark`,
  Chrome launched with `--force-device-scale-factor=2`, viewport-only
  captures (no browser chrome, no URL bar).
- **Native macOS `screencapture` failed**: the terminal lacks the
  *Screen Recording* permission (System Settings → Privacy & Security).
  Grant it and window-level captures (`screencapture -l <windowId>`)
  become possible, including real open dropdowns.
- **webreel (webreel.dev) is unusable for this app**: its config schema has
  no cookie/localStorage/storage-state injection and no JS-eval step, so it
  can never get past the Clerk gate (nor fake the repo name). Same root
  cause: agent-browser's own video recorder starts a fresh browser context,
  which lands on the sign-in page (and then Cloudflare's bot check), so
  in-app video needs the Screen Recording permission route too. The
  slideshow MP4 was assembled from the stills with ffmpeg instead.
