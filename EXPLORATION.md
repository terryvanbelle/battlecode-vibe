# EXPLORATION — what post-mortems teach about scouting, map knowledge and sharing it

Exploration means:

- finding things: resources, build sites, enemy bases, enemy armies, map edges;
- remembering them;
- getting that knowledge to the units that need it.

In Battlecode the third part is usually the hard one. Each unit sees only a small area and has a tiny
compute budget, and the shared channels are narrow, delayed, range-limited or readable by the enemy. This
file collects the techniques from 2007–2026 post-mortems that carry across rule sets. Sources are cited as
`[year team]`, and a source list is at the end. The companion files are ECONOMY.md, NAVIGATION.md, COMBAT.md
and SYMMETRY.md. Symmetry, the single most powerful exploration shortcut, has its own file.

---

## 1. Strategic lessons

1. **Simple can be enough.** "Random map location, re-picked when within distance² 5" matched every fancier
   scheme tried by a 2025 top-3 team: edges, unvisited sampling, radius around the last structure. Good
   *local* heuristics mattered more [2025 Om Nom]. Start simple and measure before building a frontier
   planner.
2. **Coverage bias decides tournaments.** A runner-up's centre-biased explorers missed corner resources on a
   large, sparse finals map and lost the deciding game [2025 confused]. Another team watched an opponent's
   edge-biased exploration take the edge resources [2025 SPAARK]. A spiral search from home favoured bases
   near the centre and found distant resources too slowly against the eventual champion [2023 Gone Fishin'].
   Balance centre and edge coverage on purpose, and test on large sparse maps.
3. **Idle units are wasted economy.** Broadcasting where the current battlefront is "cut idle exploration
   massively" [2025 Kragle]. Every unit should always know a useful place to go.
4. **Information is worth fighting for.** "Scouting is essential… there is no dominant strategy but rather
   counters for every approach" [2019 smite]. Knowing the opponent's composition lets you counter-build (see
   ECONOMY.md).
5. **Read the spec on communication range and cost before designing.** One 2021 finalist relayed messages
   unit-to-unit ("a lot of complication") before noticing that the base could read *any* known unit's
   broadcast at any range. Switching to a hub "worked much better" [2021 naalit]. A team learned at the
   finalist dinner that the API exposed what an enemy factory was building [2018 smite].

## 2. Choosing where to explore

### 2a. Target-selection schemes (cheapest first)

- **Random direction extended to the map edge.** Pick a random direction, set the target where it meets the
  map edge, and re-pick when near [2025 JWU].
- **Random location.** Pick a random location and re-pick within a small radius of it [2025 Om Nom].
  Better still, pick a uniformly random *unexplored* tile from the explored-bitset rows: count the zero bits,
  draw r, and walk the rows by popcount to the r-th zero. Give each target a timeout of `distance + 20`
  turns [2026 SPAARK, code].
- **Random quadrant tour.** Shuffle the four quadrants once per unit, visit a random tile in each in turn, and
  switch when the target comes into sensing range [2022 camel_case, code].
- **Fan out by spawn side.** A new unit's first exploration direction is the direction from its spawner to its
  spawn tile, so spawners spread units just by choosing where to place them [2025 JWU]. Alternatively, use
  `id % k` to choose one of k fan directions [2023 no thoughts head empty].
- **"Diffusion".** Move in one direction until blocked, then turn, with a crowd-avoidance bias. It is very
  cheap [2025 bytebyte].
- **Phased targets.** Early, use random "remote" targets (corners, edge midpoints, centre) for breadth.
  Later, go to the nearest unexplored location [2026 food].
- **Coarse grid of exploration points** [2016 future perfect, 1st]. Lay grid points at a fixed spacing around
  an origin and keep a local explored-bit per point. Next target: the unexplored point in the current cell,
  otherwise a ring search outward (radius 1..10) for the unexplored point nearest the origin. Skip points
  beyond known edges and pull points near an edge onto the map. Count a point as visited on arrival, or when
  adjacent and blocked.
- **Frontier search over a seen-mask.** Track seen cells (a bitset, or one bit per coarse cell) and BFS to the
  nearest unseen region [2019 plzgoeasy, 2017 E Doc Tablet, 2025 SPAARK]. Compute the frontier once,
  centrally, rather than having every unit run its own BFS [2017 E Doc Tablet].
- **Toward the enemy.** Bias toward predicted enemy structures (from symmetry), with randomness. Send
  long-idle units across the centre [2025 confused].
- **Zones and quadtrees.** Split the map into zones or Voronoi cells and assign units to them [2022
  TestSubjector notes, 2017 BTC]. Quadrant-centre targets spread units too thinly on big walled maps
  [2025 SPAARK].

### 2b. Movement patterns that widen coverage

- **Zig-zag.** When travelling along an axis, alternate diagonal steps if diagonals cost the same as
  orthogonal moves. This sweeps a wider band and catches features just out of sight [2025 JWU].
- **Vision-cone scanning.** With directional vision, alternate "move then turn" and "turn then move" to look
  90° left and right while advancing, without the strafe penalty. Keep looking around while being carried
  [2026 food, 3Mice, TSPAARK].
- **Edge-hugging scouts find things "from behind".** Scouts that followed the map edges reached enemy bases
  from the sides and rear, which became a deliberate flanking strategy [2021 Stone Tao].
- **Patrol the border.** Move perpendicular to the edge of enemy territory to scout it without stepping onto
  penalised tiles [2025 JWU].
- **Head toward unknown edges first.** Add a large bias toward whichever map bound is still unknown
  [2017 Omega Ruby].

### 2c. Spreading explorers out

- **Repulsion ("Coulomb's law").** For each known ally of the same role, add `force −= (ally − me)/dist²`, plus
  a 1/d repulsion from each known map edge. Normalise, accumulate into a momentum vector, and move along it.
  This spreads explorers without any assignment and makes attacks come from many angles [2021 wololo].
  Refinement: count only explorers that are at least as close to unexplored ground as you are, so the leading
  edge is not pushed back into explored territory. The momentum term produces straight outward sweeps instead
  of jitter [2021 wololo]:

  ```
  F = Σ over visible explorers e with distToUnexplored(e) <= distToUnexplored(me): (me − e) / |me − e|²
  heading = normalize(α·heading + F)
  ```

- **Repel from allies and walls, cheaply.** `target = me − Σ dir(me→ally) − Σ blockedAdjacentDirs`. If alone
  in the open, keep the last spread direction, with occasional random moves to escape corners
  [2026 SPAARK, code].
- **Heading explorer.** Target = `here + 100·(cos θ, sin θ)`. When the next few tiles along the heading leave
  the map, rotate to the nearest heading that does not, alternating left and right and widening
  [2022 AFinalsBot, code].
- **Potential-field explorer** [2016 future perfect], scored over 9 candidate moves:
  - −1000 × threat (penalise tiles in each hostile's range by `attackPower + 0.1·(range−d²)`, with a smaller
    falloff out to twice the range);
  - −2000 at the spot where we last took damage, remembered for 200 turns;
  - −50 × alignment with the summed directions to other scouts;
  - an edge penalty;
  - +100 for keeping the last direction (momentum, reset after 25 steps);
  - −128 for standing still;
  - a random start index to break ties.
- **Claim status per region.** Mark a sector "being explored" so others pick something else
  [2023 4 Musketeers]. Use **leases with timestamps**: a unit keeps rebroadcasting its zone, and a zone not
  refreshed for ~30 turns becomes free again, so the claims of dead units expire automatically [2017 BTC].
- **Timeouts against oscillation.** A unit that finds its target crowded picks another, but must not bounce
  between "crowded, leave" and "not crowded, return". Add a hold timer [2023 4 Musketeers].
- **Give up on targets that turn out unreachable.** Spend turns proportional to the distance, then
  switch [2020 Bowl of Chowder]. Abandon a site not reached within 20 turns and blacklist it: globally if our
  own structures block it, locally if only neutral obstacles do [2017 E Doc Tablet].

### 2d. After an objective falls

When a target is destroyed, units often "don't know what to do". Always have a next target: the nearest
unseen region, the next enemy base candidate, or the battlefront. Disperse to unvisited cells after a fight
[2019 plzgoeasy, 2017 E Doc Tablet]. Flag surviving enemies late in the game so they get hunted down
[2018 notes].

## 3. Sensing efficiently and inferring what you can't see

- **Sense only what changed.**
  - Precompute, per move direction, the tiles newly visible after that move [2012 fun gamers].
  - Update all visible tiles on spawn, then only newly visible tiles after each move. This trades a blind
    spot for bytecode, so also refresh critical lists, such as known build sites, each turn [2025 Kragle].
  - Mark tiles SEEN so they are never rescanned [2023 Gone Fishin'].
- **Every unit is a sensor** [2019 Justice of the War]. Each unit reports through the cheapest channel: an
  enemy sighting (combat vs civilian, plus location relative to itself), or a ping refreshing its own
  position. A forced self-location update every few turns keeps relative offsets accurate. Bases then see
  rushes coming before they arrive.
- **Use negative information.** A unit that pings "nothing here" three times running marks its area safe.
  That signal detects the end of a war, enemy base deaths, and safe expansion areas [2019 Justice of the War].
  A predicted enemy base location seen empty eliminates a symmetry (SYMMETRY.md).
- **Persistent forward observers.** One cheap unit parked just outside enemy base range does three jobs:
  - reports what the enemy builds;
  - screams back when a rush leaves;
  - suppresses enemy gathering under its guns.

  [2019 smite, 2019 Double J] Production reacts to the report, and falls back to a default if the observer
  dies.
- **Infer unseen threats from side effects.**
  - If damage taken exceeds what visible enemies can deal (`> 6·adjacentEnemies + 10`), an unseen
    long-range unit is present. Broadcast it and adapt production [2013 Teh Nubs].
  - If damaged with no visible enemy, turn around to find the attacker [2026 food].
- **Infer hidden enemy and team state from aggregates.**
  - Factor observed income into "number of producer type A × per-producer rate" when the coefficients make
    the factorisation unique [2025 Om Nom].
  - Infer active gatherers from the income delta after known spending [2022 Polar bears].
  - Read anything the API leaks about the enemy, such as resource reserves, production queues or upgrade
    states [2022 TestSubjector, 2018 smite].
- **Track entities by ID.** Initialise enemy base slots from the known or guessed start locations. The first
  sighting of an ID claims the nearest unclaimed slot, and later sightings update by ID [2017 AGRF]. Track
  enemy turrets by ID, so a moved turret updates its entry rather than duplicating it [2016 future perfect].
- **Candidate lists with a "missing" state.** For each enemy base slot, store the current guess index and an
  "overridden by sighting" bit. If a unit in range of the guess sees nothing, advance the guess. If all
  guesses are exhausted, mark the slot missing; with every slot missing, go to the centre. A later sighting
  re-adds the base [2022 Polar bears].

## 4. Representing map knowledge

- **One small integer per tile.**
  - A `char` per tile with bits SEEN, PASSABLE, RESOURCE, CLOUD and 4 bits of push direction; it serves
    pathing, symmetry and site computation at once [2023 Gone Fishin'].
  - An int per tile: known/empty/site/wall in 2 bits, structure type, friendly bit, goal-type and goal-colour
    bits written by a planning layer, a "contested" bit [2025 Kragle].
- **Bitmask rows.** One `long` per row per feature makes "seen", "wall", "site" and "explored" cheap to test,
  combine and BFS over [2025 Om Nom, 2025 SPAARK, 2026 GST].
- **Packed blocks.** Cache the map as 4×4 blocks of 16 bits in a frame centred on home, so a single message can
  carry a whole column of blocks [2012 fun gamers].
- **Sectors instead of tiles** for anything shared [2023 4 Musketeers]. Divide the map into small sectors
  (about 7 bits per id against 12 per exact location). Units go to the sector and decide locally with their
  own vision. Per sector, keep resources present, ownership, enemy presence ("control status"), claim status
  and a last-updated round. Keep global lists of combat, mining and explore sectors. Combat information
  expires after ~100 rounds.
- **Resource clusters as the unit of planning.** BFS-cluster resource tiles at init (link tiles within a
  small radius) and compute the centroids. Every unit derives the *identical ordered list*, so a 5-bit cluster
  index can stand in for a 12-bit location in messages [2019 smite].
- **O(1) spatial indices from spacing guarantees.** If the rules guarantee at most one site per 5×5 block, a
  grid indexed by `(x/5, y/5)` maps a location straight to its record. This replaced a 144-entry linear
  search that ran out of bytecode under message load [2025 SPAARK].
- **Retractable facts.** Shared facts must be deletable: broadcast "destroyed" when a reported turret is gone,
  delete exhausted resource entries lazily on arrival, and broadcast "drained" so others stop chasing "ghost"
  resources [2020 Java Best Waifu, 2020 smite].
- **Dedupe on insert.** Record a new resource tile only if it is far from every known entry. Lists of 20–30
  near-duplicates eat bytecode to iterate [2020 confused]. A neutral site turning enemy-owned *replaces* its
  entry [2023 4 Musketeers].
- **Staleness per type.**
  - Store `(loc, type, team, roundSeen)` with a per-type timeout: mobile hazards ~5 rounds, ordinary enemy
    units 0–1, enemy bases ~10 or until disproven [2026 Lorem Ipsum].
  - Stamp messages with a 2-bit round counter to age them out [2023 Gone Fishin'].
  - Pack location and round into one int so staleness can be checked [2013 devs].

## 5. Map edges and coordinates

- **Probe for edges early.** Test `onTheMap` at vision range along ±x and ±y, then step inward to find the
  exact edge. Probing stops at the first on-map tile, so it is cheap. Every unit can do it every turn
  [2015 the other team, 2016 future perfect]. In continuous space, binary-search the edge to high precision
  [2017 AGRF].
- **Use the maximum map size to bound unknown edges.** `otherEdge ≥ thisEdge − MAX_WIDTH` [2017 AGRF].
  Under rotational symmetry, one edge gives its opposite (SYMMETRY.md).
- **Share edges via the hub.** Units report edges to the base, which rebroadcasts them. A long-range unit
  that knows 3 edges goes to find the 4th [2021 waffle].
- **Coordinates when the origin is hidden.** Coordinates mod 128 (any power of two above the maximum map
  size), plus your own position, identify any location without knowing edges. One team said this "improved
  my bot efficiency 2 fold in comms" [2021 wstan2001, Stone Tao]. Alternatives: a torus grid mod a constant
  [2017 Omega Ruby], or coordinates relative to the home base or to the base midpoint [2015 the other team,
  2021 naalit].
- **Size broadcast cost to known bounds.** When message cost grows with radius, broadcast with radius equal
  to the maximum possible distance to any ally given the known edges [2016 future perfect].

## 6. Communication design for discoveries

### 6a. Topology

- **Hub and spoke.** The base, which has the most compute and often unlimited read range, reads every child's
  report and rebroadcasts the global picture. It continues next round if it runs out of bytecode
  [2021 waffle, 2021 naalit, 2020 smite]. "Send the results of a large computation rather than the base data
  on which it is performed" [2013 devs].
- **Rotating commander or primary duty.** The first unit each round to see a stale "commander" channel
  writes the round number and does the team-wide work that turn: build planning, target boards, donation
  decisions. No single death breaks the logic [2017 AGRF, 2017 Bruteforcer, 2024 Honey Ducklings].
- **Opportunistic sync for range-limited writes.** Keep a local database per unit (lazily initialised) and
  flush it to shared memory whenever in range. Units attacked away from home **run home to report** the
  danger [2023 4 Musketeers]. Get connected to the comms network first thing, so spawn knowledge flows
  [2025 JWU].
- **Relay with a gradient.** Re-broadcast a sighting only if you are farther from the source than the unit
  you heard it from, so the message propagates outward [2026 food].
- **Don't advertise to hazards.** If neutral threats or enemies can hear your messages, don't broadcast near
  them. Use the shared array for SOS calls instead of local shouts [2026 food, 3Mice]. A team that turned
  off chatty per-turn messages "broke half the bot", because counting allies and forming groups silently
  depended on them [2026 Lorem Ipsum]. Document what each message feeds.
- **Avoid multi-turn protocols between moving units.** Units drift out of range mid-message [2016
  victorious-secret].
- **Spawn-time handoff.** A newly built unit reads its role and target from its spawner: a flag on the
  spawner, an indexed slot, or simply its spawn position. This cuts comms for the most common message
  [2022 5 Musketeers, 2020 Battlegaode, 2019 smite].

### 6b. Encoding

- **Message header + payload.** 3–4 bits of type, then the payload. Use reserved value ranges or bit fields
  per type [2016 future perfect, 2019 Codelympians, 2025 JWU].
- **Relative coordinates and log-scale buckets.**
  - Pack several sightings in one word as offsets from the sender (4 bits of type, 4 bits each of dx+8 and
    dy+8) [2016 future perfect].
  - Use 4-bit log-scale size or health bins [2021 naalit, 2021 waffle, 2022 5 Musketeers].
  - Alternate absolute and delta updates [2019 Justice of the War].
- **Deterministic shared enumeration is compression.** If every unit computes the same ordered list
  (clusters, sectors, candidate spots), send indices, not coordinates [2019 smite].
- **Write an explicit bit budget for the shared array** and keep it in the repo. One year's budget
  (1024 bits, 821 used) [2023 Gone Fishin']:
  - 4 base locations × 12 bits;
  - 3 symmetry bits;
  - congestion bits;
  - a counter;
  - the 5 nearest resources per type;
  - 10 enemy sightings × (12-bit location + 2-bit round stamp);
  - island locations and owners.

  A layout with fixed slots for starting base, live bases, heartbeat bits, SOS bits, formation point and
  resource list also worked [2026 food].
- **Generate comms code from a schema.** Hand-editing packed layouts caused days of side-effect bugs
  [2023 4 Musketeers, 2022 smite]. Bit-addressable read/write helpers make the layout flexible [2026 3Mice].
- **Buffer-pool access** [2023 4 Musketeers]. When fields straddle words, read the whole shared array into a
  local copy at turn start (reads are cheap), write locally with dirty flags, and flush only the dirty words
  once.
- **Fixed-size ring buffers instead of collections.** A 120-slot array overwriting the oldest entry saved
  ~2000 bytecode against a list-based queue [2023 no thoughts]. Keep a ring per message group with a write
  index [2017 BTC].
- **Priority arbitration for a single broadcast slot** [2021 smite]. Each role owns several handlers. Each
  turn every handler proposes (flag, priority), and the highest wins, so an enemy-base sighting outranks a
  routine terrain report.
- **Flush on a schedule.** Queue new facts and flush them on rounds ≡ 0 mod 10, with 6 facts per message plus
  a validation hash; new units read only the rounds that matter [2020 confused].
- **Know who has already been told.** Track per fact which recipients have it, so relays don't resend
  [2025 SPAARK].

### 6c. Liveness and censuses

- **Heartbeat bits.** Each base toggles its bit every turn; a bit that stops toggling means that base is dead.
  Clear its entries and stop sending units to it [2022 Polar bears, 2026 food, 2026 Lorem Ipsum]. A base that
  overruns its bytecode is briefly "dead", so tolerate one missed toggle.
- **Decide what happens when a writer dies.** Every shared field should have a defined owner, a defined
  staleness rule, and a defined behaviour when the owner disappears [2022 5 Musketeers]. Stamp ownership
  claims with a round, and let the newer claim win, so stale echoes can't ping-pong [2021, practice notes].
- **Check-in channels for structures.** A missing check-in means rebuild it [2014 Darkpurple].
- **Approximate alive counts.** A unit increments its type counter on spawn and decrements it when its health
  drops below a threshold, counting itself dead before it dies [2017 AGRF]. Alternatively, units report in
  each turn to counters that reset when stale [2017 Bruteforcer]. A periodic census of idle support units
  tells spawners how many more to build [2016 future perfect].
- **Turn-order tokens.** A shared counter incremented by each base at end of turn decides who is "first this
  round" (clears accumulators) and "last" (commits results), and survives a base dying [2022 5 Musketeers].

### 6d. Security

When the enemy can read or write the same channels:

- **Validate messages.** Use signature bits (an 8-bit constant in the top byte), checksums or hashes
  [2013 Teh Nubs, 2020 confused].
- **Hop channels.** Map each logical channel to physical channels by hashing (type, round/cycle), and write
  redundantly [2013 Teh Nubs, 2013 devs].
- **Re-assert critical state every turn.** In one year an enemy could wipe the board mid-round
  [2013 Cory Li lecture].
- **Key encryption on map features and change constants before the final submission.** One team replayed an
  opponent's recorded messages and convinced it its own base was elsewhere [2020 Prasici; 2019 smite;
  2020 The High Ground].
- **Fingerprint the opponent from its message formats** and switch to a prepared counter. Store the
  identification so later games of the match start with it. This won the 2012 final [2012 fun gamers].

## 7. Turning knowledge into targets

- **Online clustering of enemy sightings** [2022 5 Musketeers]. Keep up to 3 running means in shared memory
  (sum x, sum y, count). Every sighting joins the nearest mean, or starts a new one if far from all. Units head
  to the nearest cluster, and the first base each round resets the accumulators. One scout poking into base
  vision barely moves a mean, which cured "distress-call oscillation".
- **Shared target board with decaying priority** [2017 AGRF, 1st]:
  - **Slots.** A few slots of (timeSpotted, priority, x, y). Units score what they see: high-value
    economic units high; enemy attackers seen by a threatened worker high (a call for help); divided by
    `sqrt(friendlyMilitaryNearby + 2)`.
  - **Replacement.** A report replaces a nearby slot if `old/(20+age) < new/20`. A unit that arrives and sees
    less there decays the slot.
  - **Consumers.** Pick the slot maximising `priority / (age+5) / (dist² + 10)` above a threshold; otherwise
    a random enemy start (20%) or a random nearby point (80%).
- **Event queue plus visited grid** [2017 E Doc Tablet]. A persistent fog-of-war grid in shared memory, one int
  per 10×10 cell, plus a queue of encoded events ("enemy spotted", "worker under attack"). Combat units pull
  from the queue, so they converge on recent sightings or rush back to defend.
- **Danger targets with expiry** [2022 camel_case, code]. Any unit that sees more enemies than allied
  attackers able to reach them writes a "danger" slot carrying an expiry counter. The base decrements the
  counters each round. Attackers go to the nearest danger target before any base target, so fights pull in
  reinforcements automatically.
- **Priority queue of enemy bases by weakness.** Attack the cheapest-to-take first [2021 3 Musketeers].
- **Battlefront broadcasting.** Share the nearest active front [2025 Kragle]. Rally on the enemy target
  closest to your army's centre of mass, and push the enemy base to last [2014 that one team].
- **Idle defaults.**
  - After N turns without seeing an enemy, charge the nearest reported threat or the enemy base, which is also
    the source of raiders [2021 Baby Ducks].
  - The base periodically fakes an enemy report at the guessed enemy base to pull idle attackers there
    [2023 no thoughts].
  - Switch the army target periodically to break stalemates [2024 Honey Ducklings].
- **Resource discovery broadcasts.** Gatherers broadcast the richest site seen, so others converge on it, and
  re-survey around themselves on arrival [2015 Ayyyyyyyylmao]. Explore through costly terrain, because
  resources hidden behind it are often unclaimed [2022 5 Musketeers].
- **Discovery memory per unit.** Each gatherer remembers its drop-off point and every resource tile it has
  seen, and walks to the nearest. The base rebroadcasts resource knowledge periodically [2020 smite]. After an
  interrupting errand such as a refill, return to the remembered task location [2025 JWU].

## 8. Checklist

1. A map memory (one int or char per tile, plus bitmask rows) updated incrementally as vision changes.
2. Edge discovery and a coordinate scheme that works before edges are known.
3. Symmetry tracking (SYMMETRY.md) feeding enemy base candidates and mirrored resources.
4. A cheap default exploration policy (random targets, fanned by spawn side), with a frontier fallback and
   centre/edge balance checked on large sparse maps.
5. A shared-memory layout with a written bit budget, message types, staleness stamps, heartbeats and
   dedupe, generated from a schema.
6. A target board or sighting clusters, so every idle unit has somewhere useful to go.
7. Forward observers and side-effect inference for the information vision can't give.

---

## Sources

Post-mortems and write-ups (read in full unless noted): 2012 fun gamers (1st) and the 2013 MIT 6.370
lectures; 2013 Teh Nubs (code); 2014 that one team (1st, code), Darkpurple; 2015 the other team (1st, code),
Ayyyyyyyylmao; 2016 future perfect (1st, README + code), victorious-secret; 2017 Arbitrary Graph Restoration
Fund (1st, code), Omega Ruby (2nd, code), Bruteforcer (4th, code), BTC (notes + code), E Doc Tablet; 2018
smite, team notes; 2019 smite (1st), Justice of the War, Double J, plzgoeasy, Codelympians; 2020 Java Best
Waifu (1st), smite (2nd), Battlegaode (3rd), Bowl of Chowder, confused, Prasici (second-hand); 2021 Baby
Ducks (1st), naalit, waffle, wstan2001, Stone Tao, wololo (code + second-hand), smite design note,
3 Musketeers (summary); 2022 5 Musketeers, Polar bears, TestSubjector, smite notes; 2023 Gone Fishin' (2nd),
4 Musketeers (3rd), no thoughts head empty; 2024 Honey Ducklings; 2025 Just Woke Up (1st), confused (2nd),
Om Nom (3rd), SPAARK, The Kragle, bytebyte; 2026 Generalized Stroke's Theorem (2nd), food, Lorem Ipsum,
3MiceWalkIntoABar, TSPAARK.
