# NAVIGATION — what post-mortems teach about moving units around the map

Navigation is the infrastructure item that post-mortems regret most often, and the one most reusable
across years. Some representative quotes:

- "One of the major causes of virtually all our losses" (a 2020 high-school finalist).
- "Really wished" they had spent more time on it (the 2020 runner-up, after a week on it).
- "Most likely a mistake" (a 2021 finalist who skipped real pathfinding because the map had no walls).
- "Every year I toy with BFS and come back to bugnav" (the 2025 winner).

This file collects the techniques and failure modes that recur across 2007–2026. Only material that carries
across rule sets is kept. Sources are cited as `[year team]`, and a source list is at the end. The companion
files are ECONOMY.md, EXPLORATION.md, COMBAT.md and SYMMETRY.md.

---

## 1. Strategic lessons first

1. **Build navigation in week 1 and keep it.** Winners built bug navigation in the first week and kept it
   all season [2020 Java Best Waifu]. Reusing last year's navigator is normal [2022 5 Musketeers,
   2025 Kragle]. Bring a tested bug and a code generator for unrolled BFS to kickoff.
2. **Pathing quality decides close maps.** "On maps with a direct path between starting locations, the team
   with the better pathing algorithm had a better chance at rushing." [2017 E Doc Tablet] A path two rounds
   shorter wins the first engagement [2012 fun gamers].
3. **Match the algorithm to the terrain** [2025 Kragle]:

   | Terrain | Fits |
   |---|---|
   | Binary walls | Bug, optionally with local BFS |
   | Variable-cost terrain (rubble, passability, paint) | Greedy or unrolled Dijkstra/Bellman-Ford |
   | Few walls guaranteed (for example, a spec cap) | Greedy-plus-bug is enough |
   | Destructible terrain | "Dig rather than path" |

   The 2016 winner wrote: "Our navigation wasn't too fancy this year, since in the worst case we could just
   dig through rubble." [2016 future perfect]
4. **Weighted terrain still needs real pathing.** Skipping it because "there are no walls" cost one 2021
   finalist map control, neutral captures and attack timing, and it went unnoticed for 1.5 weeks
   [2021 Stone Tao].
5. **Every mechanic that slows, blinds or pushes a unit is navigation.** Ignoring one "rare" mechanic cost
   one team two finals games [2020 The High Ground]. Mechanics that affect movement or vision cannot be
   skipped, unlike other side mechanics [2023].
6. **Design for the unit footprint.** Navigation written for 1×1 units "completely broke" for 3×3 units
   [2026 GST]. See §7.
7. **Don't over-engineer.** Complex planners (line-sweep obstacle analysis, greedy-simulate-then-bug with
   checkpoint stacks) repeatedly failed on bytecode and edge cases. "Over-complicating things did not help."
   [2023 Gone Fishin', 2025 Kragle]
8. **Copying a navigator is fine, as long as you understand it.** One team's navigator was widely copied in
   2025, but "blindly copying makes later modification hard" [2025 confused, 2025 JWU].

## 2. Movement primitives

Every bot needs a small set of helpers that every behaviour calls:

```
tryMoveToward(d):            // any roughly-correct move is acceptable (retreats, approaches)
    for dd in [d, d.left, d.right, (d.left.left, d.right.right if allowed)]:
        // order the two sides by which one ends closer to the destination
        if safe(dd) and canMove(dd): move(dd); return true
    return false

safe(dir):                   // pluggable safety policy passed in by the caller
    loc = here + dir
    return !inRangeOfKnownEnemyTurret(loc) && !inEnemyBaseRange(loc) && ...
```

- **Pluggable safety policy.** `NavSafetyPolicy.isSafeToMoveTo(loc)` lets each caller exclude enemy tower or
  base range, or all enemy attack ranges, without forking the navigator [2014 that one team]. Filter
  candidate squares by whether a visible enemy, or the remembered nearest enemy turret, can hit them
  [2016 future perfect].
- **Never move to increase distance in direct mode.** Skip candidates that move away from the target, and
  hand over to bug instead [2016 future perfect].
- **Cheapest-step greedy for weighted terrain.** Move to the adjacent tile with the best terrain among those
  strictly closer to the destination [2021 naalit]. A cruder version: if going straight costs more than
  twice going 45° to either side, take the 45° step [2021 Baby Ducks]. Some teams considered only 3
  candidates (target direction ±45°) to save bytecode [2021 wstan2001].
- **Dig or clear when that is cheaper than going around.** If terrain is removable, clear the lowest-cost
  blocking tile among the candidates [2016 future perfect]. Or treat diggable terrain as passable with a
  dig cost, raising the cost when resources are low [2026 GST, 2026 food, 2024 Honey Ducklings].
- **Handle the degenerate direction.** The direction to your own location is CENTER. Make sure that
  doesn't stall the unit [2025 JWU].
- **Don't let a cooldown make terrain look like a wall.** After an action that sets a long cooldown (such
  as digging), the next tile looked impassable, so the unit turned and bugged around a patch it should have
  tunnelled through. Fix: wait when the next tile is the same kind [2026 food].

## 3. Bug navigation (the universal fallback)

Bug navigation remembers only (mode, wall side, start distance, heading). It needs no map, and with a
correct leave rule it guarantees arrival when a path exists. Every year's top teams used some variant.

### 3a. Core algorithm (as used by the 2014 and 2015 winner)

```
state: mode ∈ {DIRECT, BUG}, wallSide ∈ {LEFT, RIGHT}, startDist2, rotationCount, lastDir

step(target):
    if mode == DIRECT:
        if tryDirect(target): return              // dirToTarget, then the neighbours ordered by result distance
        mode = BUG; startDist2 = dist2(here, target); rotationCount = 0
        wallSide = side whose first free rotation (≤3 steps) ends closer to target

    // BUG mode: scan from two rotations back toward the wall, take the first free direction
    d = lastDir rotated twice toward wallSide
    for i in 0..7:
        if canMove(d) and safe(d): break
        d = rotate d away from wallSide; rotationCount += (away ? +1 : -1)
    if the cell on the wall side is OFF_MAP:         // hugging the map edge is never useful
        flip wallSide; restart BUG
    move(d); lastDir = d
    if (rotationCount <= 0 or rotationCount >= 8) and dist2(here, target) <= startDist2:
        mode = DIRECT                             // back on the near side of the obstacle
```

Leave rules seen across years:

- **Closer than where bugging began.** Strictly closer than the start distance [2013 devs lecture, 2014].
- **Net-rotation count.** Leave only after net rotation returns to 0, meaning you went around the obstacle
  [2013 devs, 2014].
- **Bug 2 (m-line).** Leave when you re-cross the start-to-target line closer than when you left it. The devs
  recommended Bug 2 over simple bug [2021 lecture]. Detecting the m-line crossing robustly is the hard part:
  a cross-product test failed at varying distances [2026 nfgehrs].
- **Dist-Bug leave rule.** Leave when `dist(here, target) − (free straight-line run toward target)` is below
  the best distance ever reached. This prevents oscillation and works in continuous space as well
  [2017 Arbitrary Graph Restoration Fund, 1st].
- **Patience.** Give up after N turns or N moves without obstacle contact and return to direct mode
  [2013 devs, 2014].

### 3b. Choosing the wall side well (the main weakness of bug)

Plain bug picks a side locally and can take a very long way round. Fixes that worked:

- **Simulate both sides virtually.** On the known map, trace left-hand and right-hand wall-following for N
  steps (under a bytecode cutoff), and take the one that ends closer to the target, or first reaches a cell
  with a clear line to it. If only one finishes, take it; if neither does, pick at random.
  [2014 Darkpurple "look-ahead bug", 2023 Gone Fishin', 2026 GST]
- **Obstacle analysis.** Treat a contiguous wall set as a graph and find its two extreme endpoints by double
  BFS (diameter). Turn toward the endpoint minimising `bfsDist(me, end) + dist(end, target)`. Spread the work
  over several turns (about 15k bytecode for a 20-tile wall). Until the wall has been fully seen, always turn
  the same way so it gets observed. [2023 4 Musketeers]
- **Tangent bug.** Advance a *virtual* bug trace ahead with spare bytecode. Once the trace rounds a corner,
  head straight for the corner instead of hugging the wall. [2012 fun gamers]
- **Path smoothing.** Look two waypoints ahead on the bug path, and drop the intermediate one when the
  straight segment is clear [2025-12 pre-season notes].
- **Bug modes beyond "toward".** The same wall-follower can run AWAY (reverse the target direction) and
  AROUND (orbit between a minimum and maximum radius, flipping the circle direction when blocked). That covers
  fleeing and patrolling with one tested component [2026 SPAARK, code].
- **Prefer the side indicated by a known path**, or a precomputed coarse BFS direction, and fall back to
  random 70/30 [2017 AGRF].

### 3c. Stack-based bug (handles C-shaped traps) [2023 Gone Fishin', reused 2026 GST]

```
each turn:
    while stack not empty and canMove(stack.top): stack.pop()
    if stack empty: move greedily toward target; return        // obstacle cleared
    d = stack.top
    while !canMove(d): stack.push(d); d = rotateLeft(d)
    move(d)
```

Plus:

- if an obstacle has been followed too long, switch sides;
- forbid following the map edge;
- keep a set of "turn locations", and if you turn again at one, you are looping, so reverse.

### 3d. Bug failure modes recorded in post-mortems

- **Units as walls.** Treating allies as walls breaks bug's invariants; treating them as empty makes units
  wait forever. Allies bugging in opposite directions round a big obstacle collide and spin [2013 devs].
  Remedies:
  - classify obstacles by type (off-map, permanent, ally "wait, it will move", enemy "stop and fight")
    [2013 devs];
  - treat friendlies as walls 75% of the time and as empty 25%, which breaks deadlocks [2023 Gone Fishin'];
  - mark occupied tiles impassable for ~5 turns with a decaying queue [2026 3Mice].
- **Resetting state when the target "changes".** A moving target (a mobile base, or rotating among tiles
  around a site) resets bug every turn and loops in mazes. Don't reset if the new target is within a small
  distance of the old one [2026 Lorem Ipsum, 2025 Om Nom].
- **Bug toward a moving target.** Bug helped for fixed targets but regressed for moving ones, such as an enemy
  unit or a wandering ally. For those, use greedy plus a short oscillation escape [practice-repo measurement].
- **Loop detection by state history.** Keep a set of (tile, heading) states visited while bugging, allowing
  at most a few visits per tile. When the allowance is exhausted, make a random move and clear the history
  [2026 SPAARK, code]. A cheaper version: when re-targeting the same destination, forbid the reverse of the
  previous move [2022 camel_case].
- **Following the map edge.** It is never useful. Flip side, or bug toward the target instead
  [2014, 2023, 2025 Om Nom, 2026 GST].
- **Crowds of stuck bots flipping rotation direction oscillate.** One team lost a qualifier map this way
  [2023 4 Musketeers]. Add hysteresis and timeouts.
- **Narrow corridors confuse which wall is being followed** [2024 Honey Ducklings]. Add a local search.
- **Circling in a box with a tiny gap.** Keep a visited-location history and, on a repeat, ban directions or
  take random moves [2016 victorious-secret].
- **"Hundreds of edge cases."** Example: two units bug the same wall, meet, and one goes round; the wall it
  was following moves away and its state breaks. Write the edge cases, but don't polish before the strategy
  works. [2012 fun gamers]

## 4. Local search inside vision (unrolled BFS / Dijkstra / Bellman-Ford)

The standard modern pattern is a shortest-path computation over the tiles in or near vision, written as
straight-line code with no loops and no arrays. It is generated by a script, and the first step goes toward
the best frontier tile.

### 4a. Unrolled Bellman-Ford on a small window [2020 Battlegaode → 2021 Baby Ducks → 2023 no thoughts]

```
// generated code: each window cell i is a set of scalar fields (passable_i, cost_i, value_i, firstDir_i)
for each cell i in window:                       // e.g. 5x5 or the full vision radius
    passable_i = onMap && !wall && !occupied && extraRule(i)
    value_i    = 100 * dist(cell_i, target)      // upper bound; target may lie outside the window
repeat K sweeps (K = 1..3, chosen by bytecode left):
    for each passable cell i, in a fixed heuristic order:
        for each neighbour n: value_i = min(value_i, value_n + cost(i))
step to the adjacent cell with the smallest value
```

- **Initialise with a distance heuristic.** Off-window targets work because every cell starts at
  `100 × straight-line distance`, which acts as an exit cost. One or two sweeps converge "in most cases".
- **Why Bellman-Ford rather than Dijkstra.** It unrolls cleanly: no heap, no recursion. `PriorityQueue`
  "eats bytecode", and a hand heap at radius 2 used most of the budget [2021 Baby Ducks].
- **Scalars, not arrays.** Put every cell in its own variable or static field. The array version ran out of
  bytecode, and unrolling cut the cost by more than 2x [2020 Battlegaode]. A Python script emitted about 1000
  lines of Java.
- **Stuck fallback.** Keep the last few positions; if there is no progress, switch to bug for some turns
  (keeping the wall on the right until closer than at the obstacle). Switch back once BFS improves
  [2020 Battlegaode, 2023 no thoughts]. One widely copied 2024 navigator switches after 3 turns without
  progress [2025 confused].
- **Cost.** About 6k bytecode on average for a 5×5 window with 1–3 passes [2023 no thoughts].

### 4b. Vision-radius Dijkstra with a progress-per-cost exit [2021 Malott Fat Cats, widely copied]

```
for tiles l_i in ring order (generated; l_i = l_j.add(dir) from an already-processed neighbour):
    if onMap(l_i): p_i = cost(l_i)
        for each processed neighbour n: if v_i > v_n + p_i: v_i = v_n + p_i; d_i = d_n
choose tile maximising (initialDist − dist(l_i, target)) / v_i    // progress per unit cost
move d_i
```

- **Wrapper** [2021 Malott Fat Cats]. Keep a visited bitmap, reset when the target changes. If the chosen
  next tile was already visited, go greedy for 4 turns. Run only when ≥ 2500 bytecode remain. Generate one
  class per unit type, matched to that type's vision radius.
- **"Greedy BFS" variant.** Relax only edges that move strictly outward from the centre. Keep several radius
  versions and run the largest one the remaining bytecode allows [2023 4 Musketeers, after XSquare].

### 4c. Bitmask BFS (the cheapest form)

- **Row-bitmask BFS.** Store passability as one 64-bit word per row and expand the reachable set by shifts
  and ORs. Reachability over the remembered map costs about 300 bytecode [2026 GST]. Row bitmaps serve both
  BFS and symmetry checks [2025 Om Nom].
- **Neighbour bitflags per tile.** Store one char per tile with bits {self blocked, N/E/S/W blocked}, so
  expansion is a switch on one value with no per-direction map reads. Expand 25 iterations, stamping the
  iteration number, track the expanded cell closest to the target, and backtrack by decreasing stamps. No
  codegen is needed. [2026 3Mice]
- **Bucket queue for small integer weights.** Diagonal cost 1.5 becomes delay 3 against 2 for orthogonal.
  Re-enqueue nodes with r−1 until 0. This gives Dijkstra-like costs at BFS prices [2015 anim0ls].

### 4d. Bug + BFS hybrids (the current state of the art)

- **"Optimal Bug"** (about 1.5k bytecode per turn) [2025 Om Nom]. Both the 2026 runner-up and a 2026
  finalist said they would adopt it.

  ```
  bugTarget = prevBugTarget ?: here
  repeat K times while bugTarget is inside vision:
      advance bugTarget one bug step              // unit-occupied tiles count as passable for the virtual bug
  dist = bitmaskBFS(from = bugTarget, over the vision window)
  move to the neighbour n with dist[n] < dist[here], minimising (dist to bugTarget, terrain penalty)
  ```

  It keeps bug's arrival guarantee, cuts corners instead of hugging walls, and steers around units without
  breaking bug's invariants.
- **BugBFS** [2026 GST]. Run unrolled BFS while it reduces distance to the target, hand control to bug when
  it stops making progress, and take control back once BFS can improve again.

## 5. Global and shared pathfinding

When some unit has spare compute, or the team shares memory, a global distance field beats every local
method.

- **Base-computed BFS from the destination** [2014 that one team]. The base (and other idle structures) BFS
  outward from the rally point with spare bytecode and publish, per cell, the next direction toward the
  destination. Robots bug until their cell is covered, then follow the field.
  - **Channel layout.** Value `10000000 + dir*100000 + destX*H + destY` at `channel(page, x, y)`, so a reader
    can verify which destination it belongs to.
  - **Pages.** Up to 5 pages (one per destination), each with metadata `finished|priority|roundLastUpdated|dest`.
    Reuse a finished page and let high priority overwrite.
  - **Rules.** Never path through your own base. Near the enemy base, allow only moves outward from it.
  - **Fast terrain.** When blocked or tied, prefer side steps onto faster terrain.
- **Distributed BFS through shared memory** [2015 the other team; 2017 Omega Ruby, 2nd; proposed by
  2017 E Doc Tablet]. The whole BFS state lives in the shared array:
  - queue head and tail channels and a queue region;
  - a "was queued" region stamped with the round number, which avoids clearing;
  - a result region holding `10*round + dir + 1`.

  Any unit with spare bytecode (for example > 3000) pops and expands nodes until a floor (1500) remains, then
  writes head and tail back. Re-initialise when the destination changes; nobody works in the round of
  re-initialisation, and results older than the last initialisation are ignored. A unit that finds a
  previously unknown wall re-initialises. Use the field only for slow, expensive units; everyone else bugs.
  Omega Ruby used torus indexing (coordinates mod a constant above the maximum map size) because the origin
  was unknown, and gave units roles: the base does BFS only, scouts only write passability samples, and
  others split 50/50.
- **Coarse shared map built in the background** [2017 AGRF, 1st]. Split the world into nodes grouped into
  4×4 chunks, with one int per chunk: 16 blocked bits, a sample count, a recalculation epoch, and a fully
  explored bit. At end of turn each unit samples a chunk overlapping its vision until about 6000 bytecode
  are spent, and publishes only if it learned more. The winner then **disabled its BFS** and shipped distBug,
  keeping the grid for exploration and site reservation. A stale global path can be worse than fresh local
  bug.
- **Flow fields to key destinations** [2018, single-program year]. Precompute BFS from the enemy base, the
  rally point and the escape points, and have every unit follow them. Share paths between units.
- **Unify target choice and pathing in one search** [2018 Orbitary Graph, 1st]. Dijkstra over a cost map, choose
  the tile maximising `value[t] / (pathCost[t] + 1)`, and exit early once `maxValue/(cost+1) ≤ best`. Cache
  target and cost maps per unit type, and use a version-stamped array to avoid re-initialisation.
- **Node-capped A* with partial paths** [2019 smite, 1st]. Cap expansions (128). If the cap is hit, return
  the path to the best node found so far. Use separate variants for "avoid a radius", "assault area" and
  "worker".
- **Coarse graph search for big swarms.**
  - Down-sample the map (count obstacles in large circles ahead) and steer the swarm as one unit; this
    avoids "splat then trickle" at obstacles [2013 devs].
  - Greedy maximal-rectangle decomposition plus A* over portals [2014 schnitzel].
  - Chunk-level A* weighted by obstacle density [2013 lecture].
- **Remember your own path.** Push visited locations onto a stack to retreat along a known-clear route
  [2013 devs]. A breadcrumb trail back to base never gets stuck and tends to avoid enemies, but detours
  [2024 waffle].

## 6. Costs, hazards and terrain rules

- **Enemy threat as soft walls** [2020 Java Best Waifu, 1st]. Keep an int grid. When a static enemy turret
  is reported, add +1 to every cell in its range; when it is reported destroyed, add −1. Cells with value > 0
  are obstacles. Reference counts make add and remove O(area). Handle turrets currently in sight separately,
  since new ones appear next to you. Switch the overlay off for an all-in. The result: raiders circled the
  enemy base just outside turret range.
- **Hazards as walls in bug, costs in greedy** [2025 JWU]. Tile score = terrain or paint preference + ally
  adjacency penalty + a heavy penalty inside enemy tower range. While wall-following, treat in-range tiles as
  walls.
- **Exclusion refcounts** for any "don't build or stand here" zone [2025 JWU]: increment around each cause,
  decrement when it goes away, and a query is one array read.
- **Pushing tiles (currents, conveyors).**
  - A current tile costs the same as the tile it pushes into, and moving into an opposing current is
    forbidden [2023 no thoughts].
  - Ride currents only when they point toward the target ±45° [2023 Gone Fishin'].
  - Near the goal, walk instead of riding, because a current may carry you past it into a pit
    [2023 4 Musketeers].
  - Treating currents as walls fails on maps where they are the only path.
- **Terrain that blinds or slows.** Do not step where vision would drop below a usable minimum
  [2020 The High Ground]. Avoid clouds while kiting or chasing [2023 Gone Fishin'].
- **Movement that costs resources.** When moving consumes a shared resource, cost of a step grows with step
  length, so moving one tile at a time is cheaper per distance [2019 smite]. Budget the march before
  committing: armies ran out of fuel mid-march and "slowed to a crawl" [2019 Double J]. One team checked
  that the fuel sufficed for every attacker to reach the target before launching, with a follower unit
  signalling fast or slow pace [2019 Codelympians]. Crossing the map can cost so much that "the defender has
  an advantage", and the army arrives smaller [2012 fun gamers].
- **Retreat only onto acceptable terrain.** Retreat to tiles no worse than the current one, so units never
  back into slow ground [2022 5 Musketeers]. Settle production units on the cheapest terrain nearby.

## 7. Large units, many units, and congestion

- **Configuration-space inflation for big units** [2026 3Mice]. For an n×n unit, mark every cell within the
  footprint of each obstacle as blocked. The 1×1 pathfinder then works unchanged.
- **Keep the spawn and deposit ring clear.** This is a recurring, season-wrecking bug [2019 smite,
  2019 Justice of the War, 2019 Codelympians]. Units idling or forming a lattice next to the base block
  production and resource deposits. Exclude the ring adjacent to production structures from every formation
  or parking slot.
- **Spawn-area "pressure" rule** for narrow maps: each unit leaves one empty cell behind it and advances only
  when that cell fills, so spawns never self-block [2014 schnitzel].
- **Movement order matters.** If units act in ID order, a line of units can move away from a wall but not
  toward it, because the front unit hasn't moved yet. Swap by ID, or broadcast "I'm stuck" so allies behind
  clear the way [2013 devs].
- **Lanes at corners.** The first unit to round a corner marks it as a waypoint. Others go straight to it,
  and repulsion from marked cells creates several lanes instead of one queue [2013 devs].
- **Gather-point etiquette.**
  - A unit adjacent to an occupied target tile may move only to other tiles adjacent to the target, or stay,
    so units circulate around a resource and leave gaps [2023 no thoughts].
  - Keep a diagonal lattice around resource sites so others can pass [2023 4 Musketeers].
  - Leave when cargo is full [2023 no thoughts].
  - If ≥3 allies are adjacent, step to a less crowded neighbour [2015 the other team].
- **Ally repulsion in the move score** prevents clumping in corridors: `+1000/(0.01 + d/1.7)` for allies
  within 1.7 of the body edge. Switch it off while squeezing through a corridor on waypoints
  [2017 Omega Ruby].
- **Reserved zones.** Producers publish their footprint in a shared bitmap, and other units standing on a
  reserved cell get a score pushing them off it [2017 AGRF].
- **Spawn sanity.** Check that a spawn tile is connected to the rest of the map; one map had spawn tiles
  inside a walled pen. Avoid spawning on pushing tiles [2023 Gone Fishin'].
- **Production congestion control.** When too many units crowd a base, stop producing gatherers at every
  base; extra gatherers *reduce* income [2023 Gone Fishin', 2025 SPAARK].

## 8. Stuck detection and recovery

- **Position history.** Stuck = every sampled position over the last ~30 rounds lies within 3 strides of
  here [2017 AGRF]. Keep the last few positions and switch to a fallback navigator when stuck
  [2020 Battlegaode].
- **Progress EMA** [2017 AGRF]. `s = 0.5*s + 0.5*(prevDist − dist)`. If s stays below 20% of a stride for 40
  turns, pick a new target. Re-pick after 5 failed moves.
- **Visited penalty.** Score −1 for tiles visited in the last 5 turns [2025 SPAARK]. Penalise past locations
  to repel the unit from where it has been [2017 Segfault, 2022 Polar bears].
- **Time budget per target.** Allow a number of turns proportional to the distance, then switch target. This
  is a cheap way to abandon unreachable goals [2020 Bowl of Chowder]. Abort a mission after a fixed number of
  rounds [2020 confused].
- **Exhaust timer in combat.** A unit in micro that has taken no action for N rounds (for example, the enemy
  is behind a wall) leaves combat [2026 3Mice].
- **Stuck-unit responses.**
  - Shoot or dig through adjacent obstacles [2017 AGRF].
  - A unit stuck on hazards moves or clears after a cost check [2013 devs].
  - A carrier unit picks up a stuck unit [2020 confused; this was not a substitute for pathfinding].
  - Disintegrate a support unit stuck for 50 turns [2015 the other team].
- **Count stuck units globally.** If more than 40% of reporting attackers are stuck, build terrain-clearing
  units [2017 Bruteforcer].

## 9. Reachability checks

- **Never target something unreachable.** Several 2026 teams chased resources and enemies across walls
  until a reachability check was added. GST's bit-shift flood fill fixed micro trying to move "through"
  walls. [2026 GST, Lorem Ipsum, food]
- **Split maps.** Single-array BFS crashed on maps with unreachable regions. At turn 1, BFS from the mirrored
  enemy base; if your own base is unreachable, the map is walled off and needs a different plan
  [2018 notes].
- **Return paths can be easier.** If delivering works at range or across walls, "close enough" greedy is
  often enough on the way back, even when getting out needed a maze solver [2026 food].

## 10. Continuous-space navigation (years without a grid)

- **distBug with linecasts** [2017 AGRF, 1st]:
  - sample the straight line to the target every 0.5 units with the body radius to get the clear-run
    distance;
  - leave bug when `dist − clearRun < historicalMin − ε`;
  - while bugging, rotate in 6° steps toward the wall as far as still movable;
  - sample a random shorter stride if nothing works.
- **Angular-interval free-heading computation** [2017 BTC]. Each obstacle of combined radius l, seen from p
  with step r, blocks the heading interval `dir(p→m) ± acos((|pm|² + r² − l²)/(2|pm| r))`. Sort the interval
  endpoints with an unrolled sorting network and sweep to find the free heading closest to the desired one.
- **Sampled candidate moves under a bytecode budget** [2017 AGRF, Omega Ruby, Bruteforcer]. The candidates
  are:
  - fixed ones: toward the target, a tangent around the nearest unit, last turn's best relative move;
  - random directions at several stride lengths;

  all scored by the same function used for dodging (see COMBAT.md).

## 11. Bytecode techniques specific to navigation

- **Direction walks instead of coordinate tables.** Precompute the sequence of Directions that visits every
  cell in vision and do `loc = loc.add(dir_i)` (2–3 bytecode) instead of `new MapLocation(x+dx[i], y+dy[i])`
  (15+). This saved over 2000 bytecode per turn [2020 Java Best Waifu]. The same trick generates ring-ordered
  tile lists for unrolled Dijkstra [2021 Malott Fat Cats].
- **BFS-ordered offset tables.** Precompute (dx, dy) offsets sorted by distance. The first hit in a scan is
  the nearest, with no distance computation [2020 Bowl of Chowder].
- **Incremental sensing.** Precompute, per move direction, the list of tiles that become newly visible, and
  sense only those [2012 fun gamers, 2025 Kragle].
- **Background computation with spare bytecode** at end of turn: virtual bug traces, BFS steps, map sampling.
  This hook is the common thread behind the 2012 tangent bug, the 2014 base BFS and the 2015/2017 distributed
  BFS [2013 Cory Li lecture].
- **Size the search to the budget.** Choose the BFS radius or number of relaxation sweeps from
  `Clock.getBytecodesLeft()`. Skip the search entirely below a floor and fall back to greedy.
- **No java.util in the hot path.** Use scalar fields, static finals, reverse loops (`for (i = n; --i >= 0;)`)
  and generated straight-line code. Generators: Python or Jinja scripts kept in the repo
  [2020 Battlegaode, 2022 camel_case `dijkstra.py`, 2025 JWU `pathfind.py`, 2025 Om Nom].

## 12. Checklist

1. A safety-filtered `tryMoveToward` and a bug navigator with a correct leave rule, an edge flip and the
   "don't reset on a nearby target change" guard, all unit-tested on maze, C-trap, corridor and crowd maps.
2. A generated local BFS or Bellman-Ford sized by remaining bytecode, combined with bug as in §4d.
3. Hazard overlays (turret range) as walls or costs, maintained by reference counts.
4. Traffic rules: clear spawn ring, gather-point etiquette, temporary unit obstacles.
5. Stuck detection with target re-selection, and reachability checks before committing to a target.
6. If shared memory exists, consider a background distributed BFS for expensive slow units.
7. Run test maps that stress every mechanic that moves or blinds units.

---

## Sources

Post-mortems and write-ups (read in full unless noted): 2010 Lazer Guns; 2012 fun gamers report (1st) and
the 2013 MIT 6.370 lectures (navigation, swarms, sprint lessons); 2013 Teh Nubs (code); 2014 that one
team (1st, README + code), Darkpurple and schnitzel; 2015 the other team (1st, code), anim0ls (3rd, code);
2016 future perfect (1st, README + code) and victorious-secret; 2017 Arbitrary Graph Restoration Fund (1st,
code + commit log), Omega Ruby (2nd, code), Bruteforcer (4th, code), BTC (code), E Doc Tablet, Segfault,
Volatile; 2018 Orbitary Graph (1st, code) and team notes; 2019 smite (1st), Justice of the War, Double J,
Codelympians; 2020 Java Best Waifu (1st), smite, Battlegaode (3rd), The High Ground (4th), Bowl of Chowder,
confused; 2021 Baby Ducks (1st), Malott Fat Cats (code), naalit, wstan2001, Stone Tao, devs' pathfinding
lecture; 2022 5 Musketeers, camel_case; 2023 Gone Fishin' (2nd), 4 Musketeers (3rd), no thoughts head empty;
2024 Honey Ducklings, waffle; 2025 Just Woke Up (1st), confused (2nd), Om Nom (3rd), SPAARK, The Kragle;
2026 Generalized Stroke's Theorem (2nd), food, Lorem Ipsum, 3MiceWalkIntoABar, nfgehrs; a Dec 2025 pre-season
pathfinding brainstorm.
