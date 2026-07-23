---
name: game-feel-ui
description: >-
  Design and build game frontends that read as living game/anime SCENES, not
  card-and-panel webpages. Trigger whenever building or restyling a game UI,
  HUD, menu, shop, inventory, world map, crafting/upgrade screen, or any
  web/Phaser interface that should feel like an immersive game — especially when
  the user says it "feels like a webpage / too flat / like cards / like a
  dashboard", or wants more "game feel / juice / polish / life", diegetic UI,
  ambient idle animation, squash-and-stretch feedback, particle effects, number
  pop-ups, or spatial (map/scene) navigation instead of a nav bar. Targets
  Phaser 3 + TypeScript, and adapts to DOM/CSS, React, or canvas. Use this even
  when the user only names the symptom ("make this idle-game shop screen less
  like a dashboard") without naming any of these techniques.
---

# Game Feel UI

Most "game" frontends fail the same way: they are a webpage wearing a fantasy
skin. Equal-weight cards with rounded corners, borders and drop-shadows; a top
nav bar; every stat listed at once; nothing moving. This skill turns that into a
**scene** — an inhabited place the player looks at and reaches into.

## The one test that matters

**The silhouette test.** Blur or black out *every glyph of text*, screenshot it,
and look. If it still reads as a game scene — even an anime frame — you're on
track. If it reads as a website, you're not there yet.

Run this test in your head before you start and against the result when you
finish. Almost every failure has the same fix: **more image, fewer cards; more
motion, less stillness.** Keep it as your north star through everything below.

## The reframe

Stop thinking in **components, panels, and pages.** Start thinking in **scenes,
props, actors, cameras, and moments.**

| Webpage instinct | Game-scene instinct |
| --- | --- |
| A card holds the content | A **prop** in the world *is* the content |
| A nav bar switches pages | You **travel** to a place and the camera moves |
| Show all the info | **Reveal** info when the player touches its object |
| Animate on interaction | The world is **always alive**, even when idle |
| Button changes color | The action **squashes, bursts, pops a number, sounds** |
| The image is a thumbnail | The **image is the whole stage** |

The engine (Phaser) already speaks this language: scenes, sprites, containers,
tweens, particle emitters, cameras, postFX. You are not fighting the framework —
you are using the parts of it that a card layout ignores.

## The six pillars

Each pillar below is a directive, the reason it works, what "done" looks like,
and the webpage reflex it replaces. Internalize the *why* — you'll hit a hundred
micro-decisions this skill can't enumerate, and the reasoning is what
generalizes.

### 1. The image is the hero

**Do:** Make one large image — scene art, a character, or the key prop — the
visual subject that owns the majority of the frame. Text and controls are thin
layers on top, not the main event.

**Why:** Games communicate through a depicted world. The eye should land on a
*place* or a *character*, not on a paragraph or a button label. The moment text
is the biggest thing on screen, you've built a document.

**Done when:** delete every glyph and the screen still clearly shows a
place/character/object with mood and depth.

**Instead of:** a grid of equal cards, each with a small thumbnail boxed inside a
border.

### 2. Layered disclosure

**Do:** Never show everything at once. The resting screen is the scene. Detail
appears only when the player acts on the object it belongs to. Click the pill
furnace → *then* its level shows; *then* its refine and upgrade materials appear.

**Why:** A flat wall of stats reads as a spreadsheet and kills curiosity. Games
meter information so that engaging is rewarded and the resting screen stays calm.

**Done when:** the untouched screen shows almost no numbers; detail surfaces
contextually, out of the thing it describes.

**Instead of:** every stat, cost, and material listed simultaneously in a side
panel.

### 3. Diegetic function — functions live on props

**Do:** Attach each function to the in-world object that naturally owns it, and
present its information on an in-world surface — a wooden tag hung on the
furnace, a carved stone, a floating rune — **not** a generic rounded rectangle.
Break the reflex of reaching for `container + border-radius + border +
drop-shadow` as your content vessel.

**Why:** The card is the single biggest tell that betrays "webpage". When the
furnace's stats are engraved on a tag swinging from the furnace itself, the
interface dissolves into the world. **This is the highest-leverage pillar** — get
it right and half the battle is won.

**Done when:** a stranger could guess what each interactive element does from its
depiction alone, and no floating bordered card is doing the explaining.

**Instead of:** a tabbed settings/inventory panel with card rows.

### 4. Spatial navigation

**Do:** Move between sections by moving through space. Click a door, a signboard,
a spot on the map, an NPC — and travel there via a camera pan/zoom or a scene
transition. The map / shopfront / street **is** the navigation.

**Why:** A nav bar or tab strip is a document metaphor. Traveling to places makes
the product a world you inhabit, and gives every "page" a memorable location
instead of a menu entry.

**Done when:** there is no persistent nav bar or tab strip; you reach everything
by clicking things in the scene.

**Instead of:** a top/bottom bar of text links — Home / Shop / Inventory /
Settings.

### 5. Ambient life

**Do:** Keep something moving at all times, even when the player does nothing —
flame flicker, lantern sway, drifting spirit-energy or dust particles, a
signboard creaking, the shopkeeper breathing and blinking, a tail twitch. And
**desync every loop** (randomized delay/duration per instance) so it never
looks mechanical.

**Why:** A static screen reads as a dead document. Idle motion is what separates
a *rendered, inhabited* scene from a mockup. It's cheap and it buys enormous
perceived life. Synchronized loops, on the other hand, instantly read as "CSS
animation" — desyncing is what makes it read as "world".

**Done when:** a 10-second screen recording of the *untouched* screen still looks
alive.

**Instead of:** a perfectly still layout that only moves on interaction.

### 6. Thick feedback (juice)

**Do:** Make every meaningful action feel physical by *stacking* responses:
squash-and-stretch on press, an overshoot/elastic settle, a particle burst, a
number floating up, a short sound, and a camera nudge for big moments. Several at
once, per action.

**Why:** "Juice" — a response disproportionately larger than the input — is what
makes a button feel alive and an action feel consequential. Thin feedback (a
color change) reads as a form control; thick, layered feedback reads as a game.

**Done when:** a single tap produces motion + particles + a rising number +
sound; the reaction feels bigger than the tap.

**Instead of:** a `:hover` color shift and an instant, silent state change.

## Build workflow — world first, then wire it

Resist opening a code file and laying out boxes. Design the world, then bring it
to life in a fixed order so nothing gets skipped.

1. **Name the world and the scene.** One sentence per screen: *"This is the
   ⟨place⟩, seen from ⟨angle⟩, at ⟨time/mood⟩."* Decide the single hero image.
2. **Inventory props and actors; assign functions.** List every object. Which
   prop owns "upgrade"? Which owns "go to shop"? Which exists purely for
   atmosphere? Every function must land on a prop (pillar 3).
3. **Block layers by depth.** Order back→front: sky/background, midground
   scenery, hero prop/character, foreground occluders, particles, diegetic UI,
   transient feedback. Assign an explicit depth to each.
4. **Wire interaction with disclosure.** Make props interactive; reveal their
   info *out of the prop itself* (scale a diegetic tag up from the object), never
   all at once (pillars 2–3).
5. **Lay in ambient loops.** Add desynced idle motion to flames, hangables,
   characters, and a particle layer (pillar 5).
6. **Layer the feedback.** Route every actionable prop through the juice helpers
   — punch, burst, float-text, sound, shake (pillar 6).
7. **Grade and test.** Add a vignette / lighting / color grade for a rendered
   look, then run the silhouette test. If it fails, the fix is more image + more
   motion, less card.

## Design first, then bring it to life

You'll often lay out the scene in a design tool (Figma, a static mockup) before
writing code — and you should. Composition is far cheaper to iterate on
statically than in an engine. Just be clear about what a still frame can and
can't carry, or you'll misjudge the result.

**A static design locks:**

- Pillar 1 — the hero image and how much of the frame it owns.
- Pillar 3 — diegetic surfaces: which prop carries which function, drawn as
  world objects rather than cards.
- The *composition* of pillars 2 & 4 — depth layers, framing, where panels
  emerge from, and the spatial map of how scenes connect.

**Only code can realize:**

- Pillars 5 and 6 — ambient life and thick feedback *are* motion; a still frame
  has none of it.
- The *transitions* inside pillars 2 & 4 — the reveal that scales out of a prop,
  the camera travel between scenes.

So don't judge "game feel" from the static frame — it will always look flatter
than the finished screen, because half the feel is added in motion. Judge the
still with the **silhouette test** (that measures composition, which a still
*does* carry), and carry the motion forward as **annotations**: on each design,
note what moves and how — "lantern sways", "click furnace → tag scales out of
it", "upgrade = the big-hit combo". The still plus those notes is the handoff to
the implementation phase.

**Keep one skill, not a separate "design" copy and "code" copy.** The six
pillars are identical across both phases, so a single source of truth is
correct — and splitting them hides the whole point: good game-feel design must
*anticipate* the motion it can't yet show. The phases differ only in which
reference you open.

## Which reference to open

- **Design / static-layout phase** (Figma, a mockup — locking composition before
  code) → `references/composition.md`: hero image, depth-layer order,
  diegetic-panel design, color grading, and the anti-card / silhouette checks.
  No code.
- **Implementation phase** — writing the actual Phaser 3 + TypeScript →
  `references/phaser-recipes.md`. Paste the `Juice` helper in first; it covers
  punch, float-text, particle burst, and shake in one file.
- **The project is not Phaser** (plain DOM/CSS, React, or PixiJS/canvas) →
  `references/adapting-stacks.md`. The six pillars are invariant; only the
  primitives change.

## Definition of done

Ship only when all of these are true:

- [ ] **Silhouette test passes** — text hidden, it still reads as a game scene.
- [ ] One **hero image** dominates the frame; no equal-weight card grid.
- [ ] **No rounded-rect + border + shadow card** is the primary content vessel;
      information rides on diegetic surfaces.
- [ ] **No persistent nav bar / tab strip;** navigation is spatial.
- [ ] The **resting screen is alive** — at least a few desynced ambient loops.
- [ ] Every action has **thick, layered feedback** (motion + particles + number +
      sound), not just a state change.
- [ ] Information is **disclosed on interaction,** not dumped flat.
