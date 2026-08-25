# OREBOUND

**Play it: https://sevin47.github.io/orebound/**

Dig through a vast pixel world to find the one golden pixel.

Shoot balls into solid rock. They bounce, they break pixels, and every pixel you
break is currency to fire more balls. Buried in the world are rare **upgrade
pixels** that unlock new ball types and change how they behave — and somewhere
in the world is a single golden one. Find it and the run is over.

Single file, no build step, no dependencies. Open `index.html` in any browser.

## Play

| Input | Action |
|---|---|
| Click | Launch at that spot &middot; **drag back** to sling |
| Right-drag / two fingers | Pan |
| `1`–`7` / dock | Select ball type |
| `A` | Toggle auto-fire |
| Wheel / pinch | Zoom |
| `F` | Return camera to auto-fit |
| `U` | Tech tree &middot; `S` Stats |
| `Esc` / `R` | Start menu (pauses) |

Below 820px the layout switches to a single column with a touch bar.

## The two currencies

**Pixels** are mined and spent. Firing a ball costs a flat price and buys a
**fixed number of impacts** — not a duration. That distinction is the whole
economy: rock gets harder the further out you erode, so the pixels returned per
shot decay on their own, until a found upgrade raises your damage again. There
is no hand-tuned cost curve anywhere, because the board *is* the cost curve.

It also permanently fixes a bug from an earlier build: travel speed used to
multiply bounce rate, bounce rate multiplied damage, and damage drove income —
so speed was secretly the strongest damage upgrade in the game. Paying for
impacts rather than for time severs that link. Speed is now per-ball flavour.

**Upgrade points** cannot be bought. They are buried, and the only way to get
them is to dig. Their density falls off as `1/d`, which is not arbitrary: a
uniform density scales with *area*, so the count inside radius `r` grows as `r²`
and upgrades either flood the late game or never arrive in the early one. At
`1/d` the count grows **linearly** with `r`, so they arrive at a steady pace the
whole way out.

## The balls

Seven slots, six built, one reserved. Colour is identity — the tech tree, the
dock and the ball itself all share it.

| | Colour | Behaviour |
|---|---|---|
| **Prospector** | orange | The ball you start with. A plain bounce that breaks the softest rock in one hit. |
| **Directional** | blue | Travels only in straight vertical or horizontal lines, drifting sideways each bounce to carve wide tunnels. |
| **Smash** | red | Detonates on impact, hitting every pixel in a surrounding area. |
| **Poison** | green | Poisons pixels; poison erodes and spreads on its own. |
| **Lightning** | yellow | Crackles with static *as it flies*, arcing into nearby pixels before it ever touches them. |
| **Rush** | purple | Streaks across the cavern like a shooting star, spending its impacts in seconds. |
| **?????** | — | Not specified yet. Slot reserved. |

Ball choice is not cosmetic. Measured on the same save, switching from Rush
(single target) to Smash (area clear) moved throughput from **601 to 1664
pixels/sec**.

## Launch from anywhere, aim where you like

Press to set the launch point, drag to set the direction, release to fire. A
short click fires outward from the centre. Panning moves to right-drag (or two
fingers) so it no longer competes with aiming for the left button.

You can only launch into ground you have already opened, so **your cavern is the
set of valid launch sites**, and it grows as you mine. Auto-fire picks spots on
the working face, so idle play widens the frontier instead of re-grinding cleared
ground.

## The price is the cap

There is no ball limit. **Every ball of a colour already in flight makes the next
one of that colour dearer**, so the swarm settles wherever your income sustains it
and grows only as income grows.

| Orange balls in flight | 0 | 10 | 25 | 50 |
|---|---|---|---|---|
| Price of the next one | 8 | 14 | 28 | 92 |

Each ball has its own base price and its own growth rate, so the premium balls
self-limit hard: Rush climbs at 1.12 per ball and is unaffordable past a dozen or
so, while the starter climbs at 1.05 and can fill the arena.

This is what the reference does. Its dock price for the ball being fired visibly
inflates and then decays while the untouched colours sit at base price — which is
why the same save shows yellow at 1266 in one screenshot and 914 in another.

**Unspent impacts are refunded.** You buy impacts; a ball that drifts off into
open cavern and retires without landing them was never delivered what you paid
for. Without the refund, measured 59–73% of every ticket price was silently
burned, which stalled the opening outright.

## Speeds, measured

Every figure below was measured off reference footage frame by frame, converting
pixels-per-frame to cells-per-second using the on-screen block pitch:

| Ball | Measured | Ours |
|---|---|---|
| Green | 4.99 | 5.0 |
| Blue | 4.58 | 4.6 |
| Yellow | 4.44 | 4.4 |
| Orange | 4.36 | 4.4 |
| Red (Smash) | **2.74** — the slowest | 2.7 |
| Purple (Rush) | **58.2 median, 70 max** — a railgun | 58 |

An earlier pass recorded purple at 2.5 cells/sec and concluded there was no fast
ball. That was wrong, and wrong for an instructive reason: the tracker only
matched a ball to its next position within 90% of one block per frame, so the one
ball that crosses several blocks per frame was **filtered out of its own
measurement**. Removing the cap put it at 58.

**Balls render at a quarter of a block.** Measured 0.19, 0.27 and 0.28 across
three scenes; they had been drawn at 0.8, about three times too large.

## Aiming is a slingshot

Press where you want the ball to launch from, **drag back away from your target**,
and release. The ball flies opposite the pull, and the further back you draw the
harder it leaves. While you drag, the band stretches from the cursor to the
launch point and thickens under tension, a dashed arrow shows the shot going the
other way, and the power reads out as a percentage.

**Power is continuous: 0.5% per screen pixel**, floor 25%, full draw at 200px.
Every pixel of mouse movement is a distinct speed.

| Pulled | 0–50px | 100px | 150px | 200px+ |
|---|---|---|---|---|
| Power | 25% | 50% | 75% | 100% |
| Speed (Prospector) | 1.32 | 2.20 | 3.30 | 4.40 |

Two details that matter. It is measured in **screen** pixels, not world cells:
camera zoom ranges from about 2 to 40, so a fixed distance in cells is a flick at
one end and an arm's length at the other. And the ramp is smooth — the previous
curve jumped from full power to 30% the instant the drag passed six cells, a 3.3×
step off a single pixel of movement, which is what made it feel chunky.

Power changes only how the ball travels: a soft lob and a hard sling cost the
same and mine the same.

## The tech tree

The tree DOM is built **once** and thereafter only its classes and text are
edited in place. It used to rebuild every node with `innerHTML` on each refresh,
and the HUD refreshes several times a second while the panel is open — so the CSS
pulse on an affordable node restarted from frame zero constantly and the whole
tree flickered. Replacing the nodes also made hovering unstable, because the
element under the cursor was a different element moment to moment.

The detail card is parked against the right edge, clear of the lattice; centred
at the top it covered the very rows you were reading. On a phone there is no room
beside the tree, so it drops underneath instead.

Upgrades live in a tight node lattice. Ball unlocks run down a single spine;
each ball's perks run along its own row to the right. **Every link is exactly one
step, horizontal or vertical** — verified: 31 links, 25 horizontal, 6 vertical,
zero diagonals. Nodes sit next to one another instead of trailing long wires
across empty space. Every node belongs to a ball
(which gives it its colour) and carries an icon for what it does; nodes unlock
outward along dotted links, so spending points feels like exploring the tree the
same way finding them feels like exploring the world. Selecting one raises a card
naming it, its colour, and its cost.

Every perk alters something you can see on screen:

- **Prospector** — Fragment (first impact splits the ball in two) · Crossbreak (each impact also strikes four neighbours) · Heavy Strike (every 8th impact lands 5×)
- **Directional** — Widebore (three-wide tunnel) · Pierce (punches through four pixels behind the impact) · Freewheel (only impacts that actually *break* a pixel cost budget)
- **Smash** — Wider Blast · Aftershock (craters strike again a moment later) · Chain Blast (1-in-4 chance of a second blast)
- **Poison** — Virulence (spreads diagonally) · Fast Rot (ticks twice as often) · Contagion (spreads every tick and never dies out)
- **Lightning** — Arc (jumps to three more pixels) · Fork (every arc splits) · Overload (arcs reach twice as far)
- **Rush** — Afterburner (accelerates with every bounce) · Slipstream (drags a damaging wake) · Blur (every impact lands twice)

The rule is that an upgrade must modify ball behaviour directly rather than
nudge a percentage. Upgrades that move a number without changing how the game
feels are the idle genre's dominant failure mode. An earlier pass of this
rewrite shipped cards reading `DAMAGE +55%` and `IMPACTS +10`: exactly that
failure mode. They are gone.

**Refine** is the one deliberate exception — a repeatable global damage
multiplier, so damage can keep pace with an exponential depth curve. It is
**global, not per ball**: per-ball damage meant unlocking a new ball threw away
everything you had invested (measured as a drop from ×69 to ×4.1 the moment a new
ball came online), which punishes exactly the experimentation six ball types
exist to encourage.

## The world fights back

Two hazard pixels spread into ground you have **already cleared**:

- **Bloom** (orange) expands when struck
- **Creep** (purple) seeds a child when destroyed

They are the counterweight that lets ball count go up without the game becoming
trivial. Their growth is bounded *by construction*, not by tuning — see below.

Digging out an upgrade pixel sets off a burst of violet shards — purely
cosmetic, no damage, no game state, it just marks the best moment in the game as
something better than four grey sparks.

Also buried: **handcrafted relics**, hand-drawn bitmaps of much tougher pixels
that emerge as silhouettes while the soft rock erodes around them.

And **90 unbreakable monuments** — rings, crosses, spirals, bars and lenses,
scaled from 26 to 78 cells across, defined as distance fields rather than bitmaps
so they cost nothing to store. They take no damage ever, and they exist purely to
give the cavern a shape: without them erosion converges on a featureless disc.
None are placed within 26 cells of spawn, so a run can never be sealed in.

## Architecture

The world cannot be generated at boot — that is a multi-second freeze. It is
built one **64×64 chunk** at a time, on demand: 2500 chunks, ~4k cells each. The
canvas is a single offscreen texture updated per dirty cell, never redrawn
wholesale. Collision is bucketed on an 8-cell grid.

**Ungenerated chunks must read as solid.** An ungenerated cell holds tier 0 —
the same value a mined-out cell holds — so without an explicit check the whole
untouched world looks like space you have already cleared. That bug carved a
perfect square: mining hollowed out the generated-chunk rectangle, found
"empty" ground beyond it and stopped, leaving balls milling against walls that
were really just the edge of what happened to be generated. `solid()` now
generates on demand, which costs one array read (~26 ns/call) and builds each
chunk once. The offline erosion pass generates as it spreads, for the same
reason.

Measured with 2500 balls live: **simulation 2.7 ms/frame, draw 4.3 ms/frame** —
inside a 16.7 ms budget.

Damage is stored sparsely in a `Map` (only cells hit and not destroyed), because
a `Float64Array` over every cell would be 80 MB on its own.

## Balance, measured

Nothing here was guessed. The curve was fitted by running the real physics
headless with an auto-buying player, re-measured after every structural change:

| Build | Curve | Golden pixel |
|---|---|---|
| First working build | reach ≈ 43.9·t^0.503 | **~24 h** |
| After throughput rework | reach ≈ 76.3·t^0.620 | ~2.1 h |
| Slower balls + frontier launch | reach ≈ 104.1·t^0.404 | ~13.2 h &nbsp;*(regression)* |
| Economy-limited auto-fire | reach ≈ 174.5·t^0.561 | ~0.8 h |
| Slow balls + scaled swarm | reach ≈ 28.7·t^0.828 | ~2.1 h |
| Flat 140 cap | reach ≈ 6.0·t^1.48 (flattening) | ~8 h |
| **Priced swarm (current)** | reach ≈ 45.1·t^0.711 | **~2.4 h** |

The priced swarm **converges** rather than running away, which is the property a
hard cap was standing in for:

| Elapsed | 2 min | 5 min | 10 min | 15 min | 20 min |
|---|---|---|---|---|---|
| Balls live | 90 | 182 | 199 | 206 | **210** |
| Reach | 42 | 145 | 225 | 300 | 396 |

At twenty minutes the mix is 76 orange, 35 blue, 34 poison, 32 smash, 20
lightning and 13 Rush — every colour crowded to its own soft point, and Rush
pinned at a dozen from the five minute mark onward. One ball per ~590 cells of
open cavern, against the 1-per-4 that read as static.

The third row is worth keeping. Slowing ball speeds and launching from the
working face both looked like pure improvements, and together they cost a factor
of six — because balls spawned at the face spend their impact budget immediately
and die young, which exposed a hidden ceiling: auto-fire was **rate**-limited to
ten shots a second, so total output pinned at shots/sec × impacts/shot no matter
how rich the player got. Auto-fire now tops the swarm up until the *pixels* run
out. Your wallet is the limit, not a timer.

Those figures assume near-optimal play. A casual run lands nearer 2–3 hours.

### What simulation caught that reading the code would not

- **The economy could not start.** You began with 0 pixels and a shot cost 8, so
  you could never fire. Zero shots in five minutes.
- **A permanent soft-lock.** Auto-fire only tried the *selected* ball while the
  salvage top-up priced off the *basic* ball. Select a pricier ball, go broke,
  and nothing could ever fire again. Dead at minute 3.
- **Upgrade pixels were uncollectable.** They generated on schedule but carried
  900 base HP against surrounding rock at 5 — 250–500× tougher — so they were
  uncovered and then sat there unbreakable forever. A reward you cannot collect
  is worse than no reward.
- **Hazards grew exponentially.** Spreading was per-*hit*, and a Bloom deep in
  the world takes 160+ hits to break, so each produced ~36 offspring. Measured
  3k → 12k → 23k cells in sixteen minutes. Now each Bloom gets a single chance
  below 1 (p=0.55), so expected offspring converges to ~2.2 cells — bounded by
  arithmetic rather than by a tuning constant.

Note also that the balance is verified against the *worst case* where a player
buys every shiny perk and unlock **before** refining even once. Players buy the
interesting-looking thing first, and the design has to survive that.

## Golden pixel & seeds

One cell, uniformly placed, **no hint system** — a true needle in a haystack. It
is kept out of the opening hollow so it cannot be won in the first ten seconds.

The world seed is **rolled per run and saved**, so the golden pixel hides
somewhere new every playthrough.

## Offline

Away time is credited by eroding the cavern frontier through the same `payout()`
path a live pixel uses, so offline income can never diverge from online income.
Requires auto-fire; pays 40% of your observed rate, capped at 12 hours and
400,000 pixels per return.

## Local development

`index.html` sits at the repo root; GitHub Pages serves it from `main`.

```bash
python -m http.server 8765
```

`.claude/launch.json` lets the preview tooling start that server by name.
`file://` works too, except `localStorage` is unavailable so saves silently
no-op.

## Known limitations

- One of seven ball abilities is unspecified and shows as `?????`.
- Save embeds a run-length mask of every cell plus every partially
  damaged cell (~25 KB at 8% cleared; grows with a heavily chewed board).
- Board corners sit at depth ~2236 and are effectively impenetrable; that is the
  intended wall, there is no bedrock shell.
