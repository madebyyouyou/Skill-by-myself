# Adapting to non-Phaser stacks

The six pillars and the composition craft are **invariant** — they're about how
a scene reads, not about an API. Only the primitives change. If the project is
plain DOM/CSS, React, or a canvas engine, map the moves like this.

## Primitive map

| Move (Phaser) | DOM / CSS | React | PixiJS / canvas |
| --- | --- | --- | --- |
| Tween (`this.tweens.add`) | Web Animations API / CSS `@keyframes` | `framer-motion`, `react-spring` | GSAP, `@tweenjs/tween.js` |
| Overshoot ease (`Back`, `Elastic`) | `cubic-bezier(.34,1.56,.64,1)`; spring for elastic | motion `type:"spring"` | GSAP `back.out(1.7)`, `elastic.out` |
| Particle emitter | DOM sprites or a `<canvas>` overlay | same, in an effects component | Pixi `ParticleContainer` / `@pixi/particle-emitter` |
| Camera pan/zoom | `transform: translate()/scale()` on a world `<div>` | animate the world transform | `app.stage.position` / `scale` |
| Scene → scene | route/view swap behind a fade overlay | router + `AnimatePresence` | swap stage children with a fade |
| `setDepth` | `z-index` + explicit stacking context | same | `container.zIndex` + `sortableChildren` |
| Parallax (`scrollFactor`) | translate layers by a factor of pointer/scroll | same, in a handler/hook | move layer containers per input |
| Interactive prop | `pointerdown/over/out` + `cursor: pointer` | element handlers | `sprite.eventMode='static'`, `sprite.on('pointerdown')` |
| Disclosure container | element scaled from `transform-origin` at the prop | animated component keyed to prop | container scaled from prop anchor |
| `postFX.addGlow` | `filter: drop-shadow()`, layered box-shadow, SVG filter | same via style | `GlowFilter` (`@pixi/filter-glow`) |
| Vignette / grade | overlay gradient div, `mix-blend-mode` | same | full-screen filter / overlay sprite |
| Lighting | radial-gradient overlays, `mix-blend-mode: screen` | same | `@pixi/lights` or additive sprites |
| `camera.shake` | keyframe jitter transform on the world container | motion keyframes | jitter `stage.position` |
| Sound | Web Audio API / Howler.js | Howler in an effect | Howler / Pixi sound |

## What carries over unchanged

- **The `Juice` combo.** Rebuild `punch / floatText / burst / shake` in the new
  primitive and call them the same way. The values (1.18/0.82 squash, elastic
  settle, upward-fade float, ~14-particle burst) transfer directly.
- **Desync.** Randomize every loop's delay and duration regardless of stack.
  In CSS, vary `animation-delay` / `animation-duration` per element (inline
  style or `nth-child`); a shared class with identical timing is the tell.
- **The layer table & depth discipline** (`composition.md`) — identical; just
  becomes `z-index` or `zIndex`.
- **Diegetic-over-card** and **spatial-over-navbar** — these are design choices,
  not features of any engine. They apply verbatim.

## DOM/CSS specifics worth knowing

- **The overshoot magic numbers.** `cubic-bezier(0.34, 1.56, 0.64, 1)` is the
  reliable "back-out" for squash settle. For true elastic bounce, use a spring
  (Web Animations API has no elastic ease built in) or a keyframe with an
  overshoot-and-return.
- **Squash-and-stretch** needs a `transform-origin` matching the press point and
  animating `scale(x, y)` non-uniformly. Set `will-change: transform` on
  frequently animated props.
- **Particles at volume** — beyond a few dozen, prefer a single `<canvas>`
  overlay over many DOM nodes; DOM particle counts tank layout/paint.
- **Ambient loops** are pure CSS: `@keyframes` + `animation: … infinite
  alternate ease-in-out`, one class per motion (sway, breathe, float), with
  per-element `animation-delay`/`-duration` jitter to desync.
- **Respect `prefers-reduced-motion`** — gate the ambient and heavy-feedback
  layers behind it so motion-sensitive users get a calmer version. (Phaser
  projects should honor it too via a settings check.)

Everything else — hero image, depth layers, diegetic surfaces, staged
disclosure, spatial travel, thick feedback, the silhouette test — is the same
job in a different toolbox.
