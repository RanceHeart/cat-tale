# 猫猫物语 (Cat Tale) — Build Spec V1

## Mission

Build a Super Mario-style 2D side-scrolling platformer where the player controls a cute cat named 小咪. Use **KAPLAY** (kaplayjs, the successor to Kaboom.js) as the game engine, **Rough.js** for a hand-drawn sketchy visual style, and **anime.js v4** for smooth UI transitions. Single HTML file, no build step.

GitHub Pages: https://ranceheart.github.io/cat-tale/

## Core Mechanic

- Side-scrolling platformer, player moves left/right with arrow keys and jumps with Space
- The cat (小咪) runs, jumps on enemies, collects fish (coins), and reaches a flagpole to clear the level
- 3 levels of increasing difficulty
- Lives system (3 lives, lose one on enemy contact or falling into pits)
- Score tracking (fish collected)
- Smooth, responsive cat movement with **coyote time** and **variable-height jump**

## Visual Style — References (抄这些)

1. **Super Mario Bros (Nintendo)** — level design progression, flagpole goal, pipe obstacles, question blocks
2. **Celeste** — coyote time, jump buffering, acceleration-based movement (not instant max speed)
3. **Rough.js demos** — the entire game renders with Rough.js for a hand-drawn sketchbook aesthetic. Everything looks drawn with a slightly wobbly pencil. This is the signature visual hook.
4. **JSLegendDev's Kaboom Mario Game** (https://github.com/JSLegendDev/Mario-Game-Kaboom.js) — reference for how to structure a KAPLAY platformer (scenes, levels, player physics)

## Library Feature Checklist — USE ALL

### KAPLAY (kaplayjs) via CDN
```
https://unpkg.com/kaplay@3001/dist/kaplay.mjs
```
- `kaplay()` — init with crisp:true, background color
- `loadSprite()` — load sprites from URL or programmatically generate via canvas
- `add([sprite(), pos(), area(), body(), z(), ...])` — game object composition
- `body({ jumpForce, gravityScale })` — physics component
- `onKeyPress()` / `onKeyDown()` / `onKeyRelease()` — input handling
- `scene()` / `go()` — scene management (menu, game, gameover, win)
- `addLevel()` — tile-based level builder from string map
- `collides()` / `overlaps()` — collision detection
- `camPos()` / `camScale()` — camera follow
- `dt()` — delta time for frame-independent movement
- `tween()` — built-in tweening for smooth transitions
- `wait()` / `loop()` — timed events
- `burp()` / `play()` — sound effects
- `rand()` — random utilities
- `vec2()` — 2D vector math
- `width()` / `height()` — screen dimensions

### Rough.js via CDN
```
https://unpkg.com/roughjs@4/bundled/rough.js
```
- `rough.canvas(canvas)` — create rough canvas from <canvas> element
- `rc.rectangle(x, y, w, h, { roughness, fill, stroke })` — sketchy rectangles (platforms, blocks)
- `rc.circle(x, y, d, { roughness, fill })` — sketchy circles (collectibles, decorations)
- `rc.ellipse(x, y, w, h, { roughness, fill })` — sketchy ellipses
- `rc.line(x1, y1, x2, y2, { roughness, stroke })` — sketchy lines (background elements)
- `rc.path(d, { roughness, fill, stroke })` — sketchy SVG paths (cat character!)

IMPORTANT: Rough.js renders to a hidden canvas, then KAPLAY loads the result as a sprite via `loadSprite()` from a canvas/URL. Use this pipeline:
1. Create an offscreen canvas
2. Draw the cat and all game elements using Rough.js
3. Convert canvas to blob/URL
4. Load into KAPLAY via `loadSprite("cat", canvas.toDataURL())`

### anime.js v4 via CDN
```
https://cdn.jsdelivr.net/npm/animejs@4/lib/anime.esm.js
```
- `anime.timeline()` — title screen animations (cat slides in, title text bounces)
- `anime({ targets, keyframes, easing })` — menu transitions, level complete celebrations
- `anime({ ... })` with spring easing — UI element bounce effects

### Audio — CC Music (NOT procedurally generated)
- "Hep Cats" by Kevin MacLeod (incompetech.com) — perfect cat vibe
  Download URL: https://incompetech.com/music/royalty-free/mp3-royaltyfree/Hep%20Cats.mp3
  Licensed under CC BY 4.0 (attribution: "Hep Cats" Kevin MacLeod (incompetech.com))
- Use a second song for variety: "Electro Cabello" by Kevin MacLeod
  https://incompetech.com/music/royalty-free/mp3-royaltyfree/Electro%20Cabello.mp3

### SFX — Web Audio API
Do NOT use audio files for SFX. Generate programmatically:
- JUMP: 440Hz→660Hz sine sweep, 100ms (playful "boing")
- COLLECT: 880Hz→1320Hz sine, 80ms (bright "ding")
- STOMP (kill enemy): 220Hz square, 50ms (percussive "thump")
- HURT: 150Hz sawtooth→80Hz, 200ms (descending "bworp")
- LAND: short 200Hz sine burst, 30ms (soft "tap")
- MEOW: 600Hz→400Hz→800Hz sine, 300ms (cat meow!)

Keep SFX volume around 20% of music volume.

## Motion Principles (CRITICAL — without this the cat moves like a robot)

The cat's movement must feel **alive** and **weighty**. Codex's default output is stiff. Override that with:

1. **Anticipation**: Before jumping, the cat squats down slightly (0.08s of crouch before launch). Before stopping from a run, the cat leans forward briefly.
2. **Squash & Stretch**: On landing, the cat squashes vertically (scale.y ≈ 0.8, scale.x ≈ 1.15) for 0.1s, then springs back. On jumping, a slight stretch upward.
3. **Overlapping Action**: The cat's tail follows behind with delay — when the cat stops, the tail keeps swinging for 0.3s. Ears wiggle independently on idle.
4. **Follow-Through**: After stopping a run, the cat takes 2 extra "settling" frames (lean forward → stand upright).
5. **Variable-height jump**: The longer you hold Space, the higher the cat jumps (release early = short hop). Use a timer-based jump force: full force on first frame, reduced per held frame.
6. **Coyote Time**: The cat can still jump for 0.08s after walking off a ledge.
7. **Run Accel/Decel**: Not instant max speed. Accelerate from 0 to max over ~0.3s, decelerate over ~0.15s. This makes the cat feel like it has mass.

## File Structure

```
cat-tale/
  index.html    ← single file, everything inline
  CODEX_PROMPT.md    ← this file
  README.md     ← repo description
```

## Technical Requirements

- **Single HTML file** — CSS, HTML, JS all inline
- **ES Modules** — use `<script type="module">`
- **CDN imports** — KAPLAY, Rough.js, anime.js all via CDN (no npm)
- **Responsive** — canvas fills the viewport, scales proportionally
- **Mobile-friendly** — touch controls (on-screen buttons) + keyboard
- **Performance target** — 60fps on modern browsers
- **GitHub Pages ready** — works as static site, no build step
- **AudioContext** — created on first user interaction (browser autoplay policy)

## Visual Design Spec

### Color Palette (hand-drawn sketchbook look)
```css
--sky-bg: #87CEEB;          /* Light sky blue */
--grass: #7EC850;           /* Fresh green */
--earth: #8B5E3C;           /* Brown earth */
--cat-fur: #FF8C42;         /* Orange tabby fur */
--cat-stripe: #E0672A;      /* Darker orange stripes */
--cat-belly: #FFF3E0;       /* Cream belly */
--fish: #FFD700;            /* Golden fish */
--enemy: #E74C3C;           /* Red enemies */
--block: #C8A87C;           /* Warm beige blocks */
--ui-text: #2C3E50;         /* Dark text */
--ui-accent: #FF6B6B;       /* UI highlight */
```

### Rough.js Styling
- `roughness: 1.5` for platforms and terrain (moderately sketchy)
- `roughness: 0.8` for the cat character (cleaner but still hand-drawn)
- `roughness: 2.5` for background elements (very sketchy, adds depth)
- `fillStyle: "solid"` for the cat, "hachure" for background
- `fillWeight: 2` for medium fill lines
- `strokeWidth: 2` for outlines

### Cat Character Design
Draw the cat using Rough.js on an offscreen canvas (48×48 pixels):
- Round head with triangle ears (orange fill, darker outline)
- Big round eyes with pupils (expressive)
- Small pink nose, whiskers (3 per side)
- Oval body, slightly larger than head
- Long curved tail (swishes behind)
- 4 short legs with tiny paws
- Stripe pattern on back (2-3 darker orange horizontal lines)
- In walking animation: legs alternate, tail sways with 1-frame delay
- In jumping: legs tucked under, tail pointed down
- In falling: legs spread, tail up

### Level Design
- **Level 1** "花园" (Garden): Green grass, blue sky, simple platforms, 1 enemy type (crows), gentle learning curve
- **Level 2** "屋顶" (Rooftops): Urban theme, more gaps, moving platforms, 2 enemy types (crows + dogs), pipes to slide through
- **Level 3** "皇宫" (Palace): Indoor theme, tight platforms, spike pits, 3 enemy types, boss encounter (big dog/crocodile)

Use `addLevel()` with a tile map like:
```
"@   ^   @",
"=== ===",
"   %%%   ",
"=========",
```
Where: `@` = player start, `=` = ground, `^` = enemy, `%` = collectible, ` ` = air

### Scenes (KAPLAY scenes)
1. **Title Screen** — "猫猫物语" title text (anime.js animated entrance), cat silhouette, "Press SPACE to start" with pulsing animation, music starts here
2. **Game Scene** — main gameplay with HUD (score top-right, lives top-left, level name top-center)
3. **Level Complete** — brief celebration animation, fish count, "Next Level" prompt
4. **Game Over** — sad cat animation, score display, "Try Again" prompt
5. **Victory** — all 3 levels cleared, final score, congratulations

## Hit Feedback Layers
When the cat collects a fish:
1. Fish disappears with a pop animation (scale 1→1.5→0 over 0.2s)
2. "+1" text floats upward and fades (duration 0.8s)
3. SFX "ding" plays
4. Score counter updates with a bounce animation

When the cat stomps an enemy:
1. Enemy compresses vertically (squash, 0.1s)
2. Enemy disappears
3. Cat bounces upward slightly (reward bounce)
4. SFX "thump" plays

When the cat gets hurt:
1. Cat flashes (opacity toggle 3 times over 0.5s)
2. Cat is knocked backward
3. 1-second invincibility (no collision during flash)
4. SFX "bworp" plays
5. Life counter decrements with shake animation

## Implementation Order

### Phase 1: Foundation (make it work)
1. Set up index.html with KAPLAY + Rough.js + anime.js CDN imports
2. Create KAPLAY canvas, initialize with background color
3. Set up scene system (title, game, gameover, win)
4. Draw the cat character programmatically using Rough.js → canvas → KAPLAY sprite
5. Implement basic player movement (left/right/jump with variable height + coyote time)
6. Create Level 1 with `addLevel()` — simple flat ground, a few platforms, one enemy, fish, flagpole

### Phase 2: Gameplay (make it fun)
7. Enemy AI — walking back and forth, killed when stomped
8. Collectible fish — placed throughout levels
9. Lives system + score tracking
10. Level transitions (complete level 1 → level 2 → level 3)
11. Camera follows the cat with smooth lerp

### Phase 3: Visual Impact (make it beautiful)
12. Rough.js rendering for ALL game elements (platforms get sketchy edges, enemies look hand-drawn)
13. Cat animation frames (idle breathing, walking cycle, jump pose, landing squash)
14. Particle effects — leaf particles in the background, dust on landing
15. Parallax background (close trees, far mountains, sky gradient)
16. Title screen with anime.js (cat enter from left, title bounces in, pulsing start prompt)
17. HUD styling — hand-drawn font style for score/lives

### Phase 4: Level Design (make it complete)
18. Level 2 — moving platforms, pipes, new enemy type
19. Level 3 — spike pits, tight jumps, boss encounter
20. Victory screen with final score and celebration animation

### Phase 5: Polish (make it ship)
21. Mobile touch controls (left/right/jump buttons overlay)
22. Sound effects (Web Audio API for all SFX)
23. Background music (Hep Cats loop)
24. Browser title and favicon
25. README update with game link

## What NOT to do
- Do NOT generate music procedurally — use the Kevin MacLeod tracks listed above
- Do NOT use Unity, Phaser, or Godot — KAPLAY only
- Do NOT over-engineer — simpler is better. A tight platformer with 3 good levels beats a buggy open world
- Do NOT add story/dialogue — show, don't tell. Level design teaches the player
- Do NOT add RPG elements, inventory, or upgrades — this is a pure platformer
