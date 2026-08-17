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
| `Space` | Recall every ball |
| `Esc` / `R` | Start menu (pauses the game) |

### Touch

| Gesture | Action |
|---|---|
| Tap | Fire one ball at that point |
| Drag | Pan the camera |
| Pinch | Zoom |
| Bottom bar | Upgrades, Forge, Codex, Recall, Menu |

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
the rock around them. Any ball that crosses 70 cells of open ground without hitting
anything **re-aims at the nearest rock**, which is what stops balls idling in a
cavern that has outgrown them.

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

## Monoliths, Bedrock, Codex

Sixteen Monoliths sit on a spiral from r120 to r450, rendered as glowing landmarks
visible through unmined dark, with off-screen edge markers pointing the way. Each has
enormous HP and grants a unique **mechanic** when broken — not a percentage. Arc
Conductor makes impacts chain. Lodestone curves balls toward ore. Phase Edge lets
balls pass through rock. The sixteenth is **Bedrock Solvent**, which unseals the
outer shell and makes 100% completion possible.

Offline mining deliberately skips Monoliths and Bedrock — those must be earned live.

The **Codex** (`K`) tracks ore discovered, Monolith progress, and records.

## Energy

Energy is the hard throughput limit on the whole game, not a formality. Base
regeneration is **0.8/s** against a **0.75** cost per bounce, so roughly one impact
per second is all you can sustain at the start. Release a few balls and the tank
drains; they park, and you wait.

That is the loop the Auto-Launcher plugs into: when energy bottoms out every ball
parks, and once the tank is back above 22% the launcher trickles them out again by
itself. Energy regen is what you are really buying when you invest in that branch.

Measured uptime (share of time with balls actually in flight):

| Stage | Uptime | Drain cycles / 2min |
|---|---|---|
| Fresh start, 1 ball | 31% | 1 |
| With Auto-Launcher | 34% | 3 |
| Early-mid upgrades | 60% | 3 |
| Mid game | 66% | 2 |
| Late game, big swarm | 50% | 6 |

It tightens again late because swarm size outgrows the tank faster than regeneration
grows. Maxed regeneration sustains ~166 impacts/sec (~332 with the energy Monolith
powers) — a real ceiling rather than an unreachable one.

## Balance notes

Block HP scales exponentially with distance from the origin (`DEPTH_POW`), because an
incremental player curves exponentially and a fixed-HP board cannot keep up.

Ore **value** scales far more slowly than HP (`DEPTH_POW^0.32`). This is deliberate:
when income tracked HP, deep ore paid for the whole upgrade tree in a handful of
blocks and the late game collapsed.

A progression simulator (`sim.js`, in the scratchpad) plays these formulas forward and
estimates a full clear at about **8h10m**. After the opening ramp the mining rate
holds roughly flat at 65-90 blocks/sec all the way to 100%, with no cliff. Two things
got it there: capping energy throughput (impacts/sec tops out near 80 instead of
1,100) and making raw ore, not diamonds, the binding cost.

Its ball-travel model and its assumption of perfectly efficient buying are both
approximations — real play will differ.

## Known limitations

- No sound, no touch controls.
- The first 10% of the board takes roughly 4h45 of the 8h10 total. A slow open is
  normal for the genre, but this is the remaining rough edge; the fix if it drags is
  to delay ore gating on the first Damage levels rather than touch energy.
- The save is run-length encoded and stays small early, but grows as the board clears.
