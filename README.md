# OREBOUND

**Play it: https://sevin47.github.io/orebound/**

An incremental mining game. You fire pixels into solid rock; they bounce forever,
chewing a cavern out of a 1024 x 1024 board. Every impact costs energy and yields
diamonds, raw ore, and occasionally a Core. Sixteen Monoliths are buried at
increasing depths, each holding a unique permanent power. Break all sixteen and the
Bedrock shell at the board's edge finally yields.

## Play

Open `index.html` in any browser. No build step, no dependencies.

| Input | Action |
|---|---|
| Click | Fire **one** ball of the selected type |
| `1`–`4` / dock | Select ball type |
| Drag / wheel | Pan / zoom |
| `F` | Return camera to auto-fit |
| `U` | Upgrades |
| `C` | Forge |
| `K` | Codex |
| `M` | Monolith Chamber |
| `Space` | Recall every ball |
| `D` | Descent and the meta tree |
| `Esc` / `R` | Start menu (pauses the game) |

### Touch

| Gesture | Action |
|---|---|
| Tap | Fire one ball at that point |
| Drag | Pan the camera |
| Pinch | Zoom |
| Bottom bar | Upgrades, Forge, Codex, Descend, Lodestone, Recall, Menu |

The layout switches to a single-column, touch-sized version below 820px wide, and the
energy bar moves to full width across the top.

Every session opens on a **start menu**. *Continue* resumes and shows your run's
stats; *New Game* takes two clicks to confirm, then wipes all progression and
rebuilds the arena in memory — no reload, so nothing can write the old save back.

Balls fire independently — buy a new one mid-run and send it out immediately while
the rest keep bouncing. The camera never follows a ball; it frames the cavern.

## Deploying

`index.html` sits at the repo root and GitHub Pages serves it from `main`. Pushing to
`main` redeploys; there is no build step.

## Balls

| Ball | Colour | Behaviour |
|---|---|---|
| Basic | white | Plain bounce |
| Smash | red | Detonates a crater at every impact |
| Drill | blue | Axis-locked, and **drifts sideways each bounce** so it carves a widening swath instead of re-running a cleared corridor |
| Poison | green | Infects rock; poison erodes and spreads on its own |

Balls **collide with each other**. With the Mass Driver module, collisions shatter
the rock around them.

A ball only ever changes course from a wall bounce, a collision, or the Lodestone
Monolith power. There is one safety net — a ball that crosses more than 3x the
cavern radius without touching anything re-aims at the nearest rock — but a normal
crossing is about 1.3x the radius, so in ordinary play it never fires.

## Economy

Three currencies, each with its own job.

- **Diamonds ◆** — from every block. The generic gate.
- **Raw ore ●** — each branch is paid in its own colour, so the tree tells you where
  to dig. Ore is only demanded from a few levels in, so early progress is never
  blocked behind rock you cannot yet reach.
- **Cores ◈** — rare drop, odds climbing steeply with tier. Buys the Prime branch.

| Branch | Paid in | Ore appears past |
|---|---|---|
| Core | Amber | r18 |
| Energy | Sulfur | r80 |
| Smash | Cinnabar | r170 |
| Drill | Azurite | r270 |
| Poison | Verdite | r370 |
| Refining | Voidgem | r470 |
| Automation | Diamonds | — |
| Prime | Cores | — |

Ore depth follows the order branches are **needed**, not a palette. Core and Energy —
what you want first — are fed by the shallowest ore; Refining, which you want last,
by the deepest. Branch colour still matches its ore colour throughout.

The **Forge** (`C`) spends mixed raw ore on eight permanent modules, so every tier
stays useful long after its branch is finished.

## Descent

Clearing a stratum does not end the game and descending does not wipe your work — it
moves you **down** to a new one. Open the Descent panel with `D`.

Each descent pays **Residue ◇**, keyed to one number you can actually see: **how deep
you got**. "I reached r340 last run, this time r410" is a decision you can reason
about. Descend any time past depth r60.

Residue buys a permanent meta tree, weighted toward **head starts** rather than flat
multipliers — head starts compress the slow opening, multipliers only move it:

| Node | Effect |
|---|---|
| Legacy | Arrive with N levels of Damage, Speed and Extra Ball |
| Fleet Start | Extra balls of every colour from the moment you arrive |
| Ignition | Arrive with the Auto-Launcher already running |
| Supply Cache | Arrive holding diamonds and raw ore |
| Deep Roots | Permanent damage multiplier |
| Velocity | Permanent ball speed multiplier |
| Chamber Time / Volley / Force / Munitions | Upgrade the Monolith Chamber |
| Assay | Permanent ore and Core yield multiplier |

Lodestone **powers are kept forever**. Every stratum reseeds its ore, so no two are
the same map. Rock gets harder (x1.55 HP) and richer (x1.8 value) each stratum — gentle
enough that the meta tree stays ahead, so every run reaches deeper than the last.

Cleared strata are archived in the Codex — your excavations are finished and filed,
not deleted.

A run is not meant to exhaust a stratum — you push until the rock outruns your damage,
bank the Residue, and go again. Depth reached at the 30-minute mark:

| | Depth at 30 min |
|---|---|
| Stratum 1 (no meta) | r316 |
| Stratum 2 | r416 |
| Stratum 4 | r451 |
| Stratum 6 | r479 |

Later strata reach those depths far faster — stratum 6 passes r417 in five minutes.

## The Monolith Chamber

Monoliths are no longer buried in the arena. Descending banks a **Monolith
Attempt**, and you spend attempts whenever you like — so a fast stratum never forces
a mini-game on you, and leaving the game idle never blocks a descent.

Open the chamber with `M`. You get a timer and a volley of balls; click to fire, and
they keep ricocheting off the chamber walls until the clock runs out. Damage banks
between attempts, so a Monolith takes several visits to shatter. Break one and it
grants its permanent power, then the next — tougher — Monolith unlocks. All 16 in
sequence.

Residue buys the chamber directly:

| Node | Effect |
|---|---|
| Chamber Time | Seconds per attempt |
| Volley | Balls you may fire |
| Chamber Force | Damage per strike |
| Munitions | Unlocks Smash, then Drill, then Poison balls in the chamber |

Smash hits hardest, so Munitions changes how you aim rather than just adding numbers.

Bedrock is gone entirely — the outer shell existed to be a final wall, and under the
current pacing you are never meant to exhaust a stratum. Its reward slot became
**Anchor Stone** (+40% Residue per descent), so all 16 Monoliths grant something real.

The arena is now pure rock, edge to edge.

## Codex

Tracks ore discovered, Monolith progress, cleared strata and records.

## Momentum (there is no energy)

Energy was removed. It existed to throttle throughput, which fought directly against
short runs, and prestige made it worse: head starts handed you a swarm while Capacity
and Regeneration reset every stratum, so each descent parked you more often — by
stratum 6, every ~7 seconds.

Its branch became **Momentum**, which solves a real problem instead of taxing you:

| Node | Effect |
|---|---|
| Momentum | Balls build speed the further they fly without hitting anything |
| Rebound | Chance a bounce lands a second free hit |
| Impact Force | Flat multiplier on impact damage |

Momentum is worth nothing in a tight cavern and enormous in a wide one, so it directly
counters the travel-time bottleneck that a growing cavern creates. Sunwell now doubles
how fast Momentum builds; Twin Strike (was Flux Regulator) adds a half-damage second
hit; Kinetic Core and Velocity are permanent speed multipliers.

Balls never park now except on a manual recall, so the Auto-Launcher's job is firing
your opening volley and enabling offline mining.

## Balance notes

**Ball count is capped.** It is the worst scaling axis in this game — linear power,
superlinear cost — so the hard ceiling is 120 live balls and the reachable maximum is
about 68. Extra Ball left the Core branch for the Forge (**Multiplicity**, ore-bought,
max 8), Fleet Start is capped at 5, and Prime's Swarm became **Density**, a damage
multiplier. In its place Core gained **Shockwave**: every impact cracks a small area,
which buys throughput without adding entities.

**The renderer was the real performance problem, not the physics.** At 300 balls the
physics costs 2.4ms a frame; the killer was a `shadowBlur` glow per ball, which forces
a blur pass per draw. Glows are now pre-rendered sprites drawn with `drawImage`, trails
shorten as the swarm grows, and the spark budget is a third of what it was.

**Residue income no longer compounds x1.9 per stratum** (now x1.25). Against meta costs
growing x1.5 per level, the old rate meant later runs collapsed to four minutes and
cleared the whole board. Stratum HP rose to x1.75 to match.

Chamber Force was x400 at max against Monoliths growing x1.5; it is now x15 against
x1.62, so a Monolith takes **3-4 attempts at every stage** rather than dropping to one.

A chained simulation of eight consecutive descents:

| Run | Length | Depth | Board | Balls |
|---|---|---|---|---|
| 1 | 30m | r216 | 15% | 10 |
| 3 | 30m | r377 | 43% | 28 |
| 5 | 30m | r442 | 59% | 32 |
| 7 | 30m | r490 | 72% | 40 |
| 8 | 23m | r450 | 61% | 40 |

Runs hold 20-30 minutes, depth climbs, and a stratum is never trivially exhausted.


Block HP scales with distance from the origin, but not as a flat exponential. Depth
sits inside a power law (`exp(DEPTH_K * d^1.35)`), so the curve *accelerates*: about
1.6x at r60, 9x at r180, 500x at r400, 17,000x at r560. A flat `pow^d` could not
separate the two ends — raising it to lengthen the endgame slowed the opening by the
same factor.

Damage is exponential too (`1.34^level`), for the same reason. When it was linear the
player fell further behind the HP curve every hour, stalled at damage level 11 of 34,
and could never catch up even fully maxed.

Ore **value** climbs far slower than HP (`depthMul^0.30`). When income tracked HP,
deep ore paid for the whole tree in a handful of blocks.

A progression simulator (`sim.js`, in the scratchpad) plays these formulas forward:

| Progress | Time | Limited by |
|---|---|---|
| 1% | 19m | energy |
| 5% | 1h 05 | travel |
| 10% | 1h 58 | travel |
| 25% | 2h 08 | energy |
| 50% | 2h 33 | energy |
| 75% | 3h 45 | energy |
| 100% | **5h 48** | energy |

Early play is limited by ball count x speed / cavern radius; from the midgame on it is
limited by energy throughput. Its ball-travel model and its assumption of perfectly
efficient buying are both approximations, so real play will differ.

## Known limitations

- No sound, no touch controls.
- The first 10% still takes about a third of the total. It is far better than it was
  (5% now lands at ~1h instead of ~3h) but remains the least even stretch.
- The save is run-length encoded and stays small early, but grows as the board clears.
