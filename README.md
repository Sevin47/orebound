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

All seven built. Colour is identity — the tech tree, the dock and the ball itself
all share it.

Six of them are points. **The Boulder is the only one with a body**, and that is
its whole character: it collides when its *rim* meets stone rather than when its
centre crosses a cell, so it stops short of walls, crushes a disc rather than
chipping a pixel, and cannot squeeze through gaps the others slip past.

| | Colour | Behaviour |
|---|---|---|
| **Prospector** | orange | The ball you start with. A plain bounce that breaks the softest rock in one hit. |
| **Directional** | blue | Travels only in straight vertical or horizontal lines, drifting sideways each bounce to carve wide tunnels. |
| **Smash** | red | Detonates on impact, hitting every pixel in a surrounding area. |
| **Poison** | green | Poisons pixels; poison erodes and spreads on its own. |
| **Lightning** | yellow | Crackles with static *as it flies*, arcing into nearby pixels before it ever touches them. |
| **Rush** | purple | A railgun. Crosses the cavern in a heartbeat and carries a deep magazine (70 impacts) to spend on the way. |
| **Boulder** | slate | Oversized — seven cells across, ten with Heft. It has an actual body, so it collides on its rim and crushes everything its width touches. The slowest thing in the cavern. |

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

That last sentence used to be a description of the intent rather than of the
code. Auto-fire chose a radius uniformly between 0.55 and 1.0 of the reach, which
sounds like the frontier and is not: **measured, the average launch sat at 0.63 of
the reach**, several hundred cells inside open cavern. A ball gets twelve seconds
to find rock — 53 cells for a Prospector — so it could not possibly cross that
gap, and it retired with its magazine nearly untouched. Across a thirty-minute
run, **87–93% of every impact the player paid for was never landed.** The refund
hid the loss in the wallet; the throughput was simply gone.

It now picks a bearing, walks it end to end, and launches from one of the **faces**
along it — an open cell with rock against it, chosen at random from all of them.
Worth **4.4× the pixels mined**, with no number anywhere made bigger.

Stopping at the *first* open cell was a second bug wearing the first one's
clothes, and it took a run played all the way to the golden pixel to see it. The
outermost open cell on a bearing is the outer rim and nothing else, so once the
dig punched past a patch of stubborn rock the frontier moved beyond it and that
pocket was never worked again. Measured mid-run: **0 of 300 launches** landed
inside a ring the cavern had already passed, and the golden pixel sat in one of
those pockets with the frontier 300 cells beyond it and receding.

The tell is in ring coverage. Working only the rim, rings saturated at **82%** —
the missing fifth was abandoned pockets, not bedrock. Working every face, inner
rings finish at **96–99%**.

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
| Orange | 4.36 | 4.4 base, up to 8.9 with speed perks |
| Red (Smash) | **2.74** — the slowest | 2.7 |
| Purple (Rush) | **58.2 median, 70 max** — a railgun | 58 |

An earlier pass recorded purple at 2.5 cells/sec and concluded there was no fast
ball. That was wrong, and wrong for an instructive reason: the tracker only
matched a ball to its next position within 90% of one block per frame, so the one
ball that crosses several blocks per frame was **filtered out of its own
measurement**. Removing the cap put it at 58.

**Balls render at a quarter of a block.** Measured 0.19, 0.27 and 0.28 across
three scenes; they had been drawn at 0.8, about three times too large.

**And they are drawn as discs.** They used to be filled rectangles, which was the
one part of the picture disagreeing with everything around it: the Boulder
collides on a circular rim, and every ball carries a radial glow. The rock stays
square — it is a grid of pixels and should read as one — but nothing that moves is
a square any more, and the trails have round caps to match.

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

- **Prospector** — Swift / Fleet / Bolt (travel speed, +25% / +25% / +30%, stacking to 4.4 → 8.9 cells/sec) · Fragment (first impact splits the ball in two) · Crossbreak (each impact also strikes four neighbours) · Heavy Strike (every 8th impact lands 5×)
- **Directional** — Widebore (three-wide tunnel) · Pierce (punches through four pixels behind the impact) · Freewheel (only impacts that actually *break* a pixel cost budget)
- **Smash** — Wider Blast · Aftershock (craters strike again a moment later) · Chain Blast (1-in-4 chance of a second blast)
- **Poison** — Virulence (spreads diagonally) · Fast Rot (ticks twice as often) · Contagion (spreads every tick and never dies out)
- **Lightning** — Arc (jumps to three more pixels) · Fork (every arc splits) · Overload (arcs reach twice as far)
- **Rush** — Afterburner (accelerates with every bounce) · Slipstream (drags a damaging wake) · Blur (every impact lands twice)
- **Boulder** — Heft (another cell and a half across) · Unstoppable (never deflected; holds its heading and bores a tunnel) · Shatter (its final impact goes off at three times its own width)

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
| Priced swarm | reach ≈ 45.1·t^0.711 | ~2.4 h |
| Drift by duration | reach ≈ 31.4·t^0.826 | ~1.9 h |
| Seven balls, Boulder built | reach ≈ 37.7·t^0.817 | ~1.6 h |
| Rim-only launch + prize at 820–1020 | reach ≈ 75.4·t^0.699 | 51.6 min *(measured, missed)* |
| **Every face worked + prize at 600–730 (current)** | reach ≈ 52.9·t^0.824 | **30.3 min** *(measured)* |

The priced swarm **converges** rather than running away, which is the property a
hard cap was standing in for:

| Elapsed | 2 min | 5 min | 10 min | 15 min | 20 min |
|---|---|---|---|---|---|
| Balls live | 105 | 173 | 211 | 224 | **235** |
| Boulders | — | 10 | 14 | 15 | 16 |
| Reach | 46 | 136 | 270 | 328 | 435 |

At twenty minutes the mix is 78 orange, 36 blue, 36 poison, 34 smash, 21
lightning, 16 Boulder and 14 Rush — every colour crowded to its own soft point,
with the two premium balls pinned in the low teens from five minutes onward. One ball per ~590 cells of
open cavern, against the 1-per-4 that read as static.

The third row is worth keeping. Slowing ball speeds and launching from the
working face both looked like pure improvements, and together they cost a factor
of six — because balls spawned at the face spend their impact budget immediately
and die young, which exposed a hidden ceiling: auto-fire was **rate**-limited to
ten shots a second, so total output pinned at shots/sec × impacts/shot no matter
how rich the player got. Auto-fire now tops the swarm up until the *pixels* run
out. Your wallet is the limit, not a timer.

Those figures assume near-optimal play. A casual run runs somewhat longer.

### Where the thirty minutes comes from

Two changes, and only one of them touches throughput.

Launching on the working face rather than inside the cavern recovered the 87–93%
of impacts that were being thrown away. The rest is the prize itself.

**The first calibration was wrong, and only a completed run caught it.** Sampling
720 cells on each ring gives the fraction cleared at each minute, which *is* the
chance the run has ended if the pixel sits at that radius; read off a rim-only run
that put the band at 820–1020 and predicted a median a shade under half an hour.
But that run was abandoning every pocket it passed, so it drove *reach* outward
much faster than it actually cleared ground — ring coverage stalling at 82% was
the warning, and it read as bedrock. Playing it out: **51.6 minutes**, not 30.

Aggregate coverage is a seductive proxy. Reach passing a radius is not the same as
that radius being dug out, and the gap between them is exactly where the bug was
hiding. The only honest measurement of how long a run takes is a run.

With every face worked, one completed run gives `reach ≈ 52.9·t^0.824`, and the
prize turns up at roughly **50% ring coverage** — which lands when reach reaches
about **1.23×** its radius. Checked against the finished board: 50% coverage sat
at r=960 with reach at 1203, against 978 predicted. Half an hour puts reach at
812, so the median radius wants to be `812 / 1.23 ≈ 660`.

Hence the band **600–730**, which spans about 26–35 minutes. Verified by playing
it: golden pixel at radius 649, **found at 30.3 minutes**.

Both figures are single runs. The first ten minutes vary a good deal — 27k to 165k
cells mined by minute 5 across three runs — and that variance carries forward, so
treat 30 minutes as the centre of a fairly broad distribution rather than a
promise.

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
- **The frontier abandoned its own pockets.** Fixing the wasted impacts by
  launching on the working face introduced this: it took the *outermost* open cell
  on a bearing, so ground the dig had already flowed past was never revisited.
  Aggregate coverage looked fine and reach climbed faster than ever. It surfaced
  only by playing a run to the end and finding the prize sealed in a pocket with
  0 of 300 launches able to reach it.
- **Nine in ten impacts were never landed.** Auto-fire launched balls a long way
  inside the open cavern, further than a ball can travel before its drift timer
  retires it, so most shots expired without touching rock. Nothing looked wrong:
  balls flew, the wallet refilled from refunds, the cavern grew. Only counting
  impacts landed against impacts bought showed 87–93% going nowhere.
- **The world was rebuilt differently on every reload.** The seed was written into
  the save on every write and never once read back, so each page load rolled a new
  one: terrain regenerated under the saved mined-mask, relics and monuments moved,
  the damage map pointed at cells that were now different rock, and the golden
  pixel hid somewhere else. A run was only ever coherent within one page load.
- **The offline pass ate the monuments.** It cleared whatever cell it landed on
  without any of the checks a live impact makes, so a long enough absence
  dissolved the one thing in the world that is meant to be permanent — and it
  could dig out the golden pixel and end the run while the tab was shut.
- **Hazards grew exponentially.** Spreading was per-*hit*, and a Bloom deep in
  the world takes 160+ hits to break, so each produced ~36 offspring. Measured
  3k → 12k → 23k cells in sixteen minutes. Now each Bloom gets a single chance
  below 1 (p=0.55), so expected offspring converges to ~2.2 cells — bounded by
  arithmetic rather than by a tuning constant.

Note also that the balance is verified against the *worst case* where a player
buys every shiny perk and unlock **before** refining even once. Players buy the
interesting-looking thing first, and the design has to survive that.

## Golden pixel & seeds

One cell, on a bearing you cannot guess, **no hint system**. What is bounded is
its *depth*: the radius is drawn from **600–730**, the band the dig actually
reaches in about half an hour. Measured end to end: a pixel at radius 649 was dug
out at **30.3 minutes**.

It used to be placed uniformly over all ten million cells, which made the length
of a run a lottery rather than a design decision. The median sat near an hour, but
the spread was unbounded at both ends — and **21.5% of runs** (everything outside
the inscribed circle) put the prize in the corners, where depth runs past 2200 and
hardness is `e^(0.0035·d^1.1)`. Those runs were not long, they were unwinnable,
and the player had no way to tell which kind they were in.

It is still a needle. You do not know the bearing, the cavern is not a solid disc,
and finding it means clearing most of a ring three-quarters of a mile round. The
world is still 3163 cells across and you can keep digging as long as you like —
**what is bounded is the win, not the map**.

The world seed is **rolled per run and saved**, so the golden pixel hides
somewhere new every playthrough — and, since the seed is now actually read back on
load, it stays where it was hidden.

## Offline

Away time is credited by eroding the cavern frontier through the same `payout()`
path a live pixel uses, so offline income can never diverge from online income.
Requires auto-fire; pays 40% of your observed rate, capped at 4 hours and
**150,000 cells** per return.

The cell cap is the real control, and it is the only one that can be: it is the
only term scaled to the run rather than to the player. Mid-run the observed rate
is ~1000 cells/sec, so 40% of it reaches any plausible ceiling within minutes —
the old 400,000 was spent six minutes into a twelve-hour absence, which made every
absence past six minutes pay exactly the same, and the hours were decoration. A
full run mines ~2.4M cells, so 400k was a sixth of the game handed over for
closing a tab, along with 206 upgrade points against the ~726 a whole run yields.
At half an hour a run, away time should be a top-up: 150k is about 6% of a run.
This is the one number here picked by judgement rather than measured.

The pass itself was rewritten to be honest and to stop hitching:

- It **cannot break bedrock** and **cannot dig out the golden pixel** — the run is
  not something to come back and find already spent.
- It keeps the hazard tally, the poison map and the lighting consistent, where it
  used to leave all three wrong.
- Lighting is a **Chebyshev distance transform in two linear sweeps** rather than
  a 9×9 stamp per cleared cell. Erosion clears contiguous blobs, so the old stamp
  rewrote the same neighbourhood eighty times over — and the return-to-tab handler
  ran it across *every* cleared cell on the board. Measured in the browser: a full
  150k-cell credit now costs well under a tenth of a second.

Hazard cells are also written into the save explicitly. The mined mask cannot
describe them — a hazard that spread is sitting on ground you had cleared, so the
mask calls that cell solid and the generator refilled it with the original rock.
Cavern you had dug out healed over while you were away.

## Local development

`index.html` sits at the repo root; GitHub Pages serves it from `main`.

```bash
python -m http.server 8765
```

`.claude/launch.json` lets the preview tooling start that server by name.
`file://` works too, except `localStorage` is unavailable so saves silently
no-op.

## Known limitations

- Save embeds a run-length mask of every cell, every partially damaged cell and
  every live hazard. Measured **628 KB at thirty minutes** (2.28M cells cleared),
  written every 15 seconds; `rleMined` walks all ten million cells to build it.
- A cold load regenerates every chunk the save touched — 851 of them at thirty
  minutes — because terrain is derived from the seed and only the mask is stored.
  That is a real load screen on a mature save.
- Board corners sit at depth ~2236 and are effectively impenetrable; that is the
  intended wall, there is no bedrock shell. The golden pixel is no longer placed
  out there, so this can no longer make a run unwinnable.
