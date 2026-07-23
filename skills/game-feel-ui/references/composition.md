# Composition & diegesis

The visual-craft decisions behind pillars 1–4. This is where a scene stops
looking like a skinned webpage and starts looking *rendered*. Mostly
engine-agnostic thinking; a few Phaser config notes where relevant.

## Choose the hero, then everything serves it

Before placing anything, decide the **one** thing the frame is about — a
character, a place, or the key prop. It gets the largest area, the sharpest
focus, the strongest light, and the compositional lines pointing at it.
Everything else is set dressing at a lower contrast. A frame with three
co-equal "heroes" reads as a card grid; a frame with one hero reads as a shot.

Compose it like a shot, not a layout:

- **Rule of thirds / centering with intent.** Put the hero on a third or dead
  center for iconic props (a furnace, a throne). Don't distribute elements
  evenly to "fill space" — even distribution is the card-grid instinct.
- **Leave breathing room.** Negative space (sky, floor, haze) around the hero
  makes it read as a place. A screen packed edge-to-edge with widgets reads as a
  dashboard.
- **Foreground occluders.** A blurred lantern, a beam, or plants in the extreme
  foreground (low opacity, slightly out of focus) create instant depth and frame
  the hero. This single trick does more for "game scene" than any border ever
  will.

## Depth is the whole game

A webpage is flat: content on a background. A scene has *layers* with parallax
and light between them. Always build back-to-front and give each a job:

| Layer | Job | Notes |
| --- | --- | --- |
| Sky / far background | Establish place & time of day | Softest focus, lowest contrast, slow/no parallax |
| Midground scenery | Context around the hero | Mild parallax |
| **Hero prop / character** | The subject | Sharpest, best-lit, most parallax of the scene layers |
| Foreground occluders | Frame & depth | Blurred, darkened, strongest parallax |
| Particles | Air & life | Additive; dust/spirit motes/embers |
| Diegetic UI | Info on in-world surfaces | Rides *with* its prop |
| Transient feedback | Juice | Always on top (`depth 9999`) |

If a screen has exactly two layers (a background and a row of cards), that's the
diagnosis for "looks like a webpage." Add midground, foreground, and particles.

## Diegetic UI — kill the card

The card (`rounded rectangle + border + drop-shadow`) is the tell. Replace each
one by asking: **"What object in this world would carry this information?"**

| Webpage element | Diegetic replacement |
| --- | --- |
| Stat card | Engraving on the prop; a wooden tag hung from it; a carved stone |
| Modal dialog | The camera pushes in on the object; info unfurls from it |
| Tooltip | A whispered speech bubble; a floating rune; a lantern that lights up |
| Progress bar | A filling vial, a rising water level, a furnace heating up |
| Currency counter | Coins stacked on the counter; a ledger on the desk |
| Button | A physical lever, a bell to strike, a stamp to press, a rope to pull |
| Tab bar | Doors, signboards, paths, map pins (see spatial navigation) |
| Notification badge | A glowing item, smoke rising, an NPC waving you over |

When a real diegetic surface genuinely isn't feasible for dense text, make the
panel **look like an object from the world** — aged parchment, a bamboo scroll,
a carved plaque with the world's material and lighting — never a flat
system-chrome rectangle. Match its edges, texture, and light to the scene so it
belongs.

## Progressive disclosure — stage the reveal

Flat information is a spreadsheet; staged information is a scene responding to
you. Structure every detail view as a small sequence, not a single dump:

1. **Rest:** the prop sits in the scene, maybe with a subtle "interactable"
   shimmer. No numbers visible.
2. **Acknowledge:** on click, the prop reacts first (punch, glow) — the world
   noticed you.
3. **Reveal, in layers:** its primary fact appears (level/name), *then* a beat
   later its details (materials, costs) fade/slide in. The delay is the point —
   it reads as unfolding, not loading.
4. **Dismiss:** click-away collapses it back into the prop.

Anchoring the reveal to the prop's position (scaling up *from* it) is what makes
it feel like it came *out of* the object rather than popping over the page.

## Spatial navigation — the map is the menu

Design sections as **places**, not routes:

- Give each section a physical location and a way in that's visible in the
  scene — a door, a path, a signboard, an NPC who beckons.
- Transition by **moving through space**: step through the door (scene fade),
  or fly the camera to the location (pan + zoom). The motion teaches the spatial
  relationship between sections.
- A **world map** with interactive landmarks replaces a nav menu entirely.
  Returning "home" is walking back, not clicking a logo.
- The only acceptable persistent chrome is diegetic and minimal — a small
  hovering companion, a ring of runes — never a text-link bar.

## Color grading & light — the "rendered" pass

Even great sprite art looks like assets-on-a-page until it's graded as one
image. Apply a unifying pass over the whole scene (see `phaser-recipes.md §8`
for the Phaser calls):

- **Vignette** — darken the edges so the eye falls to the hero.
- **A unifying tint / bloom** — a warm dusk, a cold moonlight; push everything
  slightly toward one light so the layers feel lit by the same sun, not pasted
  together.
- **Point lights** near flames, lanterns, and glowing props — a warm pool of
  light around the furnace sells heat better than any orange gradient.
- **Atmospheric particles** — haze, embers, dust, spirit motes — tie the layers
  together through shared air.

## Two self-checks before you ship

- **Silhouette test (the north star):** hide all text; does it read as a game
  scene? If not: more image, fewer cards, more depth.
- **Card count:** count the rounded-rect + border + shadow rectangles on screen.
  The target is **zero** as primary content vessels. Each one you find is a
  pillar-3 violation to redesign onto a diegetic surface.
