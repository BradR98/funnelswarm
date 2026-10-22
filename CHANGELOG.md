# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### PWA install id fix (branch `feature/funnel-logic`)
- The manifest `id` was `./index.html`. Browsers resolve `id` against the ORIGIN, not the page folder, so every game on bradr98.github.io with that id (original Mob Swarm, Mob Swarm 2, the funnel) shared one identity: installing one made the others report "already installed". The funnel id is now `/mob-swarm-funnel/`, which is unique. Build label and cache: `funnel v17`.

### Funnel PWA icon (branch `feature/funnel-logic`)
- `icon-192.png` and `icon-512.png` now show an orange funnel (with falling red mobs) on top of the original blue/red target, so the funnel build is distinguishable from the original on a phone home screen. The manifest names are "Mob Swarm Funnel" / "Funnel". Build label and cache: `funnel v16`. Already-installed home-screen shortcuts keep the old icon until removed and re-added.

### Level 7 vetted: two gates from L7 on (branch `feature/funnel-logic`)
- The vetted L7 export (chute 90, density 2.0, speed 131, HP 1, gate ceiling x1.9, damage 1, bullets 1) already matched the L5-to-L9 ramp, so L8 and later keep following it (L8 87px / 2.1x / 135 / x1.9, L9 as before). The one difference was gate concurrency 2: L7 generated 3 gates. Levels 7 and up now have exactly 2 gates (`maxG`; the harness restart can still pad up to 4 for experiments).
- Build label and cache: `funnel v15`.

### Level 9 reshape: old L6 tuning moved to L9 (branch `feature/funnel-logic`)
- The vetted settings (chute 84px, density 2.2x, speed 140, mob HP 1, 2 gates, gate ceiling x1.8, damage 1, bullets 1) now describe **Level 9**. Levels 1-5 are unchanged (verified identical). Levels 6-8 ramp linearly from L5 to L9 via `effLevel` (chute 93/90/87, density 1.9/2.0/2.1, speed 127/131/135). Levels past 9 continue the old curve three levels later (L10 plays like old L7, L13 like old L10).
- `effLevel(n)` drives difficulty: tier, pour mode and its speed ramp, wave sizes, barricade count and hits, red speed, red HP growth, density and gate windows. The level number, rng seed and boss test (`n % 10`) still use the real level.
- Gate ceiling at L9 is x1.8 (old L6 window was x2.2): the window is scaled by a factor that ramps 1.0 -> 0.82 from L5 to L9 and stays there. Chute width eases 96 -> 84 over L5-L9 and stays 84. L9+ always has at least 2 gates.
- Build label and cache: `funnel v14`.

### Consistent build label (branch `feature/funnel-logic`)
- The on-screen label now reads `funnel v13` and always equals the service-worker cache suffix (`mob-swarm-funnel-v13`). Bump both together on every push. The old `v18 · c18` label belonged to the production build and was stale on this branch.

### Debug tuning harness (branch `feature/funnel-logic`)
- **`window.TUNING`** (chuteWidth, mobDensity, mobSpeed, mobRedHP, gateConcurrency, gateXRange, playerDmg, playerBullets), seeded with the L6 numbers. `reset()` re-syncs it to the math of every level that loads (including natural progression, warp and `?level=`), so the sliders physically move.
- **Floating `#debug-harness` panel** (dark, translucent, z-index 9999, collapsible, top-left over the canvas): one `<input type="range">` per variable with an adjacent live value `<span>`. The "shooter advantage + range" is a read-only `<span>` = `gateXRange - 1`. Pointer and touch events on the panel are stopped so dragging a slider never fires the cannon.
- **Live (next frame):** `chuteWidth` (`applyChute` re-lays the column, barricade plugs and gate tracks), `playerDmg` (scales red HP removed per hit, probabilistic rounding), `playerBullets` (extra projectiles per trigger).
- **On "Restart Level":** `mobDensity`, `mobSpeed`, `mobRedHP`, `gateConcurrency` and `gateXRange` (ceiling; the floor keeps the level's proportion) override the level math while the level rebuilds (`tunApply`). Gate concurrency is `min(mob-count tier cap, slider)`.
- **Copy Settings** button: copies one-line JSON of the current level plus every TUNING value, e.g. `{"level":6,"chuteWidth":96,...}`, to the clipboard (with a textarea / prompt fallback).
- **Harness fix: sliders now change the running game.** Live next frame: chuteWidth, mobDensity (interval and pipe rate rescaled), mobSpeed, mobRedHP (mobs spawned from then on), gateConcurrency, playerDmg, playerBullets. Only gateXRange needs Restart Level because gate values are baked at level build. Restart also pads the level to at least `gateConcurrency` gates (max 4, the number of gate rows) so raising the slider on a one-gate level actually adds gates. Active gates are still limited by the mob-count caps (1 / 2 / unlimited).
- **Freeze guard.** `frame()` now catches exceptions from update/render so one bad frame can no longer kill the rAF loop (which looks like a freeze until refresh). The message is shown in red inside the panel, stored in `window.__lastError` and included in Copy Settings as `error`. Restart Level is guarded the same way.
- Service worker cache: `mob-swarm-funnel-v12`.

### Gate concurrency cap + mid-game density squeeze (branch `feature/funnel-logic`)
- **Gate concurrency cap.** Gates here are sliding rows built at level start (they never "fall"), so the cap works as an activation limit. Gates start dormant (not drawn, no hit test, no motion). One more is activated per frame only while the active count is under the cap, which depends on the live blue-mob count: under 10 mobs gives 1 active gate, 10-24 gives 2, 25+ gives no extra limit. Once active a gate stays active. RNG value math is untouched. Debug: `__mob.get().gatesActive`.
- **Density tiers shifted** (`densityFor`): L1-2 1.0x, L3 1.5x (was L4), L4-5 1.8x (new bridge tier), L6-9 2.2x (was L7), L10+ 3.5x.
- Service worker cache: `mob-swarm-funnel-v8`.

### Chute overwhelm: crushed gate ceiling + late-game density tiers (branch `feature/funnel-logic`)
- **Crushed RNG gate ceiling.** `GATE_NERF` is replaced by explicit windows (`GATE_WIN`, linear between anchors, hard-clamped): `x` gates roll L4 x1.2-1.8, L7 x1.5-2.4, L10+ x2.0-3.0, so the absolute cap is x3.0. `+n` gates make n+1 mobs, so their window is the x window minus 1 (L10+ tops out at +2.0, i.e. x3). Fractional values and probabilistic rounding are unchanged.
- **Density tiers** (`densityFor`): L1-3 1.0x, L4-6 1.5x, L7-9 2.2x, L10+ 3.5x. These scale the wave interval and the pipe and pour rates. Back-pressure in `emitRed` is unchanged, so overflow waits in the spawn queue (capped by the 1000-mob pool) and the chute packs, which feeds Vertical Surge. Sandbox exempt.
- Service worker cache: `mob-swarm-funnel-v7`.

### TTK ceiling and HP curve (branch `feature/funnel-logic`)
- **RNG gate nerf.** New `GATE_NERF = 0.7` scales both the floor and the ceiling of every gate roll, and the guaranteed-multiplier fallback. Measured `x` ranges: L4 x1.4-x2.8, L7 x2.1-x4.2, L10 x2.8-x5.6 (was x2-x4 / x3-x6 / x4-x8). `+` gates scale the same way. Fractional values and probabilistic rounding in `applyGate` are unchanged.
- **Steeper HP curve.** `RED_HP_GROWTH` goes from 1.10 to 1.15, so red HP = base x 1.15^(level-5) (L7 +32% over L5, L10 about 2x).
- Service worker cache: `mob-swarm-funnel-v6`.

### Mid-game density step (branch `feature/funnel-logic`)
- **1.5x density step at Level 4.** Spawn density is now level-gated. Levels 1-3 run at exactly 1.0x. Every level above 3 (`level > 3`) spawns 1.5x: wave interval / 1.5, and pipe and pour rates x1.5. The old global 1.5x is gone, so Levels 1-3 are now 1.0x. The sandbox is exempt.
- Back-pressure is unchanged: `emitRed` still refuses to spawn inside a pile at the mouth, so extra volume queues instead of overlapping, and the chute packs tightly. That feeds Vertical Surge.
- Service worker cache: `mob-swarm-funnel-v5`.

### Single-chute pivot: RNG gate drops + Vertical Surge (branch `feature/funnel-logic`)
- **Multi-lane and spill logic removed.** The board is now exactly ONE 96px vertical chute (`columnCount()` is always 1, `MAX_COLS = 1`). Deleted: lateral spill and fluid-leveling drift, door HP and door pressure (`doorPressure`, `breachDoor`, `hitDoor`, `doorAt`, crush-angle filter), lane dividers and their rendering, the funnel-lane and `funnelBoost` logic, blue barricades and their adjacent-lane threat waves, and the multi-column barricade lane picking. The constricted dead-space margins are kept and now flank the chute. Barricades are gray plugs that each block the whole pipe.
- **Dynamic RNG gate drops.** The static pools (x2 / +2 low, x3 / +10 high) are replaced by `rollGate(kind, level)`. Each gate rolls inside a window whose floor and ceiling both rise with the level, e.g. L4 x2-x4, L7 x3-x6, L10 x4-x8. `+` gates scale the same way. Low gates favour the bottom of the window and high gates the top. Values are fractional ("messy"), e.g. x2.2 and +1.5. `applyGate` rounds probabilistically, so the average clone count equals the displayed value.
- **Vertical Surge.** When 5 or more STALLED mobs touch each other in the chute (a real jam, not free-flowing traffic), that touching cluster surges for 1.5 s: +50% fall speed (above the speed cap) and knockback immunity. A 3 s cooldown follows. Surging mobs are drawn with an orange ring. Mobs inside the spawn mouth are exempt. Debug: `__mob.get().surges` / `.surging`.
- Service worker cache: `mob-swarm-funnel-v4`. The legacy headless suite (v18 lane/door expectations) now aborts at the door tests and needs a rewrite before any merge to `main`.

### Funnel: track constriction and dead space (branch `feature/funnel-logic`)
- **Narrower track:** `TRACK_SCALE = 0.72` scales every lane width (120/150/104/80 -> 86/108/75/58px; divider width unchanged), so the packed track is 27-28% narrower: 1 lane 86px, 2 lanes 224px, 3 lanes 238px, 4 lanes 252px (was 120/308/328/344). Lane math (`colSpan`) recalculates from the new widths, so the same lane counts (1/2/3/4) are packed tightly in the centre; gates, barricades, doors, bumpers and the boss scale with the lane. The L99 sandbox keeps its own width.
- **Dead space:** everything outside the active span `[AX0, AX1]` is drawn as dark amber-hatched margin panels, with brighter funnel walls, replacing the full-width grey bed. The danger line and the HQ bar / HP trail now span only the active track.
- **Bounds:** nothing can enter the margins. Reds spawn and are clamped inside lane spans, blues clamp to the lane, the cannon clamps to `AX0+10 .. AX1-10`, the boss strafes inside `[AX0, AX1]`, and only blues inside `[AX0, AX1]` can damage the base.
- Test note: `test/headless.js` now needs `Uint16Array` in its sandbox (added). It currently shows 19 FAILs, all expectations of the original v18 design that the funnel work intentionally changed (column alternation / multi-column dispatch, barricade counts, pour rates, 120px tier-1 lane, etc.); it needs a rewrite before any merge to `main`.
- Service worker cache `mob-swarm-funnel-v3`.

### Funnel: density-trap fixes A-C (branch `feature/funnel-logic`)
- **Diagnosis:** barricades could sit only 25-60px under the spawn mouth (about a third of levels), so the "stack" was the mouth itself: 5 reds counted as a jam, the n^1.5 surge fired and doors broke within 2-4s regardless of play, while fixed-rate spawning stuffed new reds into the overlap.
- **A, runway floor:** a barricade's top must be at least `BARR_RUNWAY = 140`px below the spawn mouth (skipped if the gates leave no room). Measured over L3-39: min runway 25px -> 140px.
- **B, emission back-pressure:** `emitRed` now tries 3 spawn positions and spawns only into a clear spot (no red within 0.9x the summed radii); otherwise the wave / threat queue waits. Mouth overlap pairs fall to ~0. To recover the throughput this costs, the spawn mouth half-width cap went 24 -> `MOUTH_HALF = 44`. Net L6 steady-state on-board reds ~150 vs ~215 before (~70%), a deliberate trade for real, physical stacks.
- **C, mouth excluded from pressure:** door-pressure counting starts at `BASE_BOTTOM + 3 bodies`, so reds still leaving the mouth never count as jam pressure.
- Doors still break once and stay open (option D, re-arming doors, declined).
- Service worker cache `mob-swarm-funnel-v2`.

### Funnel: TTK baseline and HP flattening (branch `feature/funnel-logic`)
- **Base HP halved:** the HQ's starting HP is now `L.baseHP x BASE_HP_SCALE (0.5)` on every non-sandbox level, so the 1x starter cannon clears the funnel with the friction the 2x gave (L1 250 -> 125; sampled L2 128, L5 875, L9 1088, L12 1109). Red mobs themselves have 1-3 HP at base so cannot be halved: this is the HQ base HP ("Level 1 Base HP"). Boss HP, `BASE_HIT` and the sandbox are untouched.
- **HP curve flattened:** `RED_HP_GROWTH` 1.25 -> 1.10, so red HP = `RT_HP x 1.10^(level-5)` from L5 (L7 = +21% over L5, was +56%; L12 = x1.95). Density multiplier untouched.
- **Density cap verified and one leak fixed:** `SPAWN_DENSITY` is a constant 1.5 (wave interval / 1.5, pipe, threat and pour rates x1.5), not a per-level ramp. Audit found the pipe `funnelBoost` (x2 on 3+ lane boards) was also being applied to the endless pour on L3-8, where `pipeRate` is already the whole-board rate: that made those levels ~3x (2x boost x 1.5) instead of 1.5x. The boost now applies only to non-pour waves (matching the old 2-lane dispatch) and no longer to threat streams. The per-level wave-interval / wave-size ramps from the original generator are unchanged.
- Service worker cache renamed `mob-swarm-funnel-v1` so this build can be hosted beside production on the same GitHub Pages origin without sharing a cache.

### Funnel: flow, flatten, lethality (branch `feature/funnel-logic`)
- **Directional bias fixed (fluid leveling):** root cause was the lane clamp in the red solver: with open gaps on both sides of a lane it did `break` after the first matching door, so the lowest-index (left) door always won and a funnel lane could only ever flow left. It now takes the union of every open gap the lane touches. On top of that, stalled reds beside a breached door drift through it; with gaps on both sides they pick the emptier lane (probability tilts with the stack-height difference, equal lanes = fair coin), and door grind damage is weighted by the same lane-height ratio (0.25x-2x, +/-20% random tie-break) so the emptier neighbour breaks first. Headless L6 lane counts went from 335/296/0 (left-only) to ~111/190/138.
- **HP curve:** the 1.5 growth was first flattened to 1.15 and then, per the lethality blueprint, set to the final `RED_HP_GROWTH = 1.25` (red HP = `RT_HP x 1.25^(level-5)` from L5, cap 60000). L6/L9/L12 sampled max red HP: 4 / 7 / 14.
- **Speed scaling:** descent is now `RED_SPEED_BASE (122) x tier multiplier + level x 6` px/s (sandbox exempt), so L12 closes ~40px/s faster than L5. `RED_SPEED_CAP` raised 182 -> 260 so the ramp has headroom.
- **Stack surge:** door grind damage is `n` HP/s below 5 stacked reds and `n^1.5` HP/s at 5+, with the stack capped at 25 (25 reds = 125 HP/s). A full jam shatters a 125-250 HP door in roughly 1-2s (L6 doors broke ~1.6s after the jam formed, L12 ~3.3s) and floods the outer lanes.
- Balance note: base HP / TTK still untuned for the new speed, HP and density.

### Funnel: door pressure, HP and density (branch `feature/funnel-logic`)
- **Spill now requires a breach (pressure = damage):** the instant lane-hopping is removed (`trySpill` / `pressureAbove` / burst code deleted). `doorPressure(dt)` finds stalled reds (falling at < 35% of speed) stacked against a closed door; once 3+ are jammed it subtracts `1 HP/s per stacked mob` (capped at 20 mobs) from that door's HP counter, so the number drawn on the door counts down with the pile. At 0 the existing `breachDoor` fires and the pressurised stack is kicked through the gap: the only lateral spill left. Crush-angle lateral damage still applies as before.
- **Doors on every divider** (removed the 20% skip), so any lane's jam always has a counter to grind. Funnel lane is now fixed (`colN >> 1`: middle lane on odd boards, just right of centre on even), which is also where the first barricade lands, so the jam forms over a door.
- **Mob HP scaling:** from L5, red HP = `RT_HP x 1.5^(level-5)` (L6 x1.5, L9 x5, L12 x17, capped at 60000; L99 sandbox exempt). `rhp` widened to `Uint16`; new `rmax` per mob keeps wound colouring correct. Boss HP unchanged.
- **Spawn density +50%** (`SPAWN_DENSITY = 1.5`): wave interval / 1.5, pipe rate 28 -> 42/s, threat rate 14 -> 21/s, endless-pour rate x1.5.
- Headless check (cannon invincible, no fire): 100% spawns in the funnel lane; doors open after 4-8s of pile pressure (L6 doors 126/128 HP, L12 140/134/145), then the pile pours into the outer lanes; max red HP by level 6/9/12 = 5/15/51; no page errors. `__mob.get()` reports `pressureDmg`.
- Balance warning: the x1.5 compounding makes the x2 gun inadequate by ~L9 as intended, but by L15+ reds carry 100+ HP; base HP / TTK are not retuned.

### Funnel tuning: speed and pressure (branch `feature/funnel-logic`)
- **Slower descent:** `RED_SPEED_BASE` 187.5 -> 122 px/s and `RED_SPEED_CAP` 280 -> 182 (both -35%), so the player has time to carve the centre funnel before the stream reaches the cannon line. `PIPE_RATE` is unchanged, so the slower stream is denser.
- **Fewer barricades:** count per level is now `0` on L1-2, then `1 + floor((n-3)/8)` capped at 6 (was `n-1` on L1-2, then `2 + floor((n-3)/4)` capped at 12), i.e. roughly half or fewer at every level. Minimum height gap between any two barricades raised 54 -> 90px. The lane dividers and pressure doors are now the main friction. Blue-barricade threat streams scale down with the count.
- **Door-busting pressure:** a stalled red now spills only if at least `SPILL_PRESSURE = 3` reds are stacked within ~3 bodies above/behind it (`pressureAbove()`). When one breaches, the stalled mobs packed right behind it in the same lane are released on the next frames, so a jam bursts outward in a surge instead of a steady trickle. If the jam is smaller than 3 the mob waits.
- Headless check (L6/L12/L22, invincible cannon, not firing): spawns 100% centre lane(s); 460-810 spills per 40s; outer lanes still fill in bursts; no page errors. `__mob.get()` now also reports `bursts`.
- Balance note: base HP / TTK constants still assume the old speed and spread; a rebalance pass is still needed.

### Funnel logic (branch `feature/funnel-logic`, not yet on `main`)
- **Single-column spawn:** every red wave, threat-budget stream and endless-pour red now leaves the base from the centre lane only (odd lane counts: the middle lane; even counts: one of the two middle lanes, alternating per wave). The old multi-lane dispatch (`waveCursor` / `spread`) is gone. The single pipe carries the old combined rate (`funnelBoost` x2 on 3+ lanes; the pour rate is no longer divided by lane count).
- **Spill (lateral overflow):** a red whose fall speed drops below 25% of normal for 0.22s (blocked by a barricade or the mob below) checks the space beside it. If the adjacent lane has room at its height it hops over the divider and resumes falling; if both sides are full it retries after 0.12s; 0.6s cooldown per mob stops ping-pong. A backed-up centre lane now pours outward into the other lanes and chains across the board.
- **Barricades:** the first barricade always lands in the centre lane (3+ lanes); no two barricades share a height band (54px stagger, so no flat row across the lanes); the generator never plugs every lane, so at least one stays open for the spill.
- Headless check (cannon invincible, not firing, L6/L12/L22): 100% of spawns in the centre lane(s); 570-920 spills per 40s; the piles fill the outer lanes (e.g. L6 lane counts 111/161/26 at 12s). No page errors. `__mob.get().spills` exposes the counter in `#debug`.
- Balance note: base HP / TTK constants were tuned for the old spread dispatch and are untouched; expect a rebalance pass.

### Phase 15 - soft collisions and the L99 physics sandbox
- **Soft collisions:** the rigid separation (90% of an overlap per pass) is replaced by over-relaxed "soft resolving": ~20% of a light overlap per frame (3 passes of RED_SOFT/3), ramping up only for deep overlaps (> 55% of a body) so a pile compresses and flows like mud but cannot collapse into a blob. Sideways velocity also bleeds off faster (viscous).
- **Spawn jitter:** every emitted red gets +/-5px X and Y jitter, and spawns at its terminal fall speed so a 30/s stream is spaced ~6px instead of stacking on one pixel (the "shotgun blast").
- **Level 99 sandbox** (replaces the campaign L99): 1 wide (320px) column, no gates/doors/blues/crates/threats, one indestructible plug at y=470, a single pipe pouring 30 reds/s forever (until the 1000 pool is full). Win/loss disabled. Reach it with the Warp box (pause screen), `?level=99` or `#level=99`.
- Measured in the sandbox: spawn stream ~24px wide, median sideways speed ~23 px/s (p95 63) in the spawn zone; 1000 reds pile ~90px tall on the plug.
- Tests: 195 headless checks. Generic 150-level loops use L98 in place of L99; the 150-level soak was not re-tuned for this commit. Bot sims were not re-run.

### Phase 13 + 14 - dam physics, multiplier zones, pressure doors
- **UI:** the Phase 12 Main Menu is gone; the game starts immediately. Level/Warp stays on the Pause/Victory/Defeat overlay.
- **Swarm physics (Phase 13):** Brownian drift removed. Reds fall under gravity (1100 px/s^2 to a terminal speed) and are solved position-based: integrate, then 4 passes of mob-vs-mob separation (uniform grid, capped shove) + barricade tops + lane walls, then velocity = displacement / dt. A red that lands on a barricade really stops; the mobs behind it stack into a pile. Barricades are now full-width plugs (dams), so nothing slides round them.
- **Dam burst:** destroying a barricade removes the support; the whole pile accelerates down together.
- **Multiplier zones:** a Blue Barricade at 0 HP no longer bursts. It leaves a translucent x2 zone across its column for 10s; each blue passing through splits into two, once per bullet (mask bit 6).
- **Retired trapdoors (Phase 14):** the %-leak gaps are gone; dividers are solid.
- **Pressure Doors:** 1 per divider (2+ columns), HP 100-250 (by tier), 56px tall, placed just above a neighbouring barricade so the dam's pile presses on it. Colour shifts yellow -> orange -> red with cracks as HP drops. Damage only comes from a red pushed into the door with heavy lateral force (`crushHit`: |vx| >= 45 px/s and |vx| >= |vy|, i.e. > 45 deg off vertical), 0.3s cooldown per mob; free-falling mobs deal 0.
- **Breach:** a door at 0 HP is gone for the level; the stack that was pushing on it gets a sideways impulse and spills into the next lane; blue bullets can cross through the gap.
- Debug hooks: `setDoors`, `setCannonHP`, `crushHit`, `spawnZone`. Version label `v14 · c12`, cache `mob-swarm-v12`.
- Tests: 187 headless checks (dam stack, separation, dam burst, 1000-red frame cost 3.5ms avg, zone split/expiry, crush filter, breach spill, blue pass-through, no heap growth with 200 mobs through a breach).
- Sims (bot ignores doors, aims one column): bosses L10/20/30/50 4/4 (TTK 45-66s vs 45s target); L1, L10, L16, L20, L50 4/4; L6 (3 cols) 0/4 dies at ~13s. Not device-tested.
- Behaviour changes to watch: reds pile up so peak red counts are higher (soak bound raised to the 1000 pool); boss TTK and bot results changed (see sims).

### Phase 12 - aggressive pacing and threat budgeting
- **Level selector:** Level input + Warp button on the Pause/Victory/Defeat overlay, plus a new Main Menu (Start + warp) on load. `?level=N` still works but no longer needed (the SW stripped it). Debug hook `__mob.warp(n)`.
- **Columns:** L1-2 = 1, L3-5 = 2, L6-9 = 3, L10+ = 4.
- **Cleanup:** red pipe/chute graphics removed; L1-2 have no gates at all (strict 1.0x start, 9 shots/s, 1 pellet) and no supply crates; L1-2 spawn only gray barricades. First-wave grace 2s, base wave interval 5s (was 7s).
- **Threat Budget:** each Blue Barricade generates a red wave (size = round(hits x 1.0), 8-24) in an ADJACENT column. Released 10s after play starts, at 14 reds/s. Never generated on single-column levels. Boss levels get no blue barricades (the boss is the threat).
- **Anti-stacking:** at most one Blue Barricade per column per level.
- 4-column levels hit the 6-gate bit limit, so negative gates are trimmed there (known side effect).
- Tests: 163 headless checks. Sim (single-target bot): boss fights 4/4; L1 3/3, L6 1/3, L16 0/3 (too hard for a bot that covers one column), not device-tested.

### Phase 11 - discrete columns, trapdoors, barricades, level warp
- **Pivot:** the Phase 10 carved-lane terrain and open-field slide generator are replaced by a discrete column grid (1-4 vertical lanes, 8px dividers). L1-8: 1 narrow centre column (rest letterboxed/grayed); L9-16: 2; L17-24: 3; L25+: 4.
- **Cannon:** drags across the active span; shots are clamped into the column the cannon aligns with. Gates are bound to a column; every column has an `x` gate.
- **Red dispatch:** per-column queues and pipes. Waves alternate columns (1-2 columns at once on 1-4 cols, up to 3 at L41+). Boss waves come from the hull's column.
- **Trapdoors:** staggered breaks in the dividers (2+ columns). Reds crossing a gap leak sideways at 25-40% (`L.leakP`) into the neighbour column.
- **Barricades:** gray (absorbs), blue (shatters into blues upward), red (shatters into reds downward), with visible hit counters. Replace the old indestructible terrain.
- **`?level=N`:** dev warp, e.g. `index.html?level=20`.
- Balance tweaks from sims: red barricade bursts capped at 10, first-wave grace 3+2.5*(cols-1)s, big waves fan out (<=~16 reds per column).
- Known: bot-sim L16 (2 cols, 6584 HP) is still 0/3 for a single-target bot; boss TTK 53-79s vs 45s target. Not device-tested.
- Tests: headless suite now 148 checks, including level warp, column containment and the trackpad deadzone. Sims aim per column.

### Phase 8 - terrain, fluid pathing, supply crates
- **Terrain:** solid indestructible gray oriented rectangles (typed arrays, max 8). Circle-vs-OBB resolve pushes mobs out along the surface normal and strips only the velocity component into the wall, so blues and reds slide instead of bouncing. A minimum tangential slide speed (toward the nearer end of the slab) stops mobs stalling on flat faces.
- **Layouts in `generateLevel`:** central pillars, angled funnels (56px mouth) and zig-zag baffles, placed in the free bands above and below the gate rows (none on L1). Crowded boards drop overlapping bumpers rather than lose the terrain. HP derates 7% per piece.
- **Red pipe:** waves are queued and metered out (14/s) from one concentrated pipe at the top (fixed x per level), or from the boss hull on boss levels. Replaces the wide wall spawns.
- **Speed cap:** any red is capped at 280px/s and the per-tier speed multiplier is capped at 1.25x. Late tiers scale wave size and the Tank/Sprinter mix (Tank up to 45%, Sprinter up to 30%) instead.
- **Crowding:** reds push each other apart (grid broadphase, 2 passes/frame) and re-resolve against terrain, so a dense stream piles up at a choke point and overflows sideways.
- **Supply crate:** a 30 HP gray crate spawns every 15-20s, drifts down at 16px/s (sliding round terrain), absorbs blue impacts, and drops a pill when broken. Catching the pill equips Shotgun (3-way spread) or Piercing Rounds (3 HP per blue) for 8s via new Armory entries (`pellets`, `spreadVx`), then reverts to the equipped base weapon.
- Pool throughput cap lowered 600 -> 250 hits/s (terrain lengthens blue lifetimes, so the pool saturates sooner). Funnels no longer seal the screen edges, zig-zag and funnel slabs carry a preferred slide direction (toward the gap or mouth), and the upper band never gets an inverted funnel (it used to shove the whole stream to the edges, away from the boss).
- **Bot-sim:** `smartAim` now uses a debug `traceBlue` dry-run so the bot aims round terrain (the old lane-based bot stalled for 400s on many levels).

### Phase 7 - balance fixes and data-driven refactor
- **Throughput model replaces raw MTM.** Raw MTM overestimates badly (bot-sim measured efficiency falling from ~0.9 at MTM 2 to ~0.1 at MTM 700), so a flat 0.35 cannot hold TTK. HP now uses `throughput = min(9 * 1.05 * MTM^0.7, 600)` (efficiency curve + 1,500-pool cap), derated 8% per bumper and 15% per black hole. Base HP = throughput * 35 * (1 + 2% per tier); Boss HP = throughput * 45.
- **Fixed `calculateLevelMTM`:** a `+n` gate turns each mob into n+1, so it multiplies by (n+1); Phase 6 added n.
- **Mothership boss:** hull is 40% of screen width (144px), drifts at ~22px/s at the base line, hull extends below the shielded base so blues actually reach it (previously the base absorbed them first). Waves are sparse: 5-14 reds every 4.5-6.9s.
- **Spatial gate logic:** x3/+10 are narrow and high, x2/+2 wide and low, negatives wide and directly below a high gate, sliding opposite it. (`+5` replaced by `+2`.)
- **Bestiary:** `RedMobTypes` (Basic, Tank, Sprinter) with typed-array per-type lookups; reds carry a type id through all pool operations.
- **Armory:** `Armory` dictionary (`fireRate` ms, `bulletVelocity`, `piercing`) with standard / piercer / rapid presets; blues carry remaining pierce points.
- **Tiers:** `generateLevel` groups 5 levels per tier. Spawn rate, wave size, speed and enemy mix are constant within a tier; gate layouts randomize per level. T1 Basics, T2 +Tanks, T3 +Negative gates, T4 +Sprinters. Bumpers (L31+) and black holes (L51+) unchanged.
- HP scalar is now +2% per **tier** (was per level) so TTK stays near target.

### Phase 6
- Added `calculateLevelMTM(gates)` (Max Theoretical Multiplier). Base HP = `9 * MTM * 35 * (1 + 0.02 * (level - 1))`. Boss HP = `9 * MTM * 45`.
- Boss is now a hovering mothership at the base line. It strafes left/right and drops dense red waves aimed at the blue mass. It no longer marches down the lane.
- PWA: added `manifest.json` and `sw.js` (cache-first offline for `index.html` + `manifest.json`), manifest link and SW registration in `<head>`.
- Sims updated (`bot-sim.js`, `boss-sim.js`) and headless tests updated (all pass).
- KNOWN ISSUE: the sims do NOT confirm the 35s/45s TTK targets (see PROJECT_STATUS.md). Boss levels are 0/8 winnable by the perfect-aim bot.

### Added (Phase 5: procedural 100-level campaign)
- `generateLevel(n)` replaces the hardcoded levels: seeded PRNG so level N is always the same board. Base HP = 800 + 400n (L1 1,200 ... L100 40,800).
- Red spawn interval shrinks 2% per level (floor 2.5s); wave size grows +1 (min) / +2 (max) per level, hard-capped at 150 so two walls always fit the 500-red pool. Red speed +0.5%/level (max +50%).
- Tiered obstacles: L1-10 gates x2/x3/+5/+10; L11-30 add negative gates (`-10`, `/2`) sliding opposite the nearest multiplier; L31-50 add pinball bumpers (gray circles that reflect blues and knock reds back); L51+ add rotating black holes that pull in and destroy blues and reds.
- Boss every 10 levels: normal waves stop, one giant boss (HP = level x 100, 1 damage per blue impact) marches down the center lane; touching the cannon line is instant defeat. The base is shielded on boss levels; killing the boss wins.
- HUD shows "LEVEL X / 100" (+ BOSS tag). Next Level regenerates board, pools and gates in place; level 100 shows "Victory - Play Again" and wraps to level 1.
- Gates: up to 6 per level (mask bit 7 reserved for the bumped/sucked flag). Red pool gained knock-velocity arrays for bumper/black-hole physics.
- Tests: generator math/determinism/tiers, bumpers, black holes, bosses, level transitions, 150-level soak (67 checks). `test/bot-sim.js` and `test/boss-sim.js` for balance.

### Balance findings (not fixed, see BACKLOG)
- Base HP 800+400n gives pure-siege clear times of 69s (L1), 62s, 114s (L3), 21s, 61s, 24s (L7), 90s (L9) with perfect aim: the 25-40s target is not met because throughput depends on the random gate mix.
- Boss: L10 (1,000 HP) is won 20/20 but in ~1s; L20 and L30 are 0/20 at the default boss speed.

### Added (Phase 4: siege balance)
- Siege base HP: L1 1,000 / L2 3,500 / L3 10,000. Base strobes and shakes in proportion to hit rate, with a yellow chip-damage trail on the HP bar and thousands-separated HP text.
- Negative gates (red): `-10` is a tax that destroys up to 10 mobs and recharges at 5/s (charge bar shown); `/2` destroys every second mob. They slide opposite to the multiplier gate below them and play a dissonant tritone buzz. L2 has `-10`; L3 has `/2` and `-10`.
- Red shield: reds have 2 HP. The first blue hit turns a red orange, the second destroys it.
- Up to 4 gates per level; gate bitmask still fits in a Uint8.

### Balance verification (`node test/bot-sim.js`)
- Perfect-aim bot, no red spawner: L1 clears in ~58s, L2 ~57s, L3 ~167s. L3 therefore misses the 25-40s target by ~4x with 10,000 HP.
- With red walls active, idealised bots win L1 half the time and rarely/never win L2-L3.

### Fixed
- Cannon no longer auto-fires. It fires only while the player is touching/dragging and stops the instant the touch ends (also on pointer-capture loss and window blur). A 1-shot cooldown charges while idle so a tap fires at once but tapping cannot exceed the 9 shots/s rate.

### Changed (balance pass)
- Red waves are now dense clustered walls (staggered rows, ~2:1 aspect, emerging all at once, centered toward the blue cluster) instead of a single-file line.
- Wave sizes per level: L1 8-12, L2 20-25, L3 40-50 reds.
- Wave interval raised to 7.0s / 8.5s / 10.0s (was 3.0 / 1.9 / 1.1s) and first-wave grace set to 3s. At the old intervals, a 40-50 wall every ~1s would have been unwinnable (every leaked red costs 1 cannon HP).
- The red mass now drifts toward the blue mass as a single body, so walls keep their shape instead of collapsing into one column.
- Added `test/bot-sim.js` (idealised-bot balance simulator) and tests for fire gating, wave sizes/shape, wall integrity and 500-red/1500-blue stress timing.

### Added
- Cannon shield: 5 HP pips, red flash on hit; Defeat only at 0 HP (replaces instant defeat).
- Red waves funnel toward the blue cluster; red and blue masses attract laterally.
- 3-level progression (base HP 40/80/150, faster gates and spawns), level HUD, "Next Level" / "Victory - Play Again".
- Web Audio synthesized SFX (cannon pop, gate chime, collision click, hit thud, win/lose fanfares) and a mute toggle (persisted).
- Tests for shield, funneling, attraction, level progression.

### Changed
- Opponent base moved down 28px to make room for the HUD.

### Added
- Moving gates (sliding, bouncing, opposite directions) with rect-overlap trigger; no dead zones.
- Red enemy mobs (pool of 500) spawned from the opponent base, moving down at 0.75x blue speed.
- Blue-vs-red 1:1 cancellation using a zero-allocation spatial grid, with pop effects (pooled ring buffer).
- Defeat condition and "Try Again" overlay when red reaches the cannon line.
- Headless test suite (`node test/headless.js`).

### Changed
- `applyGate` parameter renamed to `mobIdx`; clones inherit parent gate bits.
- Debug hooks (`window.__mob`) only exposed at `index.html#debug`.

## [0.1.0] - 2026-10-05

### Added
- Repository created and initialized on branch `main`.
- Baseline architecture documentation (`README.md`): single-file, zero-dependency HTML5 Canvas game.
- Five-phase task roadmap (`BACKLOG.md`) and model handoff plan.
- Project status dashboard (`PROJECT_STATUS.md`).
- Phase 1 prototype `index.html`: canvas loop, draggable cannon, pooled mobs, multiplier gates, opponent base HP, Victory overlay with Restart.

[Unreleased]: https://github.com/BradR98/mob-swarm/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/BradR98/mob-swarm/releases/tag/v0.1.0

## Phase 9 - Trackpad zone, thumb cannon, entity scale, boss rebalance
- Bottom 15% (y >= 544) is a Trackpad Zone: nothing spawns, enters or renders there. Cannon line (`CANNON_Y`) sits on its border; touch works anywhere on the canvas (X-only mapping).
- Cannon redrawn as an upward crescent cradling a thumb glow.
- Global entity scale `ES = 0.82` (mobs, reds, gates, crates, pills, boss, terrain, bumpers, holes); gate rows re-spaced.
- Boss rebalance: waves 3-12 reds (0.35 x 0.4 of level wave), interval 16-24 s, pipe 7 reds/s (level pipe 14/s).
- Sim (idealised bot that never shoots reds, N=20, full fight): L10 20/20, L20 18/20 (was 2/10 and 1/10 before tuning). Pure boss TTK is 57 s (L10) / 86 s (L20) vs the 45 s target - not recalibrated.
- Headless: 131 PASS / 0 FAIL (trackpad boundary, render arcs, scale, boss pacing, input checks).

## Phase 10 - Negative-space terrain, true lanes, swarm density, pause (v10 / cache c9)
- Terrain rewritten: massive edge-hugging blocks carve continuous lanes. Layouts: Bridge (80px neck), Fork (central island), S-Curve (interlocking peninsulas) + edge rails. Placed in the free band above the gates (or below them when 4+ gates). Bumpers/holes keep clear of lanes.
- Gate slots moved up 26px (354..164) to give the lower band room for a lane layout.
- A bridge ramp must not be a single thick slab: it poked into the opposite ramp and trapped 1500 blues in a crevice (found by sim, L6 TTK 300s). Ramps are now a thin slab + stacked strips.
- Reds no longer track the blue mass; each random-walks sideways (Brownian drift, 55px/s cap). Sloped faces: reds slide downstream, blues upstream.
- Swarm: red pool 500 -> 1000, MAX_WAVE 300, pipe 14 -> 28/s, boss pipe 7 -> 14/s, Basic hp 2 -> 1, Tank 5 -> 3.
- Pause button (top right, left of mute): freezes the rAF loop (cancelAnimationFrame), PAUSED overlay with Resume; also auto-pauses when the tab is hidden.
- Throughput derate per layout (bridge 0.9, fork 0.8, s-curve 0.75; x0.75 on bosses) from bot-sim.
- Headless: 137 PASS / 0 FAIL, incl. independent 2px BFS proving a >=60px path exists on levels 1-150, row scan min open span 80px, bridge neck 80px.

## Phase 15.1
- fix: Level 99 sandbox cannon now fires (the sandbox had wrongly disabled player fire). Version v15.1 · c14.

## Phase 16 - Level 1 baseline & Tier 1 scaling (v16 / cache c15)
- Soft collisions (RED_SOFT 0.2, RED_ITERS 3) and spawn jitter were already in the shared red solver / emitRed, so they apply to every level.
- L1-8 open with ONE sustained pour instead of random waves: L1 = 500 reds @ 45/s (one column, no gates, no barricade); +8% reds and +1.5/s per level (L8 = 780 @ 55.5/s). Reds fall slower early (x0.4 at L1 -> x1.0 at L8) so the river is dense. L9+ keep the old wave system.
- Lawnmower weapon (Armory.standard): 111ms -> 50ms (9 -> 20 shots/s), piercing 1 -> 3 (a blue cleaves 3 red HP), 110px/s random sideways spray (a held trigger sweeps the whole lane). SHOTS_PER_SEC 9 -> 20 so base HP formulas keep the same perfect-stream time-to-kill.
- Tier 1 layout: L2 gets one destructible gray barricade, 2nd column at L3 (3rd at L6), barricades sit low in their free band.
- test/pour-sim.js: perfect-aim bot over L1-8. New debug hook clearReds. Headless: 200 PASS / 0 FAIL; mechanic tests run on a legacy 1.0x weapon, Phase 16 tests assert the lawnmower.

## Phase 17 - Infinite pours & the 2% attrition scale (v17 / cache c16)
- L1-8: the pour never ends (queue refills below 60); only the base HP reaching 0 ends the level (already an instant win).
- Weapon: 45ms (22.2 shots/s), cleave 3 -> 2, same 110px/s spray. Theoretical max 44 kills/s vs the 45/s L1 pour; measured, the swarm grows slowly (about 55 -> 150 reds over ~34s).
- Finding: with an infinite pour the blues are almost all spent on reds, so only ~1-3 leak to the base per second. 735 base HP was unreachable (cannon died at ~30s). Gateless pour levels (L1-2) now have a BASE_HIT of 15 per leaked blue and need ~50 leaks (HP 750 on L1); gate pour levels take base HP x0.6.
- Attrition: pour rate = 45 x 1.02^(level-1) (L8 = 51.7/s). Boards with more columns pour 1/(1+0.3 x (cols-1)) per lane, because one cannon guards one lane (L6 was 3/10 without it).
- test/pour-sim L1-8 x10: 80/80 wins, no cannon damage. Headless suite green.

## Experiment - perspective tilt (branch feature/perspective-tilt, NOT on main)
- Physics world drawn on #game inside #tilt (CSS perspective + rotateX, origin bottom, top fades out); HUD, base bar + HP, trackpad strip and hit flash on a flat #hud canvas. All collisions stay 2D.
- Touch input is read from the flat #hud canvas, so cannon X stays 1:1 with the finger.
- Tuning via query string: ?tilt=45 (deg, 0 = flat) &persp=800 &stretch=1.9 &level=N.

## Phase 18 - HP rebalance, locked tilt, merge (v18 / cache c18)
- HP curve shifted down: gateless pour levels = 250 x 1.02^(n-1) (L1 = 250, L2 = 255); every other base HP and boss HP x 1/3 (HP_SCALE). L1 bot win time about 33s -> 17s (10/10).
- Perspective tilt locked to the approved preview (45deg, 800px, 1.9x stretch), query overrides removed; merged `feature/perspective-tilt` into main (flat #hud overlay + flat input preserved).
- ARCHITECTURE.md handover added.

## Docs refresh (no version change)
- README rewritten for v18 (split rendering, endless pour, weapon, PWA, dev tools, testing). PROJECT_STATUS refreshed. Player gameplay guide added to the backlog (P18-3).
