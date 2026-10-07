# ECONOMY — what post-mortems teach about resources, production and spending

The economy is everything that turns map resources and time into units: gathering, choosing what to build
and when, where to build it, coordinating several producers, and converting surplus into whatever wins the
game. Rules change every year, but the same economic patterns keep deciding games:

- snowballing openings;
- floating resources;
- payback arithmetic;
- shared-bank coordination;
- tiebreak dumps.

This file collects what 2009–2026 post-mortems recorded about the economy, keeping only what carries across
rule sets. Sources are cited as `[year team]`, and a source list is at the end. The companion files are
NAVIGATION.md, EXPLORATION.md, COMBAT.md and SYMMETRY.md.

---

## 1. Principles that recur every year

1. **Find out what actually wins.** Read the win condition and the full tiebreak order on day 1, and play one
   starter-bot game to see how games end.
   - One team won a year on the tiebreak metric (total unit health) by buying the best HP per cost:
     "Surprisingly, almost no one copied" [2019 smite].
   - Teams lost games by not dumping their bank into the tiebreak at the end [2017 E Doc Tablet,
     2019 Wololo].
2. **Know what counts as a resource.** A quantity is worth optimising when [2021 wololo]:
   - (a) having none of it is close to losing;
   - (b) you can take it from the opponent;
   - (c) the opponent can make it hard for you to get.

   Under this test, **unit count** is often a resource separate from the currency that buys units. Forcing
   the opponent into bad currency-for-units trades opens a whole strategic layer. A quantity that fails (b)
   is a constraint, not a resource.
3. **The opening snowballs.**
   - Small early leads become harassment advantages that "snowball into a decisive economic advantage"
     [2020 The High Ground, 2020 Bowl of Chowder].
   - After a balance patch shortened games, "the beginning few workers and the first factory mattered
     greatly". An untuned opening sank a sprint finalist to 9th [2018 smite].
   - Copying the top teams' opening queue made one finalist's rank jump "instantly" [2021 Stone Tao].
   - Compare your first 50 rounds of events side by side with the strongest opponent's.
4. **Do the arithmetic before coding.** Every top team computed:
   - **Payback time and doubling time** of every income source. One example: cost 500 for +2 a turn pays back
     in 250 turns and doubles in about 175. Compare that with the remaining game length or the source's own
     lifetime [2020 Battlegaode, 2020 The High Ground].
   - **Break-even round** of an economic building. One supplier broke even at round 90; a third gained nothing
     because of rounding [2013 Cory Li lecture].
   - **Marginal return of "build another" against "upgrade"**: compare income added per unit cost. A new
     building beats an upgrade whenever a new site exists [2025 Kragle].
   - **Value per cost of each unit**: DPS per cost, HP per cost, mobility per cost, carrying capacity per
     cost. The best-value unit usually defines the meta, and one 2012 winner never built the "useless" unit at
     all [2012 fun gamers]. Compute value per resource before investing in exotic units [2023 Gone Fishin'].
   - **Cost to kill against cost to build**, including action costs. A unit that costs 50 to build but takes
     100 of the enemy's ammunition to kill bankrupts the defence [2019 Double J].
   - **HP per turn**: produced by a building (healer, producer) or removed by a weapon. Compare them all in
     one unit [2013 devs lecture].
   - **The ROI curve when unit size is a free parameter.** Compute it once and use its optimum
     [2021 naalit, paulmure].
5. **Simulate the economy offline.** A spreadsheet or Mathematica model of income, upkeep and build order,
   plotted as "army size against round" for each build, showed:
   - which builds starve themselves;
   - the window in which a rush build outnumbers an economy build (round ~100–300 in one year);
   - that interleaving research with units is worse than doing them in sequence.

   [2013 devs lecture]
6. **Floating resources are not a lead.** "Losing with loads of money" is the classic symptom. Watch for three
   kinds of float:
   - a bank that can't be spent fast enough because production is capped per turn;
   - stock split across many places, each below its usability threshold. Ten structures each holding 70 when
     building needs 100 is 700 floating [2025 Kragle];
   - resources stockpiled under decay, where stockpiling for 500 turns gained only ~30% [2013 devs].

   Fixes:
   - give surplus a secondary sink [2022 5 Musketeers "obesity" state];
   - have consumers leave exactly what producers need [2025 Kragle];
   - convert the surplus (§6).
7. **But banking for the right moment beats dribbling.**
   - Arriving with a banked burst beats a trickle [2020 Kryptonite].
   - "Bank then burst" was a 2020 top-bot pattern.
   - Strong bots defer spending until the moment of highest yield, such as the first big fight.
   - Building just in time at the point of combat lets you choose the unit mix after seeing the enemy
     [2012 fun gamers "battery mode"].

   The rule: spend continuously on things that compound; bank only for a known, near, high-yield purchase.
8. **Production rate is often the binding constraint, not currency.**
   - "One base will have a hard time beating five enemy bases even if it has 10 times the influence"
     [2021 Baby Ducks].
   - When spawning is capped per building, capturing or building more producers beats banking.
   - Add production buildings only when every existing one is busy and resources pile up [2015 the other team].
9. **Rush, economy and turtle form a triangle.**
   - "Aggressive expansion beats less aggressive expansion… rushes beat aggressive expansion" [2019 Wololo].
   - Rush beat the long-term win condition on small maps, economy beat rush on large ones, and the long-term
     plan beat economy on some maps [2013 devs].
   - In rush against rush, "the side that does NOT defend usually wins", because defending spends what offence
     needs [2020 confused].
   - Choose per map, from measurements (§4).
10. **Every action that costs a shared resource is an economic decision.** When moving, attacking or messaging
    costs the same currency as units, idle units are cheapest. One year's winner turtled ("do nothing") to
    convert every saved action into more units [2019 smite]. One year even refunded unused compute as currency
    [2012 fun gamers].

## 2. Gathering

### 2a. Assignment

- **Centralise worker-to-site assignment.** When each gatherer picked the nearest free site, two raced for the
  same one and the loser wasted ~25 turns. The fix: the spawner assigns the exact site at birth and tracks
  per-site status (open / on the way / developed / fortified / enemy) [2019 smite, 1st]. The single-program
  year's winner solved worker→job assignment with the **Hungarian algorithm**. Each big job was offered as 5
  slots of decreasing value (1.0, 0.8, …, 0.2), so several workers could share it with diminishing returns.
  It used greedy matching above 30 workers [2018 Orbitary Graph, 1st].
- **Deterministic shared enumeration.** BFS-cluster resource tiles at init. Every unit computes the identical
  ordered cluster list, so "go to cluster 7" is a 5-bit order [2019 smite].
- **A fixed per-unit type at spawn.** Assign each gatherer a resource type when it is born, using a counter
  that resets periodically, so the type ratio stays exact [2023 Gone Fishin', 2023 no thoughts].

### 2b. Site choice

- **Site scoring.** Yield × safety ÷ distance:
  - `value = distanceToMiddle · ore³ / distanceToGatherer`. Without the safety term, gatherers walked greedily
    into the middle of the map and died [2015 Ayyyyyyyylmao].
  - Mine where you stand while the yield is above a threshold. Otherwise ring-search outward, starting on the
    home side, for the closest valid cell that is unoccupied, outside enemy turret range, and has no enemy
    fighters near. If nothing is found, lower the threshold and retry [2015 the other team].
  - Score = `value − 20·dist − 10·turnsToClear − 100·occupied`, with a separate hard priority for "jackpot"
    items [2016 future perfect].
- **Developed-site placement** [2014 that one team]:
  - Sample on a coarse grid, cells closer to us than the enemy only.
  - `score = Σ local yield`, multiplied by:
    - a safety term `(1 + (distCentre − 0.5·distOurBase + 0.5·distTheirBase)/mapSize)`;
    - ×0.5 near the midpoint;
    - ×0.95 per adjacent wall;
    - an edge penalty.

### 2c. Depots and carrying

- **Depot (drop-off) placement rules.**
  - Build when no depot is within R and at least T resource is visible nearby [2020 Java Best Waifu].
  - Build when the round trip costs more than the depot pays back [2019 smite].
  - Launch further expeditions from the nearest existing depot [2019 Justice of the War].
- **Carry policy.**
  - Return at a cargo threshold.
  - While returning, keep collecting up to a cap that rises as the distance to the depot falls.
  - Ignore small piles unless already in range.
  - Tune the cap to the movement penalty of carrying. One team gained a lot by raising its cap when the penalty
    turned out small. [2026 food, 2026 Lorem Ipsum]

### 2d. Shared knowledge and denial

- **Gatherer memory and ghost resources.** Each gatherer remembers every resource tile it has seen and walks
  to the nearest. Entries are deleted lazily on arrival, and a "drained" message stops others chasing ghosts
  [2020 smite]. Broadcast the richest site seen [2015 Ayyyyyyyylmao].
- **Regrowing resources: farm at home, strip near the enemy.** If a deposit regrows only while some remains,
  leave a remainder near home and mine the enemy side to zero [2022 5 Musketeers, Polar bears,
  TestSubjector]. One rule: strip a site when `dist(our base)/dist(enemy base) > 1.2`.
- **Deny the enemy's sites.**
  - Camp the mirror images of your own sites (SYMMETRY.md).
  - Body-block enemy gatherers at contested sites [2023 4 Musketeers].
  - Harass the three best enemy-side clusters from turn 4 [2019 smite].

### 2e. Traffic and type balance

- **Site etiquette and congestion** (see NAVIGATION.md §7).
  - Too many gatherers clog the base and *reduce* income.
  - Stop gatherer production at every base when a base senses crowding [2023 Gone Fishin'].
  - Count the usable tiles around each site and mark it congested when all are occupied, clearing the mark on
    the next delivery [2023 Gone Fishin'].
  - Throttle production on local unit density, not just affordability. First check that congestion actually
    binds in the current game.
- **Type balance with adaptive ratios** when there are several resource types [2023 4 Musketeers,
  2023 Gone Fishin', 2023 no thoughts]:
  - Hold a target ratio, and each new gatherer takes the type that is under target.
  - Shift the ratio by map size and enemy distance: rush-relevant resource on small or close maps, the
    long-term one on big maps.
  - Shift it by what has been found (only one type seen → mostly that type).
  - Shift it by threat (switch gatherers to the military input when a fight starts near home).
  - Mine whichever resource is currently the bottleneck, and stop mining one above a bank cap
    [2019 Codelympians, notes].
- **Taxi and transfer.** Moving resources toward the front by unit-to-unit transfer, a resupply unit or
  "taxi" carriers works only where the rules make it cheap. Compute its value against direct delivery
  [2015 the other team, 2026 GST, 2026 food].
- **Upkeep payback.** Skip self-maintenance actions whose payback is negative early. One example: an action
  costing 5 to save 1 per turn, skipped before round 150 [2025 confused].

## 3. Build orders and production mechanisms

Hard-coded if-chains get brittle. These mechanisms scaled well:

- **Composition-driven thresholds** [2020 The High Ground]. Each buildable type has a minimum bank, raised the
  more that type is over its desired share. They reported a "cleaner build order" than the winner's.

  ```
  for t in types: threshold[t] = base[t] * f(actual[t] / desired[t])     // f increasing
  build any t with bank >= threshold[t]
  ```

- **Score-sorted purchase queue with "save for the top item"** [2018 Orbitary Graph, 1st]. Each turn, list
  every possible purchase as (score, cost, action) and sort by score. Buy while affordable. At the first
  unaffordable item with positive score, **stop buying anything that costs money** and save for it. Scores
  diminish with current counts, for example `2 + 20/(10 + count)`. Add bonuses for counters to what the enemy
  shows, and drop items that could not pay off before the game's end.
- **Additive "desire" scores run by a rotating commander** [2017 Bruteforcer, 4th]. Desire per type is built
  from shared counters: enemies seen, map size class, resources visible, stuck units. One example: a big
  bonus for a utility unit when its fast payback is visible. "More than 40% of attackers stuck" raised desire
  for terrain clearers. A floating-bank reserve "actually makes us build fewer units" and was dropped.
- **Proportional production rules.**
  - Producers ∝ income sources (gardeners ≈ 0.43 × trees) [2017 E Doc Tablet].
  - Soldiers grow sublinearly: build while `(income + 1)^0.9 > soldiers` [2017 AGRF].
  - A utility unit is built when `cost / Σ(visible value) < payback horizon` [2017 AGRF].
  - Military production is tied to economic buildings (1 attacker per income building), then switches to a
    fixed ratio after a set round [2020 Java Best Waifu].
- **Decide the next unit, save for it, then build it.** Round-window rules such as "type X only on rounds
  ≤ 20 mod 50" overproduce. Deciding first holds a fixed ratio all game [2025 Om Nom].
- **Modulo cycles with overrides** [2016 future perfect, 1st]. Cycle {A, B, A, C} with phase-based cycles.
  Overrides for "fled recently", "army below N" and "losing late → build high-variance units to make the game
  more random".
- **Income target before attacking** [2021 Baby Ducks, 1st]. Build economy units until an income target is
  reached, with guard units proportional to the economic units. Above the target, build attackers. The target
  adapts: many guards alive and idle means a slow, defensive map, so raise it. Don't spend everything on big
  attackers once an enemy base is known if the map is defensive.
- **Spending escalation for discretionary structures.** Raise the bank threshold for each extra one, and use
  a **"desperation index"** for placement: count consecutive turns in which you wanted and could afford X but
  didn't build it, and relax the allowed sites in stages as it grows [2020 Java Best Waifu].
- **Use production capacity for tech when upkeep is saturated.** Research only when more units can't be
  sustained, so the producer never idles [2013 Teh Nubs, 1st].
- **Population targets depend on map size and trade rate.** A fixed "keep ~16 alive" meant paying more per
  traded unit than the enemy on small combat maps. Scale with map size and use income rather than bank
  [2026 food]. Cap worker replication [2018 notes].

### 3a. Choosing a plan per map

- **Classify on turn 1 with cheap latches**:
  - map size;
  - distance or path cost between bases;
  - obstacle density, or free space by Monte-Carlo sampling (100 random points in vision, count blocked or
    off-map) [2017 Omega Ruby];
  - resources visible;
  - any known schedule of future events [2016 Polar Vortex].

  Map each class to a scripted opening. One 2×2 example [2017 BTC]:

  |  | Open | Cluttered |
  |---|---|---|
  | Small | soldier-first | clearer-first |
  | Large | scout/eco-first | eco + clearer |

- **Measure the rush distance as path cost, not Euclidean distance.** The devs suggested counting obstacles
  in a band along the base-to-base line [2013 devs]. The 2014 winner chose RUSH under one map size and
  otherwise scored economic sites, falling back to RUSH if the best site was poor [2014 that one team].
- **Re-evaluate continuously.** Rush feasibility came from scout reports of distance and terrain to the enemy
  [2021 wololo]. Gathering ratios flipped toward rush automatically once the guessed enemy base turned out
  close [2023 4 Musketeers]. One runner-up kept a second, more defensive doctrine for the larger final-round
  maps [2025 confused].
- **Feedback from the game itself.** "Lost early last game, so play more defensively" [2017 BTC notes].
  Infer map tempo from your own units' behaviour [2021 Baby Ducks].

### 3b. Adapting to the opponent

- **Counter-production from scouting.** A forward observer reports enemy production, and spawners switch
  types (enemy A → build B). Default when blind [2019 Double J, 2019 plzgoeasy]. Read any API exposure of
  enemy production or reserves [2018 smite, 2022 TestSubjector].
- **Time the counter to the opponent's spending** [2020 Java Best Waifu]. Build the unit that invites an
  expensive counter-purchase only when the enemy has just spent (it can't afford the counter). Make the
  enemy's answer cost more than your probe.
- **Rush response must not freeze the economy.**
  - Put a timeout on emergency modes. An "extended rush" stalemate froze one team's economy until they forced
    the build order back to normal at a fixed round [2020 The High Ground].
  - Cap reactive defensive spawning. "Emergency defence" overproduced late game, fed units into enemy lines,
    and an opponent drained them with sustained pressure [2019 smite].
  - Check that the response can actually engage the threat: one team panicked into melee units against a
    ranged unit they could never reach [2019 Double J].
  - Treat every unit type as a possible rush, not just the one you tested [2019 Double J].
- **Global all-in switch.** When an offensive becomes winnable, broadcast HALT. Every defensive producer stops,
  and ~100% of income goes to the offensive for ~100 rounds [2020 smite]. The opposite version: while a
  committed rush is alive, add an artificial cost to every non-rush purchase [2020 confused].
- **Coordination-heavy economic plans flop.** "Every time I'd implement a sophisticated strategy that requires
  a lot of coordination it would always flop" [2020 Java Best Waifu]. Prefer rules every producer can apply
  on its own.

### 3c. Several producers, one bank

When several spawners draw from a shared bank and act in sequence, greedy spending by whoever acts first
starves the priority item.

- **Designated producer.** Every spawner computes, from shared information, which of them should build the
  priority unit this turn. The others **skip their turn** so it can afford it [2019 smite].
- **Prioritised fair rotation** [2022 5 Musketeers]. Order the spawners: those nearest the fight first, then
  least-recently-chosen. Each publishes its intended next build cost and a "can actually build" bit. Then
  each runs:

  ```
  total = bankAtTurnStart                               // = current bank + spentThisRoundSoFar
  for a in order:
      if a == me: build; break
      if a.canBuild: total -= a.nextCost
      if total < myCost: skip this turn
  ```

  Update "last chosen" only if the spawner really built, or turns starve.
- **Probabilistic turn sharing (simplest).** On its turn, spawner i (0-based, in turn order) builds with
  probability `(i + 1) / spawnerCount`. Otherwise it repositions to better terrain and saves. Spending then
  spreads across spawners without any messages [2022 camel_case, code].
- **Elect the best-positioned producer.** Each base shares a "crampedness" or build score, and only the least
  cramped hires [2017 AGRF, 2017 Omega Ruby].
- **Turn-order tokens.** A per-round counter tells each base whether it acts first (reset accumulators) or last
  (commit), and keeps working when a base dies [2022 5 Musketeers].
- **Mobile or forward producers.** Move production structures toward the fight so units heal and reinforce
  with a short walk. Move one at a time, only when no enemies are in sight, and only producers not
  prioritised that turn [2022 5 Musketeers].
- **Roles go through per-spawn channels.** A single shared "role" flag limited one team to one gatherer per
  round. Encode the role in spawn position or round parity, or give each spawn its own channel
  [2023 4 Musketeers].

## 4. Expansion, territory and placement

- **Develop only the territory you can defend for the whole game** [2020 Bowl of Chowder]. Terraforming a
  capped area let its defences rise higher and survive a rising hazard longer than larger developments. The
  bot's late attack met opponents whose economies had already collapsed. Its weakness was low-resource maps,
  where the cap was too big.
- **Expansion pipeline** [2019 Justice of the War]:
  - Send gatherers quickly to near and far sites; a gatherer travelling far builds a depot there.
  - Later expansions launch from the nearest existing depot.
  - Allow at most 2 concurrent expeditions.
  - Race the *central and contested* sites first, then the close ones.
- **Escorted or forward expansion.** Escort the first expeditions [2019 plzgoeasy]. A gatherer that loses the
  race to a site builds a forward producer there anyway and contests it [2019 smite]. A forward production
  building near the enemy keeps your economy running while you attack [2018 Orbitary Graph].
- **Packing structures without self-blocking.**
  - **A global lattice of build sites.** Producers go straight to assigned grid points, which avoids congestion
    while keeping density. Always leave 1–2 gaps in a ring of structures so units can be built and leave
    [2017 E Doc Tablet, AGRF].
  - **Dynamic packing.** Any builder may start a pattern centred on itself if it doesn't conflict, and each
    completion proposes the next candidate positions 4 tiles away in each cardinal direction, disqualified as
    soon as an obstruction is seen. This beats index-based tilings on cluttered maps but is worse on open
    maps [2025 Om Nom, 2025 SPAARK].
  - **Exclusion refcounts.** A map-sized int array: increment around each cause (structure, wall, pattern
    under construction), decrement when it goes away. "Can I build here?" is one read [2025 JWU].
  - **Reserved zones.** Producers publish their footprint so other units path off it [2017 AGRF].
- **Agree on what to build without comms.** Coordinate-hash or count-parity rules flip when counts change or
  depend on luck. Instead, the first unit to arrive chooses the type from the desired ratios and **marks it
  physically** in the world [2025 JWU, 2025 SPAARK].
- **Static defences only where they pay.** Build at chokepoints near the centre, below a cap, when enemies are
  actually seen. A/B tests against your own bots can show nothing, while real opponents punish their absence
  [2025 JWU, 2025 SPAARK]. Space defensive structures apart, and relax the rules near the enemy base
  [2020 The High Ground]. Price turtle layouts in advance as templates [2016 foundation].
- **Denial is often cheaper than destruction.**
  - **Taint.** Disrupt one tile of an enemy construction pattern so it needs a special unit to finish
    [2025 confused, Om Nom, JWU].
  - **Bury a spawner.** Occupy every tile adjacent to an enemy spawner so it can't place units [2021 wololo].
  - **Mercy-kill your own dying structure** so the enemy gets no kill bonus [2014 that one team].
- **Punish over-extension.** Attack when the enemy has developed sites outside its defended zone, or is
  out-producing you from a site you can reach [2014 that one team]. When the enemy commits to a visible
  economic building it is temporarily down units. Attack it with everything [2014 schnitzel]. Farm
  "silently" if the scoring building is what draws attacks [2014 schnitzel].

## 5. Retreat, repair and replacement as economics

- **Refill or retreat versus replace.** Compare the cost of a unit's round trip home (including the turns it
  doesn't work) with the cost of a new one. Top teams let cheap units die rather than refill them, and
  preserved only expensive ones [2025 Kragle].
- **Finish the job before retreating.** Units returning at a fixed threshold were often one hit short of
  destroying a structure, wasting ~10 attacks. "Finish the kill first" was a large gain, and so was "don't
  retreat unless near a refill point" [2025 Om Nom, 2025 SPAARK].
- **Heal queues need starvation control.** "Heal lowest-HP first" left some units waiting over 1000 rounds.
  Cap the number healing and add a timeout, or heal the least-damaged first so it returns sooner
  [2022 5 Musketeers, 2022 TestSubjector].
- **Healing against rebuilding.** Compare HP bought per resource. Healing beat rebuilding 2:1 against 3:1 in
  one year [2012 fun gamers].
- **Specialists against generalists.** Fragile specialist roles died in 1-against-3 fights. Making every unit
  a gatherer that also explores improved survival [2026 Lorem Ipsum]. But when the engine rewards experience,
  concentrating it on a few specialists can unlock discounts that spreading never reaches [2024 top-bot
  replay study].

## 6. Conversions, exploits and free lanes

- **Look for mechanics that convert surplus into the scarce resource.** One example: a structure that spawns
  with free stock, so destroying and rebuilding it converts the plentiful currency into the scarce one
  ("tower flickering/farming"). Om Nom spent 13,000 banked in under 60 rounds and went from 8 units to 23.
  confused got a 70%+ win rate against its previous bot [2025 Kragle, Om Nom, confused, JWU, SPAARK]. It was
  spotted by **watching another team's replay** [2025 confused].
- **Dead units that drop resources** can bootstrap an expensive unit, or create regrowing farms near home
  [2022 5 Musketeers, smite].
- **Exponential loops in the spec.** A compounding buff was abused into effectively infinite resources before
  a patch. Exploit broken loops while they last, but keep a non-exploit path ready, because bots tuned to the
  exploit crashed after the patch [2021 Stone Tao].
- **Free side lanes.** Income from a side mechanism that never competes with the main economy is free value
  that most opponents leave untaken [2021 wololo].
- **Global versus local resources.** Produce the global one in safe places, then convert it where the local
  one is needed [2025 Kragle].
- **Infer hidden state from income.** Factor observed income into "number of producers × rate" when the
  factorisation is unique. Build the missing producer type when the inferred count hits 0 [2025 Om Nom].
  Infer active gatherers from the income delta [2022 Polar bears].

## 7. Endgame and tiebreak economics

- **Know the tiebreak order and pre-fund it.** Keep a cheap end-game routine that wins it:
  - mint a little of the higher-ranked resource [2022 5 Musketeers, TestSubjector];
  - build the counted cheap structure in the last 150 rounds [2015 the other team];
  - mass-spawn at the end [2026 food];
  - spend all resources on the final turn [2019 smite].

  On "jail" maps where every game goes the distance, the stockpiler wins [2023 4 Musketeers].
- **Compute when to switch to the best tiebreak value per cost**, from remaining rounds × income, so nothing is
  left unspent. One team lost two finals games by switching too late [2019 Wololo, Shadow Priests,
  plzgoeasy].
- **Points races.** When an alternative win condition can be bought, compute time-to-win: `remaining × price /
  income`. Go all-in once it is short or once projected income guarantees it: `VP + (bank + projected)/price
  > target` [2017 AGRF, 2017 Omega Ruby]. Don't float a bank when holding cash reduces income. Keep a few
  units alive and hidden for the tiebreak [2017 AGRF].
- **Close the game when ahead and hazards escalate.** The leading side should push to end the game rather than
  bank on an uncertain late game [2016 future perfect].

## 8. Compute is part of the economy

- **When compute costs resources, idle cheaply.** In years where unused bytecode was refunded, or bytecode
  cost power, a 69-bytecode "hibernation" loop roughly doubled the sustainable army [2012 fun gamers]. Using
  5000 bytecode per unit instead of ~0 cut a sustainable army from 40 to 27 [2013 devs].
- **Never exceed the budget on a unit's first turn.** Large allocations at spawn cost every new unit its first
  turn, at the moment snowballing matters most. Fixing it was a substantial win [2025 Om Nom].
- **A skipped turn in the economy loop is invisible.** Log bytecode per stage, and flag overruns.

## 9. Checklist

1. Tabulate the cost, income, payback and value-per-cost of every unit and building. Note the win condition and
   full tiebreak order.
2. Build a tiny offline economy model and plot army size against round for 3–4 candidate openings.
3. Implement production as thresholds, scores or desires driven by counts and map latches, not a fixed list.
   Include a "save for the top item" rule.
4. Add shared-bank coordination if there are several producers.
5. Add gatherer assignment, site scoring, depot rules and congestion control.
6. Detect floating resources and give them a sink or a conversion.
7. Add rush detection with time-boxed, capped responses, and an all-in switch.
8. Add an end-game routine that wins the tiebreak.
9. After every balance patch, re-tune the opening first.

---

## Sources

Post-mortems and write-ups (read in full unless noted): 2012 fun gamers (1st) and the 2013 MIT 6.370
lectures (numerical strategy, sprint lessons); 2013 Teh Nubs (1st, code); 2014 that one team (1st, code),
schnitzel, Darkpurple; 2015 the other team (1st, code), Ayyyyyyyylmao; 2016 future perfect (1st), Polar
Vortex, foundation, #trump2016 notes, BLEAKFORTUNE, victorious-secret; 2017 Arbitrary Graph Restoration Fund
(1st, code + commits), Omega Ruby (2nd, code), Bruteforcer (4th, code), BTC notes, E Doc Tablet, Segfault,
Volatile; 2018 Orbitary Graph (1st, code), smite, obas, team notes; 2019 smite (1st), Justice of the War,
Double J, plzgoeasy, Codelympians, Shadow Priests, Chicken, Wololo; 2020 Java Best Waifu (1st), smite (2nd),
Battlegaode (3rd), The High Ground (4th), Bowl of Chowder, confused; 2021 Baby Ducks (1st), wololo (via
practice-repo summaries and code), Stone Tao, naalit, waffle, wstan2001, paulmure notes; 2022 5 Musketeers,
Polar bears, TestSubjector, smite notes; 2023 Gone Fishin' (2nd), 4 Musketeers (3rd), no thoughts head
empty, don't @ me (snippets); 2024 replay studies of top bots (practice repository); 2025 Just Woke Up (1st),
confused (2nd), Om Nom (3rd), SPAARK, The Kragle; 2026 Generalized Stroke's Theorem (2nd), food, Lorem Ipsum,
TSPAARK, nfgehrs.
