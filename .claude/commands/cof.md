# Circle of Feelings — project context

## Repo & hosting
- GitHub: `https://github.com/barthwahlen/circle-of-feelings`
- Live: `https://barthwahlen.github.io/circle-of-feelings/`
- Workflow: edit `index.html` → commit → `git push` → Pages redeploys in ~60 s
- Issues: `gh issue list` / `gh issue view N` — user logs issues on GitHub; reference with `fixes #N` in commit messages

## File
Single self-contained file: `index.html`
All CSS, HTML, and JS in one file. No build step. History lives in git — use `git log`, `git diff`, `git restore` instead of backup files.

## Wheel architecture
- Segments are `<div class="seg">` with `clip-path` polygons, built entirely in JS — no hardcoded HTML for segments
- Colors are inline `background` styles set by JS — CSS cannot change them directly, use `filter` overrides on `.seg`
- `lighten(hex, factor)` interpolates toward white to generate mid/outer ring colors from `palette.bg`
- Each emotion has `palette.bg` (hex), `coreText`, `midText`, `outerText`, `midFactor`, `outerFactor`
- Ring font weights: core 600 (Host Grotesk), ring 1 600 (Host Grotesk), ring 2 500 (DM Sans), ring 3 400 (DM Sans)

## Theming
- **Dark mode = default** — `<html>` has NO `data-theme` attribute
- **Light mode** = `data-theme="light"` on `<html>`
- ⚠️ `html[data-theme="dark"]` **never matches** — always use `html:not([data-theme="light"])` for dark-only CSS
- CSS tokens defined in `:root` (dark defaults) + `html[data-theme="light"]` overrides
- Theme persisted in `localStorage` key `cof-theme`; OS `prefers-color-scheme` used as fallback
- Wheel base shadow is inline JS style — suppressed in light mode via `html[data-theme="light"] .wheel-base { box-shadow: none !important }`

## Language system
- `currentLang` / `setLanguage(lang)` — triggers re-render of all localised content
- Data objects: `LANG_DATA` (emotion names + UI strings), `HELP_DATA` (how-to tab), `INFO_DATA` (about/purpose/resources tabs)
- Language persisted in URL `?lang=xx` search param (English omitted)
- `tEmo(name)` / `tUI(key)` — translation helpers
- `tED()` caches the resolved translation object in `_tED_lang` / `_tED_data` — reset these if adding a new language mid-session
- When adding a new `tUI()` key, add it to all 11 languages in `LANG_DATA`: en · fr · es · de · pt · nl · it · ja · zh · ar · fa
  - ⚠️ French strings may contain ` ` (non-breaking space) — edit each language block individually, not in bulk

## Overlays
- **Emotion overlay**: `#overlay` / `#overlay-card` / `.overlay-body` — has peek + expand states
- Emotion overlay element refs are cached at module level: `ovBar`, `ovTitle`, `ovSimilar`, `ovSensations`, `ovTelling`, `ovHelping`, `ovHSimilar`, `ovHSensations`, `ovHTelling`, `ovHHelping`, `ovShare` — add new ones here if extending the overlay; don't use `getElementById` inside functions
- **Share button**: `#share-feeling` / `.share-feeling-btn` — copies `location.href` (already contains `?lang=xx#slug`) to clipboard; `.copied` state shows `tUI('linkCopied')` for 2 s then reverts; timer stored on `ovShare._revertTimer`; reset in `openOverlay()` before each open
- **Intro hint**: `#intro-hint` — floating pill shown on first visit only; dismissed on first `pointerdown`; seen-state stored in `localStorage` key `cof-hint-seen`; text = `tUI('dragOrTapToExplore')`; rendered by IIFE at end of script
- **Info overlay**: `#help-overlay` / `#help-card` — 4 tabs (how/about/purpose/more) using `.info-panel`
- Tabs use `data-tab` attribute; panels use `id="info-panel-{tab}"` pattern
- `renderHelpOverlay()` renders all 4 panels (how from `HELP_DATA`, rest from `INFO_DATA`)
- `openHelp()` / `animClose(overlay)` for open/close
- Sheet drag: `addSheetDrag(overlay, card, onCollapse, onDismiss?, onExpand?)`
- `clearAll(force?)` dismisses emotion overlay; `force=true` = instant, no animation

## Key gotchas
- `html[data-theme="dark"]` never matches — dark mode has no attribute on `<html>`
- Segment colors are inline JS styles — use CSS `filter` on `.seg`, not color changes
- Do not confuse `#help-overlay` (the info overlay) with a title bar — there is no app title element. First-use guidance is handled entirely by `#intro-hint`
- Orientation-change crash fix: resize is debounced 120 ms + `cancelMotion()` + `wheelScale` clamp in `fitWheel`
- NEVER apply CSS `filter` to `#wheel` itself — a container-level filter creates a single huge GPU layer that unconditionally OOMs iOS Safari on orientation change or overlay open
- `.seg` filters (`:hover`, `.hi` drop-shadow, `.dim`) are fine on desktop but MUST be suppressed on iPhone-class viewports via `@media (max-width: 932px)` (orientation-agnostic) — each `.hi` segment gets its own GPU compositing layer; combined with the overlay's `backdrop-filter` this blows iOS Safari's per-tab GPU budget. Don't scope this rule to portrait-only — landscape is equally constrained.
- Peek height (`--peek-h`) is recalculated in the debounced resize handler so orientation changes don't jam the overlay card position
- `allLabels` items have a `.span` property (cached at build time) — use `l.span.textContent`, not `l.el.querySelector('span')`
