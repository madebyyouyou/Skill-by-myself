---
name: ios-pwa-fullscreen
description: Use when an iOS home-screen web app (PWA) will not render edge-to-edge — fullscreen and safe-area work, status-bar overlap, viewport-height traps (100dvh / 100vh / height:100%), a blank white screen at launch, letterboxing or a band of background colour at the top or bottom, a colour seam under the status bar, or when this has to be diagnosed on a real device where Web Inspector is unavailable.
---

# iOS PWA Fullscreen

Field notes from a real device (iPhone 16 Pro Max, iOS 18.7, WebKit 605.1.15). Every number below was
measured on-device, not inferred. Desktop Chrome reproduces none of these problems.

## Know this before touching anything

**An iOS home-screen web app does not get the status bar's 62pt. Do not try to take it.**

Measured inside a running standalone PWA:

```
innerHeight     894        <- the visible viewport
100vh           956        <- what CSS thinks the screen is
100dvh          894
100svh          894
100lvh          956
clientHeight    894
visualViewport  894
screen          440 x 956  (avail 956)
safe-area       top=0  right=0  bottom=34  left=0
```

`inset-bottom = 34` is non-zero, which **proves `viewport-fit=cover` is working**. So `inset-top = 0`
together with `innerHeight = 894` does not mean insets are broken — it means the page box genuinely
starts *below* the status bar. That 62pt is not the page's to use.

No combination of `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style: default`,
or `black-translucent` changes this. Tuning those meta tags is wasted effort.

## The fix: shift up, don't fight for it

Designs for fullscreen mobile usually reserve a solid-colour band at the top for the status bar to sit
over (in the reference case: 120 design px = 62pt). Given that band:

> Size the container to the **full** screen height, then move it up by `full − visible`,
> and let `overflow: hidden` clip the overflow.

What gets clipped is exactly the band the status bar was going to cover. **Covered and clipped look
identical.** The aspect ratios then line up by themselves — not a coincidence, just the same 62pt
expressed two ways:

```
851 / (1849 − 120) = 0.4922
440 / 894          = 0.4922      -> zero letterboxing on all four sides
```

```ts
function measureUnit(unit: string): number {
  const probe = document.createElement('div');
  probe.style.cssText =
    `position:fixed;top:0;left:0;width:0;height:${unit};visibility:hidden;pointer-events:none;`;
  document.body.appendChild(probe);
  const h = probe.getBoundingClientRect().height;
  probe.remove();
  return h;
}

function syncRootSize(): void {
  const el = document.getElementById('root');
  if (!el) return;

  const standalone =
    window.matchMedia('(display-mode: standalone)').matches ||
    (navigator as Navigator & { standalone?: boolean }).standalone === true;

  const visibleW = window.innerWidth;
  const visibleH = window.innerHeight;
  if (visibleW <= 0 || visibleH <= 0) return;

  // Safari has toolbars and 100vh reports the *large* viewport there;
  // applying the shift outside standalone throws the picture off-screen.
  const fullH = standalone ? Math.max(visibleH, measureUnit('100vh')) : visibleH;
  const shortfall = fullH - visibleH; // ~62; becomes 0 if the OS ever hands over the full screen

  el.style.width = `${visibleW}px`;
  el.style.height = `${fullH}px`;
  el.style.top = `${-shortfall}px`;
}

syncRootSize(); // must run BEFORE the renderer is constructed, so it measures a correct parent on frame 1
window.addEventListener('resize', syncRootSize);
window.addEventListener('orientationchange', () => setTimeout(syncRootSize, 120));
```

```css
html, body {
  margin: 0; padding: 0; overflow: hidden; overscroll-behavior: none;
  background: <the solid band's colour>;
}
html { height: 100%; }
body { position: fixed; inset: 0; height: 100vh; } /* first-frame fallback only */
#root { position: absolute; top: 0; left: 0; }     /* size and top come from JS */
```

## Meta tags and manifest

```html
<meta name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
<link rel="manifest" href="./manifest.webmanifest" />
<meta name="theme-color" content="<the solid band's colour>" />
<meta name="apple-mobile-web-app-title" content="…" />
```

```json
{ "display": "standalone", "orientation": "portrait",
  "start_url": ".", "scope": ".",
  "background_color": "…", "theme_color": "…" }
```

- Standalone mode comes from the manifest's `display: standalone`. Nothing else is needed.
- **Do not add** `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style`, or
  `mobile-web-app-capable`. On current iOS they shorten `innerHeight`; they are a net negative.
- **These tags and the manifest are snapshotted when the user taps "Add to Home Screen."** After
  changing them the icon must be **deleted and re-added** — force-quitting the app is not enough.
  Verification: the diagnostic panel must report `navigator.standalone = true` **and**
  `matchMedia('(display-mode: standalone)')` = true. If the second is `browser`, the old config is
  still in effect and any test result is meaningless.

## Three height-propagation traps

All three are invisible on desktop Chrome.

| Written as | What happens |
|---|---|
| `height: 100dvh` | Resolves to **0** at some moments (notably PWA cold start). The container collapses, the canvas is sized 0×0 — **white screen**. |
| Parent stretched by `inset: 0` + child `height: 100%` | The stretched height's *computed value* is still `auto`, and percentage heights require a containing block with a **definite** height. Chrome is lenient; **WebKit is not** → child collapses to `width × 0` — **white screen**. |
| `height: 100%` in standalone | Resolves to the short value (894) **and overrides the `inset: 0` stretch** → top-aligned, ~62pt of background exposed at the bottom. |

**Do not propagate height through CSS. Write pixel values from JS** (see `syncRootSize` above).

A related trap: **double centring**. If the container centres with flex *and* the renderer centres
itself (e.g. Phaser's `autoCenter: CENTER_BOTH` sets the canvas margin directly), both apply. It is
invisible while the fit is exact and skews the picture the moment any letterboxing appears — measured
case: a 400px-wide canvas in a 430px container ended up with 29px on the left and 1px on the right.

## On-device diagnostics

Web Inspector is not reachable from a home-screen app, so the page must be able to report on itself.
Inline a **non-module** script in `index.html` so it runs before any build output:

- **White-screen watchdog** — after ~10s, if the canvas is still missing or 0-sized, render the
  diagnostics as a full-screen panel. It **must self-heal**: keep re-checking every second and remove
  itself once the canvas is ready. Otherwise a slow load is misreported as a crash, and the panel
  permanently covers the app.
- **Manual trigger** — two-finger long press (~1.5s). Two fingers avoids clashing with single-finger
  taps and drags. Needed because once the app runs, the watchdog never fires, yet layout questions
  still need live numbers.

The panel should print at minimum:

```
innerHeight / 100vh / 100dvh / 100svh / 100lvh / clientHeight / visualViewport / screen
safe-area insets (all four)
body, container and canvas measured size + position
canvas margin, and letterboxing on each side
navigator.standalone + display-mode
resources completed and bytes so far, plus the most recent files
captured errors (window.onerror + unhandledrejection, capture phase)
```

Pick the predicates carefully. Two that produced false readings:

- Do **not** infer "stylesheet applied" from `body.marginTop === '0px'` — a fallback inline style in
  `<head>` makes it always true. Use the **measured height of body/container**: a height of 0 means the
  stylesheet has not arrived, i.e. the entry module has not finished loading.
- If the container's width is exactly `viewport − 16`, that 16 is the browser's default
  `body { margin: 8px }` — again meaning no stylesheet yet.

## Status bar colour

The status bar strip is painted by the **system**, using `theme-color`. It must equal the colour of the
band at the top of the artwork, or there is a visible seam at the join.

**Sample it from the image; do not eyeball it.** In the reference case an eyeballed `#eee7d7` was off;
the measured value was `#e5dbd2`.

```js
// average a row of pixels, and check the spread to confirm it really is flat colour
const raw = await sharp(file).ensureAlpha().raw().toBuffer();
// sample the target row across x, average RGB, and require a mean deviation <= 2
```

Use the same colour for the `body` background so any letterboxing outside standalone matches.
**Never use a dark colour there** — it frames the artwork in a black border.

## Also worth knowing

- Elements the design places hard against the safe-area edge need to be moved down. In the reference
  case the mockup put the back button and resource bar at `top = 107`, i.e. 13px inside the 120px band;
  after the upward shift those 13px are clipped. They were moved to `top = 136` (band + 16px of
  breathing room), preserving their original alignment with each other.
- The 34pt home indicator at the bottom is **not** reserved — it floats over the content. Keep anything
  important away from the bottom edge.

## Method notes

Three habits that would have found all of this much faster:

1. **Check existing write-ups before tuning parameters.** This exact failure (`894 < 956`, ~62px band
   at the bottom) had already been documented in another project's `index.html` header comment, in
   wording that matched the observed symptom line for line. Two rounds of meta-tag guesswork happened
   anyway.
2. **If one number can falsify a hypothesis, go get that number first.** A stalled load was blamed on
   asset weight (61MB) when the observed rate — 9KB over 265s, about 34 B/s — already ruled out
   bandwidth. The real cause was a deadlocked request chain.
3. **Desktop Chrome cannot reproduce this class of bug** (`innerHeight === 100vh` there, and
   percentage-height propagation is lenient). For anything touching the iOS viewport, trust only
   numbers from the on-device panel.
