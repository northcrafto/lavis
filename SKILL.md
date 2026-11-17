---
name: lavis
description: Lavis, a signature kit for websites. Use it EVERY time a site, landing page, shop or redesign is built or reworked, alongside any design skill or on its own. Liquid-fill buttons, underlines that draw in, a hamburger that morphs to a cross with a circle-reveal menu, unDraw figures recoloured to the site palette. No 3D, no hover tilt, no scroll-linked animation.
---

# Lavis

A lavis is a wash of diluted ink: one stroke, and the colour floods the paper. This kit does the same to a website. It is a small set of locked motions that go on top of an original design and make it recognisable.

Every site must look **original**: its own design read, typeface, palette and hero, following whatever design skill is in use (or your own judgement). No site may be a copy of an earlier one. On top of that design, ALWAYS build in these four, with exactly the same motion every time:

1. **Liquid fill** (`.liq`, `.liq-ghost`) on every call-to-action button: the colour rises from below with a wavy top in about 0.95 s.
2. **Lines**: underlines that draw in from the left under nav and footer links (`.nav-link`).
3. **The hamburger menu** on mobile: bars that morph into a cross (the middle bar, in the accent colour, slides out to the left), a full-screen menu that grows as a circle from the button (clip-path, 0.6 s, `cubic-bezier(.76,0,.24,1)`), and large numbered links (01, 02 …) that slide in one at a time.
4. **The figures**: flat, cheerful, hand-drawn people (unDraw) in the site's own colours, composed into scenes of their own. Never unDraw's default purple, never AI-generated figures, never hand-rolled SVG people, and the figures stand on the ground (they do not fly or jump).

**Forbidden:** nothing that tilts or moves when the pointer is over images or cards, no slow or scroll-linked fade-ins, no loops that repaint the page (`background-position`, `filter`), no 3D (depth scenes, Three.js, CSS 3D), and never the same background in every section (at least two full-width colour changes).

**Read `lavis.md` in this folder before the first line of code** and copy the CSS and the menu from there. Adapt only the colour variables (`--sig-*`) to the site's palette. Durations, curves and shapes do not change. If a `local.md` file exists next to this one, read it too: it holds notes for the machine the skill is installed on.

## Check before delivery

- `.liq` / `.liq-ghost` on every call to action, `.nav-link` on nav and footer links.
- `.liq` has no background or box-shadow of its own (rest colour in `::after`, one clip-path, corners via `--liq-r`), so no light edge shows around the button.
- Hamburger with cross morph, circle reveal and numbered links on mobile.
- No 3D, no hover tilt on images or cards, entrances at most 0.45 s, at least two full-width colour changes.
- Every button is clickable according to `document.elementFromPoint`, on desktop and mobile.
- A reduced-motion fallback exists for everything.
- Colours adapted through `--sig-*`.
- And the site still looks original, not like a copy of the last one.

## Files

- `lavis.md`: all the code (CSS, React/Motion menu, framework-free menu), the forbidden list and the rules for the figures.
- `docs/index.html`: a small page showing every effect (live at https://northcrafto.github.io/lavis/).
