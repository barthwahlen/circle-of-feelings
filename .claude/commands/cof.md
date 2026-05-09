# Circle of Feelings — project context

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
- Segment colors use no CSS filter (filters on wheel/seg cause iOS Safari GPU OOM crash)
- Wheel base shadow is inline JS style — suppressed in light mode via `html[data-theme="light"] .wheel-base { box-shadow: none !important }`

## Language system
- `currentLang` / `setLanguage(lang)` — triggers re-render of all localised content
- Data objects: `LANG_DATA` (emotion names + UI strings), `HELP_DATA` (how-to tab), `INFO_DATA` (about/purpose/resources tabs)
- Language persisted in URL `?lang=xx` search param (English omitted)
- `tEmo(name)` / `tUI(key)` — translation helpers

## Overlays
- **Emotion overlay**: `#overlay` / `#overlay-card` / `.overlay-body` — has peek + expand states
- Emotion overlay element refs are cached at module level: `ovBar`, `ovTitle`, `ovSimilar`, `ovSensations`, `ovTelling`, `ovHelping`, `ovHSimilar`, `ovHSensations`, `ovHTelling`, `ovHHelping` — add new ones here if extending the overlay, don't use `getElementById` inside functions
- **Info overlay**: `#help-overlay` / `#help-card` — 4 tabs (how/about/purpose/more) using `.info-panel`
- Tabs use `data-tab` attribute; panels use `id="info-panel-{tab}"` pattern
- `renderHelpOverlay()` renders all 4 panels (how from `HELP_DATA`, rest from `INFO_DATA`)
- `openHelp()` / `animClose(overlay)` for open/close
- Sheet drag: `addSheetDrag(overlay, card, onCollapse, onDismiss?, onExpand?)`
- `clearAll(force?)` dismisses emotion overlay; `force=true` = instant, no animation
## Key gotchas
- `html[data-theme="dark"]` never matches — dark mode has no attribute on `<html>`
- Segment colors are inline JS styles — use CSS `filter` on `.seg`, not color changes
- The `#info-bar` is the top title bar ("Circle of Feelings"), unrelated to the info overlay
- Orientation-change crash fix: resize is debounced 120 ms + `cancelMotion()` + `wheelScale` clamp in `fitWheel`
- NEVER apply CSS `filter` to `#wheel` itself — a container-level filter creates a single huge GPU layer that unconditionally OOMs iOS Safari on orientation change or overlay open
- `.seg` filters (`:hover`, `.hi` drop-shadow, `.dim`) are fine on desktop but MUST be suppressed on iPhone-class viewports via `@media (max-width: 932px)` — each `.hi` segment otherwise allocates its own GPU compositing layer, and combined with the overlay's `backdrop-filter` blows the per-tab GPU budget. Landscape was missed once (rule was inside `(orientation: portrait)`) and overlay-open crashed in landscape with the "a problem repeatedly occurred" banner.
- Peek height (`--peek-h`) is recalculated in the debounced resize handler so orientation changes don't jam the overlay card position
- `allLabels` items have a `.span` property (cached at build time) — use `l.span.textContent`, not `l.el.querySelector('span')`
- `tED()` caches the resolved translation object in `_tED_lang` / `_tED_data` — reset these if adding a new language mid-session
