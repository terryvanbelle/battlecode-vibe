# ADVICE.md — problems you will meet writing a Battlecode bot, and ideas for each

This is the cross-year reference for future attempts at Battlecode. It distils five practice
seasons (each built on the ones before, so the later lessons weigh more), the published
post-mortems of strong teams from those seasons, and the strategy literature that turned out to
apply. Everything is stated as a general principle. There are no unit names, no rule numbers, no
build names and no tool names: each season has its own, and the point of this file is what
survives the change of rules.

**How to read it.** It is a list of problems. Each problem says how it shows up, then lists ideas
for getting past it. Skim the index, go to the problems you have, and treat every idea as a
hypothesis to measure rather than a rule to obey. Where a lesson was earned by losing (most were),
the failure is named so you can recognise it early.

**Conventions.** "Base" is whatever produces units; "producer" any structure or unit that makes
units or income; "the score term" whatever the season's tiebreak or points formula counts;
"the field" the external opponents; "the incumbent" your current best build; "a candidate" a
proposed change; "the mirror" a game between candidate and incumbent; "a gate" the test that
decides whether a candidate is kept.

---

## Index

**Part I — Foundations, before any strategy**

1. You do not actually know the rules
2. You cannot run enough games, or see what happened in them
3. Your units silently run out of compute
4. The code becomes impossible to change
5. Identical code wins on one side of the map and loses on the other
6. Your instruments lie to you

**Part II — Capabilities of the bot**

7. Units get stuck, oscillate, or wander
8. You do not know where the enemy or the resources are
9. Units cannot share what they know
10. Units interfere with each other
11. The economy stalls, or resources pile up unspent
12. You do not know what to build, or when
13. You lose fights at equal numbers
14. You win fights and lose wars
15. You get rushed, raided, or sieged
16. You lose games you were winning
17. One strategy for every map
18. The opponent does something you cannot answer
19. Contested scoring terms and shared objectives
20. Multi-party and cooperative twists

**Part III — Learning to get better**

21. Deciding what to work on next
22. Knowing whether a change helped
23. Self-play cannot see the thing you changed
24. A change that does nothing
25. Diagnosing a single loss
26. Correlations that mislead
27. Overfitting: to yourself, to one opponent, to one map
28. Measuring your strength against the field
29. The plateau
30. Rewrites and structural swings
31. Ideas that are obviously right and lose anyway
32. Compute, time and disk
33. Records, memory and handoffs
34. Running the project as an AI agent
35. Working with the human owner
36. Rules that keep the measurement honest
37. Tournament realities

**Appendices**

- A. Strategy literature mapped onto a grid game
- B. Checklists
- C. The rules that recur most

---

# Part I — Foundations, before any strategy

## 1. You do not actually know the rules

**How it shows up.** The spec says one thing, the engine does another, and a mechanism you built
on the spec never fires or fires wrongly. Across five seasons, most of the non-obvious facts
that decided games came from the engine, not the spec: which tile a movement cost is charged on;
whether a unit built this round acts this round or next; the order in which units act; the scan
order of sensing calls; whether a cost is charged before or after a legality check; what exactly
happens when a structure changes hands; which actions emit which replay events. Each was found
after a candidate built on the assumed version had already failed.

**Ideas.**
- Treat the engine source as the truth. Decompile or read the engine for every mechanic a
  mechanism depends on, and tag each fact in your rules digest with where it was verified. Two
  digests agreeing is not verification; a summary is a cache that goes stale.
- Read the legality assertion of every engine call before you rely on it. List its clauses and
  check each against your code or against a number you have already measured. Unread
  preconditions voided whole evaluation runs repeatedly.
- Play one game with the trivial starter bot and read the ending reason before designing
  anything. The season's real win condition is usually visible in the first replay, and the first
  structural iteration should aim at it, not at generic economy, navigation and combat.
- Tally how games are actually decided (which ending, which score term) over your first real
  evaluation. One season's founding thesis was refuted by a single tally: nearly all games were
  decided by a term the bot was not optimising.
- Sweep the whole controller API against your call sites at a fixed cadence. Methods you never
  call are candidates; mechanics you never use are the field's advantage. Sweep unused *state and
  rules*, not only unused methods.
- Check the map corpus for regularities before probing for them in-game: symmetry types, where
  the origin sits, spawn geometry, how often a map property occurs. A fact true on every map is a
  constant, not a sensor reading.
- Verify a ported technique's premise before its conclusion. Kiting is right when the target can
  die; it inverts when damage dealt is the score. Cross-year advice comes with preconditions.
- Note a mechanic's hidden side effects. A defensive structure can cost a capability or change how
  the game is scored; an action legal for one unit type can be illegal for another; a mechanism
  can restart the code of anything that changes hands.
- Re-read the engine when the loop stalls. Every season found a mechanic nobody had noticed after
  weeks of play: an extra win condition, an action exempt from a penalty, a cost curve that
  inverted to give a free census.

## 2. You cannot run enough games, or see what happened in them

**How it shows up.** Games are slow, run one at a time, on the machine hosting your session; you
have win/loss counts and nothing else; a hypothesis about why you lost cannot be checked.

**Ideas.**
- Build the instruments before the bot, in this order: a rules digest; a headless runner that
  invokes the engine with a bare JVM (no build daemon per game), runs games in parallel within
  memory, and records `(opponent, map, side, seed) -> winner, rounds, reason`; a determinism
  check; a replay-to-text reader; a bytecode monitor in the bot; snapshot, mirror and sparring
  tools; unit tests; standing charts. Do not start the strategy loop until each has passed a check
  on a trivial bot.
- Run games on a separate compute machine, never on the box hosting the agent session. Games are
  memory-heavy; a small host swaps and every measurement slows tenfold. Keep the host for one
  small diagnostic game at a time.
- Measure the machine: how many parallel games before throughput stops rising, how big one game
  is, how long a small map takes against a large one. Plan evaluations in those units.
- The replay reader is the microscope. It should give: per-round team aggregates (resources, unit
  counts by type, actions, deaths, the score terms) as a CSV; an event window; one robot's whole
  life; an ASCII board at a round; your own log lines; per-type bytecode maxima and overruns;
  navigation statistics (moves, oscillation, coverage, first contact). Grow it by one mode every
  time a hypothesis needs a quantity no aggregate carries. Nearly every root cause in five seasons
  was found in this output, not by code review.
- Log decisions at the decision point, once per decision, with a tag you can grep. Count the
  choice, not its downstream effect. A mechanism's firing should be a grep count.
- Log what a unit *hears* and *knows*, not only what it does. Several root causes sat one hop
  upstream of every metric in the set, in what a newborn unit could learn.
- Snapshot every accepted build as a renamed package so it can be an opponent forever. Keep the
  working bot byte-identical to the incumbent between candidates.
- Freeze the dev runner's seed so a diagnostic rerun is like-for-like, and give every evaluation
  game a fresh seed so a random draw of map and side is a different game. Record the seed.
- Fix determinism early and document it. Identical code twice must give identical results, with
  the engine's final coin flip the only allowed variance. With determinism, a re-run adds no
  information, a paired comparison on the same seed is exact, and a policy-identical perturbation
  tells you the noise floor.
- Silence the opponent's output so your log lines are the only ones in a replay, and give every
  spawned game its own stdin.
- Archive one replay per accepted iteration in the same commit as the acceptance.

## 3. Your units silently run out of compute

**How it shows up.** A unit over its per-turn budget does not throw; its turn spills into the next
round and the actions that would have followed simply do not happen. A base that overruns builds
nothing and scores nothing for hundreds of rounds, and the candidate it belongs to gates at 50%
whatever the change was worth. Two seasons blamed outcome shifts on bytecode without measuring;
both retractions came after the monitor was read.

**Ideas.**
- Put a monitor in the turn loop from the first commit: compare the round number before and
  after your logic (a change means overrun) and read the bytecode counter against the type's limit
  at the end of the turn (near miss above 90%). Report both in logs and have the replay reader
  summarise them for every run. Zero overruns is the standard.
- Treat any overrun in a candidate as voiding its evaluation. A unit that misses a turn is a
  different bot, not a worse one.
- Profile by stage before cutting. The obvious cost is often not the main one.
- Cheap patterns: static fields over instance fields; arrays over collections; unrolled loops over
  a fixed radius; precomputed direction walks instead of constructing locations; a hand-rolled
  xorshift instead of the library random; lookup tables indexed by coarse coordinates; bit-packed
  ints or longs per tile; a switch over a chain of ifs. No object allocation in an inner loop: one
  allocating terrain scan put units over budget a hundred times a game.
- Amortise: update only newly visible tiles after a move; run expensive scans every few rounds;
  spread a large computation over several turns; skip a scan when the budget is already spent so
  the unit still acts this turn.
- Use free compute where it exists: an immobile unit before the game develops, a producer's
  off-turns when it acts every other round. Read siblings' state there, not on the acting turn.
- A unit's first turn is its most expensive turn. Static initialisers and lookups run then; keep
  them small or lazy.
- Do not optimise first. Budget is rarely the binding constraint until the basics work; when it is
  not binding, the headroom is decision quality per turn, and a real search over the visible
  region may be affordable.
- Check the instrumented build, not only your local one: the instrumenter can double your cost.
- Keep the debug class trivial. Reflection-flavoured calls can be rejected by the instrumenter
  with an unhelpful message, and if your loop catches exceptions the real cause never prints.
- Count thrown exceptions per game in the harness and print the total above the results. A base
  that throws at round 200 silently stops building and reads as a strategic collapse.

## 4. The code becomes impossible to change

**How it shows up.** Every constant is load-bearing, a change to one unit's movement flips fifty
games, and you cannot tell which of your accepted features are still worth anything.

**Ideas.**
- Layer it: an abstract robot, then mobile versus immobile, then one class per type; utility
  classes for map memory, communication, exploration, navigation and micro; one file of tunable
  constants, each with the measurement that set it.
- Per-unit state machines: sense, determine state, execute state, then state-invariant
  post-actions. A unit's purpose should be clear from birth and re-derived only on a named
  trigger.
- Give units memory: a current goal with a remembered return point, or a small goal stack with
  start, run and stop conditions. Without it a unit interrupted by a fight forgets its task and
  abandons half-built work.
- Cache what a unit senses once per turn and pass it around; never re-sense in a hot loop.
- Static fields are per robot. Team-wide state lives only in the shared channel; a "team counter"
  in a static is a private counter.
- A map-property latch computed once at spawn (rich or poor start, small or large map, close or
  far enemy) is worth more than any global constant. Branch many decisions on a few such signals
  and gate every bold change on one of them.
- Distinguish committed investment (always pursue) from discretionary surplus (escalating bars),
  and never let them share one gate or one reserve.
- Denominate reserves in the thing they buy, not in raw resource, when cost curves are nonlinear.
- Prefer rates and ratios to fixed thresholds when you do not know the distribution a threshold
  sits on. Every constant is a threshold on a distribution; plot each against the range it actually
  sees in real games.
- Return whether an action actually happened, and let every handler fall through on failure.
  The most damaging single class of bug in one season was "camp forever on an unreachable target"
  because a handler returned success after a failed move.
- In a chain of early-return stages, an early action is never local: consuming an action changes
  which later stages run. Adding an early attack made one bot attack less.
- When adding a condition to an existing branch, read the guard it nests inside; a new clause
  inside a guard that already excludes the case is unreachable.
- Keep the bot small enough to re-read function by function. Structural reviews of a small bot
  found real bugs (unused parameters, dead branches) that no test caught.
- Audit old policy comments against current facts after every structural accept; a premise from
  iteration 11 can still steer iteration 209.

## 5. Identical code wins on one side of the map and loses on the other

**How it shows up.** A mirror of your own bot against a byte-identical copy splits 100/0 on a map.
The cause is in your code, not the map: a fixed compass order for tie-breaks, "the first result
that satisfies" from a sensing call whose scan order is fixed, a hard-coded corner or heading.
Whichever side's forward direction aligns with the fixed preference gets a compounding tempo edge,
and which side that is depends on the map's own geometry.

**Ideas.**
- Audit every tie-break on the day it is written: iteration over a direction array, first-found
  from a scan, defaults correlated with team identity. Break ties relative to the unit's own
  geometry (toward a target, by score), never by an absolute order.
- Mirror-match the bot against itself on every map at every accept and demand roughly even
  splits; a map that splits lopsidedly under identical code is a bug in the code.
- Do not assume fairness is free. A consistent arbitrary preference can be giving the army
  cohesion; a random replacement can lose strength while gaining fairness. Fix it early, when
  little has been tuned around it, and measure the cost.
- Play every evaluation cell from both sides and report maps swept (won from both sides) alongside
  totals, so spawn-side luck cancels.

## 6. Your instruments lie to you

**How it shows up.** A metric column is misaligned and "coverage" is really cumulative moves; a
resampling tool inverts under one convention; a replay mode drops the last N rounds; a census
counts both teams; a regex reports the wrong side; a stale sparring partner scores 95% when the
truth is 62%. Every instrument built in one season was wrong on first use, and every error was
caught by a number looking odd, never by inspection.

**Ideas.**
- Run every new instrument on a case whose answer you already know, and re-run it after every
  change including cosmetic ones.
- Write integrity checks on live data first: a share must lie in [0, 1]; a count cannot exceed
  the population; a utilisation cannot exceed 100%. The errors that cost you land in a plausible
  range, so also cross-check one number against the engine's own field.
- Unit-test the bot's pure logic (encodings, geometry, map memory, sizing functions) and the
  relationships between tuning constants (a cap that must exceed another cap). Break an invariant
  on purpose and check the test fails.
- Unit-test every analysis script on synthetic inputs, and run the whole suite after every change
  to the bot or to a tool, with one command.
- Every check must be shown able to fail: a self-test mode that forces the failure branch, plus
  one real defect it caught.
- Make tools refuse rather than remind. A rule that lives as "remember to check" will be broken;
  the ones that held were scripts that abort: a gate that will not print a number without its
  unit, an arm builder that deletes an arm whose diff is empty, a sparring-partner sync that hard-
  fails on a mismatch, a staleness warning printed inside the summary block where it cannot be
  grepped away.
- A label inferred from the environment is not provenance. Name every result row by the code that
  played, never by the newest directory present; mislabelled rows corrupted several seasons'
  records.
- Check which end of an action a replay event's id refers to, whether the engine emits the event
  at all, and that a dump is non-empty before you count anything.
- When a sequence of unrelated changes shifts per-opponent numbers the same way, test a
  deliberately inert change first; if it reproduces the baseline, the shared pattern has a real
  cause.

---

# Part II — Capabilities of the bot

## 7. Units get stuck, oscillate, or wander

**How it shows up.** A unit loops between four tiles for three hundred rounds with no log line;
two units ping-pong between two equidistant targets and never work; a role with a fixed list of
stations walks toward a taken or unreachable post all game; an explorer walks into a corner and
stays; a unit boxed in between its own structure and an obstacle holds a post forever.

**Ideas.**
- Do not write a pathfinder from scratch. Bug navigation on binary passability, greedy movement
  with a good tie-break on graded terrain, and an unrolled fixed-radius search where affordable
  are the proven set. Adapt a known one, keep a cheap greedy fallback, and measure before you
  replace it: one season wrote a cost-aware pathfinder and reverted it unrun once the navigation
  statistics showed movement was not the gap.
- Bug-follow only when the condition is a fixed target and an obstructed approach; greedy is
  strictly better when the path is clear. Give each robot its own wall-following handedness.
- Every walker needs a stall exit, and the stall counter must survive target flicker. The largest
  single gain of one season was one line: a target re-pick that reset the stall counter every turn,
  so the stall that would have marked the target unreachable never counted.
- Keep a short position history for oscillation detection, scoped to the behaviour it protects; a
  shared stuck counter that fires on benign congestion near the base regresses everything.
- Commit to a target until reached or depleted, with a clear release condition, but keep the
  ability to redirect to something materially better. Absolute stickiness lost heavily; per-tick
  re-optimisation ping-pongs.
- Filter any "nearest tile" target for reachability (elevation, passability, occupancy) and
  blacklist regions proved unreachable.
- Treat friendly units as soft obstacles, and side-step when the planned step is blocked by a
  friend.
- Read the movement API's cost rules exactly: facing penalties, the tile a cost is charged on,
  cooldown stacking. A hidden penalty on off-facing moves sat unnoticed for a hundred and fifty
  iterations and cost a quarter of all movement.
- Exploration: assign each explorer a heading and reassign it only when confirmed stuck; move
  toward the map edge along the heading and re-roll on arrival; zig-zag on diagonals to cover more
  ground; use repulsion between explorers instead of assignment; use an unexplored bitset so
  targets are novel tiles. "Move away from home" exploration walked units into the enemy.
- Log a unit that has not moved in N turns. No log line for three hundred rounds is itself the
  bug.
- Two short guards against degenerate movement outweighed the rest of one season's bot in
  ablation. Complexity and value are unrelated here.

## 8. You do not know where the enemy or the resources are

**How it shows up.** The bot guesses the enemy base by one assumed symmetry and is wrong on half
the maps; a raid gathers for a thousand rounds beside an empty tile; units born inside a sealed
area know nothing; scouting costs more than the information returns.

**Ideas.**
- Never hard-code one symmetry. Keep every candidate symmetry alive, record what you see,
  eliminate candidates as terrain or sightings contradict them, and extrapolate the unseen half.
  Scouting toward the map centre disambiguates fastest.
- Look before you commit. Fly or walk to within sensing range of the guessed location, outside its
  attack range, and prune the guess if nothing is there.
- Sightings of "nothing there" are information; broadcast negatives as well as positives so stale
  danger and target maps decay.
- Make scouting nearly free: piggyback on units already moving, spread explorers by spawn
  direction, and make the consumer of the information robust to a corrected answer. Dedicated
  scouts paid only where the resource sought was sparse; abundant resources are found incidentally.
- Information has a price. Foreknowledge that costs more to obtain than it buys is not
  foreknowledge. Measure the return, and note that a wrong early guess sometimes carried an
  incidental benefit (passivity) that the correction removed.
- Check the corpus first. If every map places the origin or the enemy the same way, that is a
  constant; probing for it every game cost one season's rush three hundred rounds of arrival time.
- Runtime map classification from local sensing alone can be badly imprecise (a base near one
  mirror axis reads as "close" under the wrong symmetry). Trigger posture on observation, such as
  an enemy actually arriving, rather than on predicted geometry at round one.
- Pool sightings so short-sighted units benefit from long-sighted ones; vision and attack range
  asymmetries between unit types are real and exploitable in both directions.
- Symmetry inference implemented and firing was still worth nothing in one season because no
  decision consumed the answer. Name the decision that will use a fact before building the sensor.

## 9. Units cannot share what they know

**How it shows up.** The channel is tiny and contested; a flag set this round is gone before the
newborn that needed it can read it; a signal set after every build blocks every other broadcast so
new structures are born deaf; a stuck flag never clears; a message that must reach everyone
reaches a third of the units alive that round and nobody born later; ownership claims ping-pong
between stale echoes.

**Ideas.**
- Read the channel's rules first: who may write, who may read, when a write becomes visible,
  whether writes cost, whether readers see the current round or the previous one, whether a
  newborn can read anything on its first turn. One season had mobile units writing a channel only
  the base could write; every call threw and aborted the turn.
- Design the schema before the logic: fixed slots per purpose, one writer per slot, an explicit
  clearing condition you have seen fire in a long real game. A signal that only clears under a rare
  state is a permanent switch, not a conditional; "no contact recently" almost never becomes true in
  a sustained war.
- Type header plus payload; sectors instead of coordinates; bit-pack; leave spare bits. Read the
  array once per turn into a local copy, set dirty flags, flush only what changed.
- Stamp every claim with the round of the observation behind it and let the newer claim win
  whoever relays it. Refusing echoes throws away real relayed sightings; accepting them unstamped
  loops forever.
- Anything a unit needs at birth must be re-posted on a schedule, and a newborn should read for
  its first few turns before committing to a role. A unit built this round usually acts next round,
  so a one-round order is never read.
- Roles should be assigned by the one robot that knows the count (the spawner), not chosen by the
  newborn from an empty view. Newborns choosing alone produced a dozen of the wrong role.
- Define what happens when a writer dies. Census by accumulator plus first-actor-publishes-last-
  round's-total gives a live count when the API gives none; a cumulative "ever built" counter is
  not a census and becomes a lockout under attrition.
- Look for side channels: a cost formula that depends on live population inverts to an exact
  census for free; physical markers whose position encodes intent; a unit's state readable by id.
- Broadcast the few things that reduce idle time: friendly structure locations, battlefronts, help
  requests. Connect new units to the network at spawn even at a small cost.
- Deduplicate: track who already knows what, so a hub does not repeat one fact twenty times.
- Under a lossy channel, prefer positive claims. "No targets remain" inverts under loss: silence
  from reporters is indistinguishable from reports not arriving.
- Authenticate if the channel is shared with the enemy or replayable: a salted hash of payload
  and round, and change the constants before submitting.
- Audit priority chains for unconditional early returns. A correctly set signal is useless if no
  consumer can reach the branch that reads it.
- Do not broadcast a target to everyone nearby: pooled sightings caused stampedes onto one tile
  and a distress call scattered defenders exactly when they were needed. Coordination through
  comms pays for knowledge, rarely for reaction.

## 10. Units interfere with each other

**How it shows up.** Several units head for the same tile, all but one re-pick, and each re-pick
costs rounds; units parked near the base stand on every tile a producer could spawn onto and
production stops without a log line; workers alter or build on tiles a friend needs to pass; an army
strings out through a corridor; a shared pool is spent by one producer before a sibling can act.

**Ideas.**
- Emergent coordination beats commanded coordination. Spawn order as an implicit formation,
  repulsion between explorers, momentum terms and role-by-distance all worked; every elaborate
  commanded plan flopped. The strongest teams say so every year.
- Settle claims once: first to arrive keeps it, or assign deterministically from a canonical list
  everyone computes alike. Never let a unit yield to a "closer" unit while walking; that rule makes
  everyone re-pick forever.
- The tiles adjacent to a producer are infrastructure. Every station picker, parking rule and
  structure placer must skip them; permanent structures compete for the same tiles and the problem
  gets worse the more of them there are.
- A role with a fixed list of stations needs a stall exit and a check that the list fits the map
  (an edge or corner base has fewer neighbours). Count idle units per role at a fixed round in
  diagnostics.
- Shared pools across several producers are a turn-order problem before they are an income
  problem. Ten income-side fixes never moved a starved producer's build count; a fairness yield
  gated on sibling hunger did.
- Fixed hard-coded claims outperform "nearest free" rules whenever the population is large enough
  to race.
- Boundary-following and digging rules must leave a door: a unit boxed in by two structures can
  never leave and blocks every spawn.

## 11. The economy stalls, or resources pile up unspent

**How it shows up.** Income stops on a fifth of the maps; a large bank sits idle in most losses;
half the gatherers gathered nothing in a two-hundred-round window; production shut off forever at
round five hundred because a census counted dead units as alive; a "danger" gate paused the
economy for one trivial enemy; a reserve nobody spends holds back bodies that arrive two hundred
rounds late or never; the treasury is pinned just beneath whatever threshold you set.

**Ideas.**
- Before tuning how much to spend, census what your units actually do. An idle resource or a
  frozen unit is a defect to trace, not a knob to turn. Repairs of defects and removals of binding
  caps transferred to the field in every season; reallocations between unit types mostly did not.
- Audit the production decision chain for silent shut-offs: stale censuses, over-broad danger
  rules, caps that bind while resource piles up, a base physically sealed by its own idle army, a
  gate threshold above the band the resource ever occupies. Three of one season's largest wins were
  such gates.
- Find which resource is binding by checking whether it is accumulated anywhere. A resource never
  at cap is being spent; a resource always at cap is not the constraint. Money that piles up while
  a second resource sits in absorbing states means the second resource is the economy.
- Never let a rule stop the economy for something that cannot hurt it. Distinguish a real threat
  from an enemy scout in sensor range; opponents build cheap scouts by the hundred.
- Income first, then population. Every attempt to buy units the economy could not feed failed,
  and the mechanism (upkeep, cost curves) made it certain.
- The opening is worth more than any later reallocation. Compare your first fifty rounds' event
  stream side by side with the strongest opponents'; the whole-game gap was often decided before
  round fifty and no aggregate showed it.
- Compute return curves from the engine formula before believing a replay correlation about unit
  size or count. Whether fewer bigger or more smaller pays is arithmetic, not doctrine.
- Keep "bank" and "income" distinct in diagnosis. Opponents' large banks are usually a by-product
  of a snowball, not a policy you can copy by withholding spending.
- Never float. Make consumers leave exactly what producers need and spend the rest; a lead on
  paper is not a lead.
- Measure a winning game's production cadence before designing a throttle; two measurements
  replaced seven failed guesses. Keep a proven opening byte-for-byte and gate only replacement
  spending behind a deeper reserve.
- A rate limit has a ceiling and a period; test both. Every ceiling raise in one season failed;
  halving the period was accepted.
- The band a converged bot's treasury settles into is an equilibrium of its gates, not idle
  capital; lowering a gate moves the equilibrium, not the share of time above it.
- Every economic decision may be made in the first few hundred rounds if the base later cannot
  spawn; know when your production windows close and do not tune constants that only bind after
  they do.
- The unit that expands is the economy. If the win condition is territory or production, the
  number of producers is the master variable; a strong field often has several times your producer
  count with each unit individually less productive than yours.

## 12. You do not know what to build, or when

**How it shows up.** A fixed build order breaks on half the maps; a standing guard is bought before
the economy; a structure you never build is what the field wins with; infrastructure gates never
fire in a contested game or fire everywhere and starve the army.

**Ideas.**
- Priority-driven build orders: a desired composition, each type's priority a function of actual
  over desired plus a minimum-resource bar, or spawn weights instead of sequences. Composition is
  then tunable and map-adaptive.
- Enumerate the few legal openings, compute the fastest to first expansion, give it a fallback if
  the nearby expansions are contested. Ship a working opening early and iterate.
- Escalating thresholds for discretionary structures (each extra one needs a higher bank) let
  investment scale with game length without one flat number.
- A "desperation index" (rounds you have had the resources but no acceptable site) relaxes
  placement constraints in stages.
- Make sure every gate is reachable in a contested game: measure the quantity's real range over a
  few real games before setting a bar on it.
- Check the surplus branch does not buy standing bodies before economy or expansion; a surplus that
  becomes idle units is a surplus wasted.
- A structural doctrine of a strong opponent (mass static defence, a different economy layout)
  cannot be tested by bolting a small version onto an unchanged strategy. Integrate it from the
  start or accept the test is inconclusive.
- Copying a whole architecture is right shape and wrong size when the economy that pays for it is
  the thing it protects. Check the prerequisite before the copy.
- Sequence capabilities: movement as a body, a live census, retreat and repair, focus fire, target
  priority, economy scaling, then exotic mechanics. Micro tweaks before the army can move together
  measured zero.
- React to threats in production, not only in movement: a rush detector that switches the build,
  time-boxed so a stalled rush does not freeze your economy.
- A fresh capture or a new structure is an empty shell; budget for what it needs at birth, or it
  is lost within a hundred rounds.

## 13. You lose fights at equal numbers

**How it shows up.** Units trade one hit for their life; nobody retreats or everyone retreats;
each unit picks its own target; a shared target is reset every round and the formation scatters;
units arrive one at a time and die one at a time.

**Ideas.**
- Evaluate all candidate squares with a heuristic struct: enemies in range from there, distance
  to nearest enemy, allies adjacent, can-attack-after-move, HP; compare with a scoring function
  in one common currency. This discretised potential field is the micro every strong team copies.
- Consider both orderings, attack-then-move and move-then-attack, and let the framework express
  both; frameworks that cannot have bad micro.
- Kite when movement and action cooldowns are separate: out of range, step in and hit; in range,
  hit and step out. Attack even blind if the cooldown resets anyway. Verify kiting's premise first
  (the target must be killable and survival must be what converts).
- Target selection: kill what shoots back first, then production, then the win-condition
  structure. Within a class, lowest HP first unless that is overkill, then the highest; prefer
  kills per action. A threshold that scales with the unit's own size makes big units passive; a
  flat bar fixed it.
- Focus fire through one shared target slot that any unit promotes to when its local pick is
  better; clear it only on confirmed staleness, never every tick.
- Concentrate; never trickle. Aimed ranged fire follows Lanchester's square law, so two to one in
  numbers is roughly four to one in effect. Wait or regroup when outnumbered locally; a wave works
  where a stream dies. Synchronise approaches so only one unit is exposed per turn.
- Retreat only near death, and as a group; step to favourable terrain after acting; avoid stacking
  where there are adjacency penalties or splash.
- Combat volume matters as much as efficiency. Measure attacks per game and kills per action for
  both sides; a bot with better per-hit efficiency and a third of the volume loses.
- Per-unit target preferences are bounded by what one unit can sense. If vision is narrow,
  "weakest visible" and "nearest visible" are the same target and team priorities need comms.
- Before blaming trade efficiency, check unit-count and producer-count parity in the exact window
  blamed; an even kill exchange with a base-count divergence is an economy problem.
- Do not buy survival with inactivity or damage with a resource you need. Leashes that halved
  deaths and mechanisms that raised damage both lost when they cost the thing that converts.
- Type-aware thresholds when several unit types share code; and a "free hit before fleeing" when
  already in range.

## 14. You win fights and lose wars

**How it shows up.** Your units are individually more productive and you lose; the army is spread
across a diffuse front; a siege chases the enemy base past defenders; reinforcements march to a
stale guess-point; you refuse an inefficient trade the opponent happily takes.

**Ideas.**
- Identify the opponent's centre of gravity, the one thing its system depends on (a production
  structure, an income generator, a hub), and direct the collective blow there. Killing production
  before combat units, or holding a resource site, decides more than winning skirmishes.
- Identify the real scarce resource and deny it cheaply: occupy spawn tiles, partially complete a
  shared objective so the enemy cannot finish it, put one unit on an unclaimed site so the enemy
  must bring a specialist.
- Hold the centre and move on interior lines: the side whose forces travel shorter paths between
  fronts concentrates first. Spawn units on the enemy-facing side; broadcast battlefronts so
  reinforcements go to the live fight, not a static objective.
- Attack where the opponent is unprepared: along map edges, at the economy rather than the
  fortifications, before defences exist. Do not renew a failed attack along the same line; re-
  fortify and send a second wave.
- Tempo: a rush exploits a production loop that reacts too slowly. Shorten your own observe-
  orient-decide-act loop with in-game detectors, and lengthen theirs by forcing reactions.
- Know the culminating point. An attack that outruns its strength flips to the defender's
  advantage; engage on favourable local counts and stop when the count turns.
- Economy of force: mass at the main effort is paid for by minimal force elsewhere, with security
  as the floor. Keeping one defender home versus sending everyone is a real trade; measure it per
  archetype of opponent.
- A "rally before reinforcing" rule failed repeatedly because one diffuse-front map rewards a
  spread army. Make grouping conditional on the front, not universal.
- Decompose your deficit and check the direction you open moves the binding term. One season
  spent a session on eight directions all aimed at the factor it already led.
- For a contested proportional score term the question is never "is this trade efficient" but "is
  the opponent making it too"; refusing a bad trade the opponent takes concedes the whole term.

## 15. You get rushed, raided, or sieged

**How it shows up.** The base dies before the economy exists; a timed raid at an opponent-specific
round removes defenders and puts attackers on the emptied ground; single large attackers convert a
base from range; every defensive feature reads as pure cost on your own evaluation.

**Ideas.**
- Standing defences that cost no actions once placed are the highest-value class of feature:
  static structures, traps, a wall, placed before the threat arrives. Bodies delay a rush;
  fortifications break it. Three variants that waited for evidence of a rush all lost more.
- Sizing must be relative: an absolute "one guard per base" strips small-army maps; a home-threat
  override gated on total army size held.
- Defence is the stronger form: the defender has terrain, preparation and shorter lines, so
  attacks culminate. But pure turtles lose to the clock and to snowballs, so defend the core and
  expand.
- Make defensive spending conditional on a measured trigger and verify the trigger fires only
  under the doctrine it answers. An unconditional counter loses everywhere the doctrine is absent;
  the conditional version was neutral in the mirror and turned the losses to swarm opponents.
- Prefer defences that change a decision rather than buy bodies: shoot the attacker whose payload
  matters, not the nearest one; guard the exact tile the opponent exploits. Read the replay
  round by round to find that decision.
- A base-level reaction usually cannot arrive in time; the base sees the attacker a few rounds
  before it strikes. Defence has to be a standing posture or an early-warning network from long-
  sighted units to blind ones.
- Layer it: danger map to steer clear of static defences, an outer harassment screen, an inner
  wall, a last-resort fallback when the attack fails.
- Killing the enemy's spawner is the wrong answer to a spawner that is rebuilt cheaply; defend at
  home instead.
- Defensive features cannot be valued against a mirror that never attacks. Build the sparring
  opponent that does the thing, and price the defence against it and against the plain mirror
  (for its cost where the threat is absent).
- Any defensive structure may carry a hidden strategic cost (a forfeited capability, a changed
  scoring regime). Price it.

## 16. You lose games you were winning

**How it shows up.** A won board loses on the round-cap tiebreak; you lead at the decisive round
and collapse; a beaten, located enemy is never finished off; long near-mirror games flip from
unrelated perturbations.

**Ideas.**
- Know the tiebreak order from day one and make closing out a beaten enemy an explicit behaviour.
- Plot the master variable past the round at which it predicts the outcome and ask, in losses,
  "was I ever ahead?" The answer splits losses into two different defects; half of one bot's
  losses were games it led.
- A structure count that peaks and declines while the opponent's rises monotonically is a defence
  or replacement defect, not an economy defect.
- An accepted mechanic can change what game you are playing (when a shared pool dies, when the
  match ends). Re-dose constants tuned under the old shape.
- Long chaotic games are amplifiers: one flipped long game is not evidence; a one-directional
  sweep across many opponents on one map or side is.
- A win-more mechanism (a second base, an upgrade that consumes the army) is not a comeback
  mechanism; classify each candidate as one or the other and test it in the games it is for.

## 17. One strategy for every map

**How it shows up.** Every fixed formula for an opening count fixes one cluster of maps and breaks
another; a defence tuned on a central base fails on an edge base; the diagnostics you chose all
flatter the candidate.

**Ideas.**
- Adapt rush versus turtle, and expansion versus army, from measured signals (distance to enemy,
  passability, resource density, spawn geometry) at run time, never at compile time.
- One or two cheap latches computed at spawn drive many decisions; that beat every global
  formula.
- Diagnose on maps the candidate was not designed for, and on both an edge base and a central
  one. Two chosen diagnostics are not a forecast; a redesign that won two hand-picked maps by
  ten to fifteen percent lost the random-map gate three times.
- Check where your probe map sits in the corpus distribution; one lineage ran every stage-zero
  test on the single most outlying map of seventy-five.
- Sample the regime you trace, not only the regime you score. Sixteen iterations attacked the
  failure mode of the only two maps anyone had opened while the neglected regime held the inverse
  failure.
- Expect the tournament map pool to shift larger and slower than the defaults, which nerfs rushes
  and favours economy; make custom maps that expose your problems.

## 18. The opponent does something you cannot answer

**How it shows up.** A rush kills you before round three hundred; a strong opponent's whole
doctrine is different from yours; you cannot see why you lose because you never watch the games.

**Ideas.**
- Whenever the enemy uses a tactic against you, learn to use that tactic against other bots. Copy
  the offence and submit it first; the offence paid immediately and was one season's only step
  outside the error bars. Then build the defence against your own copy, which gives you a
  reproducible attacker to test against.
- Copy what better bots do, but only what you understand; steal automation freely. Understand the
  prerequisite before copying a structure.
- Build a sparring archetype for every opponent behaviour that beats you, from measurement of
  their replays, and run it before you trust it: two of one season's partners did not reproduce
  the condition they were built for. A partner must reproduce the behaviour and be competitive
  (roughly a quarter to three quarters against the incumbent), or it cannot rank two builds.
- Replay recorded opponent games from a chosen round with your candidate on your side. It is the
  best harness for opponent-specific mechanisms and the worst strength measure: it rewards
  difference, not strength.
- Read the loss census with action histograms normalised per round and per unit. When per-unit
  rates are at parity, the whole gap is unit-turns; when they are not, it is behaviour.
- Reading a strong opponent's source (where the rules of your project allow it) is worth several
  iterations of inference; three accepted micro fixes in one season were direct ports. Where the
  rules forbid it, replays and boards are the source.
- Classify the field into archetypes and decide which to adopt; the general plan converges and the
  edge comes from one or two extras.

## 19. Contested scoring terms and shared objectives

**How it shows up.** The score is a proportional share of a common pool; a race is decided by
throughput per body; every redesign that adds surface loses by arithmetic before it is coded.

**Ideas.**
- Count the ceiling first. If every body works at its maximum rate, the only levers are more
  bodies or fewer tiles, and geometry loses before code does. Write the arithmetic bound into the
  ledger so nobody reopens the line without a new premise.
- Prefer rules that withhold effort from waste over rules that add work: only reinforce what is
  exposed, do not spend on a position that cannot be threatened, cover the one gap nobody covers.
- Look for the cheapest conversion of resource into the score term and for actions exempt from a
  penalty; the largest gain of one season came from reading the engine's scoring code for exactly
  that.
- For a proportional term, match the opponent's rate of taking it before optimising efficiency.
- Optimising a score term is worthless in games that never reach scoring; decompose losses by
  ending first.
- Do not spend on defence of a shared objective that cannot concentrate: "nobody is there" made a
  static-defence line null because defenders could not be brought together.

## 20. Multi-party and cooperative twists

**How it shows up.** A neutral party whose damage is scored, a cooperate-or-defect choice, a
penalty for being the first to break a peace.

**Ideas.**
- Treat the switch of objectives as an explicit state transition with named triggers, not as
  emergent behaviour; scoring weights change at the switch, so everything tuned before it must be
  re-dosed after it.
- Understand the year's core constraint, build an economy around it, commit fully to that
  strategy and adapt tactically within it.
- Check what a "defensive" or "neutral" action does to your status under the cooperation rules;
  a purely defensive placement made one bot count as the aggressor and cost it a capability.
- The neutral party's behaviour is code; read it rather than infer it from the flavour text.

---

# Part III — Learning to get better

## 21. Deciding what to work on next

**How it shows up.** A replay shows twenty things wrong; every loss suggests a fix; you fix
everything a little and nothing well; you spend three weeks on a pathfinder that was never the
gap.

**Ideas.**
- Prioritise by expected win-rate gain, not by replay to-do lists. Pick one or two changes and do
  them very well. The best tasks are usually the simplest.
- Three sources of candidates, used in rotation so none dries up: absolute degeneracy in your own
  replays that needs no opponent (a resource pinned in a band, production at zero, a unit
  oscillating, an overrun); the census of a scrimmage block reduced to per-round metrics with the
  earliest-onset correlation as the pointer; and the capability gap (a doctrine the field shows
  that you never produce, an API method never called, a perennial lever from cross-year research).
- Highest yield, in order: repairing your own defects; removing binding caps; tracing a stall
  within one replay; ablating accepted features; reading the engine for unused mechanics. Lowest:
  theorising from where you lag the opponent; sampling replays only from the loss list; copying
  the opponent's ratios.
- Keep a functional-area map (economy, production mix, navigation, exploration, combat,
  communication, defence, map adaptation) and a rule: after three consecutive rejects in one area
  the next candidate comes from another area or from the structural track.
- Before implementing a fix diagnosed on one replay, estimate how often its trigger fires across
  other recent games. "Helps the diagnosed case, hurts broadly" was the most common rejection
  shape in two seasons.
- Check you are actually behind on a metric before optimising it; the comparative check against
  the bot that beats you killed several directions for zero games.
- Rank decision sites by opportunity count in the deciding window, and check a lever's time
  constant against that window; a mechanism that works too late is a losing mechanism.
- Fix idle time and floating resources before adding features; a unit-count lead means nothing if
  half the units are idle.
- A rejected iteration is a delivered result if it closes a direction with a number. Log what it
  closed.

## 22. Knowing whether a change helped

**How it shows up.** Eight rejections in a row that were the instrument's fault: a fixed panel
resolving only effects above seventy-five percent; a twenty-four-game head-to-head with a five-
game noise floor; a batch read while running that showed the losses first; a candidate compared
against the wrong baseline; a gate scoring the map-and-side draw rather than the change.

**Ideas.**
- Measure the noise floor once per lineage: two builds differing by an inert constant over the
  full corpus, both sides. State every gate in games over a named opponent and map set; never quote
  a win-minus-loss margin as if it were a win count (its standard deviation is twice as large).
- Know the arithmetic. Separating fifty-five percent from fifty needs on the order of eight
  hundred games; detecting a five-point difference at conventional power needs over a thousand a
  side; a ninety-six-game block resolves roughly twenty points. Most of what you can build is a two
  to eight point effect. Cost each experiment's resolution against its expected signal before
  spending games, and if every lever prices below resolution, the next decision is the evaluation
  design.
- Use a sequential probability ratio test against the incumbent on random maps and random sides,
  in batches, capped. It stops when the evidence is decisive and spends games only while they
  decide something. A fixed-N test is either too small to see real effects or wastes games on
  obvious rejects.
- Pair every candidate game with a control on the same seed (incumbent against itself). Where the
  change did not fire the games are identical and contribute nothing; only discordant pairs enter
  the test, read by a sign test with a minimum pair count. Unpaired, a gate scores the draw: one
  read thirty-four to forty-six with seventy-four of eighty pairs identical and the rest in the
  candidate's favour.
- Decide in advance what to do with the inconclusive band. Keeping small positive results
  provisionally as a stack, tested as a stack against the incumbent and snapshotted only when the
  stack clears the gate, is how a low-budget loop improves at all; keeping each on faith
  accumulates noise, discarding each never improves.
- Diff game by game on (opponent, map, side), and read the shape. A small, scattered, mixed-
  direction diff is chaos; a one-directional diff concentrated on one map or side across many
  opponents is a real regression. Judge by asymmetry per flip and dose coherence, not totals.
- Never read a running batch as a result: games that end early are the losses (or, against an
  annihilator, the wins). A zero-to-five read that ended eleven-to-five voided a gate.
- Compare candidates against control blocks of the incumbent played in the same period on the
  same pool, never against the incumbent's pooled history; an accepted build's acceptance games
  rate higher than its later games (the winner's curse).
- Controls expire on every accept: re-measure, never quote a stale number. Verify a revert file
  by file against the snapshot; source control does not undo a committed change and a stash does
  nothing to committed code.
- Small maps for diagnostics (they are cheaper), never for the gate (it biases every decision
  toward them). Match the sample scale to the change's blast radius: round-one, every-unit code
  needs full-scale verification from the start; a clean eight-game sample missed a six-flip
  regression.
- On a fixed corpus repetition never adds power; the only new information is a different build,
  different opponents or different maps. Use dose-response curves instead of re-runs: a parameter
  is a dose only if it changes the condition evaluated; an inverted high dose rejects that dose,
  not the mechanism; always include a byte-identical zero arm; a curve that peaks in the middle is
  stronger evidence than any single point.
- Sibling cells from one run are not corroboration; a multiplicity correction applies across
  cells and across arms.
- A fixed-seed single game is a filter, never a verdict; single-seed wins failed to carry to the
  field at least four times in one season.
- Run a control-versus-control gate when several rejections arrive in a row, to prove the harness
  is fair before you believe them.
- Pre-register everything the numbers could tempt you to move: the counter, the gate, the
  falsifier, the selection rule, which instrument decides, the subgroup, the branch order. Then do
  not move a bar after seeing a number, and never reinterpret a null.

## 23. Self-play cannot see the thing you changed

**How it shows up.** A candidate wins its mirror by nineteen points and moves the external ladder
by nothing; a defensive change reads fifty percent against a twin that never attacks; four
accepts on the mirror make the bot worse against real opponents; the mirror and the ladder
disagree in sign.

**Ideas.**
- Know what each instrument can and cannot see. The mirror measures the marginal value of one
  change against your own previous build; it is blind to any weakness both builds share and prices
  anything aimed at the field only for its cost. Sparring archetypes see the one behaviour each
  reproduces. The external ladder sees strength and what the field punishes. Keep all three and
  never confuse them.
- The mirror can prove a feature is not paying for itself; it cannot prove a feature is
  unnecessary. Before ablating a defensive feature, check the protected counter has any variance on
  the instrument you plan to use.
- Economic changes and repairs of your own defects show in the mirror and transferred. Repairs of
  defects and removals of caps carried; reallocations and copied ratios mostly did not.
- A self-play dose ladder finds a mirror optimum, not a field optimum.
- When a change is built to answer something only the field does, pre-register a second arm
  against the archetype that has that property, decided before the gate runs. A null in the mirror
  plus a win in that arm is a finding about an opponent class; a null in both is a change that does
  not pay. If you reach for this argument after a disappointing gate, you have already failed the
  test.
- "My instrument is blind" is a hypothesis: build the external instrument that would refute it
  and run it. Two lineages that claimed blindness were refuted both times.
- Distrust the proxy when a series of self-play accepts never moves the external rating, never on
  a single block. If the external line is flat for twenty accepts, the loop is optimising the wrong
  objective.
- Self-derived opponents decay: by staleness (rebuild them from the newest accepted snapshot with
  policy edits stored as data) and by convergence (once you adopt their doctrine they are you).
  Frozen snapshots are the one self-derived instrument that cannot drift.
- Two bots co-adapted to each other's weaknesses can each correctly conclude the other's axes are
  closed while both lose three quarters of games to the field. Being ahead of a rival is not
  evidence of being near optimal.
- Independent lineages are useful as instruments (frozen cross-architecture rungs, a periodic
  round robin), not as a development model; one well-instrumented lineage beat three isolated ones.

## 24. A change that does nothing

**How it shows up.** A mechanism compiles, runs, and is completely inert: its caps exclude every
target on the maps tested; a rule posts guards and another rule walks them away; a free-tile rule
is unreachable; a trigger fires on the wrong condition; a role flag is set the round before the
newborn can read it. Statistical tests then measure nothing for hours.

**Ideas.**
- No gate starts until one logged diagnostic game shows the pre-registered decision counter
  firing and acting as claimed, at roughly the claimed rate. One logged game costs minutes; a
  wasted gate costs a night. This rule caught three inert candidates in one season and many more
  later.
- Check reachability in the pre-registration: the branch is taken at observed values, the choice
  set has more than one member, the property optimised is visible at the scale of the decision.
- Stage zero answers "does it fire", never "does firing pay". Diff counters between candidate and
  baseline on one map; all-identical means the mechanism is dead.
- Check an upstream dependency before concluding a feature is a no-op. A correct exploit tested
  as zero effect for twenty-seven iterations because the economy never produced the unit it
  applied to; it worked once that changed.
- A confirmed defect is not a defect that is costing you; measure the price before repairing it.
- "It is implemented" is not evidence that it happens; only the counters are.

## 25. Diagnosing a single loss

**How it shows up.** You know you lost and roughly when; you do not know the mechanism, and two
different mechanisms leave the same trace.

**Ideas.**
- Trace, then theorise. Read the replay with the reader before forming a hypothesis; enumerate
  the mechanisms that could produce the symptom ("chooses badly" and "never sees it" look alike)
  and find which the data supports. Check the same symptom in a second game against a different
  opponent; a shared symptom is not a shared cause.
- Follow one unit's whole life for every mechanism you add. Indicator truncation, an abandoned
  build, a frozen fallback, a counter that never decremented were all found this way and none by
  code review.
- Read intermediates against a control run of the incumbent against itself on the same map and
  side, never against the other side of the same game; the side effect alone can be larger than
  most candidates' effects.
- Read both sides. The two best findings of one season came from the opponent's side of games
  already paid for.
- Attribute a rate to its actor before calling it waste; check the direction of every id in a
  replay event; normalise per round and per unit before comparing counters.
- A quantity that tracks the win condition collapses in every loser; check the mirror before
  calling it a mechanism. You may be reading the scoreboard and calling it a defect.
- Verify the mechanism fired before explaining why it underperformed; otherwise the explanation
  is decoration on noise.
- Distinguish slow from stopped: a rate statistic cannot. Look for absorbing states (a producer
  that once drained can never save back up).

## 26. Correlations that mislead

**How it shows up.** By mid-game every metric correlates with winning because the winner is ahead
on everything; a metric that identifies weak opponents correlates with winning; the opponents'
unit mix correlates with their wins and a bot rebuilt around that mix loses every game.

**Ideas.**
- Act on the earliest onset only: the first round at which a metric's lead correlates with the
  result and holds. Temporal precedence is the one causal hint a correlation honestly gives; late-
  round correlations are the scoreboard.
- Stratify within opponent and map so a metric cannot score by identifying easy games.
- Merge blocks before reading; with forty-seven games everything looks significant.
- A correlation earns a diagnostic game, not a code change. It finds symptoms as readily as
  causes, and only building and testing separates them.
- Two things scaling together is not a mechanism; find which is upstream. Coverage predicted wins
  early in one season and more scouting did not win games: it marked a winning position without
  being a lever.
- A relationship measured under your own policy is an equilibrium of that policy, not a law.
- Confirming every link of a chain does not establish the terminal effect; a figure that closes
  the gap if assumed is a requirement, not evidence.

## 27. Overfitting: to yourself, to one opponent, to one map

**How it shows up.** Wins fall on three maps out of twenty-seven for no structural reason; a fix
works on the exact recorded opponent and not on the field; a fixed map list you hand-picked is an
overfitting surface; strategies cycle rock-paper-scissors between your own versions.

**Ideas.**
- Random map and random side per game for anything that judges strength; a full corpus for the
  census; a pinned list only for attribution within one screen, and then the same list for every
  arm of that screen.
- Keep a ladder of frozen past versions and play it on a schedule tied to accepts; a head-to-head
  chain of positive margins does not chain (plus eight, plus six, and plus eight overall), and
  only a frozen roster sees a lineage walking downhill.
- Test against a basket of archetypes (rusher, turtle, expander, harasser), not only your last
  version; self-play alone cycles.
- Grade on expected score against the whole field, not on the target opponent. A recorded-game
  harness proves a mechanism; only a large random block proves it generalises. Nontransitivity is
  real: an internal rusher beat the incumbent and rated well below it on the field.
- Watch for per-opponent trades (one opponent up sharply, two unchanged, one down) and for accepts
  that only match the baseline.
- A saturated rung (beaten ninety percent or more) is censored, not blind; add a harder rung,
  never retire one.
- Randomise what an opponent could learn from you (communication constants) before submission.

## 28. Measuring your strength against the field

**How it shows up.** Raw win rates across builds are incomparable because each met a different
pool; a sequential rating depends on play order and a batch of easy calibration games lifts you
sixty places; a "just above us" pool drifts to bots that beat you ninety percent of the time.

**Ideas.**
- Play external opponents only as scrimmages: random map from the corpus, random side, rotating
  opponents drawn from the rating band around your build, so results are informative in both
  directions. Never choose a map or side against an external bot; never play external bots against
  each other.
- Rate all games at once with a batch pairwise fit (Bradley-Terry on the Elo scale), each of your
  builds a separate player, a weak prior, and repeated cells under the same seed counted once.
  Grade a build by its rating with an interval, its rank, and its expected score against the whole
  field.
- Withdraw a submission whose rating interval falls below the previous submission's; accepting
  means submitting, and a real team cannot scrimmage without submitting.
- Compare candidate and incumbent by alternating blocks in the same period. A ninety-six-game arm
  cannot see twenty rating points; two hundred and forty games is the floor for a ladder verdict,
  and more for a submission.
- Expect the pool to move under you as you climb; a striking arm result may be against the wrong
  pool.
- A rule that hides games against bots you cannot yet beat allocates attention well early and
  starves the hypothesis pipeline late; let the rating band allocate attention instead.
- One small map is not a screen; tier opponents on several map sizes.
- Ladder positions in the middle of a real tournament are separated more by scrimmage luck than
  by strength; rank by your own many-game tests.

## 29. The plateau

**How it shows up.** Thirteen arms, thirteen closures, no accept; every incremental change in one
area rejected; the log declares convergence and the next gain comes from somewhere else.

**Ideas.**
- Before any escalation, audit correctness: the bot's and the instruments'. One basic found broken (an
  inference that guesses where it could be exact, a mechanism that never fires) means others exist at
  the same level. Run a read-only audit through independent lenses (dead code; rules and API; shared
  state; behaviour in replays; the measurement pipeline), each finding with evidence, a fix and a
  regression test, and have separate reviewers try to refute every finding. Fix the instruments
  first and prove them with an identity control (identical code must disagree with itself zero times
  on the harness that decides); then put each bot fix behind a switch, judge the combined build with
  exact paired tests, confirm on fresh seeds, and keep the checks permanent. One such audit ended a
  plateau that dozens of tactical experiments had not: the basics were broken and the tests mostly noise.
- Escalate in order, without skipping to the last step: ablate what you already carry (features
  accepted on thin margins are often worth nothing, failure-mode preventers are often worth the
  most); sweep the API for methods never called; re-read the allowed field games for what you were
  not looking for; re-read cross-year research; a structural attempt; a rewrite.
- When every single change fails its gate, look for a jointly necessary pair; several accepts were
  two halves each inert alone. Pre-register counters per half so a reject still says which failed.
- Exhausting one axis is not exhausting the space. Convergence claims are relative to the
  instruments in hand; twice a declared local optimum was broken by a new metric or instrument,
  not a new tactic.
- A plateau in accepts is not a plateau in strength; check the absolute instrument before
  concluding the loop is dead, and register "indistinguishable from N accepts ago at census power"
  as a result.
- A closure says an implementation failed; a ceiling says no implementation can. Only the second
  licenses abandoning a line.
- Re-test old rejections when both the mechanism was verified to fire at the time and the margin
  sat inside the old instrument's resolution, or when a since-fixed defect blocked it. Base rate:
  about one in three reopens paid, and the failures converted weak rejections into firm ones.
- Regularities that fill ledgers: metrics that move without converting to wins; survival bought
  with inactivity; and the winner's recurring profile of capability preserved at zero marginal cost
  (standing defences, spending idle resources, removing pure waste).
- Never idle. Waiting on a run is work: read replays you already own, prepare the next candidate,
  ablate. Stopping because nothing comes to mind is not a state the loop is allowed to be in.

## 30. Rewrites and structural swings

**How it shows up.** The strategy has stopped moving and the code carries more closed directions
than live ones; every top opponent runs a design you do not; a rewrite starts from the copied
design and rates far below the incumbent.

**Ideas.**
- Schedule structural attempts deliberately: at least one in every four attempts, immediately
  after the three-rejects rule fires, and whenever the external line has been flat for five
  accepts. A rejected structural attempt is an ordinary outcome.
- A rewrite is warranted when the strategy changes drastically. It is usually faster than expected
  and better in the long run because the infrastructure, the ledger and the plumbing survive it.
- Keep the plumbing, change the roles. Start a rewrite from the proven defaults (the opening, the
  defence, the navigation) and add the new design on top; the rewrite that began from the copied
  design rated far below the incumbent until the proven parts were restored.
- Set a stage budget and a closing criterion before a structural programme starts, with stages
  that separate execution failure from premise failure. Each stage needs its diagnostic before its
  gate; one structural programme ran thirty-six stages with every mechanism eventually
  working and was closed by the criterion written at stage thirty.
- Design a multi-session programme on paper first from the census numbers, not in code in an
  evening.
- Keep mechanics decoupled from the strategy on top so a pivot after a balance patch is days, not
  a rewrite of everything.

## 31. Ideas that are obviously right and lose anyway

**How it shows up.** A fix for a measured, real defect loses; a copied doctrine loses; a symmetry
fix costs strength; a knowledge repair stalls a mechanism that relied on ignorance.

**Ideas.**
- "It is obviously a bug" is not evidence that fixing it helps. A unit whose task is aborted and that falls back to
  guarding keeps its value; the fix that removed the waste lost until it was packaged with a use for
  the freed capacity. Fixes that free a resource paid; fixes that free only time did not.
- A mechanism that paid when knowledge was poor can stall once knowledge is good. Re-audit every
  mechanism downstream of a repaired channel.
- Fixing a real asymmetry can cost strength because sixty iterations were implicitly tuned around
  it. Fix early or price the fix.
- A defect confirmed is not a defect costing you; measure the price before repairing.
- A low cost metric can mean not playing: the bot with the lowest per-unit drain was losing to the
  one paying most. Measure the winner's value of a cost before calling it waste.
- A signal pointing the wrong way is refuted, not weak.
- An accept attributes the mechanism, never the reason you built it; if the number holds and the
  story fails, keep the number and log the attribution as open.

## 32. Compute, time and disk

**How it shows up.** The disk fills three times; two runs share a class tree and one recompiles
under the other; a sync deletes a directory mid-gate; two drivers take the same run id from the
clock; a background job queued from a tool call dies with the call's shell; a completed run is
lost because its script was edited while running.

**Ideas.**
- Kill ideas as cheaply as possible, in this order: engine read, census of replays already on
  disk, one-map probe, small screen, full evaluation. Most attempts fail; the loop's job is to make
  failure cheap and informative. Your own past rejects give a real exchange rate for pricing new
  ones.
- Futility stops: end a screen once the candidate cannot reach the gate.
- Private class tree per run; atomic sync by rename, never delete-then-extract; unique run ids
  from more than the clock; a lock or semaphore across runners on a shared machine; never edit a
  shell script while a background job runs it (the shell resumes at a stale byte offset).
- Check disk before every gate; prune replays once a block's study is fetched; size the disk for
  the season (a gate keeps every loss).
- Detach long runners from the session shell so they survive it, and make finished-but-uncollated
  runs the normal casualty you recover from, not a loss.
- Verify a wait predicate matches a live process before queueing on it; a runner that re-executes
  under another name matched nothing and a queued job started early.
- Never kill by a pattern that appears in your own command line; inspect the process list, then
  kill by id.
- Count processes correctly (a timeout wrapper doubles them) and read a batch only when complete.
- Prepare the next candidate while the current gate runs; keep several arms queued and an idle
  filler that plays the shipped build on fresh random maps when the queue is empty.
- Run overnight volume; it finds real gains.

## 33. Records, memory and handoffs

**How it shows up.** A fresh session re-derives a defect its own log holds; a story replaces the
number it was told about and is later found false; a stale chart misleads the owner; thirty-one
commits sit unpushed; a lesson written down is repeated within the hour.

**Ideas.**
- An append-only training log, one entry per attempt: target, trace, pre-registration, diagnostic
  counters, gate numbers, decision, what was learned, and a "next" pointer. In-flight runs named by
  run id and gate so a fresh session can resume from the log alone. Corrections as dated entries in
  place, never edits of history.
- A closed-directions ledger: each closed avenue with the measurement that closed it, its kind
  (refuted, priced below the gate, blocked on a prerequisite, engine-impossible, unevidenced), and
  a checkable re-open condition, power-checked when written. Grep it by name before opening
  anything; a re-open condition that fires is written back to the entry.
- Durable lessons in a separate file, each naming the measurement behind it; a lesson without one
  is a belief and is marked as such. Consolidate on a schedule and audit for withdrawn claims.
- Record the measurement, not the explanation. Numbers survive their session; stories usually do
  not. Three catalogue entries were withdrawn because a story was written down instead of a
  number. A quotable line is the most likely thing in a report to be an explanation.
- A handoff document that is a closure map: current incumbent and its grade, what is in flight,
  every closed axis with kind and number, the gotchas that cost time, and a section of facts a
  context-free reader will misread.
- A lesson you wrote is not a control; install the check where the mistake happens. Ask: could
  the next session make this mistake without reading anything?
- Nothing stale stays: a chart or document that no longer matches the data is regenerated or
  deleted, at every accept.
- Keep mandatory reading small and everything else grep-only. A hundred-thousand-word log must
  not be read whole; a one-line-per-lesson index over an archive is what a new session needs.
- Every engine fact carries its provenance; every settled fact its population and opponent.
- Every user prompt recorded verbatim; every commit pushed. Committing is not delivering; anything
  not pushed is invisible and expendable.

## 34. Running the project as an AI agent

**How it shows up.** The session idles waiting on a run; context grows until compaction and the
sharpest work comes after it; two sessions of one lineage run concurrently against the same tree;
a status snapshot is trusted as a state; a fetched document is summarised into confident fabricated
quotes; the owner has to say "not much is happening".

**Ideas.**
- Instruments first, and prove each on a trivial case before it judges anything real.
- Never idle. Nothing resumes you automatically; polling a run is execution. Do not wait for a
  scheduled check to start work you already know you need; the periodic check is a failsafe, not a
  pacing device.
- Cycle sessions on a context budget, not on failure; state lives in committed artefacts, not in
  context. Resume prompts should say what not to re-read.
- One session per working tree and per source-control index; stop the old agent explicitly before
  relaunching; "completed" is a snapshot, and messaging a finished agent resumes it.
- Delegate reading, not deciding. Panels of sub-agents produced accurate low expectations and one
  real finding; the loop still had to run the measurement. Parallelise games, not agents.
- Stage explicit paths in commits, never everything; an untested edit shipped under a chart
  commit.
- Write process rules into scripts, not memory; every "remember to" rule failed at least once.
- Read documents from extracted text, not through a summarising fetch that invents quotes.
- Keep a machine-checkable resume point (run id and gate, not intention); a clean working tree is
  not evidence that an iteration never ran.
- Make progress visible: a status line, charts refreshed by the post-block script, a heartbeat a
  watchdog can see. From the owner's seat "waiting" and "doing nothing" look identical.
- Honour a pre-registered rule's spirit: it is a commitment about reasoning, not a licence to skip
  verification when the number lands favourably.
- Say plainly when you are at a local optimum under your instruments, and name what a different
  instrument would have to show.

## 35. Working with the human owner

**How it shows up.** The owner follows from the repository and a phone; asks where the work is;
supplies the risk appetite, the never-idle rule, the "copy the tactic" rule and the "if two
instruments disagree, run the third" rule that the agent would not have found alone.

**Ideas.**
- The owner's role is advisory and asynchronous. Act on bot design without approval when told to;
  the worst that can happen is a measured failure. Reserve questions for decisions that change the
  objective or the rules of measurement.
- Surface disagreements between instruments to the owner as disagreements, not as a verdict; and
  surface method decisions (what to grade on, when to retire a lineage) proactively rather than
  waiting for them.
- Keep the documents the owner reads current at every accept: the standing, the chart, the
  handoff. Report ladder blocks as they land.
- Record time in the owner's timezone; record every prompt verbatim.
- The most valuable interventions in five seasons were about method, not strategy: restate the
  objective as absolute strength, not beating a rival; play the frozen roster on every accept;
  copy an opponent's tactic and submit it; re-read the principles of other years when stuck; keep
  trying and rewrite if that is what it takes.

## 36. Rules that keep the measurement honest

**How it shows up.** A benchmark's source is read and the bot overfits to it; a chosen map is
played against an external bot; ladder games are spent to answer a development question; a
candidate under trial pollutes the team rating.

**Ideas.**
- Never read external opponents' source when the point is to approximate a contest; compile it
  automatically and study replays. Never read post-mortems of the season you are practising on.
- External opponents only as scrimmages under contest rules; external bots never play each other;
  the ladder is built from your own games.
- A candidate never plays the ladder; you submit what you believe in. Accepting means
  submitting; a rating drop withdraws.
- A recorded game may be examined and replayed between your own bots; nothing may stand in for
  playing an external opponent in a way the contest would not allow.
- Enforce these in the tools, fail-closed; a rule enforced by memory was breached once by
  forgetting.

## 37. Tournament realities

**How it shows up.** Placement in an early sprint means nothing and the meta shifts with balance
patches; a bot that loses badly on a fifth of maps loses best-of-three sets; a last-minute change
carries a fatal bug; a dominant gimmick is nerfed.

**Ideas.**
- Treat early tournaments as reconnaissance: classify the field into archetypes and choose.
- Prefer a higher floor to a higher ceiling; reduce map-specific failure modes.
- Parametrise anything the developers might nerf (patterns as data, tile scores as constants,
  spawn weights); assume dominant gimmicks get nerfed; re-read the spec after every patch.
- Never submit an untested last-minute change; before a deadline make one or two impactful
  changes very well.
- Pace the effort; two bot iterations a day for weeks beats a burst and a fizzle.
- Do not reveal a last-minute counter to its target in scrimmages, but test it against someone.

---

# Appendix A — Strategy literature mapped onto a grid game

Use these as a checklist of where to look and as names for failure shapes, not as design
authority. In one season the framework was written after the fact and validated nine of eleven
maxims; the two it did not (scouting, prolonged war) failed on price, not principle.

- **Sun Tzu.** Know the enemy and yourself: scrimmage widely and watch losses; extensive self-
  knowledge coexisted with zero enemy knowledge until the first external tally. Attack where
  unprepared: rush before defences exist, approach along edges, harass the economy. All warfare is
  deception: vision-versus-range asymmetries and information denial. Put yourself beyond defeat
  first: standing defences before offence, sized relatively. Move only for advantage: verify
  before committing. Spies: information has a price; scouting that costs more than it returns is
  not foreknowledge.
- **Clausewitz.** Centre of gravity: find the one thing the enemy system depends on and strike
  it collectively. Friction and fog: everything that can go wrong does; the bots that perform
  best do the basics robustly. Culminating point: an attack that outruns its strength flips;
  engage and retreat on local counts. Defence is the stronger form, but the defender must still
  end the war before the clock does.
- **Jomini.** Interior lines and decisive points: hold the centre, spawn toward the enemy,
  broadcast fronts so reinforcement travels short paths; place static defence at chokepoints.
- **Liddell Hart.** The indirect approach: flank with cheap units, harass rather than assault
  fortifications, and never renew a failed attack along the same line.
- **Boyd.** The per-turn sense-orient-decide-act loop is an OODA loop; advantage lives in
  orientation (map memory, symmetry, danger maps). Tempo applies twice: in-game against slow
  production reactions, and in development, where two iterations a day beats one a week.
- **Lanchester.** Aimed fire scales with the square of numbers: concentrate, never trickle, split
  the enemy, and use predicted outcomes to decide engage or retreat.
- **Principles of war.** Objective (every unit has a goal at all times), mass and economy of
  force (main effort paid for elsewhere), security (the floor), simplicity (plans that survive
  friction), unity of effort (emergent, not commanded).

# Appendix B — Checklists

**Day one.** Rules digest tagged with engine provenance. Trivial bot through the runner, the
replay reader, the bytecode monitor, the mirror, the snapshot tool and the unit tests. One game
read for its ending reason. Determinism checked; seed per game. External opponents compiled and
frozen. Play-symmetry audit of every tie-break. Charts and a handoff file that exist and are
regenerated by the post-accept script.

**Before a candidate.** Trace the motivating replay; check a second game. Grep the ledger.
Pre-register: mechanism, decision counter, reachability, trigger frequency across other games,
price against what it displaces, gate, falsifier, which instrument decides, dose ladder with a
zero arm. Run one logged diagnostic and read the counters and the overruns. Only then gate.

**After an accept.** Snapshot. Archetype regression. Submit a scrimmage block; rebuild the
ratings and charts; mine the block for the next candidate. Update the ledger, the learnings, the
handoff. Re-audit old premises if the accept was structural. Commit explicit paths and push.

**Before a submission deadline.** Nothing untested. Overruns zero. Mirror even on every map.
Frozen roster played. Communication constants changed.

# Appendix C — The rules that recur most

1. Build the instruments before the bot, and prove each on a case whose answer you know.
2. The engine is the truth; read it for every mechanic you rely on and tag the fact with where.
3. Census your own units before tuning: idle resources and frozen units are defects, and repairs
   of defects transferred every time while reallocations mostly did not.
4. No gate until a logged game shows the decision counter firing; count the choice, not its
   effect.
5. Pre-register the counter, the gate, the falsifier and the deciding instrument; never move a bar
   or reinterpret a null after the number.
6. Pair every candidate game with a control on the same seed; judge by discordant pairs and by
   the shape of the game-by-game diff, never by a total.
7. Self-play prices cost and finds your own defects; it cannot judge anything aimed at the field.
   Keep the mirror, the archetypes and the external ladder, and never confuse them.
8. Copy the opponent's offence and submit it; build the defence against your own copy; make
   defences conditional so they cost nothing where the threat is absent.
9. Emergent coordination beats commanded coordination; standing defences that cost no actions
   beat bodies; withholding waste beats adding work.
10. Record the measurement, not the explanation; keep a ledger of closed directions with their
    arithmetic and re-open conditions; hand off from those, not from memory.
11. Put every process rule into a tool that refuses; a lesson written down is not a control.
12. Never idle, never go silent, push everything, and when every small change fails, look for the
    jointly necessary pair or change the design.
