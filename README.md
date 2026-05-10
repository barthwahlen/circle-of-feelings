# Circle of Feelings

An interactive wheel for finding the right word for what you're feeling — with similar words, typical bodily sensations, what the feeling might be telling you, and how it might help.

**[Try it →](https://barthwahlen.github.io/circle-of-feelings/)**

<!-- Add a screenshot here later: ![Circle of Feelings](./screenshot.png) -->

## Why naming feelings matters

Naming an emotion precisely — known as **affect labelling** — measurably reduces activity in the amygdala, the brain's threat centre. You feel less overwhelmed not because the feeling went away, but because language gives your prefrontal cortex something to work with.

"I feel bad" is a dead end. "I feel overlooked and quietly resentful" points directly to what needs addressing. **Emotional granularity** — the ability to distinguish subtle shades of feeling — is linked to better decision-making, stronger relationships, and greater resilience under stress.

You don't have to be in crisis to use this. Curiosity about yourself is enough.

## Features

- **Eight core emotions** (Plutchik) with mid- and outer-ring variations — many dozens of named feelings in total
- **Per-emotion detail**: similar words, typical sensations, what it might be telling you, how it might help
- **Deep-link sharing**: every feeling has a URL — paste `https://barthwahlen.github.io/circle-of-feelings/?lang=fr#joie` and the recipient lands on _Joie_ in French
- **11 languages**, with right-to-left layouts for Arabic and Persian
- **Light & dark themes**, persisted per device
- **No accounts, no tracking, no build step** — one HTML file you can open offline

## Languages

English · Français · Español · Deutsch · Português · Nederlands · Italiano · 日本語 · 中文 · العربية · فارسی

Switch via the language menu in the top-right; selection persists in the URL (`?lang=xx`).

## Local development

No bundler, no dependencies, no install step. Either:

```bash
# Open the file directly
open index.html

# …or serve it (preserves history.replaceState behaviour for deep-links)
python3 -m http.server 7734
# → http://localhost:7734
```

Edit `index.html`, refresh the browser. That's the whole loop.

## Architecture

Everything lives in a single `index.html`:

- **Wheel rendering** — segments are `<div class="seg">` elements with `clip-path` polygons, generated in JS from the `DATA` array
- **Theming** — CSS custom properties on `:root` (dark default) with `html[data-theme="light"]` overrides
- **Internationalisation** — `LANG_DATA` (UI strings) and `_T` (emotion names); language persisted via `?lang=xx`
- **Deep linking** — `#emotion-slug` in the URL hash; resolved on load via `slugIndex` and `applyHashOnLoad`
- **Overlays** — emotion sheet (`#overlay` / `#overlay-card`) and info sheet (`#help-overlay` / `#help-card`) share a sheet-drag controller (`addSheetDrag`)

When working on this in [Claude Code](https://claude.com/code), `.claude/commands/cof.md` is loaded with `/cof` as concise project context.

## Acknowledgements

The eight-emotion structure draws on **Robert Plutchik's wheel of emotions** and decades of clinical and neuroscientific research. Specific influences:

- _How Emotions Are Made_ — **Lisa Feldman Barrett**'s book on the science of emotion and why naming them precisely matters
- _Permission to Feel_ — **Marc Brackett** (Yale Center for Emotional Intelligence) on RULER, the evidence-based approach to emotional literacy
- _Emotional Intelligence_ — **Daniel Goleman**'s book on why emotional intelligence can matter more than IQ
- **Atlas of Emotions** — Paul Ekman's interactive map of emotion states, built with the Dalai Lama
- _Affect labelling_ — Lieberman et al. (2007), the original research showing that naming emotions reduces amygdala activation
- _Emotional granularity_ — Barrett &amp; colleagues on why distinguishing emotion states matters for wellbeing

## License

MIT — see [LICENSE](./LICENSE).

— Barth, 2026
