# OREBOUND

**Play it: https://sevin47.github.io/orebound/**

An incremental mining game, stripped to its core loop. You fire pixels into solid
rock; they bounce forever, chewing a cavern out of a 1024 x 1024 board. Every
block you break pays diamonds. Diamonds buy five upgrades. That is the entire
game.

Single file, no build step, no dependencies. Open `index.html` in any browser.

## Play

| Input | Action |
|---|---|
| Click / tap | Aim and launch one ball |
| Drag | Pan the camera |
| Wheel / pinch | Zoom |
| `F` | Return camera to auto-fit |
| `U` | Upgrades |
| `S` | Stats |
| `Space` | Recall every ball |
| `Esc` / `R` | Start menu (pauses the game) |

Below 820px wide the layout switches to a single column with a touch-sized
bottom bar: Upgrades, Stats, Recall, Menu.

Every session opens on a **start menu**. *Continue* resumes and shows your run's
stats; *New Game* takes two clicks to confirm, then wipes all progression and
rebuilds the arena in memory — no reload, so nothing can write the old save back.

Balls fire independently. Buy a new one mid-run and send it out immediately while
the rest keep bouncing. The camera never follows a ball; it frames the cavern.

## The five upgrades

| Upgrade | Effect | Cost | Cap |
|---|---|---|---|
| **Impact** | x1.30 damage per level | x1.70 per level | none |
| **Velocity** | x1.18 ball speed | x1.75 | 25 |
| **Swarm** | +1 ball | x2.10 | 24 |
| **Refining** | x1.25 diamonds per block | x1.65 | none |
| **Auto-Launch** | parked balls deploy themselves | x1.72 | 12 |

Effect growth is slower than cost growth on every line, so each level buys
strictly less per diamond than the last. Progress comes from the board paying
more as you get deeper, never from an upgrade outrunning its own price.

**Impact and Refining are uncapped**, and that is load-bearing. Income in a game
like this is exponential, so any finite tree gets emptied and the game ends. A
tree with no last level means diamonds always have somewhere to go.

## The curve

Two constants decide everything, and they live together at the top of the script:

```js
const DEPTH_K = 0.0215, DEPTH_E = 1.12;
const depthMul = d => Math.exp(DEPTH_K * Math.pow(d, DEPTH_E));
const VAL_POW  = 0.30;
```

Block HP is `tierHP × depthMul(d)`. Block value is `tierVal × depthMul(d)^0.30`.
Both derive from the same depth term, so difficulty and income can never drift
apart — and because value climbs at only 30% of the rate hardness does, a deeper
block always costs more to break than it pays back. Depth has to be bought with
upgrades; it can never be earned by waiting.

The exponent sits just above 1, so the curve is close to a straight exponential
in depth: gentle over the first twenty cells, then compounding hard.

| Depth | Hardness |
|---|---|
| 20 | x1.9 |
| 100 | x42 |
| 200 | x3.4k |
| 300 | x360k |
| 500 | x7.2e9 |

## One owner per stat

Damage is multiplied by Impact and by nothing else. Income is multiplied by
Refining and by nothing else. No second system stacks another exponential on top
of either.

This is the rule the previous version broke in five places at once: damage was
`1.34^dmg × 1.35^overch`, then x9 on crit, then x`lens`, then x`vein` and
x`tough` against ore — five independent multipliers on one number with no shared
budget. The result was x127k rock hardness against x2.1M ore value and a game
that emptied itself in minutes.

## Balance, measured

The curve above was not guessed. It was fitted by running the real physics
headless with a "buy the cheapest affordable upgrade" player and reading the
depth it reached:

| Elapsed | Depth | Blocks/sec | Levels (impact / velocity / swarm / refining / auto) |
|---|---|---|---|
| 2 min | 18 | 3 | 3 / 0 / 0 / 0 / 0 |
| 5 min | 30 | 2 | 4 / 2 / 0 / 2 / 0 |
| 10 min | 50 | 22 | 9 / 7 / 3 / 6 / 4 |
| 15 min | 152 | 520 | 27 / 24 / 16 / 26 / 12 |
| 20 min | 214 | 172 | 36 / 25 / 22 / 35 / 12 |
| 30 min | 254 | 76 | 40 / 25 / 24 / 40 / 12 |
| 45 min | 279 | 42 | 43 / 25 / 24 / 42 / 12 |
| 65 min | 297 | 20 | 44 / 25 / 24 / 44 / 12 |

The jump between minutes 10 and 15 is Swarm, Velocity and Auto-Launch all
landing at once — the engine igniting. It self-corrects as depth bites. After
the capped upgrades run out, only Impact and Refining remain, and the game
settles into roughly one level per ten minutes, indefinitely.

The first tuning attempt used `K=.0055, E=1.25`. That only reached x7 hardness by
depth 112, damage outran the board, and the entire tree bought out in six
minutes — the same failure as the old build, in a smaller game. The fix was a
much steeper curve in the 0–250 range, not more systems on top.

## Why Velocity is capped and the others are not

A ball's step loop discards its remaining travel when it bounces, so a ball can
land at most one impact per substep — three per frame, about 180 hits a second.
Past that, speed buys nothing. The cap is where the stat stops working, not a
balance decision.

Swarm is capped for frame budget: past 25 balls the swarm costs more to draw
than it earns. Everything beyond it has to be bought as Impact.

## Offline and tabbed away

`requestAnimationFrame` stops in a hidden tab and on a sleeping phone, so the
simulation cannot simply keep running. Elapsed wall-clock time is credited
instead, by eroding the cavern frontier through the same `payout()` path a live
block uses — so offline income can never diverge from online income.

Requires Auto-Launch. Pays 40% of your observed block rate, capped at 12 hours
and 250,000 blocks per return.

## What was removed

The previous version layered five progression systems onto this loop: an ore
tree with six colour currencies, a Forge running real-time research on Cores, a
Residue meta tree, a stratum/descent prestige reset, and a Monolith minigame
with sixteen unique powers. All of it is gone, along with Smash/Drill/Poison
balls, crits, shockwaves, poison spread, and the Codex.

Old saves (`orebound.v2`, `orebound.v1`, `pixelbouncer.v3`) are **not migrated**
— they encode currencies that no longer exist, and a half-conversion would hide
bugs rather than fix them. They are cleared on first load with a notice on the
start menu.

The file went from 2161 lines to 1151.

## Local development

`index.html` sits at the repo root and GitHub Pages serves it from `main`.
Pushing to `main` redeploys; there is no build step.

To serve it locally:

```bash
python -m http.server 8765
```

A `.claude/launch.json` is included so the preview tooling can start that server
by name. `file://` works too, except that `localStorage` is unavailable, so saves
silently no-op.

## Known limitations

- `paintAll()` walks all 1,048,576 cells. It runs on boot, on reset and after an
  offline credit — a visible hitch on slow machines, but not in the frame loop.
- The save embeds a run-length mask of the whole board plus every partially
  damaged cell. It stays small in practice but grows with a heavily chewed board.
- Corners of the board sit at depth ~724 and are effectively impenetrable. That
  is the intended wall; there is no bedrock shell.
