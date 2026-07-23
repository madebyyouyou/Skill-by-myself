# Phaser 3 + TypeScript recipes

Copy-paste patterns for each pillar, written for **Phaser 3.60+** (the current
particle/FX API). Notes flag anything that differs on older versions or needs
the WebGL renderer.

**Paste the `Juice` helper (§1) into your project first** — every screen reuses
it. Everything else references it.

## Contents

1. [The `Juice` helper — reuse everywhere](#1-the-juice-helper)
2. [Pillar 1 — image as the hero: staging & parallax](#2-image-as-the-hero)
3. [Pillar 3 — interactive diegetic props](#3-interactive-diegetic-props)
4. [Pillar 2 — layered disclosure: reveal from the prop](#4-layered-disclosure)
5. [Pillar 4 — spatial navigation: camera moves & scene travel](#5-spatial-navigation)
6. [Pillar 5 — ambient life: desynced idle loops](#6-ambient-life)
7. [Pillar 6 — thick feedback in depth](#7-thick-feedback)
8. [Rendered look: postFX, glow, vignette, lighting](#8-rendered-look)
9. [Sound without friction](#9-sound)
10. [Gotchas](#10-gotchas)

---

## 1. The `Juice` helper

One static utility class. It encapsulates the four juice moves you'll call on
almost every interaction. Tune the magic numbers to taste — the overshoot easings
(`Back`, `Elastic`) are what make it feel like a game rather than a form.

```ts
// Juice.ts
import Phaser from 'phaser';

export class Juice {
  /** Squash-and-stretch press with an elastic overshoot settle. */
  static punch(scene: Phaser.Scene, target: Phaser.GameObjects.Components.Transform, base = 1) {
    scene.tweens.chain({
      targets: target,
      tweens: [
        { scaleX: base * 1.18, scaleY: base * 0.82, duration: 90, ease: 'Quad.easeOut' },
        { scaleX: base, scaleY: base, duration: 460, ease: 'Elastic.easeOut', easeParams: [1.15, 0.4] },
      ],
    });
  }

  /** A number/label that floats up and fades — damage-number energy. */
  static floatText(scene: Phaser.Scene, x: number, y: number, text: string, color = '#ffe8a3') {
    const t = scene.add.text(x, y, text, {
      fontFamily: 'Georgia, "Songti SC", serif',
      fontSize: '30px',
      color,
      stroke: '#3a2410',
      strokeThickness: 5,
    }).setOrigin(0.5).setDepth(9999);
    scene.tweens.add({
      targets: t,
      y: y - 64,
      alpha: { from: 1, to: 0 },
      scale: { from: 0.8, to: 1.1 },
      duration: 900,
      ease: 'Cubic.easeOut',
      onComplete: () => t.destroy(),
    });
    return t;
  }

  /** A one-shot particle burst at a point. Requires a small 'spark' texture loaded. */
  static burst(scene: Phaser.Scene, x: number, y: number, tint = 0xffcc66, count = 14) {
    const e = scene.add.particles(x, y, 'spark', {
      speed: { min: 60, max: 220 },
      angle: { min: 0, max: 360 },
      scale: { start: 0.7, end: 0 },
      alpha: { start: 1, end: 0 },
      lifespan: { min: 350, max: 650 },
      blendMode: 'ADD',
      tint,
      emitting: false,
    }).setDepth(9998);
    e.explode(count);
    scene.time.delayedCall(700, () => e.destroy());
  }

  /** Short camera shake for weighty moments. Keep it subtle. */
  static shake(scene: Phaser.Scene, duration = 180, intensity = 0.006) {
    scene.cameras.main.shake(duration, intensity);
  }

  /** The full "something big happened" combo. */
  static bigHit(scene: Phaser.Scene, obj: Phaser.GameObjects.Sprite, label: string) {
    Juice.punch(scene, obj);
    Juice.burst(scene, obj.x, obj.y);
    Juice.floatText(scene, obj.x, obj.y - obj.displayHeight * 0.4, label);
    Juice.shake(scene);
    scene.sound.play('impact', { volume: 0.5 });
  }
}
```

If you have no `spark` art yet, generate a 1-pixel dot texture at boot so
`burst` works immediately:

```ts
// in a boot/preload scene create()
const g = this.make.graphics({ x: 0, y: 0 }, false);
g.fillStyle(0xffffff, 1).fillCircle(8, 8, 8);
g.generateTexture('spark', 16, 16);
g.destroy();
```

---

## 2. Image as the hero

Full-bleed background, hero prop layered on top, thin UI last. Depth constants
keep the stack legible.

```ts
const D = { SKY: 0, FAR: 10, MID: 20, HERO: 30, FORE: 40, FX: 50, UI: 60, POP: 9999 };

// Full-bleed scene art — cover the whole camera, don't letterbox.
const bg = this.add.image(0, 0, 'shop-interior').setOrigin(0).setDepth(D.SKY);
bg.setDisplaySize(this.scale.width, this.scale.height);

// The hero prop owns the center of attention.
const furnace = this.add.sprite(640, 470, 'pill-furnace').setDepth(D.HERO);
```

**Parallax on pointer move** — cheap depth that makes flat art feel 3D. Shift
each layer by a factor of the cursor's offset from center:

```ts
const layers: Array<[Phaser.GameObjects.Image, number]> = [
  [far, 0.02], [mid, 0.05], [furnace as any, 0.09], [fore, 0.16],
];
this.input.on('pointermove', (p: Phaser.Input.Pointer) => {
  const dx = p.x - this.scale.width / 2;
  const dy = p.y - this.scale.height / 2;
  layers.forEach(([obj, k]) => obj.setPosition(obj.getData('ox') + dx * k, obj.getData('oy') + dy * k));
});
// store each layer's origin once: layer.setData('ox', layer.x).setData('oy', layer.y)
```

Prefer real painted/AI scene art and character sprites — that's what makes the
silhouette test pass. When art isn't ready, use bold silhouettes + gradients as
placeholders but keep the same layered composition so swapping real art in later
is a one-line texture change.

---

## 3. Interactive diegetic props

A prop that glows on hover, punches on press, and does its job on release — with
a generous hit area and a hand cursor.

```ts
furnace.setInteractive({ useHandCursor: true });
// A forgiving hit area beats pixel-perfect for pointer targets:
furnace.setInteractive(new Phaser.Geom.Circle(furnace.width / 2, furnace.height / 2, furnace.width * 0.6),
  Phaser.Geom.Circle.Contains);

let hoverGlow: Phaser.FX.Glow | undefined;
furnace.on('pointerover', () => {
  hoverGlow = furnace.postFX.addGlow(0xffb347, 4);         // WebGL only — see §8
  this.tweens.add({ targets: furnace, scale: 1.04, duration: 160, ease: 'Sine.easeOut' });
});
furnace.on('pointerout', () => {
  if (hoverGlow) furnace.postFX.remove(hoverGlow);
  this.tweens.add({ targets: furnace, scale: 1, duration: 160, ease: 'Sine.easeOut' });
});
furnace.on('pointerdown', () => Juice.punch(this, furnace));
furnace.on('pointerup', () => this.revealFurnacePanel(furnace));
```

The rule that keeps it diegetic: interaction affordances (glow, scale, cursor)
belong on the *depicted object*, not on a button chrome drawn over it.

---

## 4. Layered disclosure

Reveal info **out of the prop** using a `Container` scaled up from the object's
position with a `Back` overshoot. Build the panel from in-world art (a hanging
wooden tag), not a rounded rectangle.

```ts
private panel?: Phaser.GameObjects.Container;

revealFurnacePanel(furnace: Phaser.GameObjects.Sprite) {
  this.panel?.destroy();

  // Anchor above the furnace so it reads as belonging to it.
  const c = this.add.container(furnace.x, furnace.y - furnace.displayHeight * 0.55).setDepth(500);

  const tag = this.add.image(0, 0, 'wooden-tag');            // diegetic surface, not a card
  const level = this.add.text(0, -18, 'Refinement · Lv.7', {
    fontFamily: 'Georgia, serif', fontSize: '22px', color: '#f3e0b0',
  }).setOrigin(0.5);
  c.add([tag, level]);

  // Stage the sub-items: materials fade/slide in slightly after the tag — disclosure in layers.
  const mat1 = this.add.image(-60, 40, 'herb-spirit-grass').setAlpha(0);
  const mat2 = this.add.image(60, 40, 'ore-star-iron').setAlpha(0);
  c.add([mat1, mat2]);

  c.setScale(0);
  this.tweens.add({ targets: c, scale: 1, duration: 340, ease: 'Back.easeOut' });
  this.tweens.add({ targets: [mat1, mat2], alpha: 1, y: '+=6', delay: 180, duration: 260, ease: 'Sine.easeOut' });

  this.panel = c;
}
```

To dismiss, tween scale back to 0 with `Back.easeIn` then `destroy()` in
`onComplete`. Close on a click outside, not a diegetic "X" if you can avoid it.

---

## 5. Spatial navigation

Two patterns. Both replace a nav bar.

**Travel by clicking a prop (scene → scene) with a camera fade:**

```ts
const door = this.add.sprite(1100, 520, 'shop-door').setInteractive({ useHandCursor: true });
door.on('pointerup', () => {
  Juice.punch(this, door);
  this.cameras.main.fadeOut(380, 0, 0, 0);
  this.cameras.main.once(Phaser.Cameras.Scene2D.Events.FADE_OUT_COMPLETE, () => {
    this.scene.start('StreetScene');           // the "page change" is stepping through a door
  });
});
// In the destination scene's create(): this.cameras.main.fadeIn(380, 0, 0, 0);
```

**Move within one big scene by panning/zooming the camera to a location:**

```ts
// The world is larger than the viewport; clicking a landmark flies the camera there.
this.cameras.main.setBounds(0, 0, 3000, 1600);
stall.on('pointerup', () => {
  this.cameras.main.pan(stall.x, stall.y, 700, 'Sine.easeInOut');
  this.cameras.main.zoomTo(1.4, 700, 'Sine.easeInOut');
});
```

A **world map** is the same idea: a map image with interactive location pins;
clicking a pin pans/zooms or transitions. The map *is* the menu — no text-link
bar anywhere.

---

## 6. Ambient life

The resting screen must move. The critical detail is **desync** — randomize
delay and duration per instance, or many objects will pulse in lockstep and
read as "CSS animation".

```ts
// Lantern / signboard sway — pivot at the top.
lantern.setOrigin(0.5, 0);
this.tweens.add({
  targets: lantern, angle: { from: -4, to: 4 },
  duration: 2400 + Phaser.Math.Between(0, 600),         // desync
  delay: Phaser.Math.Between(0, 500),
  yoyo: true, repeat: -1, ease: 'Sine.easeInOut',
});

// Character breathing — pivot at the feet.
shopkeeper.setOrigin(0.5, 1);
this.tweens.add({
  targets: shopkeeper, scaleY: 1.03,
  duration: 1800 + Phaser.Math.Between(0, 400),
  yoyo: true, repeat: -1, ease: 'Sine.easeInOut',
});

// Blinking — a separate eyes sprite squashed to nothing, on a random cadence.
const scheduleBlink = () => this.time.delayedCall(Phaser.Math.Between(2200, 5200), () => {
  this.tweens.add({ targets: eyes, scaleY: 0.08, duration: 60, yoyo: true, ease: 'Quad.easeOut', onComplete: scheduleBlink });
});
scheduleBlink();

// Drifting spirit-energy / dust — a slow continuous emitter (additive, low alpha).
this.add.particles(0, 0, 'spark', {
  x: { min: 0, max: this.scale.width },
  y: this.scale.height + 10,
  lifespan: 6000,
  speedY: { min: -18, max: -34 },
  speedX: { min: -8, max: 8 },
  scale: { start: 0.25, end: 0 },
  alpha: { start: 0.5, end: 0 },
  blendMode: 'ADD',
  frequency: 320,                                        // one every 320ms, gentle
}).setDepth(50);
```

**Flame flicker** = two or three overlapping flame sprites on `ADD` blend, each
with its own desynced scale+alpha jitter, plus an upward spark emitter and a
glow (§8). Layering different frequencies is what sells fire over a single
pulsing sprite.

---

## 7. Thick feedback

`Juice.bigHit()` (§1) is the stacked combo. Reach past it when an action
deserves its own character:

```ts
// Successful upgrade: punch + burst + rising "+1 Lv" + shake + a flash of the whole scene.
upgradeBtn.on('pointerup', () => {
  Juice.bigHit(this, furnace, '+1 Lv');
  this.cameras.main.flash(160, 255, 240, 200);          // brief white bloom
});

// Insufficient materials: a "no" wobble instead of a punch — feedback also communicates failure.
const denied = () => this.tweens.add({
  targets: buyBtn, x: '+=8', duration: 60, yoyo: true, repeat: 3, ease: 'Sine.easeInOut',
});
```

Principles: (1) **stack** at least three channels — motion, particle, number,
sound; (2) make the reaction **bigger than the input**; (3) give *distinct*
actions *distinct* feedback so the player reads outcome from feel alone.

---

## 8. Rendered look — postFX, glow, vignette, lighting

These lift flat art toward "rendered scene". **postFX/preFX require the WebGL
renderer** (`type: Phaser.AUTO` gives it when available; they no-op on Canvas).

```ts
// Glow on a hot/magical prop (also used for hover in §3).
furnace.postFX.addGlow(0xff9a3c, 6, 0, false, 0.1, 24);

// Camera-wide vignette + a touch of bloom for the whole scene's mood.
const cam = this.cameras.main;
cam.postFX.addVignette(0.5, 0.5, 0.75, 0.55);           // center x, y, radius, strength (0..1)
cam.postFX.addBloom(0xffffff, 1, 1, 1.1, 1.1);

// Real 2D lighting (lamps, furnace glow): switch sprites to the Light pipeline.
this.lights.enable().setAmbientColor(0x2a2233);
bg.setPipeline('Light2D');                               // needs a normal map for full effect; still tints
this.lights.addLight(furnace.x, furnace.y, 300, 0xff8a3d, 2);
```

Even without normal maps, an ambient-color + a couple of warm point lights near
flames/lanterns instantly reads more like a lit interior than flat sprites.

---

## 9. Sound

Load short clips in `preload()`; play them on interaction. Browsers block audio
until the first user gesture — Phaser unlocks its context on the first pointer
input automatically, so tie the first sound to a click and you're fine.

```ts
// preload()
this.load.audio('click', 'audio/click.ogg');
this.load.audio('impact', 'audio/impact.ogg');

// on interaction
this.sound.play('click', { volume: 0.4, rate: Phaser.Math.FloatBetween(0.95, 1.05) }); // slight pitch variance = less robotic

// global mute toggle (wire to a diegetic bell/gong prop, not a checkbox)
this.sound.mute = !this.sound.mute;
```

Randomizing `rate` slightly per play stops repeated sounds from feeling
machine-stamped — the audio equivalent of desyncing loops.

---

## 10. Gotchas

- **Particle API changed in 3.60.** Use `this.add.particles(x, y, texture,
  config)` which *returns the emitter*. The old
  `this.add.particles(tex).createEmitter(cfg)` is removed. `emitter.explode(n)`
  for bursts; `frequency`/`emitting` for continuous.
- **postFX / preFX need WebGL.** They silently do nothing under the Canvas
  renderer. Use `Phaser.AUTO` and don't rely on FX for legibility — treat it as
  polish on top.
- **`postFX.addGlow` returns the FX object** — keep the reference to `remove()`
  it on `pointerout`, or repeated hovers stack glows and wash out.
- **Desync or it looks fake.** Identical duration + zero delay across many
  tweens is the #1 tell that betrays "web animation". Always jitter both.
- **Depth, not source order.** Rely on explicit `setDepth()` (the `D` constants
  in §2) rather than add-order; disclosure panels and popups must sit above
  everything (`9999`).
- **`this.tweens.chain()` is 3.60+.** On older versions, sequence with nested
  `onComplete` callbacks or a `timeline` (deprecated later).
- **Clean up transient objects.** `floatText`, `burst`, and one-shot panels must
  `destroy()` on completion or they leak and slow the scene.
