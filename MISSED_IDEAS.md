# MISSED_IDEAS — cross-year principles that would have caught what we missed

Seven practice seasons (2020-2026) each ended with a list of ideas the top teams used and we never found:
`yearly/MISSED_IDEAS_<year>.md`. Read side by side, the misses are not random. The same few habits of thought
hid them every year. This file names those habits. Each principle is stated year-agnostically, then lists
the misses it would have prevented, then says how to apply it. The checklist at the end turns them into a
week-one routine.

The principles complement ADVICE.md and the topic guides (COMBAT, ECONOMY, NAVIGATION, EXPLORATION,
SYMMETRY). Those say what works. This file says why we did not find it.

**The pattern in one paragraph.** We were good at measuring and poor at imagining. Our gates were
stricter than any top team's, so almost nothing we tried was a false accept. But the ideas never entered
the gate: we read an engine fact without pricing it, closed a direction after one implementation, judged
field-facing ideas against ourselves, and tuned fixed constants where top teams computed state-dependent
rules. Most of what we missed was visible in replays, recurred across years, or was one engine call away.

---

## Index

1. Price every action by the state it creates, not the state it destroys
2. An idle resource means a conversion exists; find it
3. A rejection closes an implementation, not an idea
4. An obstacle you measured is a design requirement, not a verdict
5. Judge field-facing ideas against the field
6. Every constant should be asked what it depends on
7. Estimate hidden global state from what you can observe
8. Control to targets, not caps
9. Model exactly what the enemy can do next turn
10. Attack the opponent's economy, not only its win condition
11. Give parked units a lattice
12. Symmetry is free information; get it cheaply and use it everywhere
13. Communication should carry what units cannot see
14. Plan production; reserve resources for the plan
15. Small fixed specialist sets beat proportional roles
16. Units are expendable; survival is a means
17. Script the end of the game against the tiebreak
18. Unused bytecode is unused intelligence
19. Copy structures, not tactics
20. Recurring tricks are a checklist, not a discovery

Appendix: the week-one checklist.

---

## 1. Price every action by the state it creates, not the state it destroys

**The habit that hid it.** We read self-destruct and conversion calls as losses. "Disintegrate is a pure
suicide with no refund" was a correct engine reading and a wrong economic one.

**What it would have caught.**
- 2025: tower flickering. A money tower rebuilt every 50 turns converts chips into 500 fresh paint. Five
  of five top teams did it while we sat on 100k+ chips and starved for paint.
- 2022: lead farming. A builder that disintegrates on an empty tile plants a deposit that regenerates
  for the rest of the game. Every top team used it; our bot never called `disintegrate`.
- 2022: sacrificing a far, idle archon for its lead.
- 2020: disintegrating units to clear drop tiles before a crunch; drowning dirt-carrying landscapers to
  build an island.
- 2026: a carried rat disintegrating to deny the landing kill.

**How to apply.** For every engine action, write down the full world state after it, including what the
engine spawns, refunds, resets or regenerates as a side effect. Ask "what new object or tile does this
create?", not "what do I lose?". Do it for actions that look useless or harmful first: they are the ones
nobody else in your head has already priced.

## 2. An idle resource means a conversion exists; find it

**The habit that hid it.** When one resource piled up unspent, we treated it as surplus and kept tuning
the binding resource. We even wrote "the binding resource is X; Y is not" as a lesson.

**What it would have caught.**
- 2025: chips idle, paint binding. Flickering, money-tower-first builds, and drains from friendly towers
  all turn chips into paint.
- 2022: thousands of lead unspent while gold bound. More laboratories, placed where transmutation is
  cheap, and watchtowers as the lead sink.
- 2020: thousands of soup unspent after the flood. Vaporators, net guns and drones were the sink.
- 2021: the bank idle in losses. Buffrakers, a standing safety budget, and income-targeted builds.
- 2023: banked mana with no endgame plan.

**How to apply.** Instrument every resource's stock over time from the first day. If a stock grows while
another binds, list every engine path from the idle one to the binding one, including indirect ones
through building, destroying, rebuilding or upgrading. Treat "no conversion exists" as a claim to prove
from the engine, not a default.

## 3. A rejection closes an implementation, not an idea

**The habit that hid it.** After one or two failed implementations we wrote "direction closed". The top
teams made the same direction work with a different gate, timing or representation.

**What it would have caught.** Each year's section 2 lists these. The common cases:
- SRPs (2025), messaging (2025), squeaks (2026), amplifiers (2023): our gate or trigger was wrong, not
  the mechanic.
- Multiple kings (2026): ours were defensive musters; theirs were placed on cheese mines and gated on
  population.
- Throws and ratnaps (2026), blind fire (2023), kill steps (2023): theirs were scored enumerations; ours
  were single rules.
- Launcher cohesion (2023), escorts (2024): theirs were conditional and spaced; ours were always on and
  crowded.
- Symmetry inference (2022, 2025): closed on cost; theirs was a cheap by-product of every unit's turn.
- Defense towers (2025), vaporators and net guns (2020), labs (2022): the condition for building them,
  not the unit, was what we had wrong.

**How to apply.** Before closing a direction, write three alternative designs that differ in trigger,
target, and representation, and say why each would also fail. Record the closed implementation, not the
closed idea. Re-open any closed idea that a stronger opponent is visibly using in replays.

## 4. An obstacle you measured is a design requirement, not a verdict

**The habit that hid it.** We were very good at measuring why something could not work, and then we
stopped. The measurement usually named exactly the constraint the top teams designed around.

**What it would have caught.**
- 2025: "a unit is in message range of a tower only at the moment it spawns". The top teams used that
  moment to load each newborn with the tower's whole world model.
- 2025: symmetry inference overran the bytecode limit. Bitmask rows made it about 1,000 bytecodes.
- 2026: the 90-degree vision cone made rats blind behind them, so we called it "structural". The top teams
  managed facing as a resource: defensive facing, alternate looking, reading damage signatures.
- 2025: "units are blocked by robots, not terrain", so navigation work was closed. One finalist designed
  its pathing precisely for robot clogging.
- 2026: the cat-trap pool was capped, so speculative traps were rejected. The top teams picked their own
  traps back up to recycle the cap.

**How to apply.** When a measurement says "impossible because X", write the next sentence: "a design
that works within X would ...". Ask whether a different data representation, a different moment in the
unit's life, or a different engine call removes X. Search the API for any call that relaxes X.

## 5. Judge field-facing ideas against the field

**The habit that hid it.** Our accept gates were mirrors, peers built from our own code, or self-play
tournaments. An idea that only pays against how the field plays read as neutral or negative there.

**What it would have caught.**
- 2020: vaporators were rejected in the mirror against our wall-first incumbent, while every top team ran
  on 20-45 of them.
- 2021: buffrakers were rejected in the mirror, where the guards were already cheap.
- 2026: half our peer archetypes cooperated, while every real opponent attacked on first contact. Our
  catDamage work defended a term that decided almost no real game.
- 2023: the winner of a self-play parameter tournament scored worse on the field panel.
- 2024: the mirror could not price a defence against rushes.

**How to apply.** Classify each candidate as self-facing (navigation, bytecode, bugs) or field-facing
(anything that reacts to or exploits an opponent's choices). Judge field-facing candidates only on field
games, or on archetypes built to mimic what the field does. Before building a peer set, tally how real
opponents actually behave: their opening, how often they attack, what they spend on.

## 6. Every constant should be asked what it depends on

**The habit that hid it.** We tuned fixed numbers: carrier caps, guard sizes, heal thresholds, trap
minimums, retreat HP. Top teams wrote the same quantities as functions of map size, round, income,
bank, threat size, or local force ratio.

**What it would have caught.**
- Map size: our final bots in 2021, 2023 and 2024 had no map-size term, and 2022's only map branch was
  "rich home". Top teams switched resource ratios, aggression, army size and lab counts on map size.
- Bank: 2024 trap thresholds fell as the bank grew; ours were fixed.
- Threat: 2021 guards were sized to the muckraker actually seen; ours were a flat 15.
- Local force: 2024 retreat used a sum of incoming damage; ours used a fixed HP.
- Round: 2021 bidding fractions rose by game phase; ours bid from round 1.

**How to apply.** For every tunable constant, write down what it should depend on before tuning its
value. A sweep over a constant that should be a function will find a mediocre compromise. Prefer
"enough to beat what I see" over "N".

## 7. Estimate hidden global state from what you can observe

**The habit that hid it.** When the API did not return something directly, we concluded we could not know
it.

**What it would have caught.**
- 2025: the number of money and paint towers, decoded from the change in the chip count each turn.
- 2026: the number of live rats, decoded from the current rat cost.
- 2021, 2022, 2023: income per round, tracked from deltas and used to time builds, labs and anchors.
- 2024: enemy alive counts from a kill recorder plus the respawn delay.
- 2022: an estimated "warzone" location that drifts toward the enemy between sightings.

**How to apply.** List the global quantities a good decision needs: unit counts, composition, income,
enemy strength. For each, find any observable that is a function of it: prices, deltas, cooldowns,
timers. Build the estimator early; it feeds principles 6 and 8.

## 8. Control to targets, not caps

**The habit that hid it.** We limited production with caps and cumulative windows. Top teams set a target
for a live quantity and replaced losses the turn they happened.

**What it would have caught.**
- 2026: a standing-army target read from rat cost. Ours built in windows and left rounds with zero rats
  and over a thousand cheese, which is the rat-turn deficit we measured.
- 2021: an income target that rose with the number of living politicians.
- 2025: fixed production ratios with saving-up for the next unit.
- 2022: a miner count driven by whether existing miners had work.

**How to apply.** For each production decision, name the live quantity it should hold (army size,
income, coverage, employment) and build when that quantity is below target. A cap only says when to stop;
a target also says when to start again.

## 9. Model exactly what the enemy can do next turn

**The habit that hid it.** Our micro scored tiles with threat counts. The top teams' micro modelled the
actual engine: who can see a tile, who can hit it after a move, who is on cooldown, who is stunned, who
is killable, who is unreachable behind a wall.

**What it would have caught.**
- 2023: kiting out of enemy vision rather than out of attack range. Vision exceeded attack range, so a
  tile could be seen without being shot, and contact broke only beyond vision.
- 2024: not counting an enemy you can kill this turn as a threat; tracking which enemies a trap just
  stunned; checking whether a hurt teammate's escape is blocked.
- 2026: scoring the cat's path and cone, not distance; ratnapping from outside the victim's cone.
- 2022: weighting each tile by rubble cooldown and by enemies that can shoot it.
- 2023, 2024, 2026: ignoring enemies that cannot reach you.

**How to apply.** Write the micro's threat model from the engine's legality checks, not from intuition.
For each tile, compute what each visible enemy can legally do to you next turn given its cooldowns and
position, and what you can do from that tile. Re-check the model against replays of top bots where they
won fights at equal numbers.

## 10. Attack the opponent's economy, not only its win condition

**The habit that hid it.** Our offence marched on whatever the win condition named: the enemy base,
flags, islands. The top teams also hit the enemy's income early.

**What it would have caught.**
- 2023: launchers went first to the enemy's home mana well, killing carriers before they produced.
- 2020: an early drone harass on enemy miners and soup.
- 2025: tainting every unclaimed ruin with one tile of paint, which blocks a tower until a cleaner arrives.
- 2021: burying the enemy's spawn tiles with spaced muckrakers.
- 2022: mining lead to zero in enemy territory so it never regenerates for them.

**How to apply.** List the opponent's production chain from resource to unit. For each link, ask what the
cheapest action is that breaks it, and whether symmetry tells you where that link is before you see it.
Denial is often cheaper than defence.

## 11. Give parked units a lattice

**The habit that hid it.** We placed stationary units in rings, clusters or adjacent to home, and then
measured congestion, blocked spawns and self-walls.

**What it would have caught.**
- 2020: buildings and walls on a raised parity lattice, which kept the economy alive after the flood.
- 2021: slanderers on a checkerboard, politicians on a sparse lattice.
- 2022: watchtowers on a checkerboard that never blocks movement.
- 2023: carriers on separate checkerboard lanes going out and coming home.
- 2024: builders digging a parity lattice.

**How to apply.** Whenever units stop moving (buildings, guards, miners, economy units), place them on a
parity pattern that keeps every unit reachable and every lane open. Do it in the first design, before
measuring congestion.

## 12. Symmetry is free information; get it cheaply and use it everywhere

**The habit that hid it.** We treated symmetry inference as a costly job, a dedicated scout or an
expensive scan, and used the result only to find the enemy base.

**What it would have caught.**
- 2022: elimination from one mirrored tile pair per unit per round, at no cost to any unit's job.
- 2025: bitmask rows that made full elimination about 1,000 bytecodes.
- 2026: we guessed rotation, which was wrong on most maps, and never attempted inference.
- 2023: mirroring the whole map memory, so pathing knew unseen walls; danger zones around predicted
  enemy bases; the enemy's home wells as raid targets.
- 2026: mirrored cheese mines, and traps placed on the enemy-facing side of a mine.

**How to apply.** Infer symmetry from every tile every unit already senses, share the eliminations, and
then fill every map memory with the mirror: walls, resources, spawn points, and danger zones. Our own
SYMMETRY.md calls it basic. Treat it as day-one infrastructure.

## 13. Communication should carry what units cannot see

**The habit that hid it.** Our messages mostly carried static facts like home location and resource
sites. Top teams spent their bandwidth on live enemy positions, team state and timed requests.

**What it would have caught.**
- 2026: every rat in a fight squeaked its health and the nearest enemy, so allies saw behind themselves.
  Our final bot squeaked nothing.
- 2024: a decaying heat map of where the fights are.
- 2021: dead-reckoning children from their last move, so reports could be short offsets.
- 2025: towers pushing their whole world model to newborns.
- 2023: sightings that expire within a few rounds and are erased when a unit sees the spot empty.
- 2022, 2023, 2024: read the shared state once per turn and write only what changed; heartbeats so
  dead units' slots are freed.

**How to apply.** Ask what each unit would decide differently if it knew one more thing, and send that.
Stamp everything with an age and expire it. Look for the moments when bandwidth is free (spawn,
adjacency, idle turns) and use them.

## 14. Plan production; reserve resources for the plan

**The habit that hid it.** Our producers spent whatever they could afford, by an if-chain, and multiple
producers fought over a shared pool with cooldown hacks.

**What it would have caught.**
- 2022: explicit lead reservations across archons, ordered by urgency.
- 2025: save up for the next unit in a fixed spawn plan.
- 2020: one priority formula that every spawner runs on shared counts, so the team converges on a
  composition without a coordinator; a hold-production message so one unit can reserve for another.
- 2023: refuse to spawn units that will be killed at once, and spawn on the far side from the enemy.

**How to apply.** Write the build order as a target composition plus a reservation protocol. Never spawn
into a fight you are losing. Check in replays whether resources arrive at the moment the plan needs them.

## 15. Small fixed specialist sets beat proportional roles

**The habit that hid it.** We assigned roles as fractions (one in ten builds) or let everyone do a little
of everything.

**What it would have caught.**
- 2024: exactly three builders who farm build XP to the top level and lay every trap; other units may
  not trap except in emergencies.
- 2025: one tower in three designated as the flicker tower.
- 2024: defenders and attackers reassigned by need, with counters in shared state.

**How to apply.** When a skill improves with use, or a job needs exclusivity, assign it to a small fixed
set of named units from the start. When the need for a role varies, let units claim roles by measured
need rather than fixed proportions.

## 16. Units are expendable; survival is a means

**The habit that hid it.** We spent many iterations keeping units alive. Several top teams instead let
units die when that was cheaper, and spent the savings elsewhere.

**What it would have caught.**
- 2025: refill trips only when the team was poor; a rich team replaces units instead.
- 2024: our survival arm cut deaths by 12% and did not raise captures.
- 2020, 2022, 2026: units that disintegrate on purpose (principle 1).
- 2022: wounded units that cannot be healed in time sacrifice themselves to plant lead.

**How to apply.** Price a unit's remaining life against its replacement cost and the resources a rescue
consumes. Survival work earns a place only when a dead unit costs more than the rescue.

## 17. Script the end of the game against the tiebreak

**The habit that hid it.** Our bots behaved the same in the last rounds as in the middle game.

**What it would have caught.**
- 2023: stop building fighters and bank for the tiebreak resource.
- 2026: spend everything on units in the last 50 rounds; bite cats late for the damage term.
- 2021: politicians convert at maximum radius in the last rounds.
- 2024: dig everywhere in the last 50 rounds for the level-sum tiebreak.

**How to apply.** Read the tiebreak order on day one. For each term, write what an unthreatened unit
should do in the last N rounds to raise it. Count how many of your losses were close on the tiebreak.

## 18. Unused bytecode is unused intelligence

**The habit that hid it.** We treated bytecode as a limit to stay under. In 2026 our bot used about 4% of
its budget, and we read that as health.

**What it would have caught.**
- Every year: generated, unrolled pathing and micro; constant strings converted to arrays for free
  initialisation; generated comms accessors from a schema.
- 2020, 2023: spare base bytecode spent on map analysis in the first rounds.
- 2025: bitmask symmetry; inlined micro that needed no bytecode checks.

**How to apply.** Measure headroom per unit type. If most of the budget is idle, ask what better decision
the idle budget could buy: a search, a simulation, a full threat model. Build a code generator early, so
an expensive idea costs a script, not hand-written loops.

## 19. Copy structures, not tactics

**The habit that hid it.** When we studied a stronger opponent from replays, we copied one tactic at a
time onto our own design. When the opponent's design differed structurally, the tactic failed inside ours.

**What it would have caught.**
- 2020: we reverse-engineered a lattice from a top bot's boards but kept our wall-first skeleton, so the
  lattice never became the base. One finalist's first lesson was to copy the best bot's whole design.
- 2023, 2024: blind fire, cohesion, escorts and heal thresholds were copied as switches onto a base
  that lacked the conditions that made them work.

**How to apply.** When a stronger opponent wins by something your design has no slot for, draw its
architecture: build order, role structure, economy loop. Rebuild your skeleton to match before porting
tactics. Replays show structure: unit mix by round, where units park, what they hit first.

## 20. Recurring tricks are a checklist, not a discovery

**The habit that hid it.** Many missed ideas had appeared in earlier years' post-mortems we had read, but
we waited to rediscover them in the new game.

**What it would have caught.** The same tricks recur across rule sets: self-destruct for value
(2020, 2022, 2025, 2026), lattices (2020-2024), nine-tile micro with a full threat model (every year),
symmetry by-product inference (every year), income estimation (2021-2023, 2025), economy denial (2020,
2021, 2023, 2025), endgame tiebreak scripts (2021, 2023, 2024, 2026), generated code (every year).

**How to apply.** Run the checklist below in the first week of every new season, before the first
strategy decision. Each item is a question about the new rules; each "yes" is a candidate to build.

---

## Appendix: the week-one checklist

Answer each question from the engine and the first replays. A "yes" or a number is a candidate.

**Engine and economy**
1. Which actions destroy a unit or structure, and what does the engine create, refund, reset or
   regenerate when they do? (1)
2. Which resources can pile up unspent, and what are all the engine paths from each to the others? (2)
3. What does a newly built structure or unit start with, and can rebuilding farm it? (1, 2)
4. Which global quantities can be decoded from prices, deltas or timers? (7)
5. What is the tiebreak order, and what raises each term in the last rounds? (17)

**Strategy**
6. What is the opponent's production chain, and which link can be broken most cheaply and earliest? (10)
7. Which decisions should depend on map size, round, income or threat size? (6)
8. Which live quantities should production hold at a target? (8)
9. Which skills improve with use and deserve a small fixed specialist set? (15)
10. Where do units stop moving, and which lattice keeps lanes open? (11)

**Information**
11. Which symmetry eliminations does every unit's normal sensing already allow, and which map memories
    should be mirrored? (12)
12. What would each unit decide differently if it knew one more fact, and when is bandwidth free? (13)

**Combat**
13. From the engine's legality checks: for each tile, what can each visible enemy legally do to me next
    turn, and what can I do from it? Include vision versus attack range, cooldowns, stuns, kills and
    reachability. (9)
14. When is a unit's death cheaper than its rescue? (16)

**Engineering and method**
15. How much bytecode does each unit type leave idle, and what would a code generator let us spend it on? (18)
16. Which candidates are field-facing, and which opponents in our gate actually play the way the field
    does? (5)
17. For every closed direction: what three alternative triggers, targets or representations were not
    tried? (3)
18. For every "impossible because X": what design works within X? (4)
19. Where does the strongest visible opponent's architecture differ from ours? (19)
20. Which items on this checklist have recurred in past seasons, and have we tested each? (20)
