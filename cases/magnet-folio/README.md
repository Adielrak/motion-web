# Case · magnet-folio

A freelance-portfolio home page rebuilt from a live reference's **source**, not from screenshots:
magnetic controls that ring when released, a giant name marquee that reverses with the scroll
direction, a square project preview that trails the cursor with a second, faster dot on top, and
curtains whose edges are curved.

`index.html` — single file, DOM + inline SVG, rAF, **no dependencies**. Ladder **rung 1.5**.

Reference: `dennissnellenberg.com` (home), read on 2026-11-16. The page and its own bundle were
fetched and read: `style-new.css`, `styleguide.css`, `index-new.js`, plus a `rea analyze-web-bundle`
pass over the live page through CDP (3 scripts, 0 routes, 0 endpoints, so it is a static brochure
and all of the behaviour is in the client JS). The reference runs GSAP 3.9.1 + ScrollTrigger,
Locomotive Scroll, Barba 2 and jQuery 3.5. This case runs on none of them.

**What is faithful:** every mechanism, timing, easing and geometry ratio below comes from the
reference source. **What is adapted:** the identity. The name, copy, palette, portrait, project names
and project screens are all invented here. The reference's photograph is replaced by a drawn portrait,
and its project videos are replaced by drawn device screens. Per `SKILL.md` § Replicate Mode, nothing
identifying the reference's owner is carried over.

---

## 1 · 设计系统

### Tokens

| Role | Value | Reference had |
|---|---|---|
| ink | `#18191c` | `#1C1D20` (`--color-dark`) |
| ink-deep | `#111214` | `#141517` (loader) |
| paper | `#ffffff` | `#FFFFFF` |
| mist | `#ecebe7` | `#E9EAEB` (lightgray) |
| sand | `#b7b0a2` | `#999D9E` grey behind the photograph |
| accent | `#d9461f` / deep `#b8391a` | `#455CE9` / `#334BD3` blue |
| lines | `rgba(24,25,28,.16)` / `rgba(255,255,255,.2)` | `.175` / `.2` |

**One** saturated colour, used only for things that answer the pointer: the fill blob, the
"Get in touch" disc, the cursor dot and the open-menu button. That is the reference's rule (the blue
appears nowhere static) and it is the part that transfers. The hue itself does not.

### Type

| Tier | Size | Reference |
|---|---|---|
| marquee name | `max(9em, 15vw)` → **216px** at 1440 | identical |
| footer headline | `clamp(3.25em,7vw,8em) × .875` | identical |
| work rows | `clamp(3.25em,7vw,8em) × .75` | identical |
| intro statement | `clamp(1.55em,2.3vw,2.25em)` | h4 tier |
| body | 16px, weight 450 | `Dennis Sans` 450 (Neue Montreal) |

Measured ratio **13.5×** (harness). The face is Neue Montreal in the reference. Here it is a
`"Neue Montreal", "Helvetica Neue", Inter, Arial` stack with no web font shipped. The page is legible
on that stack, but a person will notice the difference. Shipping a real grotesk is the first upgrade.

### Space

`--section-pad: clamp(5em,21vh,12em)` · `--container-pad: clamp(2.5em,8vw,8em)` ·
`--gap: clamp(1.5em,4vw,2.5em)`. All three are the reference's own values. Below 720px they switch to its
mobile set (`max(2.5em,12vh)`, `clamp(1.25em,4vw,2.5em)`).

### Dialect

**Creative.** No shadows except the drawn device screens, so 0 shadow levels on UI. Radius is
either 0 or 50% (discs and pills) with nothing in between. The nav CTA is a link with a dot, not a
filled button. The footer is full-bleed and dark with display-size type. Buttons are sentence case.

### Section shapes (4 `<section>` + header + footer)

| Shape | Ground | Why it differs |
|---|---|---|
| hero `<header>` | sand, full-bleed, 100vh | the only image-led screen; everything else is type |
| intro | paper, 2-col grid (statement / aside + disc) | the one place to read a sentence |
| work | paper, full-width rows, hairlines | a list that needs the cursor to show its pictures |
| more | paper, single centred pill | a breath, and a second magnet at a different strength |
| strips | **mist**, two offset rows of screens | the pictures the list hid, now shown at rest |
| footer | ink, curved top edge | the dark ground is the "end of page" signal |

### Components and the states each owes

- **Magnetic disc** (`.btn-round`): rest · tracking · ringing on leave · fill rising · focus.
- **Pill** (`.btn-normal`): rest · hover (fill + text to white) · tracking.
- **Nav link** (`.btn-link`): rest · hover dot · current-page dot (in the menu).
- **Hamburger**: hidden (`scale 0`) · popped (after 30% of a viewport) · open (accent, X).
- **Work row**: rest · hovered (title −10% of the gutter, meta +10%, siblings at .3).
- **Preview**: closed (`width 0`) · open · switching project (strip slide) · pressed.

---

## 2 · 动效系统

### Ladder

Rung **1.5**: DOM + CSS transitions for state changes, one rAF loop for everything continuous.
No scroll library. The reference smooths scroll with Locomotive. This page leaves native scroll alone
and reads `window.scrollY` every frame, which keeps keyboard, find-in-page, anchors and the real-wheel
smoke test all working. **What would force a climb:** a pinned horizontal section, which this page
does not have.

### Mechanics

| # | Mechanic | Trigger | Driver | Reference source → here |
|---|---|---|---|---|
| 1 | **Magnet** | `pointermove` on the control | `((p − left)/w − .5) × data-strength`, and the label gets its own smaller strength | `gsap.to(…, 1.5, power4.out)` on move and `elastic.out` on leave → a spring: move k=80, c=2√k (critically damped); leave k=70, c=4.5 (ζ≈.27, rings) |
| 2 | **Fill blob** | `pointerenter` / `leave` | ellipse 150%×200%, parked at `−76%` | `startAt:{y:"76%"} → 0%` .6s power2.inOut, out to `−76%`. The jump back to +76% before each enter is what makes it **always rise from below** |
| 3 | **Name marquee** | time + scroll direction | two copies on one track, one clock | `xPercent:-100` over **18 s** in a repeating timeline, with `timeScale` tweened to ±1 on a direction change → `ts += (dir − ts)(1 − e^(−6dt))` |
| 4 | **Cursor preview** | hover a row | three followers | `pos += (mouse − pos)/d` at **120 Hz** with d = 12 (image), 7 (dot), 6 (label). Here it is stepped by `1 − (1 − 1/d)^(dt·120)` so the lag does not depend on the frame rate |
| 5 | **Preview strip** | row index | `translateY(−i·100/count %)` | .6 s power2.inOut, identical |
| 6 | **Loader** | load | one timeline | greeting words every .15 s; curtain `top → −100%` .8 s power4.inOut; its curved tail 10vh → 0 over 1 s; content `y: 50vh → 0` 1.5 s expo.out, stagger .07 |
| 7 | **Curved curtain** | (6) and footer scroll | height of a clipping box | a 745%-tall ellipse inside a box whose **height** animates. The bulge is a side effect of clipping, so no path morph is needed |
| 8 | **Menu panel** | click | `translate(100% + 6vw) → 0` .8 s `(.7,0,.2,1)` | its left bulge is the same trick, sideways: `width 6vw → 0` over .85 s, slightly slower than the panel |
| 9 | **Word rise** | enters the viewport | each word `y 100% → 0` | 1 s power3.out, stagger **10 ms**. Reset when it leaves below (`toggleActions: play none none reset`) |
| 10 | **Scrubbed tracks** | scroll | position | header arrow 0→90° · footer curve 7.5vh→0 · footer arrow 15°→0 · hamburger pops at 30% of a viewport · strips ±12vw in opposite directions |

### Scroll schedule (1440×900)

```
scrollY →  0 ──── 900 ─────────────────────── strips ──────── footer ──── end
arrow      ████████                                                       (0→90°)
burger        ▲ pops at 270
marquee    ═════════ runs on time; direction set by the last scroll delta ════════
words                ▲ intro enters → rise, once
strips                                    ████████████████
curve                                                      ████████████  (7.5vh→0)
```

The overlaps are deliberate. The marquee runs on its own clock, so the scroll only ever *steers* it
and never *drives* it. That is why the hero never looks dead, even with the wheel at rest.

### Reduced motion

The loader is removed from the DOM, not hidden, so its words cannot be counted as stuck. The marquee
is parked at 0, the words are painted at rest and the globe stops. The magnets still track, because
that is a direct response to input rather than ambient motion, but every CSS transition is cut to .01 ms.

---

## 3 · 判据

```bash
python3 scripts/verify_case.py cases/magnet-folio/index.html --folio
```

| Check | Measured |
|---|---|
| console errors | none |
| h-overflow 320/375/414/768/1440 | 0 / 0 / 0 / 0 / 0 |
| type ratio | 13.5× (216 / 16) |
| distinct section shapes | 3 of 4 `<section>` |
| dead custom properties | none (25 referenced) |
| real wheel / real hover | both react |
| reduced-motion hidden | 0 / 19 |
| **`--folio` magnet track** | (25.0, 20.0) for want (25, 20); label (12.5, 10.0), so it lags the disc |
| **`--folio` magnet ring** | min x **−10.2 px** after release from +25, rests at (0.01, 0.01) |
| **`--folio` marquee** | ts 1.00 → **0.21** 90 ms after reversal → −1.00, position running backward |
| **`--folio` preview** | 60 ms after a jump: image gap 241 > dot gap 141, both settle to 0.00 |

`--folio` is three pairs of assertions that **pull against each other**, every one of them driven by
the real mouse and wheel:

- the magnet must **track** (within 15% of strength) *and* **ring** (overshoot past 0 by > 1px), *and*
  **settle** (< .5px). An over-damped leave spring passes tracking and fails ringing. A spring with no
  damping fails settling.
- the marquee must **reverse** *and* **not snap** (|ts| < .95, 90 ms after the reversal).
- the followers must **lag in order** (image > dot > 1px) *and* **converge** (< 2px).

Verified to fail on purpose: a critically damped leave → `magnet returned without ringing (min x 0.00)`.
`ts = dir` → `marquee SNAPPED on reversal (ts -1.00 90 ms later)`.

---

## 4 · 可以照搬 / 不许照搬

**可以照搬 (mechanism):** the magnet spring pair and its transform-cancelling target. The fill-blob
re-park. The two-copy marquee with an eased timeScale. The frame-rate-independent `1/d` follower. The
`−i/count` strip. The curved curtain built as an ellipse in a clipping box. Every easing token in
`:root`. The loader timeline numbers.

**不许照搬 (skin), and the reasoning to take instead:**

- *The palette.* Take the rule, not the hue: one saturated colour, spent only on things that react.
  On a neutral page the accent *becomes* the affordance.
- *The sand hero with a person in it.* What transfers is that a freelancer's first screen is the
  person: a face at the scale of the type. Use the person's own photograph. The drawn one here is a
  stand-in.
- *The project screens.* The reference uses project video. A list that shows its pictures only under
  the cursor needs pictures worth hovering for, and grey boxes would kill mechanic #4.
- *The section order* (hero → statement → list → more → strips → contact). It follows from a
  freelancer's argument: who I am, what I believe, proof, more proof, ask. Reorder it for a different
  argument.
- *The copy and name.* Invented. Never ship a clone's identity.

---

## 5 · 踩过的坑

1. **The oracle aimed outside the button.** The first magnet check moved the mouse to (.9, .8) of the
   bounding box and reported "did not track: (−0.1, 0.1) vs (40, 30)". On a circle, that corner is
   outside the disc, so `elementFromPoint` returned the wrapper. The page was right and the probe was
   wrong. Aim inside the shape: (.75, .7) is .32 of the radius from the centre.
2. **A closed off-canvas menu still catches the pointer.** The harness's first hittable `a[href]` was a
   link in the closed side panel. It was translated off-screen but still had a > 4 px rect, so the real
   hover landed on nothing and "hover changed something" read **False**. `visibility: hidden` with a
   delayed visibility transition fixed the oracle, *and* it removed four tab stops a keyboard user
   could not see.
3. **`display:none` does not save hidden children from the reduced-motion check.** The nine loader
   words kept `opacity: 0` inside a `display: none` parent and were counted as **9 stuck hidden**.
   Removing the loader node when it is done (or never needed) is correct either way.
4. **The magnet must subtract its own transform.** `getBoundingClientRect()` includes the translate
   the magnet has already applied, so the normalised offset feeds back into itself and the button
   creeps toward the cursor. The target is computed against `rect.left − m.x`.
5. **Section-shape key.** Four white sections with left-aligned headings read as **2 distinct
   shapes**. Giving the strips band the mist ground is the reference's own move (its `about-services`
   tints to `#E9EAEB`) and it lifted the count to 3.
6. **The source fetch needed the network, not cleverness.** The sandbox's proxy refused the host,
   and the first plan was to rebuild from memory. A personal environment that allows exactly
   `dennissnellenberg.com` turned guesses into quoted values, for example `.btn-fill` parked at **76%**
   and the follower divisors **12 / 7 / 6**. None of those would have been remembered correctly.

---

## 6 · 改造方向

- **Agency case index:** the same row list with the preview showing a looping clip per project, and
  `active-big` (×1.66 dot) on rows that link out.
- **Product changelog:** rows are releases and the preview is the UI that changed. The `−i/count` strip
  makes stepping between neighbouring releases read as one continuous reel.
- **Event site:** the marquee carries the date and city, its direction steered by scroll, and the
  hamburger becomes "Tickets".
- **Any CTA-heavy page:** take the magnet alone. The ringing leave is what makes a big round CTA feel
  touchable rather than animated.
