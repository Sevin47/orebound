# OREBOUND

**Play it: https://sevin47.github.io/orebound/**

Ten million pixels. One of them is golden.

Shoot balls into solid rock. They bounce forever, every pixel they break is one
pixel of currency, and that currency buys more balls. Buried in the rock are
upgrade pixels that grow your skill tree. Golden-ball pixels give you the one
ball that can break the golden pixel.

Single file, no build step, no dependencies. Open `index.html` in any browser.

## Where the rules come from

Every mechanic and number here was read out of the shipped **10 Million Pixels
demo** (Steam, Unity 6000.0.23f1). The sources were:

- `Assembly-CSharp.dll`, decompiled
- the scene and prefab data in `level0` and `sharedassets0`
- the demo's own 360×360 world mask
- a real generated demo world from a save file

Earlier builds of Orebound were tuned by measuring trailer footage frame by
frame. Several of those guesses turned out wrong once the code could be read:

| | Earlier Orebound (from footage) | The demo (from its code) |
|---|---|---|
| Balls | Bought a fixed number of impacts, then retired | **Immortal.** Bounce forever at a fixed speed |
| Price | Rose with balls in flight, decayed when they died | **+fixed step per ball bought, forever** (Basic 4 then +2) |
| Rock | Hardness grew exponentially with depth | **Flat noise bands, HP 1–7.** Depth scaling is coded and then zeroed |
| Golden pixel | Any ball could break it | **Only a golden ball can** |
| Gray ball | An oversized Boulder | A **Gravity** ball |
| Speeds | Red slowest at 2.7, purple 58 cells/s | 8 × multiplier: Basic 3.2, Lightning 5.6, Poison 6.4, Gravity 8, Smash 9.6, Directional 9.6, Speed 32 |

The demo save matches the model exactly. Its orange price of 116 is 4 + 2 × 56,
and it has 56 orange balls in flight. Its green price of 85 is 30 + 5 × 11, with
11 green balls. Its orange damage of 4 and speed of 0.7 are the sums of the tree
nodes it has bought.

## Play

| Input | Action |
|---|---|
| Press in open ground, drag back, release | Shoot. The ball flies opposite the drag |
| Right-click a ball on the dock | Recycle one of that ball (once unlocked) |
| Drag from rock / right-drag / two fingers | Pan |
| `1`–`7` / dock | Select ball · `G` golden ball |
| Wheel / pinch | Zoom (a pinch zooms about the point between your fingers) |
| `F` / Space | Return camera to auto-fit |
| `U` / `Tab` | Skill tree · `S` Stats |
| `M` / menu | Sound on or off |
| `N` / menu | Next sound style: Bubbles, Raindrops, Crystal |
| `Esc` / `R` | Start menu (pauses) |

A drag shorter than a tenth of a cell cancels the shot. How far you pull does
nothing beyond aiming. The demo snaps every ball to its fixed speed within a
tenth of a second, so slingshot power was never a real mechanic.

## The world

Terrain follows `WorldManager.GetTypeFromNoise`. One Perlin layer sits at about
10.7 cells per noise unit, with a finer layer at a tenth of its weight. The
result falls into these bands:

| Noise | ≤.12 | .12–.15 | .15–.18 | .18–.28 | .28–.30 | .30–.32 | .32–.35 | ≥.35 |
|---|---|---|---|---|---|---|---|---|
| HP | 1 | 2 | 3 | **7** | 6 | 5 | 4 | 1 |

The hardest rock sits in the middle of the range, so seams of HP 7 run between
softer ground. The noise amplitude was calibrated against a real demo world:

| HP | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| Demo world | 40.0% | 11.5 | 10.9 | 3.7 | 3.1 | 3.6 | 27.0 |
| Orebound | 40.0% | 12.0 | 10.8 | 3.2 | 3.7 | 5.0 | 25.3 |

Rock colour follows the HP left, so a damaged pixel steps down to the colour of
softer rock. Visibility is the demo's: breaking a pixel fully reveals its four
neighbours, half-reveals the next ring, dimly reveals the one after, and shows
nothing further out.

**The opening is a cave picture.** Like the demo's hand-drawn world, an
unbreakable picture is stamped around spawn. Every new game picks one of ten at
random: the Anchor, the Ammonite, the Hourglass, the Web, the Kraken, the Old
Tree, the Crown, the Eye, the Lantern and the Labyrinth. Each is drawn from
simple shapes in code and always leaves a way out of the spawn chamber. Rock a
picture seals off completely never holds a special pixel. Worlds saved before
the pictures existed keep the demo's fishbone. Further out, unbreakable
monuments stand in for the silhouettes the full game's trailer masks show
buried deeper.

The run opens the way the demo's does: the 3×3 around spawn cracks open, and
those 9 pixels are your starting money. That's enough for two balls.

## Special pixels

- **Upgrade pixels** (purple, 1 HP, one skill point each). There are 78, one per
  point the tree costs. The first 38 use `SetSpecialTiles` as written: two at
  12–13 cells from spawn, then a spiral whose radius blends linear and geometric
  growth. The other 40 carry that spiral out across the full world.
- **Golden-ball pixels** (cyan, 1 HP, one golden ball each). Only 12 in the
  whole world, one per ring of a widening spiral from 60 to 1300 cells out, so
  each one is a real find. (The demo's density would put thousands here.)
- **Beacons.** When the dig pushes its bounding box outward, any hidden upgrade
  or golden-ball pixel within 15 cells of the new edge lights up from a distance
  (`UpdateFurthest`).
- **The golden pixel.** It is placed uniformly in the world, outside a box
  around spawn (the demo's ±100 of ±176, scaled up). Its depth is a lottery,
  exactly as in the demo. Ordinary balls bounce off it, and **every bounce gives
  you a golden ball.** Only a golden ball can break it. A golden ball is a hint,
  not a digger: fired from anywhere, it curves toward the pixel, breaks nothing,
  and fizzles out after a second and a half or on the first wall or rock it
  touches. Where it goes out, an arrow pointing at the pixel lingers for a few
  seconds, so you know which way to dig. The world goes dark while one is out. Breaking the pixel
  takes three seconds. Then you choose: keep digging the same world, or start a
  new game in a new cave.

## The balls

Every ball is a circle 0.3 cells across: a frictionless, perfectly elastic body
moving among unit squares. Balls pass through each other, because the demo's
ball layer is masked off from itself.

| Ball | Price | Speed | Behaviour |
|---|---|---|---|
| **Basic** | 4, +2 | 3.2 | Chips 1 HP. Piercing lets it carry on through pixels it breaks |
| **Gravity** | 10, +3 | 8 max | Accelerates back toward its launch heading at 10 cells/s² (+30 per speed point) |
| **Poison** | 30, +5 | 6.4 | Poisons what it hits: 1 HP per tick. With spread chance, can jump to one neighbour per tick |
| **Lightning** | 50, +8 | 5.6 | On a global tick, strikes the first rock within 4 cells in a shuffled compass direction |
| **Smash** | 70, +12 | 9.6 | Also hits *Hit spread* random visible neighbours for *Spread damage* |
| **Directional** | 90, +14 | 9.6 | Locked to the axis it was fired along. Drills one line back and forth forever |
| **Speed** | 120, +20 | 32 | Fast |
| **Golden** | free | 18 | See above |

**Ball recycling.** You unlock it per ball in the tree. Right-click that ball's
dock button to pull back the ball that has gone longest without touching
anything. Its next shot is free and doesn't raise the price. You can hold one
at a time, with a 30-second cooldown.

## The skill tree

This is the demo's own lattice, read node by node out of the scene: 66 buttons
on a 40-unit grid. A bought node reveals each neighbour that is the same colour,
or is a New Ball node. A revealed node shows faint outlines of where it leads.
Most nodes cost 1 point. New Ball nodes cost 2–3. One Basic piercing node costs 5.

The demo ships 32 of those nodes enabled (38 points). The other 34 are built
into the scene but switched off: the whole Red and Gray branches, plus more
piercing, poison, lightning and speed. All of them are enabled here, for 78
points in total. Three of the disabled nodes overlapped enabled ones and two
could not be reached, so those five moved by one cell.

**One node is weakened.** The demo's last Basic piercing node gives +6, for 9
piercing in all. Basic damage tops out at 8, so the ball then breaks every rock
and passes through all of it. One cheap ball beats every other kind. Here that
node gives +1, so piercing tops out at 4. A Basic ball still goes through the
soft rock it breaks, but the hard seams (HP 5–7, about a third of all rock)
always turn it back. Making the node more expensive wouldn't help: there is one
upgrade pixel per point, so every node gets bought in the end.

## Architecture

The world is 3163² cells, generated one 64×64 chunk at a time on demand.
Generation is a pure function of the seed, so a save only needs these:

- the seed
- a run-length mask of what has been mined
- the damage on standing cells
- the revealed beacons
- every ball's position and velocity (balls are permanent, so they come back
  where they were)

Rendering uses one offscreen texture, updated per dirty cell. Ungenerated cells
read as solid, so balls can never see unbuilt world as open space.

## What is not the demo

- **World size.** The demo is the 50k-pixel picture. Orebound keeps the picture
  and extends it to ten million pixels. Upgrade placement past the demo's 38 and
  the golden-pixel exclusion box are scaled to fit.
- **Collision.** Unity resolves circle-vs-box contacts with true normals. Here,
  moves are swept per axis, so corner hits reflect on both axes. A piercing ball
  passes only when every cell on its leading edge breaks.
- **Noise.** This uses improved Perlin noise, not Unity's exact
  `Mathf.PerlinNoise`. It's calibrated to the same rock mix, but the patterns
  are not the same.
- **Sound.** The sound effects are Orebound's own, not the demo's, in three
  styles you pick from the menu (or cycle with `N`):
  - **Bubbles:** soft pops
  - **Raindrops:** high plinks with the tick of landing, rain that comes on
    harder while the golden pixel breaks, and distant thunder
  - **Crystal:** struck-glass chimes, all on one pentatonic scale, so mining
    turns into a slow random melody

  Each one is synthesized with Web Audio when it plays, so there are no audio
  files. At most
  six sounds play at once. The constant rock sounds get at most two of those, at
  most about eleven a second, and only for rock on screen, so finds and the
  golden pixel always come through. Sounds pan toward where they happen. The
  menu's toggle or `M` turns sound off. Both choices are kept separately from
  the world save.
- **No tutorial, no offline progress.** The demo has no offline progress
  either. Balls simply resume from the save.

## Local development

`index.html` sits at the repo root, and GitHub Pages serves it from `main`.

```bash
python -m http.server 8765
```

`.claude/launch.json` lets the preview tooling start that server by name.
`file://` works too, but `localStorage` is unavailable there, so saves silently
do nothing.
