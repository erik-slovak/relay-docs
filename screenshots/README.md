# Marketing assets — Markdown Review

Dark-mode, **true retina 3200×1800** (1600×900 viewport at devicePixelRatio 2),
captured with agent-browser from **production** (`markdown-review.kanzen.sh`)
reviewing the two demo PRs in this repo. All visible references to
`erik-slovak/relay-docs` were rewritten in the DOM to the fictional
**`relayhq/relay`** before capture.

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

## Notes

- **Retina lever:** `agent-browser set viewport 1600 900 2` (CDP
  device-pixel-ratio = 2) → 3200×1800 output. The
  `--force-device-scale-factor` launch flag does **not** work and is silently
  ignored if the daemon is already running.
- **`07-range-picker.png` caveat:** the commit picker is a native `<select>`,
  whose OS-drawn popup no headless tool can capture — so the open dropdown in
  that shot is a styled DOM overlay standing in for the real popup (faithful
  content, synthesized presentation).
- **Video is deferred on purpose.** A proper motion demo of the commit picker
  needs the native `<select>` replaced with a custom dropdown component first;
  once that lands, the picker (and a full scroll-through) can be screen-recorded
  live. Until then this folder is stills only.
