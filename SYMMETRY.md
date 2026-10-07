# SYMMETRY — what post-mortems teach about symmetric maps

Battlecode maps are almost always symmetric so that neither side has an advantage. This means half the map
is known as soon as the other half has been seen, and the enemy base is one of a small number of known
candidates. Every strong team treats "which symmetry is this map, and what does it imply" as basic
infrastructure. One recent post-mortem calls it "Battlecode 101". Build it before the season starts, and
do not pay for it with a lost sprint.

This file collects what post-mortems from 2009–2026 recorded about symmetry. Only the parts that carry
across rule sets are kept. Sources are cited as `[year team]`, and a source list is at the end. The
companion files are ECONOMY.md, NAVIGATION.md, EXPLORATION.md and COMBAT.md.

---

## 1. Why symmetry matters

Symmetry pays off in seven ways, roughly in order of value:

1. **Locating the enemy base before seeing it.** Mirror your own base(s) under each candidate symmetry.
   That gives at most three places to look, and usually one once the symmetry is known. Many teams rushed or
   rallied to the predicted base from turn 1. [2012 fun gamers, 2021 Baby Ducks, 2023 4 Musketeers]
2. **Doubling map knowledge.** Every tile you have seen tells you its mirror. Pathfinding can treat an
   unseen tile as equal to its seen mirror and get "nearly full map knowledge from half the exploration".
   [2015 anim0ls] Mirror every discovered static feature too: resource sites, neutral structures, hazards.
   [2016 mid high diamonds]
3. **Finding enemy resources.** Your own resource sites mirrored are the enemy's. Send raiders there to
   deny the enemy economy, and add the mirrors to your own list of mining options. If a site you want is
   occupied or hard to reach, try its symmetric twin. [2023 4 Musketeers, 2023 no thoughts head empty]
4. **Splitting the map into "our half" and "their half".** The perpendicular bisector of the two bases (the
   "midline") is the natural front. Resources on your side are safe to develop. Resources across it are
   contested or can be harassed. [2014 that one team, 2019 smite, 2015 Ayyyyyyyylmao]
5. **Income asymmetry as a win condition.** On a symmetric map with fixed resources, holding everything on
   your side plus even one site on the enemy side produces "an asymmetry in resource incomes that should,
   with optimal play, lead to a win." [2019 smite]
6. **Deficit detection.** If you expanded fully and still own less than half the resources, some
   enemy-held pocket must be on your side. Find it and take it back. [2019 Justice of the War]
7. **Map geometry.** Knowing one map edge gives the opposite edge, and the midpoint of mirrored spawns is
   the map centre. [2015 the other team, 2016 mid high diamonds, 2016 Polar Vortex]

## 2. The candidate set

The usual candidates are:

| Name | Maps (x, y) to | Notes |
|---|---|---|
| Rotation (180°) | (W−1−x, H−1−y) | Most common default. Point symmetry about the centre. |
| Horizontal reflection (flip top/bottom) | (x, H−1−y) | Mirror line is horizontal. |
| Vertical reflection (flip left/right) | (W−1−x, y) | Mirror line is vertical. |
| Diagonal reflections (rare, square maps only) | (y, x) or (W−1−y, H−1−x) | Appeared in some early years. [2012 fun gamers, 2015 anim0ls] |

Naming of "horizontal" and "vertical" is inconsistent between teams and between spec documents. Define the
names by their coordinate formula in code and never by the English word.

Points about the candidate set:

- **Read the spec and the map corpus.** Some years allowed only reflections [2019]. Most allowed all three.
  Some early years allowed diagonals, and a few maps were 4-way symmetric, so several candidates are true at
  once [2016 #trump2016]. Check every official map at kickoff for which symmetries occur and how often.
- **Never assume one symmetry.** One team hard-coded rotation because all three scrimmage maps were
  rotational, and it "backfired quite heavily" at the first tournament [2023 Gone Fishin']. A competitor
  complained that not every map was rotational [2017 IRC]. Assume rotation as the *prior*, never as a fact.
- **Several candidates can be true at once.** A map can be both rotation- and reflection-symmetric. Base
  guesses from different candidates can then coincide. That is fine: the guesses agree.

### Integer-safe formulas

Work with doubled coordinates so that centres on half-tiles stay integers [2016 mid high diamonds,
2015 anim0ls]:

```
// cx2 = minX + maxX, cy2 = minY + maxY   (twice the centre; equals W-1, H-1 when origin is 0)
mirror(ROT,   x, y) = (cx2 - x, cy2 - y)
mirror(HREF,  x, y) = (x,       cy2 - y)      // horizontal mirror line
mirror(VREF,  x, y) = (cx2 - x, y)            // vertical mirror line
mirror(DIAG+, x, y) = (c + y, -c + x)         // c from the base pair, only on square maps
mirror(DIAG-, x, y) = (c - y, c - x)
```

When absolute coordinates carry an unknown offset (several years added 10,000+ to every coordinate or hid
the origin), the mirror needs the map edges, or the spawn pair, to fix the centre:

- **From edges.** Find the edges first, then compute cx2 and cy2 [2021 Baby Ducks, 2021 waffle].
- **From one's own and the enemy's spawn positions** (when both are known). For the rotation centre:
  `c2 = ourBase + theirBase`.
- **Bounded modulus for addressing.** Coordinates mod 128 (or mod a power of two above the maximum map size),
  together with your own position, identify any location uniquely without knowing the edges [2021 wstan2001,
  2017 Omega Ruby torus indexing]. This solves *addressing* but not *geometry*: reflection still needs the
  relevant edge pair.

## 3. Detecting the symmetry

The method depends on what the game reveals at turn 0.

### 3a. Both teams' spawn positions are given

Test each candidate exactly against the full spawn sets, and stay UNKNOWN if more than one fits (for
example, when bases are collinear). [2016 mid high diamonds]

```
// A = our spawns, B = their spawns (multisets of points), doubled coordinates
for b in B where b.y == A[0].y:   s = A[0].x + b.x
    if multiset(A) == { (s - p.x, p.y) for p in B }: candidates.add(VREF(s))
for b in B where b.x == A[0].x:   s = A[0].y + b.y
    if multiset(A) == { (p.x, s - p.y) for p in B }: candidates.add(HREF(s))
for b in B:                        c2 = A[0] + b
    if multiset(A) == { c2 - p for p in B }:        candidates.add(ROT(c2))
symmetry = (candidates.size == 1) ? the one : UNKNOWN
```

**Geometric shortcut** [2015 the other team]. A reflection maps our base to theirs only if the segment
joining them is perpendicular to the mirror line. With grid-aligned mirrors, that segment must be
horizontal, vertical or diagonal. So if the line between bases is none of those, the map *must* be
rotational. If the bases share an x coordinate, the candidates are horizontal reflection and rotation; if
they share a y, vertical reflection and rotation. [2015 anim0ls]

**Eliminate with other known symmetric object pairs.** Map each of your known structures (towers, outposts,
resource nodes) through each surviving candidate. Look the image up in the set of known enemy structures. A
miss eliminates that candidate. [2015 anim0ls]

### 3b. The whole static map is known at turn 0

Test every candidate against a distinctive layer over the whole map. Sparse layers such as resource
deposits make better tests than terrain, because they are more distinctive and produce fewer false
positives. [2019 smite]

```
isVREF = all rows r: all i: layer[r][i] == layer[r][W-1-i]
isHREF = all cols c: all j: layer[j][c] == layer[H-1-j][c]
isROT  = all (x,y): layer[y][x] == layer[H-1-y][W-1-x]
```

Checking only a few rows is cheaper but can accept the wrong symmetry. It is acceptable only if the rules
guarantee exactly one of two symmetries and the sampled rows are informative. [2019 Codelympians]

### 3c. Fog of war: eliminate from observations (the common modern case)

Keep a set of candidates. Every time a tile is sensed, compare it with its mirror under each surviving
candidate if that mirror has already been seen. Any mismatch eliminates the candidate.
[2023 Gone Fishin', 2025 confused, 2026 food]

```
candidates = {ROT, HREF, VREF}            // or whatever the spec allows
onTileSensed(t):
    record(t)                              // map memory, one small int per tile
    for s in candidates:
        m = mirror(s, t)
        if seen(m) and staticFeatures(m) != expectedMirror(s, staticFeatures(t)):
            candidates.remove(s)
    if candidates changed: queue a 3-bit "eliminated" report for the shared channel
```

Rules that every team that did this learned:

- **Compare only static features:** walls, impassable terrain, resource deposits, neutral structures, the
  terrain type of a tile. Never compare units, paint or ownership, resources that deplete or regrow, or
  anything a player can change. Movable bases are not static either [2022: bases could move].
- **Mirror directional features.** A current, conveyor or "facing" tile pointing east must appear pointing
  west at its mirror under VREF or ROT, and still pointing east under HREF. Mirror the direction before
  comparing.
  [2023 Gone Fishin']
- **Elimination is monotone.** A symmetry, once eliminated, never comes back, so the state is a bitmask that
  units can OR together. Three bits in shared memory are enough. [2023 Gone Fishin', 2025 SPAARK,
  2025 Om Nom, 2026 food]
- **Make it cheap and incremental.**
  - **Check only on the far side** [2025 confused]. Keep a set of cells already checked. For a reflection,
    check only cells on the far side of the mirror line; for rotation, only cells in the opposite quadrant.
    This avoids checking each pair twice.
  - **Row bitmasks** [2025 Om Nom]. Store each static feature as one 64-bit word per row, plus a pre-reversed
    copy, so a whole row is checked in a few word operations (about 9x faster than per-tile checks, about 1k
    bytecode for an exhaustive check):

    ```
    for each row y in or near vision:
        seen2 = seen[y] & seenRev[mirrorRow(y)]           // tiles whose mirror is also seen
        if (wall[y] & seen2) != (wallRevMirror(y) & seen2): eliminate(s)
        if (ruin[y] & seen2) != (ruinRevMirror(y) & seen2): eliminate(s)
    ```

  - **XOR form** [2026 SPAARK, code]. With `R(x) = Long.reverse(x) >> (64 − W)` mirroring the bits within a
    row, test the rows in the current vision band:
    - horizontal reflection: `E = explored[y] & explored[H−1−y]`; invalid if
      `((wall[y] ^ wall[H−1−y]) & E) != 0`;
    - vertical reflection: `E = R(explored[y]) & explored[y]`; invalid if `((R(wall[y]) ^ wall[y]) & E) != 0`;
    - rotation: `E = R(explored[y]) & explored[H−1−y]`; invalid if `((R(wall[y]) ^ wall[H−1−y]) & E) != 0`;
    - repeat each test for every other static feature layer.

    Flag eliminations a unit found itself, as opposed to heard, so it sends them first.
- **Fuse with the map recorder.** The per-tile record used for pathing (seen, passable, feature bits) is
  the same data the symmetry check reads. [2023 Gone Fishin', 2025 Kragle]
- **When map edges are unknown, find edges first.** Reflections are undefined until the relevant edge pair
  is known. Under rotation, discovering one edge gives the opposite one:
  `maxX = theirBase.x + (ourBase.x − minX)`. [2015 the other team, 2021 waffle]

### 3d. Eliminate by visiting predicted base locations

With k own bases, each candidate symmetry predicts k enemy base locations. Visit one predicted location for
candidate S. If no base is there, S is eliminated, along with **every other location S predicted**. Units
clear candidates as a side effect of hunting. [2023 4 Musketeers, 2020 confused, 2026 food]

```
targets = []
for s in candidates: for b in ourBases: targets.add((mirror(s, b), s))
visit targets in order (nearest first, or the rotation candidate first)
on arrival at (loc, s): if no enemy base visible at loc: candidates.remove(s); drop all targets tagged s
```

Remember a "seen at round R" timestamp if bases can move or be rebuilt [2022].

## 4. Who should find the symmetry, and where

- **The map centre is the best vantage point.** The three hypotheses diverge fastest there, and it is on
  the way to every enemy base candidate. One team's rush unit ran to the centre first to settle the symmetry
  and then went straight to the enemy. It was slower when the default guess was right, but "more consistent
  overall". [2020 The High Ground, 2025 confused]
- **Prefer units already travelling.** A dedicated symmetry scout that walked to the centre and back cost
  early mining. Letting attackers check symmetry on the way to the guessed base cost nothing.
  [2023 Gone Fishin'] Units that follow the army (support units, relays) are good passive symmetry checkers.
  [2023 no thoughts head empty]
- **Report as soon as you can.** If writing to shared memory is range-limited, a unit that has eliminated
  two candidates can walk home to report it. [2026 food] Re-squeak or relay the bits toward a unit that can
  persist them.
- **Do not send units one at a time to symmetry targets.** An over-eager "symmetry explore" sent single
  units into 1-vs-5 fights. Pair symmetry targets with group movement. [2026 TSPAARK]
- **Fall back gracefully.** Always handle "the guess was wrong", and re-target when the first candidate turns
  out empty. [2018 notes] On a split or walled map, check reachability of the predicted base at turn 1: an
  unreachable base means a different plan. [2018 notes]

## 5. Using the symmetry once known

```
class Symmetry:
    bits candidatesAlive                       // shared, OR-merged elimination mask
    mirror(s, loc)
    bestGuess(): if exactly one alive: it; else the prior (usually ROT) among alive ones

    enemyBaseGuesses():
        return [mirror(s, b) for s in alive for b in ourBases], ordered by (s == bestGuess, distance)

    effectiveTile(loc):                        // for pathfinding and planning
        if seen(loc): return map[loc]
        if bestGuess known and seen(mirror(bestGuess, loc)): return map[mirror(...)]
        return UNKNOWN (treat as passable)

    onFeatureDiscovered(f):                    // resources, neutral structures, hazards
        record(f); if symmetry known: record(mirror(f)) as "enemy-side twin"
```

Until the symmetry is decided, hedge rather than guess. For decisions that depend on where the enemy is,
such as where to put something precious, use the worst case over every live candidate: for example, maximise
the distance to the nearest enemy base candidate under *every* surviving symmetry. In one practice season a
blind rotational guess was wrong on 4 of 10 maps, and against one opponent the guess was wrong in 28% of
games. Observed symmetry was settled by round 1 in about half of games and by round 30 in about three
quarters [practice-repo measurements].

Name the decision that will consume the answer before building the detector. Symmetry inference that works
but feeds no decision is worth nothing.

Typical uses of the result:

- **Rally and rush targets** from turn 1 [2012 fun gamers]. Late improvements built on this took one
  first-year team from the top 20 into the top 15. It regretted not doing it in week 2 [2021 naalit].
- **Mirror own resource sites** to find enemy sites for denial or camping [2023 4 Musketeers]. Flag your own
  structures that sit near the mirror of an enemy-side resource as "in danger" and garrison them early
  [2019 Codelympians].
- **Choose sites by side.** Prefer economic sites on your own side of the midline and far from it, so your
  reinforcements arrive first [2014 that one team, 2014 schnitzel]. Use distance to the base midpoint as a
  cheap danger proxy in site scoring [2015 Ayyyyyyyylmao].
- **Doppelganger marking.** Each unit marks the mirror of its home site as hostile [2019 smite].
- **Count holdings against the mirror** to detect enemy pockets on your half [2019 Justice of the War].
- **Fill unseen terrain** in BFS from mirrored tiles [2015 anim0ls]. Bound searches early using inferred
  edges [2015 the other team].
- **Seed exploration lists** with symmetry guesses, plus stale combat locations [2023 4 Musketeers]. Send
  splashers or raiders toward the inferred enemy territory [2025 Om Nom, 2025 SPAARK].
- **Know which side you are on.** If results flip when you swap sides on the same map, suspect a
  side-dependent or symmetry bug. Run every test map from both sides. [2024 It's A Trap?, 2022 camel_case]

## 6. Pitfalls recorded in post-mortems

- **Hard-coding one symmetry** [2023 Gone Fishin'] or assuming rotation forever.
- **Comparing dynamic state** (units, paint, depleted resources, moved bases).
- **Forgetting to mirror direction-valued tiles.**
- **Accepting a symmetry from a partial check** that the rules do not guarantee [2019 Codelympians].
- **Ambiguous spawn configurations.** Collinear bases, or maps symmetric under several transforms, leave
  more than one candidate. Code must tolerate several "true" candidates.
- **Unknown edges or offset coordinates.** Reflection formulas silently produce garbage until the edges are
  known. Guard them.
- **Symmetry work that never reaches the units that need it.** If only a few units can write shared memory,
  the knowledge must be carried or relayed. Budget for that.

## 7. Minimal checklist for a new season

1. At kickoff, read the spec's guarantee about symmetry and tabulate the symmetry of every released map.
2. Implement `mirror(s, loc)` with doubled-centre integers and unit tests on asymmetric probe points.
3. Pick the detection method for the year: given spawns (3a), full map at turn 0 (3b), or fog elimination
   (3c) plus base-visit elimination (3d).
4. Put a 3-bit eliminated mask in shared memory. Every unit ORs into it.
5. Wire the result into: enemy base targets, resource mirroring, pathfinding fill-in and own-half tests.
6. Test on maps of each symmetry type, from both sides.

---

## Sources

Post-mortems and write-ups (read in full unless noted): 2012 fun gamers strategy report (1st);
2014 that one team and 2015 the other team (1st, README + referenced code); 2015 anim0ls (3rd, code);
2015 Ayyyyyyyylmao; 2016 mid high diamonds, #trump2016 and Polar Vortex (code); 2017 Arbitrary Graph
Restoration Fund and Omega Ruby (code); 2018 team notes; 2019 smite (1st), Justice of the War and
Codelympians; 2020 The High Ground and confused; 2021 Baby Ducks (1st), waffle, naalit, wstan2001 and
3 Musketeers (summary); 2022 5 Musketeers, Polar bears and TestSubjector; 2023 Gone Fishin' (2nd),
4 Musketeers (3rd) and no thoughts head empty; 2024 It's A Trap? harness notes; 2025 confused (2nd),
Om Nom (3rd), SPAARK and The Kragle; 2026 food and TSPAARK. Tags such as "(code)" in the research notes
mean the detail came from the team's published bot source that its write-up pointed to.
