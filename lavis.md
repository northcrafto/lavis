# Lavis: the kit

Every site gets its **own, original design**: its own typeface, palette, layout and hero. Four things come back **every time**, because they are the signature:

1. **Liquid fill** on buttons (`.liq`): colour rises from below with a wavy top, in about one second.
2. **Drawn lines**: underlines that draw in from the left under links (`.nav-link`).
3. **The hamburger menu**: three bars morph into a cross, a full-screen menu grows as a circle out of the button, and numbered links slide in one at a time.
4. **The figures**: flat, cheerful hand-drawn people from unDraw, recoloured to the site palette and composed into scenes of their own (section 5).

The motion (durations, curves, shapes) is locked. Only the **colours** adapt to the site, through CSS variables. Never swap in a faster fill, a circular blob, or a slide-out drawer in place of the circle.

---

## 1. CSS (paste into globals.css / your main stylesheet)

Set the variables to the site's own colours. Copy the rest verbatim.

```css
:root {
  --sig-accent: #e8622c;    /* the site's accent: underlines, the middle bar of the hamburger */
  --sig-btn: #ffffff;       /* button at rest */
  --sig-btn-ink: #16161a;   /* text at rest */
  --sig-fill-top: #2b2b30;  /* the liquid, top */
  --sig-fill-bottom: #121214; /* the liquid, bottom (text turns --sig-btn when filled) */
}

/* Liquid fill: the button fills from below with a wavy top.
   IMPORTANT: the button itself has NO background and no box-shadow. The rest colour and the edge ring live in
   ::after (bottom layer), the liquid in ::before, and the whole button is clipped with ONE clip-path. With the rest
   colour on the button itself it leaked out as a thin light line around the rounded edge under GPU rendering
   (Chromium; it does not show in headless screenshots). Set the corner radius with --liq-r, never border-radius
   directly. Never put background or box-shadow on .liq. */
.liq {
  --liq-r: 999px;
  position: relative; isolation: isolate; overflow: hidden; border: 0;
  border-radius: var(--liq-r); clip-path: inset(0 round var(--liq-r));
  background: transparent; color: var(--sig-btn-ink);
  transition: color .6s ease, transform .2s ease;
}
.liq::after {               /* rest colour + edge ring */
  content: ""; position: absolute; inset: 0; z-index: -2; pointer-events: none; border-radius: inherit;
  background: var(--sig-btn);
  box-shadow: inset 0 0 0 1.5px color-mix(in srgb, var(--sig-btn-ink) 20%, transparent);
  transition: opacity 0s;    /* back at once when the pointer leaves */
}
.liq::before {              /* the liquid */
  content: ""; position: absolute; z-index: -1; left: -15%; width: 130%; bottom: -2px; height: 170%;
  transform: translateY(100%);
  background: linear-gradient(180deg, var(--sig-fill-top) 0%, var(--sig-fill-bottom) 100%);
  border-radius: 46% 46% 0 0;
  transition: transform .95s cubic-bezier(.2,.75,.25,1);
  pointer-events: none;
}
.liq:hover, .liq:focus-visible { color: var(--sig-btn); }
.liq:hover::before, .liq:focus-visible::before { transform: translateY(0); }
/* Once the liquid reaches the top the rest colour is removed, so it cannot show at the edge */
.liq:hover::after, .liq:focus-visible::after { opacity: 0; transition: opacity 0s .95s; }
.liq:active { transform: translateY(1px); }
.liq:focus-visible { outline: 2px solid var(--sig-accent); outline-offset: 3px; }

/* Outline button that fills with the accent (secondary buttons). Same layering: the ring lives in ::after. */
.liq-ghost {
  --liq-r: 999px;
  position: relative; isolation: isolate; overflow: hidden; border: 0;
  border-radius: var(--liq-r); clip-path: inset(0 round var(--liq-r));
  background: transparent; color: inherit;
  transition: color .6s ease, transform .2s ease;
}
.liq-ghost::after {
  content: ""; position: absolute; inset: 0; z-index: -2; pointer-events: none; border-radius: inherit;
  box-shadow: inset 0 0 0 1px color-mix(in srgb, currentColor 18%, transparent);
  transition: opacity 0s;
}
.liq-ghost::before {
  content: ""; position: absolute; z-index: -1; left: -15%; width: 130%; bottom: -2px; height: 170%;
  transform: translateY(100%); background: var(--sig-accent); border-radius: 46% 46% 0 0;
  transition: transform .95s cubic-bezier(.2,.75,.25,1); pointer-events: none;
}
.liq-ghost:hover, .liq-ghost:focus-visible { color: var(--sig-btn); }
.liq-ghost:hover::before, .liq-ghost:focus-visible::before { transform: translateY(0); }
.liq-ghost:hover::after, .liq-ghost:focus-visible::after { opacity: 0; transition: opacity 0s .95s; }
.liq-ghost:active { transform: translateY(1px); }
.liq-ghost:focus-visible { outline: 2px solid var(--sig-accent); outline-offset: 3px; }

/* Underline drawn in from the left (nav, footer, "view all" links) */
.nav-link { position: relative; }
.nav-link::after {
  content: ""; position: absolute; left: 0; right: 0; bottom: -5px; height: 2px; border-radius: 2px;
  background: var(--sig-accent); transform: scaleX(0); transform-origin: left center;
  transition: transform .3s cubic-bezier(.4,0,.2,1);
}
.nav-link:hover::after, .nav-link:focus-visible::after, .nav-link[aria-current="page"]::after { transform: scaleX(1); }

@media (prefers-reduced-motion: reduce) {
  .liq::before, .liq-ghost::before { border-radius: 0; bottom: 0; height: 100%; transition: transform .2s ease; }
  .liq:hover::after, .liq:focus-visible::after, .liq-ghost:hover::after, .liq-ghost:focus-visible::after { transition: opacity 0s .2s; }
  .nav-link::after { transition: none; }
}
```

**Where the effects go:**
- `.liq` on every primary call to action (Buy, Book, Send, Download, Play). `.liq-ghost` on secondary buttons. Never both variants for the same intent.
- `.nav-link` on every text link in the header and the footer.
- On dark sites: a light or accent-coloured button that fills dark, or a dark `--sig-btn` with a light liquid. Check contrast in both states.

---

## 2. The hamburger menu: React / Next (Motion)

Shown below `md`. The menu sits **outside** `<header>`, because `backdrop-filter` on the header would otherwise make `position: fixed` relative to the header. The header (logo and cross) has a higher z-index than the menu, so both stay visible on top of it.

```tsx
"use client";
import Link from "next/link";
import { useEffect, useState } from "react";
import { AnimatePresence, motion, useReducedMotion } from "motion/react";

const LINKS = [
  { href: "/#services", label: "Services" },
  { href: "/#pricing", label: "Pricing" },
  { href: "/about", label: "About" },
  { href: "/contact", label: "Contact" },
];
// Centre of the circle = centre of the hamburger button (right edge minus ~42px, half the header height)
const ORIGIN = "calc(100% - 42px) 34px";

export function Nav() {
  const [open, setOpen] = useState(false);
  const reduce = useReducedMotion();

  useEffect(() => {
    document.body.style.overflow = open ? "hidden" : "";
    return () => { document.body.style.overflow = ""; };
  }, [open]);
  useEffect(() => {
    const h = (e: KeyboardEvent) => e.key === "Escape" && setOpen(false);
    window.addEventListener("keydown", h);
    return () => window.removeEventListener("keydown", h);
  }, []);

  return (
    <>
      <header className="sticky top-0 z-40 border-b border-line bg-bg/85 backdrop-blur-md">
        <nav className="mx-auto flex h-[68px] max-w-[1280px] items-center justify-between px-5 md:px-8">
          <Link href="/" className="relative z-50" onClick={() => setOpen(false)}>{/* logo */}</Link>
          <ul className="hidden items-center gap-8 md:flex">
            {LINKS.map((l) => (
              <li key={l.href}><a href={l.href} className="nav-link text-[15px] font-medium">{l.label}</a></li>
            ))}
          </ul>
          <a href="/#book" className="liq hidden h-11 items-center rounded-full px-5 text-[15px] font-semibold md:inline-flex">Book</a>

          {/* Three bars morph into a cross; the middle bar, in the accent colour, slides out to the left */}
          <button
            onClick={() => setOpen(!open)}
            aria-expanded={open}
            aria-label={open ? "Close menu" : "Menu"}
            className="relative z-50 -mr-2 flex h-11 w-11 items-center justify-center md:hidden"
          >
            <span className="relative block h-[16px] w-[24px]">
              <span className={`absolute left-0 h-[2.5px] w-full rounded-full bg-current transition-all duration-[450ms] [transition-timing-function:cubic-bezier(.6,.05,.2,1)] ${open ? "top-[6.75px] rotate-45" : "top-0"}`} />
              <span className={`absolute left-0 top-[6.75px] h-[2.5px] w-full rounded-full bg-[var(--sig-accent)] transition-all duration-[350ms] ${open ? "-translate-x-8 scale-x-[.35] opacity-0" : ""}`} />
              <span className={`absolute left-0 h-[2.5px] w-full rounded-full bg-current transition-all duration-[450ms] [transition-timing-function:cubic-bezier(.6,.05,.2,1)] ${open ? "top-[6.75px] -rotate-45" : "top-[13.5px]"}`} />
            </span>
          </button>
        </nav>
      </header>

      <AnimatePresence>
        {open && (
          <motion.div
            key="menu"
            className="fixed inset-0 z-30 flex flex-col bg-bg md:hidden"
            initial={reduce ? { opacity: 0 } : { clipPath: `circle(0px at ${ORIGIN})` }}
            animate={reduce ? { opacity: 1 } : { clipPath: `circle(150% at ${ORIGIN})` }}
            exit={reduce ? { opacity: 0 } : { clipPath: `circle(0px at ${ORIGIN})` }}
            transition={{ duration: 0.6, ease: [0.76, 0, 0.24, 1] }}
          >
            {/* Optional: the site mark, large and faint, in the bottom corner */}
            <nav className="relative flex flex-1 flex-col justify-center px-8 pt-[68px]">
              <ul className="space-y-2">
                {LINKS.map((l, i) => (
                  <motion.li
                    key={l.href}
                    initial={reduce ? false : { opacity: 0, x: -28 }}
                    animate={{ opacity: 1, x: 0 }}
                    exit={reduce ? undefined : { opacity: 0, x: -16 }}
                    transition={{ duration: 0.5, delay: 0.18 + i * 0.07, ease: [0.16, 1, 0.3, 1] }}
                  >
                    <a href={l.href} onClick={() => setOpen(false)} className="group flex items-baseline gap-4 py-2 font-display text-[40px] font-bold leading-none tracking-tight">
                      <span className="text-base font-semibold tabular-nums text-[var(--sig-accent)]">0{i + 1}</span>
                      <span className="relative">
                        {l.label}
                        <span className="absolute -bottom-1 left-0 h-[3px] w-full origin-left scale-x-0 rounded-full bg-[var(--sig-accent)] transition-transform duration-300 group-hover:scale-x-100 group-active:scale-x-100" />
                      </span>
                    </a>
                  </motion.li>
                ))}
              </ul>
              <motion.div
                initial={reduce ? false : { opacity: 0, y: 20 }}
                animate={{ opacity: 1, y: 0 }}
                exit={reduce ? undefined : { opacity: 0 }}
                transition={{ duration: 0.5, delay: 0.18 + LINKS.length * 0.07, ease: [0.16, 1, 0.3, 1] }}
                className="mt-10"
              >
                <a href="/#book" onClick={() => setOpen(false)} className="liq inline-flex h-14 items-center rounded-full px-8 text-base font-semibold">Book</a>
              </motion.div>
            </nav>
          </motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```

## 3. The hamburger menu without a framework (static HTML, Shopify themes)

Same motion. The circle starts from the button's real position (`--cx`/`--cy` are computed on click).

```html
<button id="menu-btn" class="relative z-50 grid size-11 place-items-center md:hidden" aria-expanded="false" aria-controls="menu" aria-label="Menu">
  <span class="hb"><span></span><span></span><span></span></span>
</button>
<div id="menu" class="fixed inset-0 z-30 flex flex-col justify-center bg-[var(--bg)] px-8 md:hidden" aria-hidden="true">
  <ul class="space-y-2">
    <li class="m-item" style="--i:0"><a href="#services" class="flex items-baseline gap-4 py-2 text-[40px] font-bold leading-none"><span class="text-base text-[var(--sig-accent)]">01</span>Services</a></li>
    <li class="m-item" style="--i:1"><a href="#pricing" class="flex items-baseline gap-4 py-2 text-[40px] font-bold leading-none"><span class="text-base text-[var(--sig-accent)]">02</span>Pricing</a></li>
  </ul>
</div>
```

```css
.hb { position: relative; display: block; width: 24px; height: 16px; }
.hb span { position: absolute; left: 0; width: 100%; height: 2.5px; border-radius: 9px; background: currentColor;
  transition: top .45s cubic-bezier(.6,.05,.2,1), transform .45s cubic-bezier(.6,.05,.2,1), opacity .35s ease; }
.hb span:nth-child(1) { top: 0; }
.hb span:nth-child(2) { top: 6.75px; background: var(--sig-accent); }
.hb span:nth-child(3) { top: 13.5px; }
#menu-btn.open .hb span:nth-child(1) { top: 6.75px; transform: rotate(45deg); }
#menu-btn.open .hb span:nth-child(2) { opacity: 0; transform: translateX(-32px) scaleX(.35); }
#menu-btn.open .hb span:nth-child(3) { top: 6.75px; transform: rotate(-45deg); }

#menu { clip-path: circle(0px at var(--cx, calc(100% - 42px)) var(--cy, 34px)); visibility: hidden;
  transition: clip-path .6s cubic-bezier(.76,0,.24,1), visibility 0s linear .6s; }
#menu.open { clip-path: circle(150% at var(--cx, calc(100% - 42px)) var(--cy, 34px)); visibility: visible;
  transition: clip-path .6s cubic-bezier(.76,0,.24,1), visibility 0s; }
#menu .m-item { opacity: 0; transform: translateX(-28px); transition: opacity .3s ease, transform .3s ease; }
#menu.open .m-item { opacity: 1; transform: none;
  transition: opacity .5s ease calc(.18s + var(--i) * .07s), transform .6s cubic-bezier(.16,1,.3,1) calc(.18s + var(--i) * .07s); }
@media (prefers-reduced-motion: reduce) {
  #menu, #menu.open { clip-path: none; transition: opacity .2s; }
  #menu:not(.open) { opacity: 0; }
  #menu .m-item, #menu.open .m-item, .hb span { transition-duration: .15s; transition-delay: 0s; }
}
```

```js
const btn = document.getElementById('menu-btn'), menu = document.getElementById('menu');
const set = (open) => {
  const r = btn.getBoundingClientRect();
  menu.style.setProperty('--cx', `${r.left + r.width / 2}px`);
  menu.style.setProperty('--cy', `${r.top + r.height / 2}px`);
  btn.classList.toggle('open', open); menu.classList.toggle('open', open);
  btn.setAttribute('aria-expanded', open); menu.setAttribute('aria-hidden', !open);
  document.body.style.overflow = open ? 'hidden' : '';
};
btn.addEventListener('click', () => set(!menu.classList.contains('open')));
menu.addEventListener('click', (e) => e.target.closest('a') && set(false));
addEventListener('keydown', (e) => e.key === 'Escape' && set(false));
```

---

## 4. Forbidden

These rules come from real complaints about real sites: "I can't click anything", "the animations take forever", "the same colour everywhere", "it's hard to use when the mouse is over the images".

- **Nothing tilts, rotates or moves when the pointer is over images or cards.** No tilt cards, no pointer-driven scene, no parallax on hover. Allowed on hover: a slight zoom of the image (at most `scale(1.04)`, at most 0.5 s).
- **Fast animations.** Entrance on page load at most 0.45 s with at most 12 px of travel. No scroll-linked animations (`animation-timeline: view()`), and no content waits to fade in while you scroll. The liquid fill (about 0.95 s) is the one exception.
- **Nothing repaints the page continuously.** Never animate `background-position`, `filter` or `box-shadow` in a loop. Loops may only animate `transform` and `opacity`.
- **No 3D.** No depth scenes, Three.js models, CSS 3D figures or WebGL canvases. After the build, check every button and link with `document.elementFromPoint` on desktop and mobile.
- **Not the same colour everywhere.** Every page has at least two full-width sections that change surface, for example one band in the text colour (inverted) and one block in the accent colour. If every section sits on the same background, the design is not finished.
- **No pill badges.** Never put a word in a small rounded bubble with a tinted background: plan names ("Pro"), "New", "Beta", status tags, counters dressed up as chips. It is the fastest way to make an interface look generated. A label is plain text: the same typeface as its surroundings, set apart by weight, size or the accent colour, with no background and no border. A real count (unread messages) may sit in a solid circle, nothing else.
- **No accent-tinted boxes.** Do not fill a box with the accent colour at low opacity (the brown or peach panel behind a code, a card number, a notice). Boxes sit on the neutral surface with a hairline border, and the accent is kept for the one thing that can be pressed.
- **No gradient avatars.** A placeholder avatar is a flat circle in the text colour with the initial in the background colour. No rounded squares, no gradients.
- **No default-blue controls.** Radios, checkboxes, range sliders and focus rings never stay in the browser's blue. Set `accent-color` and the focus outline from the site's own colours.

## 5. The figures

Flat, cheerful, hand-drawn people give a site a warm, human feel that stock photos don't.

- **Source:** unDraw (undraw.co, by Katerina Limpitsouni, free to use commercially).
- **Always recolour.** unDraw's default purple (`#6c63ff`) is replaced with the site's accent. Clothes and details that would vanish against a coloured surface get a light tone. A figure still in purple is not finished.
- **Scenes of your own.** Compose the figures into scenes only that site has. Cut them apart and combine them freely, but do not draw new figures.
- **Forbidden:** AI-generated figures, hand-rolled SVG people, and figures that fly or float. They stand on the ground.
- **Keep them through a redesign.** Never remove a site's figures when it is redone; new sections use the same style and preferably the same figures.

## Exceptions to general design-skill rules

Lavis is meant to sit on top of whatever design guidance you use. Where they disagree, these win:

- **The numbers 01, 02, 03 in the hamburger menu are allowed**, even if numbered labels are otherwise banned.
- **The liquid fill and the underlines count as motivated motion.** Both are feedback on hover and focus.
- **The menu is not replaced** by other menu patterns (gooey menus, mega menus and the like).
