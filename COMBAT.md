# COMBAT — what post-mortems teach about fighting

Combat splits into two parts:

- **micro**: what each unit does turn by turn near enemies (where to step, whom to hit, when to run);
- **macro**: when and where armies fight (rushes, harassment, sieges, defence, all-ins).

Post-mortems from 2009–2026 agree on a small set of laws, and on a standard micro "kernel" that almost every
top team converged on. This file collects them, keeping only what carries across rule sets. Sources are
cited as `[year team]`, and a source list is at the end. The companion files are ECONOMY.md, NAVIGATION.md,
EXPLORATION.md and SYMMETRY.md.

---

## 1. Laws that held every year

1. **Micro has the highest marginal return.**
   - "A slightly improved macro strategy might increase our win rate by 5%, but micro can do a 30–50%
     increase" [2023 Gone Fishin', 2nd].
   - Combat-code improvements beat older versions 50–0 and 31–0, where navigation, economy or strategy
     improvements gave about 28–22 [2012 fun gamers, 1st]. Their priority order: attack micro > swarm
     cohesion > navigation > high-level strategy.
   - Macro still decides when armies are unequal: "Even with the best micro, a single soldier cannot stand up
     to a mass of 10" [2017 E Doc Tablet]. Get both to "good", then grind micro.
2. **Concentrate. Fights are strongly non-linear (Lanchester).** A simulated force-ratio table
   [2013 devs lecture]:

   | Unit advantage | Better kill ratio |
   |---|---|
   | 5% more | ~20% |
   | 10% more | ~40% |
   | 15% more | ~80% |
   | ~33% more | you lose nobody |

   "Never engage when outnumbered unless next to your base." Never trickle units in one at a time: teams that
   lost their clump and then sent reinforcements singly lost [2013 devs]. "Fighting one-by-one loses"
   [2014 schnitzel].
3. **Model the exact turn order and cooldowns; the asymmetry decides who wins a duel.** It cuts both ways:
   - If attacks resolve *after* movement in the same turn, **the gap closer wins**: whoever steps into range
     first hits first, and the other cannot escape [2013 Cory Li lecture, devs].
   - If a unit can attack *and then* move, or attack range is close to vision, **the first to move into range
     dies**. Let the enemy walk into you [2019 smite, 1st; 2014 schnitzel; 2014 Darkpurple].
   - With separate move and attack cooldowns (for example, attack every turn but move every other turn),
     kiting lands about 3 hits for every 2 taken [2023 Gone Fishin'], or 2 hits per hit taken against static
     targets [2025 JWU, Om Nom, SPAARK].

   Derive which regime you are in from the engine, not the spec prose, and write it down.
4. **Gaps between one unit's reach and another's awareness are exploitable.**
   - Attack from outside the target's vision: "assassinate the soldier from the shadows"
     [2022 5 Musketeers].
   - Attack static defences from just outside their range [2025 confused].
   - Keep firing at a last known location after backing out of sight. The enemy stops firing because it can't
     see you, "giving a huge combat advantage" [2017 Segfault].
5. **Defence is hard, attack is easy.** "Hence spam and rush bots tend to always surface in Battlecode"
   [2021 Stone Tao]. Every season had a rush meta early that faded after balance patches or better
   defences. Rush strategies that win sprint 1 rarely survive to the finals [2025, 2026].
6. **Commanded coordination flops; emergent coordination works.** "Every time I'd implement a sophisticated
   strategy that requires a lot of coordination it would always flop" [2020 Java Best Waifu, 1st]. What
   works instead: spawn order, repulsion, shared target slots, and local rules every unit applies alone.
7. **Start with the dumb version.** "Don't try to invent Battlecode Stockfish without having the basic dumb
   'run straight at the nearest thing' bot done" [2023 don't @ me]. "Just taking all the game constants at
   face value is enough to be competitive with top teams" [2025 SPAARK].

## 2. The micro kernel: score every candidate move

Almost every top team since about 2013 evaluates each of the (up to) 9 candidate moves, including staying
put, with a small struct of features, and picks the best. Attack targeting is usually a separate, simpler
step. Sources: [2013 devs], [2016 future perfect], [2019 smite], [2021 Malott/XSquare style, copied by many],
[2022 smite], [2023 Gone Fishin'], [2025 SPAARK, Om Nom, Kragle], [2026 food, GST].

```
microTurn():
    enemies = sense(); if none and no remembered threats and hp ok: return false   // fall through to macro
    best = null
    for d in [CENTER] + 8 directions:
        if d != CENTER and !canMove(d): continue
        m = MicroInfo(here + d)
        for e in enemies:                                   // threat side
            if e.canAttackNextTurn(m.loc): m.dangerCount++; m.expectedDamage += e.dps
            m.minDistToEnemy = min(m.minDistToEnemy, dist(m.loc, e))
            if canAttackFrom(m.loc, e): m.canAttack = true; m.bestTargetValue = max(..., value(e))
        for a in allies near m.loc: m.support += a.canHit(sameEnemies)
        m.terrainCost = cost(m.loc)
        if best == null or m.isBetterThan(best): best = m
    if best.canAttack and attackFirstIsBetter: attack(); move(best); else move(best); attack()
    return true

isBetterThan(a, b):                                       // lexicographic, from the top teams
    if a.inLethalRange != b.inLethalRange: return !a.inLethalRange
    if weaponReady: prefer canAttack, then fewer dangerCount, then higher support
    else:           prefer fewer dangerCount (kite), then larger minDistToEnemy
    then lower terrainCost, then closer to the macro target
```

Variants worth knowing:

- **Single-currency additive score** [2025 SPAARK, HS 1st]. Convert everything into one unit (5 score = 1 of
  the resource).
  - Desired direction: +20 for the exact direction, +15 for each neighbour.
  - Terrain: −5 per resource lost to bad terrain.
  - Allies: −5 per resource lost to ally adjacency.
  - Enemy defences: a large minus inside enemy defence range, larger if you could be one-shot.
  - Enemy units: −20 inside an enemy's move-plus-attack range.
  - Anti-oscillation: −1 for recently visited tiles.

  Attack targets are scored in the same currency. The 2026 version of the same team weighted self HP and
  enemy HP equally, and scaled the resource weight inversely with how much the team had
  (`k/(bank + c)`) [2026 SPAARK, code].
- **Jointly evaluate move then act.** For each candidate move, compute the best action available *from that
  tile*, so "step then hit" competes fairly with "hit then step" [2026 SPAARK, 2025 SPAARK]. Many frameworks
  make one order implicit; refactor so both are tried [2025 SPAARK]. When orders matter: if moving away from
  the fire direction, shoot first then move; if moving toward it, move first, then shoot, to avoid walking
  into your own projectiles [2017 AGRF].
- **Expected-damage minimisation** [2019 smite, 1st]. For each move (including staying), sum
  `damage(enemy)` over enemies whose range band (min ≤ d² ≤ max) covers the tile. Area attackers get a little
  slack. Pick the minimum. Non-combat units use a "hide" variant: only tiles outside every enemy's *vision*.
- **Distance bands instead of simulation** [2013 Teh Nubs, 1st]. Count enemies at "1 step", "2 steps" and
  "3 steps" (by squared distance). Count a unit fully only if it can act within 3 rounds, and others as 0.2.
  Count the same bands around the closest enemy and the closest ally. Rules:
  - adjacent enemy: fight;
  - no enemy within 3 steps, or allies ≥ 3× enemies: advance;
  - closest enemy at 2 steps: advance if it is already engaged, or our 2+3-step count exceeds theirs + 1;
    otherwise step back.

  This is cheap and gives most of the value of simulation.
- **Ideal range per enemy type** [2017 Omega Ruby, 2nd]. Penalise `(d − ideal(type))² · weight(type)`, plus a
  huge penalty for any tile a projectile will cross. Hold ranged duels at just beyond the range where shots
  can't be dodged.
- **Attract/repel potentials per type** [2017 AGRF, 1st]. Use `+a/(d+1) − b/(d²+1)` (attract at range, repel
  up close). Raise b when hurt ("coward mode"). Add −1000 inside a melee unit's strike radius, and a small
  pull toward allies' centre. The potential is used as the secondary key after safety.
- **Feature lists that mattered** [2022 smite notes]:
  - enemies that can attack the square, their HP, and whether they can move;
  - allies weighted by how much their attack area overlaps the square;
  - healing benefit;
  - terrain on our square and on enemy squares;
  - enemies hittable from the square, preferring a high `myHP − enemyHP`.
- **Directional vision and facing** [2026 food]. With vision cones, penalise tiles orthogonally adjacent to an
  enemy. It has more approach tiles than you can face, so it can always reach your blind side. Evaluate both
  "face where I'm going" and "face the threat", and prefer ending the turn facing danger.
- **Influence maps** [2018 Orbitary Graph, 1st]. Stamp ~20 named kernels at unit positions ("enemy ranged
  threat", "healer proximity" and so on). Compose them arithmetically into per-type target and cost maps,
  then pick moves by value/(cost+1).

Implementation lessons:

- **Make the kernel cheap.** Fully unrolled with templates, with no objects (field access costs an extra load)
  and no calls, the kernel came out about 2x cheaper and needed no bytecode checks [2025 Om Nom].
- **Trigger micro on** enemies visible, low HP, or remembered recent enemies [2025 Kragle].
- **Skip micro when the enemy is unreachable** (behind a wall). Use an exhaust timer: no action for N turns →
  leave combat [2026 food, 3Mice].
- **Refresh state after every action.** One team computed attack range from a stale pre-move object, so melee
  units stepped in and waited a turn to attack, and died [2018 smite].

## 3. Engagement decisions: fight, hold, or leave

- **Local numeric superiority.** Count enemies that can hit you against allies that can hit the closest such
  enemy. Stay if allies > enemies, or if equal and (more than one enemy, or a 1v1 with HP ≥ theirs);
  otherwise retreat [2016 future perfect, 1st].
  `guessIfFightIsWinning`: allies within range ≥ {2, 4, 5, 1.5n} for n = {1, 2, 3, ≥4} enemies
  [2014 that one team, 1st].
- **Exact 1v1 cooldown race** [2016 Polar Vortex, 2015 the other team]:

  ```
  myTurnsToKill    = turnsUntilWeaponReady + attackDelay * ceil(enemyHP / myDamage - 1)
  theirTurnsToKill = theirTurnsUntilReady  + theirDelay  * ceil(myHP / theirDamage - 1)
  fight iff myTurnsToKill <= theirTurnsToKill            // adjust by 1 for the turn spent moving
  multi-enemy: kill them one at a time, accumulating the DPS of those still alive; accept if damage taken < 0.7*HP
  safe tile = outside static defence range and in range of at most one enemy that we beat 1v1
  ```

- **Strength estimate over a wider area.** Sum heuristic strength of allies and enemies from local sensing
  plus shared sightings ("extended radar"). Advance if stronger; otherwise kite or retreat
  [2012 fun gamers].
- **Contextual aggression** [2026 food]. Halve the enemy-proximity penalty when allies outnumber enemies 2:1.
  Invert it (charge) when a high-value ally is under threat. Their theory: passive beats aggressive locally,
  because the aggressor walks into range and traps, but passive against passive stalls and costs map control.
  So be aggressive only with a numbers, position or economy edge, and trade units when your economy is better.
- **Chase rule** [2023 Gone Fishin']. Chase a visible but out-of-range target only if nothing else is
  attackable, or the target can't shoot back, or you have 2+ more attackers than the enemy locally. Never chase
  a kiter with an equal unit [2012 fun gamers].
- **Step-in discipline** [2024 top-bot replay study; matches 2019 smite]. Don't make a lethal step into range
  while wounded unless the strike kills. Hold a ready attack and let the enemy step into you. A strong cycle:
  **strike → leave reach → hold one step outside reach → strike** when ready again. Holding there is safe only
  with supporting allies nearby.
- **After losing sight of enemies, move to just outside the vision radius of the closest remembered enemy.**
  The enemy must then step into your vision to advance, which gives you the first shot. This beat "wait N
  turns" [2023 4 Musketeers].
- **Hysteresis in state.** Use explicit states (OFFENSIVE / DEFENSIVE / RUNAWAY / RUSH) with restricted
  transitions (RUNAWAY → DEFENSIVE only), so units don't dither [2016 what_thesis]. Use stances (AGGRESSIVE /
  DEFENSIVE / SAFE) chosen from the macro goal and the distance to the rally point [2014 that one team].
- **Retreat rules must not be baitable.** An opponent advanced one unit at a time to trigger a team's retreat
  rules and herded its whole army back to base [2014 Darkpurple].

## 4. Kiting and cooldown dancing

- **Canonical skeleton** [2016 future perfect, 1st]:
  1. If the weapon is ready and a target is in range, step away from adjacent slow melee enemies first, then
     shoot.
  2. If the weapon is not ready, back up to keep maximum range: choose the move maximising the minimum
     distance to attackers, preferring orthogonal moves, triggered when the closest enemy is near.
  3. Otherwise hold position.

  Kite slow enemies. Don't try to outrun faster ones: stand and fight.
- **Kite back after every attack, unconditionally** [2023 Gone Fishin', and that year's sprint winner]. Attack,
  then step to the tile that **minimises the number of enemies that can see or reach you**. Remember where the
  enemy was, and step back in when ready. Conditional kiting (only when a skirmish evaluation said so) lost to
  "always kite". Check the premise first: kiting helps only when the enemy can die and cooldowns are
  asymmetric. When damage dealt *is* the score, the advice inverts.
- **Kite while reloading** [2022 camel_case, code]. If the visible target can attack and our weapon isn't
  ready and we're damaged, move to safety: the tile maximising the sum of squared distances to visible
  attackers, among tiles with acceptable terrain.
- **Step onto better terrain while attacking.** Before hitting a non-structure target, step to an adjacent tile
  with strictly cheaper terrain that keeps the target in range [2022 camel_case]. Shoot from low-cost terrain;
  never retreat onto worse terrain [2022 5 Musketeers, 2022 Baby Ducks ideas].
- **Synchronise attackers against single-target defences.** Several attackers enter range on the same turn, so
  a structure that can hit only one per turn wastes its shots [2025 JWU].
- **Tie-break orthogonal over diagonal** when distances are equal. Diagonal steps go farther in Euclidean
  distance and break formation [2023 Gone Fishin'].
- **Retreat in a direction that keeps you facing the enemy** when facing matters. Backing off diagonally can
  keep the enemy in your arc while you leave theirs [2012 fun gamers].

## 5. Targeting and fire control

- **Kill efficiency first.**

  ```
  for e in attackable enemies:
      hitters(e)  = allies that can hit e this turn (+1 for me)
      turnsToKill = ceil(e.hp / (hitters(e) * dmg))
  attack argmin turnsToKill, prefer one-shot kills; tie-break by threat (lowest action delay), then value
  ```

  Variants:
  - minimise `enemyHP / (1 + allies able to hit it)` [2014 that one team];
  - minimise turns to kill [2023 Gone Fishin'];
  - maximise DPS per remaining HP, `attackPower/(health·attackDelay)` [2016 future perfect].
- **Type priority tables.** Kill what shoots back first, then production, then the win-condition structure:
  "Soldier > Laboratory > Sage > Watchtower > Builder > Archon > Miner" [2022 camel_case]. Within a class,
  lowest HP, unless it is below one hit and overkill would be wasted [2025 SPAARK]. Also prefer:
  - targets that haven't moved since first seen (being built, or stuck), ×2 [2017 AGRF];
  - carriers whose payload matters over empty ones [2020 practice-repo replay study];
  - helpless targets: economic buildings > construction in progress > lowest HP [2014 that one team].
- **Score actions on a common value scale.**

  ```
  score = 100 * (kills − 2) + (enemyResourceDestroyed − ownResourceSpent)
  // kills weighted by type: high-value attackers ×2, bases ×10
  act only if score > 0 (except to protect key allies)
  ```

  This prioritises multiple kills but never uses a huge unit on two tiny ones [2021 Baby Ducks, 1st]. Some teams
  converted unit count into the currency through an exchange rate proportional to income, and judged every
  action by `Δcurrency + rate·Δunits`. That alone avoided bad trades [2021 wololo].
- **Area attacks.**
  - Score each aim point by +10 per enemy, −10 per ally, and a small bonus per unseen tile. Fire if the score
    is above a threshold [2019 smite].
  - Rewarding unseen tiles once made a team hit its own base just outside vision [2019 Double J].
  - Fire only if the weighted sum of enemies and enemy-controlled tiles exceeds the action's cost
    [2025 confused].
  - Swing only if it hits ≥ 2 targets, checking both before and after moving [2025 JWU, Om Nom].
  - Splash damage that is split evenly among all units in range can be **diluted**: cheap bodies next to your
    base soak a blast aimed at it [2021 waffle].
- **If you can attack, attack, even blind.** When an attack would otherwise go unused, fire at:
  1. an enemy reported in comms;
  2. the last seen enemy location;
  3. the tile toward the enemy base.

  [2023 Gone Fishin', 4 Musketeers, no thoughts] Always attack, even when idle [2024 Honey Ducklings].
- **Line-of-fire checks.** Fire only if a sampled line cast's first hit is an enemy [2017 AGRF]. Compute the
  hittable angular cone minus allies and obstacles in front [2017 BTC]. Make sure no ally is adjacent to the
  impact point [2015 the other team].
- **Spread versus single shots.** Use spread shots when close or outnumbered, and when rich enough; single
  shots otherwise [2017 AGRF, Segfault]. Cycle the aim slightly left/centre/right/centre ("flickering") so the
  target can't dodge [2017 E Doc Tablet]. Very fast projectiles can snipe beyond vision, at friendly-fire risk.
- **Focus fire through shared memory.** Units read posted target IDs and shoot one nearby, or post their own
  [2014 Darkpurple, 2019 Shadow Priests]. Use one shared slot cleared only when confirmed stale, not every
  turn.
- **Guided munitions.** If projectiles are separate units with tiny budgets, the launcher does all the
  thinking. It writes `(targetID, dx, dy)` into a channel keyed by the projectile's spawn cell, and the
  projectile reads it on birth [2015 the other team].

## 6. Retreat, healing and survival

- **Healing state with hysteresis** [2014 that one team, 1st]. Enter when HP < 20%, or when (attackers able to
  fire within 3 turns + 1) × damage ≥ HP. Leave when HP > 80%. Healing units flee any enemy with more HP and
  shoot only when the weapon is ready.
- **Retreat thresholds near death only.** Retreating at the first scratch lost more damage than it saved.
  Retreat at critical HP, and not when deep in a fight you're winning [2023 Gone Fishin', camel_case].
- **Retreat and heal as a group.** If 3 of 4 units peel off, the healthy 4th dies alone. "If any nearby ally
  needs to retreat and you're not clearly winning without them, retreat together" [2023 4 Musketeers].
- **Retreat direction.**
  - Sum the unit vectors from each visible enemy to you, quantise to 8 directions, and try ±1..±4 rotations
    while skipping static defence range. Take the first move that leaves every enemy out of range, else the one
    maximising minimum enemy distance [2014 that one team].
  - Ban directions toward a nearby map corner, so you don't get cornered [2016 future perfect].
  - **VIP flee arc** [2012 fun gamers]. Score each of 8 directions (+1 per armed enemy, −1 per armed ally,
    plus a wall term), smooth by adding the neighbours (a circular convolution), and flee toward the centre of
    the widest low-score arc. Count disabled enemies too: they can be revived at once.
- **Escape a straight-line chaser** by moving perpendicular to its path while still increasing distance. That
  gives two chances: it turns, or it passes by [2026 food].
- **Spawn blockers behind a fleeing VIP**, or cheap decoys every few hundred rounds. One team's decoys escaped
  chasers ~85% of the time [2016 future perfect, victorious-secret]. Drop obstacles behind a fleeing slow unit
  [2026 3Mice].
- **Die productively.**
  - If damage this turn would kill you, charge the enemy centroid, or move away from allies if death harms
    them [2016 future perfect].
  - A carrier that can't get home throws its cargo at the attacker as damage [2023 4 Musketeers,
    don't @ me].
  - Self-destruct when `Σ min(enemyHP, dmg) − dmg × adjacentAllies > 1.5 × myHP` (or > myHP if about to die
    anyway), using precomputed charge-path tables. Use a shared lockout channel so several units don't
    detonate at once [2014 that one team].

## 7. Groups, formations and coordination that works

- **Spawn order is formation** [2023 Gone Fishin']. Units act in creation order. Spawn the first group in a
  square with the earliest-spawned (first to act) *farthest* from base, so outer units move first and inner
  units never see them as walls. That alone won ~2/3 of games against an identical bot.
- **Grouping without leaders** [2023 Gone Fishin']. Lowest-ID leader-follow failed: followers blocked the
  leader. What worked: for a few turns after a fight, rally on the visible ally closest to the enemy base. When
  adjacent, the lower-HP unit stops and the higher-HP one continues, so wounded units end up at the back.
  Squads of ~5 that merge but never split [2014 schnitzel]. Gather until a threshold count, then advance
  [2015 Ayyyyyyyylmao, 2018 notes].
- **Swarm physics (boids)** [2013 devs]. A spring toward the waypoint, with weight by distance band;
  repulsion from the nearest ally; attraction or repulsion by enemy count; spread out when area attackers are
  seen. The base reshapes the cloud by changing the force profile, for example a ring at radius r. Counting
  allies and enemies per candidate cell and moving to "most allies, fewest enemies" produced a concave that
  collapsed on an equal enemy army and beat it decisively.
- **Surround ("squishing").** Units at the back of a ball deflect sideways so more weapons reach the front
  [2017 E Doc Tablet]. Concave attack squads appeared among several finalists [2019].
- **Lattices for defence.**
  - Checkerboard ((x+y) even) slots assigned by the spawner, spiralling out and skipping resource tiles and the
    ring next to the structure.
  - Switch contested areas to a **dense lattice** with no buildable or passable gap for the enemy
    [2019 smite, Double J].
  - A sparse guard lattice also works as a scouting net [2021 naalit].
  - A dense friendly checkerboard lets your own units pass diagonally.
  - Guards circling a base on a ring at fixed spacing ("electrons") naturally balance coverage [2021].
- **On-call reserves.** Defenders stand by and sprint to a broadcast location when called, then return
  [2019 Shadow Priests]. A "high-priority attacker spotted" channel pulls units within a radius
  [2017 AGRF].
- **Distress without oscillation.** Pulling soldiers home whenever an enemy flickers into base vision
  oscillated. Clustering sightings (EXPLORATION.md §7) fixed it [2022 5 Musketeers].
- **Reinforce or regroup.** Decide explicitly whether reinforcements join a lost battle or regroup at base.
  Recall everyone if the base is attacked [2013 devs].
- **Multi-pronged pressure.** Each attacker goes to its own closest objective rather than one blob
  [2015 Ayyyyyyyylmao]. Raiders flank along map edges so they reach targets from several directions; the
  planned upgrade was to converge the flankers simultaneously [2021 Stone Tao]. While armies fight, a small
  raid hits a lightly held objective elsewhere [2024 top-bot replay study].
- **Use enemy bodies as cover** against projectiles: stand behind an enemy unit relative to the shooter
  [2017 Omega Ruby].

## 8. Macro: harassment, sieges and all-ins

- **Hit the economy and production, not just the army.**
  - Early harassers deny enemy-side resources [2019 smite].
  - Raiders hunt gatherers and economic units [2017 E Doc Tablet, 2021 Stone Tao].
  - Kill production before the main target: fewer defenders, and new spawns sit idle [2020 confused].
  - Ordered priority lists make readable rush logic (move adjacent to the target → destroy the producer →
    destroy the base → …) [2020 confused].
- **Harassment that blocks movement is as valuable as kills.** Circle the enemy base just outside known
  turret range (a threat overlay; see NAVIGATION.md §6) and trap units inside [2020 Java Best Waifu, 1st].
- **Mass arrival beats per-turn kill caps.** A defence that kills one unit per turn can't stop many arriving
  at once ("the crunch"). Estimate the swarm ratio needed to saturate a defence: about 10 attackers per
  defensive turret in one year [2020 Java Best Waifu, smite].
- **Base camping.** Crowd the enemy base to kill each new spawn and block spawn tiles. If the base hurts
  nearby units, stand one step outside its range. Rotate around the base so the far side is covered and slots
  open for newcomers [2023 4 Musketeers].
- **Timed all-ins against deadlines.** Attack when your economy peaks, or before an enemy timer completes
  (`moveOutRound = deadline − travelTime`) [2013 Teh Nubs, devs]. Schedule waves: ignore turret danger from
  round X, send a fortified second wave at Y [2020 Java Best Waifu].
- **Close the game when ahead** before a hazard escalates or a tiebreak gets uncertain [2016 future perfect].
- **Exhaustion attacks.** When defenders spend a resource to shoot, cheap high-HP units can drain it before
  the main attack [2019 Chicken, Double J].
- **Body-blocking walls.** Units can't share tiles, so a ring of hard-to-kill units around a base can't be
  breached without area weapons or terrain changes. Check for this pattern whenever units occupy tiles
  [2020 confused, Bowl of Chowder, smite].
- **Static defences.** Place them before the threat. Bodies delay a rush; fortifications break it. Space them
  apart (no overlap) and put them at chokepoints near the centre. Build them conditionally on a measured
  trigger, so they cost nothing where the threat is absent [2020 The High Ground, 2025 JWU].
- **Break stalemates.** Switch targets periodically [2024 Honey Ducklings]. Have a fallback target list:
  nearest danger, then known enemy base, then possible enemy base, then wander [2022 camel_case].
- **Anti-turtle code may never be used.** One winner built a "detect turtle, surround, mass charge" mode and
  never used it in the tournament [2016 future perfect]. Build what the field actually does.

## 9. Defence against rushes

- **Detect early.** Every unit reports enemy sightings to the base, so rushes are seen before they arrive
  [2019 Justice of the War]. A forward observer screams back when a rush leaves [2019 smite].
- **Respond proportionately and in time.**
  - Build long-range defenders while the threat is far and short-range ones when it is close.
  - Units near the base converge on the enemy's approach point.
  - Spawn in an alternating left-right pattern to spread out [2019 Justice of the War].
  - Spawn units toward the attackers to body-block, and move the VIP away [2026 food, Lorem Ipsum].
  - "Just spawn more units when in danger" was enough against every rush bot one team met [2026 Lorem Ipsum].
- **Cover every threat type,** not just the one testing showed strongest [2019 Double J]. Check that the
  response can engage the threat. Track the largest enemy unit size seen, so guards are sized to kill it: when
  unit size is variable, defences tuned to the minimum get exploited [2021 Stone Tao, naalit].
- **Cap and time-box emergency responses** (see ECONOMY.md §3b).
- **Use the base's own fire** and stay near friendly structures, which heal or shoot [2023 4 Musketeers,
  2015 the other team].
- **Defend starting production.** Losing the first income structure was unrecoverable in one year
  [2025 JWU].

## 10. Neutral hazards and third parties

- **Steer neutral hostiles into the enemy.** If a neutral faction targets the nearest unit, a fast unit can
  lead it toward the enemy base. Check relative speeds. This was the dominant creative strategy of one finals
  [2016 future perfect, foundation, what_thesis]. Kiting the neutral at the enemy did *not* work in another
  year [2026 3Mice]. Test it.
- **Avoid them when you can't use them.** Stay out of the hazard's vision, or at least out of its immediate
  forward path. Enemies take priority over hazards, being smarter and more predictable [2026 GST, food].
  Treat a hazard that hasn't moved as a wall and ignore it [2026 3Mice]. Don't broadcast near hazards that
  home in on signals [2026 food].
- **Score the hazard.** If damage to the neutral counts in the tiebreak, farm it deliberately [2026 food,
  Lorem Ipsum].

## 11. Projectiles and continuous space

- **Sampled dodging under a bytecode budget** [2017 AGRF, 1st]:

  ```
  prefilter bullets that cannot reach bodyRadius + stride this turn
  candidates = [toward target, tangent around nearest unit, last turn's best relative move]
             + random directions at stride (80%), half stride (10%), tiny (10%) while bytecode > 3000
  primary key  = −estimated damage:
      will hit this turn → full damage
      else 0.5 * (R − perpendicularDist + margin) * damage / (turnsToImpact + 1)   // urgency decays
      ignore bullets already blocked by an obstacle (cached raycast per bullet id)
  secondary key = per-type positioning potential (§2)
  ```

- **Gap-seeking candidates** [2017 Omega Ruby]. Add the midpoint between the predicted crossing points of the
  two incoming bullets, and ±90° from the first bullet's heading.
- **Desire zones** [2017 Bruteforcer, 4th]. Each projectile becomes a chain of avoid-circles at predicted
  future positions, with decreasing penalties. Add attract and repel zones for units. Evaluate 8 directions at
  several stride lengths. Trim zones (weakest first) to fit the remaining bytecode. Widen the safety margin
  when there are few bullets and narrow it when there are many.
- **Ignore bullets moving away from you** (velocity more than 90° from the bullet→unit vector)
  [2017 Segfault].

## 12. Ability combos and action economy

- **Look for anything that multiplies actions per turn.** Two examples:
  - a support unit that resets another's cooldowns (blink + attack, reset, again: "devastating")
    [2018 Orbitary Graph, 1st];
  - spawn chains where a new unit acts in the same round ("church lightning") [2019].

  Both years' biggest surprises came from reading the turn queue and cooldown rules for this.
- **Carry and throw mechanics.** Enumerate move + turn + throw combinations and score each by outcome: hits an
  enemy, a hazard, a wall, an ally, or leaves vision. Prune bad combinations empirically. Hold a grabbed enemy
  until a good throw appears [2026 GST, Lorem Ipsum]. Grab the enemy you can carry that has the most HP, or
  one carrying your ally [2026 food].
- **Relay objectives.** A carried objective handed between adjacent allies every turn or two moves faster than
  any single carrier. Escorts re-grab it the round after a carrier dies [2024 top-bot replay study]. The
  chance of delivery is a chain, `p^n`, so small changes in p or n compound. Freeze the carrier *and* move the
  objective farther away.
- **Traps and placed weapons.** Place them where several enemies are adjacent, after direct attacks, for
  holding ground or retreating. Compare trap cost with the enemy's replacement cost [2026 food]. Save scarce
  ones for big groups; the yield rises with enemies near the tile [2024 replay study]. Place traps only once
  real fights have begun [2024 Honey Ducklings].

## 13. How top teams improved their micro

- **Test maps with pre-placed armies.** An empty "plains" map, prespawned formations, and an N-against-N map
  for tuning large fights [2012 fun gamers, 2017 E Doc Tablet]. Then 3–5 purpose-built maps per behaviour
  [2026 food].
- **Watch one unit.** Follow a single unit into combat, find its stupidest decision, and fix it in a way that
  disturbs the good decisions as little as possible [2026 GST].
- **Copy better micro, then understand it.** "Rob XSquare's micro and turn it against him" took ~20 minutes
  and gained +100 rating [2024 cout for clout, second-hand]. Reverse-engineer the teams that out-fight you
  [2026 GST]. Copying without understanding made later changes hard [2025 JWU, confused].
- **Self-play tuning works for micro** (one team ran it "a million times") [2023 Gone Fishin']. Confirm against
  real opponents: self-play misses what the field exposes [2025 JWU, 2026 food].
- **Imitation bots of the top opponents** as permanent sparring partners [2016 future perfect, 2017 Volatile].

## 14. Checklist

1. Write down the engine's turn order, attack timing and cooldowns, and decide whether this is a
   "gap-closer-wins" or "first-to-enter-dies" year.
2. Implement the 9-move MicroInfo kernel with a lexicographic or single-currency comparison, evaluating
   move+act jointly.
3. Add target selection by turns-to-kill and type priority, with focus fire through a shared slot.
4. Add a healing or retreat state with hysteresis, group retreat, and retreat direction away from the enemy sum.
5. Add a local superiority check and a 1v1 cooldown-race predictor, plus a step-in rule.
6. Add kiting while reloading, and blind fire at the last known position.
7. Build test maps with prespawned armies, and A/B micro against your previous version *and* imitation bots.
8. Macro: harass economy and production, mass arrival, timed all-ins, rush detection with proportionate
   responses.

---

## Sources

Post-mortems and write-ups (read in full unless noted): 2012 fun gamers (1st) and the 2013 MIT 6.370
lectures (swarms, numerical strategy, sprint lessons); 2013 Teh Nubs (1st, code), bovard; 2014 that one team
(1st, code), Darkpurple, schnitzel; 2015 the other team (1st, code), Ayyyyyyyylmao, MattJohnerson; 2016 future
perfect (1st, README + code), Polar Vortex (2nd, code), foundation, what_thesis, BLEAKFORTUNE, #trump2016
notes, victorious-secret; 2017 Arbitrary Graph Restoration Fund (1st, code + commits), Omega Ruby (2nd,
code), Bruteforcer (4th, code), BTC (code), E Doc Tablet, Segfault, Volatile; 2018 Orbitary Graph (1st, code),
smite, Sudoers; 2019 smite (1st), Justice of the War, Double J, Shadow Priests, Codelympians, plzgoeasy,
Chicken; 2020 Java Best Waifu (1st), smite (2nd), Battlegaode (3rd), The High Ground (4th), Bowl of Chowder,
confused, Kryptonite and Bagger288 (second-hand); 2021 Baby Ducks (1st), wololo (via practice-repo summaries
and code), Stone Tao, naalit, waffle; 2022 5 Musketeers, smite micro notes, camel_case (code), Polar bears,
TestSubjector, Baby Ducks ideas; 2023 Gone Fishin' (2nd), 4 Musketeers (3rd), no thoughts head empty,
don't @ me (snippets); 2024 Honey Ducklings, cout for clout (second-hand), and replay studies of top bots
recorded in the practice repository; 2025 Just Woke Up (1st), confused (2nd), Om Nom (3rd), SPAARK, The
Kragle; 2026 Generalized Stroke's Theorem (2nd), food, Lorem Ipsum, 3MiceWalkIntoABar, SPAARK (code).
