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
| Click | Launch at that spot &middot; **drag** to aim |
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

## A calm swarm

Balls drift at **4.4–8.8 cells per second**, and the live cap is a **flat 140**.
Both numbers are chosen for the same reason: the game is something you watch
erode, not something that demands attention. An earlier build scaled the cap
with cavern size and reached 2000+ balls, which is chaos to look at.

**Rush is the deliberate exception** at 75 cells/sec with a long tail — a
shooting star crossing an otherwise slow arena.

Special balls are capped separately and hard: Rush 10, Lightning 18, Poison 20,
Smash 22, Directional 30.

**Balls never steer themselves.** A ball flies exactly where you sent it until it
bounces off rock, and if it crosses 55 cells of open ground it simply retires and
auto-fire replaces it at the working face. An earlier build re-aimed idle balls
at the nearest rock, which looked like they were flying themselves and quietly
took the aiming decision away from the player.

**Ball-to-ball collision is gone.** Hundreds of elastic collisions a frame read as
noise rather than physics, and they kept knocking balls off the headings you
chose. Balls pass through one another.

Because the swarm is small and fixed, throughput cannot come from more impacts —
so it comes from each impact clearing more ground. **Smash craters widen with
Refine**, up to +7 cells of radius, which is the late-game accelerator and is
calm to watch.

## Aiming

Press to set the launch point, drag to aim, release to fire. **Pull further back
to throw harder** — drag length scales launch speed from 30% up to the ball's own
maximum, and never past it. A plain click with no drag fires at full speed.

Power changes only how the ball travels: a soft lob and a hard throw cost the
same and mine the same.

## The tech tree

Upgrades live in a sprawling node graph, not a list. Every node belongs to a ball
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
| Slow balls + scaled swarm | reach ≈ 28.7·t^0.828 | **~2.1 h** typical, 3.2 h worst case |

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
