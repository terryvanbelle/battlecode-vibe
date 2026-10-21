# MISSED_IDEAS_2026 — Battlecode 2026 ("Uneasy Alliances") -- ideas the top teams used that we never found

Written 2026-10-21. Post-hoc comparison of our bot (`battlecode26-vibe`, final snapshot
`g_iter34`) against the published 2026 post-mortems and the public source of three
finalist bots. During practice the agent was forbidden from reading either; this is the
first read. Every claim about THEIR bots below is from the cited file/section; every
claim about OURS is from our repo. Anything inferred rather than read is marked (inferred).

## 1. Sources and where we stand

**Post-mortems read in full (PDF, `pdftotext -layout`):**
- *Generalized Stroke's Theorem* (2nd overall; Urav Tanna, Seth Lifland, Andre Mao, UVA), 10 pp.
  Found vendored as `uravt/Battlecode26/Battlecode_2026_Postmortem.pdf` (the battlecode.org copy is network-blocked).
- *food* (Alex Thummalapalli, solo, Georgia Tech; Top 12 finalist, 9th-12th), 31 pp.
  `AlexT101/battlecode26/Battlecode_Postmortem_2026.pdf`. Note: our `BENCHMARK.md` calls this
  bot `bench_finalist` and credits it with the "Best Postmortem Award"; the author is team **food**,
  so the "food" post-mortem the task lists as blocked is in fact fully read here.

**Post-mortems NOT reached:** *Lorem Ipsum*, *3MiceWalkIntoABar*, *Sloppy Joe Pirates*
(battlecode.org PDFs blocked; eight WebSearch queries returned only the index page, no content).
The winner, *subscribe to caterpillow* (Naman Doshi, Alexander Wang, Andy Wu, Yiheng You), published
no post-mortem on battlecode.org/past.html; the GitHub user `Caterpillow` has only `caterpillow.github.io`,
and twelve repo-name guesses (`git ls-remote`) found nothing. The food PM records that food beat
Lorem Ipsum 3-2 and lost 2-3 to Mango Tango (5th-6th) and Muscallonge (3rd).

**UNSW "UnderTheC" write-up** (`Battlecode-UnderTheC/battlecode-underthec.github.io`, 8 markdown posts):
this is **UNSW Battlecode 2026, a different game** ("dragons" -- snake-like chains on a torus with kelp
and portals, `unswbc` toolkit), not Uneasy Alliances. Only its process lessons (an LLM-agent loop) are
comparable; see 2.8.

**Code read (shallow clones of the public repos):**

| repo / package | lines | read |
|---|---|---|
| `uravt/Battlecode26` `src/Version41` (GST, final) | 5,050 core + ~24,000 generated (`Pathfinding/*BFS*.java`, `Communication/Communicator.java`) | all core files in full: RobotPlayer, RatKing, SymmetryManager, BabyRat (hand-written part), both StateUpdaters, all 14 Behaviors incl. BRMicro decision tree, BRCatMicro, Utilities key methods, Communicator layout/encodings, Pathfinding.attemptMove, Reachability head, `Tester.py` |
| `AlexT101/battlecode26` `src/finalsbot` (food, final) | 5,984 | RobotPlayer (2,504) in full, Globals, Nav, Map in full, Helpers key methods, Comms index |
| `erikji/battlecode26` `src/SPAARK` (HS 4th) | 11,336 (Comms 6,268 mostly generated) | RobotPlayer, G, RatKing, RatKingMicro, Sustain, BabyRat, BabyRatMicro in full; Motion explore/microMove/doMicroAction; Comms squeak/shared-array/form/defend sections; `runmatches.py`, `copybot.py`, `changeseed.py` |

**Where our bot stands** (`BENCHMARK.md`, `TRAINING_LOG.md` Iterations 238-252): final accepted
snapshot `g_iter34` (Iteration 251). Benchmarks **10/162** against the vendored finalist bots
(`bench_lecture`, `bench_anicolao`, `bench_finalist`=food, `bench_spaark`, `bench_stroke`=GST V41);
peers **162/216**. Recorded weaknesses the comparison should explain: the benchmark gap is
**rat-turns 2104 vs 4187** with identical per-rat action rate (Iteration 238); catDamage is a
zero-sum pool we take 43.5% of in cooperative losses, and our cat-trap pool is EMPTY 76% of turns
(Iteration 252); the population cap gate (`MAX_POPULATION = cheese > 1500 ? 60 : 25`) is dead on
one map and load-bearing on another (Iteration 252, screened not decided); 91% of benchmark losses
are King destruction, and the enemy King is untouched (600 HP) in 7 of 8 (TRAINING_ALGORITHM
"Closed directions").

## 2. Ideas we never figured out

### 2.1 Macro strategy and win condition

**Backstab is the default, not a decision.** All three bots fight any enemy rat they can reach with
no `isCooperation()` check: GST's baby-rat state updater is literally `DefectBRStateUpdater`
(`BabyRatStateUpdaters/DefectBRStateUpdater.java`: `nearestSeenEnemy != null` -> `BRMicro`), and
`Utilities.attemptAttack`/`attemptRatnap` carry no cooperation guard; SPAARK `BabyRat.exploreCheckMode`:
`if (G.opponentRobots.length > 0) mode = ATTACK`; food runs `doAttack()` (ratnap, throw, bite) every turn
an enemy is adjacent and its PM states "in practice, most games ended in backstab mode; the payoff
for early aggression ... pretty much always outweighs the uncertain outcomes of cooperation" (food PM
§Scoring; GST PM §1.2 says the same). *We* never bite first (`RobotPlayer.java` `!rc.isCooperation()`
gates on every rat attack; Iteration 210 revoked the last first-bite licence; Iteration 211 gates
even the trap ring on `!isCooperation()`), and `TRAINING_ALGORITHM.md` §"Cooperation and backstab
strategy" treats it as a policy axis to cover with peer archetypes. Why it mattered: against every
real opponent the war starts on their terms, and our peer set (half of which cooperates) measured a
game that tournament bots do not play -- 19 of 28 peer losses are points games (Iteration 204)
while only 14 of 154 benchmark losses reach points (Iteration 146). Our catDamage work (Iterations
128, 204-206, 213, 218, 252) defends a 0.5-weight term that decides almost no real game. (inferred)

**Multiple Rat Kings formed ON cheese mines are the economy.** food: a King writes a formation
location (shared slot 9) when `allCheese - 50 >= (numKings + 2) * 600` and `sharedArrayMineCount >=
numKings` (`RobotPlayer.checkRequestNewRatKing`, `summonKingThreshold`); the location is a discovered
mine at least `MIN_SQUARED_DIST_FROM_KING_TO_SPAWN = 40` from every King (`findFormRatKingLocation`);
COLLECT rats switch to `BUILD_RAT_KING`, and a rat standing on a mine with `sensedAllies > 3` may
`squeakBecomeKing()` itself (`runBabyRat`). PM: "many matches were won simply by who formed a rat king
first" and the first King "would be stuck in a bad spawn location". GST: `maxRatKings = area <= 2500 ? 2 : 3`;
the chief King writes slots 20/21 with `Communicator.getBestFormationLocation()` (the known mine with
the greatest mean distance from existing Kings and threat level <= 60) once
`getCurrentRatCost() > RAT_COST_THRESHOLD * (1.8 + (numKings-1)/3)`, i.e. once the standing army is
~22-25 rats (`RatKingBehaviors/RKHeadquarter.java`); rats with `globalCheese > 1500` go to
`BRFormRatKing` and call `becomeRatKing()` only when > d^2 25 from the nearest King (so the old King's
3x3 body never disqualifies the 3x3). SPAARK: `Comms.writeFormRatKingLocations` when
`numberOfMines >= numberOfRatKings*2 && globalCheese > cheeseThreshold-100 && minNumberOfBabyRats >= 24`,
choosing the mine maximising `-sum(dist^2 to other mines) + 2*dist^2(self)`, plus an emergency rule
`numberOfRatKings == 1 && (round > 1900 || health < 100)` that forms a second King three tiles away.
*We* tried eight times (Iterations 18, 50, 74, 75, 106, 107, 146, 173) and closed the direction:
opportunistic upgrades never fired because our only cluster is around our own size-3 King (106),
and musters cost the army (146, 173 at -28 peers). The difference is that theirs is an *economic*
move -- the new King is placed where cheese is, becomes the delivery point, and is gated on
population not treasury -- while ours were defensive musters at arbitrary points.

**Population is a target in rats, read off `getCurrentRatCost()`.** All three derive the live
census from the price: food `estimateAliveRats() = (cost-10)/10*4 + 2` with floors/ceilings
`MIN_ALIVE_RATS 8`, `MAX 16`, `22` above 2000 cheese, `32` above 2500, `42` above 3800
(`Globals.java`, `shouldSpawnRats`); GST builds whenever `getCurrentRatCost() <= RAT_COST_THRESHOLD`
(36 on maps <= 23x23, else 40, i.e. ~12-15 rats) or `inDistress` or cheese >= 2000 after r100
(`RatKing.java`, `RKHeadquarter.execute`); SPAARK keeps a 100-round rolling mean of the cost
(`Sustain.java`) and spawns while `getCurrentRatCost() <= spawnCost`, where `spawnCost = mean+10`
when rich and `globalCheese/100 + 25` otherwise, capped at 30 before round 200 (`RatKing.run`).
`Comms.minNumberOfBabyRats = (cost-10)/10*4` is also what gates King formation. *We* count
`builtCount` cumulatively per 200-round window against `MAX_POPULATION` (25, or 60 when cheese > 1500)
and a reserve of `8 * getCurrentRatCost()` (g_iter34); Iteration 252 measured the King
cheese-blocked on 83.5% of turns and Iteration 228 traced "25 builds in rounds 0-99 then NOTHING
until r400 with 1095 cheese and zero rats". A standing-army target replaces every death the
round it happens, which is exactly the rat-turn deficit of Iteration 238. The food PM adds the
caveat we would have hit next: 16 standing rats was too many on small combat maps and should
scale with map size (GST does: `RAT_BUILD_THRESHOLD = 0.002*area + 6.2`).

**Endgame tiebreaker spending.** food spawns everything above 600 cheese from round 1950
(`shouldSpawnRats`); GST has `SPAM_BOTS_FRENZY_ROUND = 1900` (build while cheese >= 667) and enters
cat micro after `ATTACK_CAT_ROUND = 1700` "for tiebreaker reasons"; SPAARK sets `scorePerCatHp = 100`
from `TARGET_CAT_ROUND = 1900` so rats start biting cats. *We* have no round-gated endgame at all
(Iteration 196 "endgame cheese push -- VOID on the pre-check"). Iteration 251 found nine
points-decided losses within 1.0 point and `safelycontained` lost by 115 cheese of 25,755. (inferred
that a last-50-round spend flips several of those.)

**Cats are a hazard first and a cheap damage source second; cat traps are recycled.** food bites an
adjacent cat only when no enemy is sensed or squeaked (`doCatAttack`), and lays cat traps only when
`kingSOS || round > 600`, cooperating, and cheese >= 900; the PM regrets not doing more. GST bites a cat
within d^2 8 only with no enemies in view, lays a cat trap in the actionable tile inside the cat's
180-degree cone (`Utilities.attemptCatTrap`, `isWithinDistanceSquared(loc, 7, cat.direction, 180)`),
and -- the piece we never knew existed -- **picks its own cat traps back up** when cooperating and no
cat is visible (`Utilities.attemptRemoveCatTraps` -> `rc.removeCatTrap`, called from `BabyRat.runBabyRat`),
recycling the 10-trap team cap. SPAARK's `trapScore` prices a cat trap at `-10*scorePerCheese +
50*scorePerCatHp` per adjacent cat and only while `getNumberCatTraps() < 10 && round - 100 <
backstabRound`. *Our* Iteration 218 rejected speculative traps because "the binding constraint is
CAT_TRAP.maxCount 10 ... a speculative trap crowds out one that fires", Iteration 135 concluded "the
shared cap was the answer all along", and Iteration 252 measured the pool empty 76% of turns and
full 15.5%; `removeCatTrap` has zero call sites in `src/bot/RobotPlayer.java`.

### 2.2 Economy

**Carry a batch, not a crumb; cap the load.** GST returns at `RETURN_CHEESE_THRESHOLD = 20` once no
cheese is visible, always at 100, and refuses pickups above 100 (`BabyRat.java`, `Utilities.attemptCheesePickup`);
food returns at 40 if no cheese was seen last round (else at 80), collects up to 100, 140 within
d^2 48 of a King and 500 adjacent, and will not detour for < 5 cheese (`Globals.java`, `cheeseThreshold()`,
`decideState`); SPAARK retreats above 30 raw (or 149 while cheese is still visible) and uses
`pickUpCheese(loc, amount)` to cap at 150 (`BabyRat.attemptCollectCheese(int)`). *We* deliver the moment
`getRawCheese() > 0` (`runBabyRat`: `deliverCheese` runs first); Iteration 115 tried batching and
rejected it (section 3). The slowdown is `0.01 * rawCheese` per `RULES.md`, so 20-40 cheese costs a
rat 20-40% speed, which all three accept in exchange for halving trips.

**Transfer cheese to any of the King's nine body tiles, including across walls.** food tries
`transferCheese` on the centre and all eight `translate(dx,dy)` tiles (`senseNearbyAllies`); the PM
(Fig. 11, map "Hike") says rats escape a maze once and then deliver over the wall without re-entering
it. GST `Utilities.attemptCheeseTransfer` tries the near face tile and its two neighbours. *We* call
`rc.canTransferCheese(kingLoc, ...)` on the centre only (`deliverCheese`).

**Cheese-mine memory with cooldowns, assignment and symmetry mirroring.** GST keeps
`SymmetryManager.seenMines` (location -> threat level), inserts the symmetric twin with sentinel 127
the moment a mine is seen, squeaks new mines to the King on delivery (`Communicator.findMineToStore`),
and the King packs ten mines into shared slots 0-19 (x:6, y:6, threat:7 bits); rats returning home
pick `targetCheeseMine` (70% the nearest mine in their explore direction, 30% random) and
`updateMineThreatLevels` counts enemies/cats within d^2 41 of each mine; the King drops mines into
`unsafeMines` (reset every 75 rounds). food stores mines in slots 11-63 (lossy encoding when the map
exceeds 1024 tiles, `Comms.initLossyEncoding`), adds mirrored mines once symmetry is known
(`Map.addSymmetricalMines`), puts a mine on `mineCooldowns` after 3 rounds without cheese or when a
King is near, and spreads rats by starting the scan at `rc.getID() % size` (`findMine`). SPAARK
tracks `lastVisited` on a 12x12 grid and revisits a mine only after `20 + 5*sqrt(area) + 10*numMines`
rounds, choosing among stale mines with probability `1/(dist+1)` (`BabyRat.exploreCheckMode`). *We*
had zero `hasCheeseMine()` call sites until Iteration 232, whose unbounded "walk to the remembered
mine" lost on the open map (pickups 593 -> 233) and was rejected along with 233. Their versions work
because of the cooldown/assignment layer, which ours lacked.

**Dig through dirt as part of pathing.** GST PM §4.2: "treat all dirt as passable, simply stalling
the bot if its action cooldown became too high ... allowed us to win decisively on maps where vital
cheese mines were completely buried", later a BFS dig cost; code: `Pathfinding.attemptMove` sets
`digDirt = (rawCheese > 10 || allCheese > 1000) && canTurn && sawDirt` and selects among
`BabyRatBFS`, `...ClearDirt`, `...AvoidCat`, `...AvoidCatClearDirt`. food `Nav.canPass` treats dirt as
passable when `canRemoveDirt && globalCheese > 600` and no enemy is sensed, and waits a turn when the
next tile is also dirt (PM Fig. 12). SPAARK `Motion.move`: `if (canRemoveDirt(adjacent)) removeDirt; move`,
and the King clears the five tiles in front before stepping. *We* dig only as a last resort when
every direction is blocked (`digTowardOpenSpace`, Iterations 2 and 4).

**Mobile King that collects.** food Kings navigate to the nearest discovered mine not within d^2 20
of another King (`findKingMine`, `runRatKing`) and pick up cheese on the way; GST's `RKHeadquarter`
walks the King to `closestCheeseMine` (or map centre after `LEAVE_CORNER_ROUND`) and spawns rats two
tiles out along the longest open ray with centre and mine bias (`doDirs`, `getBestDir`, `getMineBias`);
SPAARK's King has EXPLORE / COLLECT_CHEESE / FLEE modes and bug-navs onto mines. *We* never move the
King except a d^2 20 cat flee; Iterations 126/127 (relocate onto a mine) were rejected and
`TRAINING_ALGORITHM.md` closes "Mobility". The King being the delivery point is what makes
moving it worth 40 movement cooldown. (inferred)

### 2.3 Combat micro

**Score all nine tiles, with the action you could take from each.** GST PM §6 ("XSquare style
micro"); `BRMicro.microInfo.isBetter` is a lexicographic decision tree: `safe()` first, then
`inRange()` if the action is ready, then standing on our own rat trap, then kite the enemy King
when alone and on cooldown, then prefer diagonal adjacency (d^2 2) to cardinal (d^2 1), then
closer when `bold` / farther when `inRetreat`, then fewer `enemies8`, then no strafe. food
`calculateMoveHeuristics` with named constants: `CAT_ADJACENT -200` (divided by 10 when the cat is
not looking), `ENEMY_RAT_ADJACENT -10` (-15 if higher HP), `IS_ENEMY_RAT_CARDINAL -5`,
`FLYING_RAT_DANGER -10`, `RATNAP_POSITION_BONUS +50`, `RATNAP_HP_BONUS +35`, `ATTACK_KING +30`,
`FINISHER +20`, `READY_ATTACK +10`, `DESTINATION_MULTIPLIER 3*sqrt(dist)`, `STRAFE -5`; enemy
penalties halved when allies outnumber 2:1 and inverted under a King SOS. SPAARK `BabyRatMicro.micro`:
+300 toward the bug-nav direction, -500 for cardinal adjacency, -250 for diagonal adjacency to a
healthier rat, -300 for being inside an enemy's cone within d^2 8, -1000 for the two tiles in front
of a cat, plus a per-tile `MicroActionScore` (attack / cheese-boosted attack / trap / cat trap /
ratnap / throw) added to the move score (`actionScores[]`, `Motion.microMove`). *We* have
`engage()` (bite if adjacent else step toward) and `flee()` (step directly away); every single-rule
alternative (Iterations 8, 9, 42, 62, 100, 104, 105, 149, 150, 188, 234) was rejected. The 0.02:1
kill ratio and 47-0 throw asymmetry (Iterations 210, 246) are what one-dimensional micro looks like
against nine-tile scoring. (inferred)

**Manage the 90-degree cone: defensive facing, alternate looking, read the damage signature.**
food `calculateDefensiveDirection` chooses the facing that leaves the fewest enemies outside the
cone, `rotateRandomlyAfterMove` alternates left/right while walking (PM Fig. 20), and `checkAmbush`
reacts to an unexplained HP drop: exactly 20 = cat (step forward, turn round), otherwise rat (turn
round). GST `Pathfinding.attemptMove` uses a four-step `turnCadence` (move then turn +/-90) so an
explorer sweeps both flanks, and `Utilities.attemptRatnap` deliberately steps to a tile *outside the
victim's three-tile cone* before grabbing. SPAARK adds +10 to the alternating left/right turn score
every two rounds and +100 toward the last seen opponent, and turns to face any opponent within d^2 8
after moving. *We* only ever face our movement direction (`stepTo`); Iteration 123 "turn to face an
unseen attacker" and 234 "retreat without turning our back" were rejected, and Iteration 246
concluded the blindness is "structural ... not addressable at all". It is addressable; it is
managed as a resource.

**Ratnap and throw are the main weapons, and the throw is a scored path.** GST
`Utilities.attemptThrow` enumerates stay/move-back/left-90/right-90/diagonal/forward combined with
three throw headings and scores each path with `scoreThrowPath`: enemy hit `8 * remaining`, cat 200,
wall or dirt `4 * remaining`, ally `-4 * remaining` (x5 for the King), early exit at >= 48, throw when
score >= 15 or `turnsCarrying >= 8` or own HP <= 20, never toward our King within d^2 25. GST PM §6.3
calls this "super important"; §6.1 says enemies behind you "can ratnap you regardless of health".
food prefers ratnapping the highest-HP enemy or one carrying an ally, scores throw lines
`200/iters` enemy, `100/iters` wall, `50/iters` map edge (`findBestThrowDirection`), marks the two
tiles ahead of any flying rat as danger (`flyingRatDangerSpots`), and dumps an expiring carry away
from the King (`tryLastResortThrow`). SPAARK `ratnapScoreStrafe`: +1700 for a legal grab, +1000 +
`i*100` for a throw that lands on an enemy, +1000 + `i*50` into a wall, -100 if an ally is in the
line. *We* have zero call sites for `carryRat`/`throwRat` (Iteration 235); Iterations 17, 54, 71,
72, 143, 245 were rejected and 246 declared the mechanic unusable. Iteration 245 measured 1642 throw
damage taken per game against 0 dealt; Iteration 235, 52% of our rat deaths are throws.

**Spend cheese on a bite only to finish or to flip the HP comparison.** GST `bestCheeseAmount`:
`cheeseToKill = (hp-10-1)^2+1` when affordable; otherwise spend to erase an HP deficit of <= 15 so
the enemy loses its ratnap right (`canCarryRat` needs `targetHealth < grabberHealth`, `RULES.md`);
against a King, `min(raw, 17)`. food `getCheeseForAttack`: finisher if <= 3 extra damage needed,
deficit if <= 3, only above 600 cheese; the PM dropped an always-+2 rule as "essentially meaningless".
SPAARK `attackScore`: hpDiff 0 -> 1 cheese, 1-2 -> 2 cheese, and `doMicroAction` has the
`ceil(sqrt)` breakpoints 1,2,5,10,17,26,37,50,65 as ATTACK_1..9. *We* spend a flat 4 cheese on
**cats** when cheese > 1000 (`BITE_BOOST_CHEESE`, Iterations 45/83), which buys +2 against a
4000-HP target, and never on rats.

**Cat micro is about the cone and the "cat path", not distance.** GST `catMicroInfo` marks
`inCatVision` via `isWithinDistanceSquared(loc, 17, cat.direction, 180, true)`, `inCatPath` from a
table of the 2-3 tiles ahead (`Utilities.catPath`), measures distance from the cat's centre
(`x+0.5, y+0.5`), and `BRCatMicro.betterCatMicro` orders: leave vision > leave path > (if action
ready) bite > ratnap an enemy to feed the cat > trap > avoid being ratnapped > enemy DPS > stand on a
cat trap > distance. The PM (§6.2) names the cat path "by far the most dangerous squares". In rescue
mode GST also drops dirt on the cat's actionable tiles (`tryBuild(getCatActionableLocations)`) and
throws enemies into the cat (score 200; fed cats sleep two turns per `RULES.md`). food's King
escapes *perpendicular* to the cat's heading (`evadeCat`, `rayMetric`), and SPAARK scores the two
tiles in front of a cat at -1000. *We* flee straight away from the cat and bite when adjacent;
Iteration 21 (flee only inside the cone) and 23 were measured inert, which with hindsight says the
flee rule, not the cone test, was the problem. (inferred)

**King distress is a broadcast that changes everyone's micro.** food: SOS bit (slot 8) set when an
enemy or a cat is within d^2 16; rats within d^2 16 enter RETURN and `kingSOS` inverts their
enemy-proximity penalties so they charge (PM Fig. 19); the King spawns toward the enemy
(`trySpawnInDir`), lays rat traps toward it, and moves away. GST packs `inDistress` into the King
record, rats within `ANSWER_DISTRESS_DIST_THRESHOLD 25` go `bold` (`BRMicro`), `RKFlee` builds on
the tile nearest the threat direction, traps in front of the enemy, and squeaks a `PresenceSqueak` so
rats learn the attacker's position; the King also publishes the attackers' mean location (slots
46/47). SPAARK writes a (location, 1 for cat / 2+n for enemies) pair per King
(`writeDefendRatKingLocations`), rats within `(10+n)^2` switch to DEFEND_RAT_KING, and defenders
nearer the cat than the King **squeak to pull the cat off the King** (`BabyRat.defendRatKing`:
`G.rc.squeak(...)`, since cats turn toward squeaks per `RULES.md`). *We* have a static 16-tile trap
ring and the King bites after building; Iterations 183 (rats prioritise the cat at our King),
184 (King broadcasts a cat bearing) and 187 (King census) were closed *before* a run, and 166/237
used squeaks to lure cats toward traps far away rather than off the King. Iteration 182 found a
parked cat grinding the King from outside its reach "unanswerable"; the lure is the answer SPAARK
chose.

**Reachability before engagement.** GST computes 7x7 reachability by bit-parallel BFS in ~300
bytecodes (`Reachability.bitBFS`, PM §4.2.4) and ignores enemies that `!Reachability.query(loc)`
when choosing micro targets (`BabyRat.updateInfo`); food `canReachEnemyInMoves(myLoc, enemies, 3)`
is a bounded bitset BFS and the PM says "we don't use micro if enemy rats are unreachable (on the
other side of a wall)". *We* chase anything visible within `CHASE_RADIUS_DSQ` regardless of walls.

**Aggression keyed to local numbers that include allies you cannot see.** GST sets `inRetreat` when
`rawCheese >= 40 || allies - enemies <= -2` and `bold` near a distressed King, where `numberAllies`
comes from presence squeaks (`BabyRat.processSqueaks`); food halves enemy penalties at a 2:1
advantage and its PM §Combat Theory explains why passive beats aggressive 1v1 (invisible traps) and
why standoffs must be broken by numbers; SPAARK weights enemy penalties 0.75 vs 1.0 by
`opponentRobots.length >= numberOfAlliesNearby`. *Our* ally counts come only from our own cone
(`countAlliesNear`), which is why the `allies >= 3` gates of Iterations 63-67 "never fired".

**Sacrifice while carried.** GST `BRThrownCarried`: a rat carried or thrown toward its own allies
calls `rc.disintegrate()` when below 20 HP so the landing kill is denied; a carried rat turns to keep
seeing. food has `CARRIED_DISINTEGRATE_THRESHOLD = 12`. We never use `disintegrate`.

### 2.4 Navigation

**Bug-nav with loop memory, plus a BFS front end.** GST PM §4.1: unrolled radial BFS (6-8k
bytecode), fall back to Gone-Fishin'-style stack bug-nav when BFS stops making progress, simulate
both turn directions, turn away from map borders, and reverse when a "turn location" repeats. food
`Nav.goTo` (XSquare bug-nav): a `states[x][y]` char = `(bugPathIndex<<14 | obstacle.x<<8 |
obstacle.y<<2 | rotate bit)` resets the path when the same state recurs at the same tile,
`checkRotate` picks the side by simulating up to 8 rotations each way, and a King blocked by its own
rats "acts as if it moved" instead of rerouting. SPAARK `bug2Helper` with `simulateMovement`. *We*
use Bug2 only for `deliverCheese` (Iteration 35) and greedy stepping elsewhere; Iterations 36 and
231 (bug-nav for collection) were rejected; our stuck escape is a 2-tile-oscillation detector with a
random shuffle. Mazes (`corridorofdoomanddespair`, `minimaze`, `jail`) are our swept losses
(Iteration 249).

**Explore toward targets, not along a compass heading.** food: before round 200 pick one of nine
goals (corners, edge midpoints, centre) not within d^2 20 of a King, afterwards the nearest unvisited
3x3 block (`Map.getRemoteLocation`, `getUnvisitedLocation`). GST `BRExplore`: initial direction away
from the parent King, target = the farthest on-map tile along it (`assignExplore3Dir`), re-pick
weighted by distance to the edges when the heading runs off-map (`checkDirection`), and after round
75 with symmetry known, head for mines. SPAARK picks random unexplored locations from `long[60]`
explored bitmaps and keeps the King out of `catSeenLocs`. *We* give each rat a compass heading from
`rc.getID()` and re-roll it after two stalled explore calls (`explore()`); Iteration 121 ("steer
explorers off the map edges") was rejected and "explains the whole plateau".

### 2.5 Communication

**Squeak is the baby rats' network, with a typed header.** GST PM §3.2: three header bits select
among 7 `SqueakInfo` records (`Communicator.SqueakTypes`); §6.1 "squeak maximization": every rat in
micro squeaks its health plus nearest enemy location, direction and health every turn
(`PresenceSqueakInfo`, 7+6+6+3+7 bits), and `BabyRat.processSqueaks` merges them into `allRobots` /
`allEnemyLocations` so a rat sees behind itself; cat squeaks carry the cat's facing; mine squeaks are
sent on delivery; symmetry is squeaked to the King. SPAARK packs one 32-bit squeak with enemy
relative location (7 bits), id mod 256, HP (6 bits), facing, type, own state and travel direction
(`Comms.sendSqueakMessages`), derives `numberOfAlliesNearby` from distinct squeakers over two rounds,
sends "critical information" (new mines, cat sightings, symmetry) as type 3 when within d^2 16 of a
King, and **suppresses squeaks within d^2 80 of a known cat or of its own King** so it never lures
cats home. food resqueaks the enemy King's location outward (PM Fig. 9), squeaks the two nearest
enemies when bytecode allows, avoids squeaking when a cat is sensed, and says squeaked enemies "greatly
improved my combat micro". *We* have zero `rc.squeak` call sites in the final bot; Iterations 109,
110, 166, 170, 237 were rejected, and 237/238 profiled 1086 vs 0 squeaks in a game we won.

**Shared array as a King registry with heartbeats.** food slot 7 is a heartbeat bit per King,
slot 8 the SOS bits, slots 2-6 positions; a King whose bit stops toggling is erased by another King
(`manageKingLifecycle`). GST packs x, y, turn mod 128, distress, health and a valid bit into three
slots per King (`Communicator.storeRatKingInSharedArray`) and evicts records not refreshed for 2
turns. SPAARK writes `round/2` per King and a new King claims the first stale id
(`RatKing.init`). *We* use slots 0/1 for the one King, 2 for desperation, 3/4 for a guessed enemy
King -- there is no representation for a second King, which every multi-King attempt also lacked.

### 2.6 Symmetry and exploration

**Infer the symmetry, then use it everywhere.** All three eliminate H/V/R by comparing each sensed
wall/mine tile with its three mirror images (GST `SymmetryManager.checkSym`, bitmaps `long[60]` for
walls, mines and vision, run with leftover bytecode >= 400; SPAARK `Comms.symmetryValid`; food
`Map.checkTileSymmetry`), publish it (GST slot 48 via a `SymmetrySqueakInfo` to the King; food slot
0, rats RETURN to report it; SPAARK slot 0 with `criticalSymmetry` squeaks). Uses: food's FIND_KING
visits the three `possibleEnemySpawns` and marks them checked (PM §Timeline; `pickSpawnTarget`);
GST places rat traps on the mine's enemy-facing side,
`cheeseMine.add(mine.directionTo(getSymmetric(ownKing)))` (`Utilities.attemptTrapMine`), and
`placeDirtMine` walls that side; both mirror known mines. *We* guess 180-degree rotation for the
enemy King (`runRatKing` slots 3/4), which Iteration 138 found wrong on 16 of 27 maps and disabled
(`DESPERATE_RAID = false`). `RESEARCH.md` §4 already called inference "Battlecode 101" and §7 item 2
asked for it; it was never attempted.

### 2.7 Bytecode and engineering

**Spend the budget on information: codegen, bit-parallel maps, bytecode-gated optional work.** GST:
generated unrolled BFS (`Pathfinding/BabyRatBFS.java` 4,960 lines and three variants), `updateMapInfo`
unrolled over the sensed-tile count, `FastBitMap` with a 7x7 `long` mask BFS, `FastLocSet` /
`FastIterableLocMap`; PM §6.1 accepts going over bytecode in crowded fights as "a very worthy
tradeoff". SPAARK: `updateExploredBabyRat0..7` generated per facing (most of the 6,268-line
`Comms.java`), `addCat` stamps a radius onto `long[]` with precomputed masks, `Clock.getBytecodesLeft()`
gates throughout, and a hard-coded first-turn path "to prevent bytecode running out on turn 1". food:
a `visionChanged` flag so robots re-sense only after a turn or move, optional squeaking/cat work
behind `getBytecodesLeft() >= 1000`, and "BYTECODE EXCEEDED LAST TURN" prints. *We* use ~4% of 17,500
(TRAINING_LOG "Measurement -- the bytecode explanation is FALSE"); the unused 15,000 bytecodes per
turn were never converted into anything.

**Modular strategy code.** GST PM §2: Strategy pattern with `StateUpdater` and `Behavior` singleton
interfaces, explicitly to escape "3000-line, non-codegen files"; food PM §Macro: a five-state machine
with an ordered priority list and a one-page rationale for having few states; SPAARK: mode integers
with paired `checkMode()` / `execute()` functions. *Ours* is one 1,531-line `RobotPlayer.java` whose
comments are the history of 250 iterations; `RESEARCH.md` §6 recommended a rewrite, never done.

### 2.8 Tooling, testing and process

**Parallel runners with seed variation, frozen versions and purpose-built maps.** GST `Tester.py`
(8-way `ThreadPoolExecutor`, 47 maps x both sides via `gradlew massTest`), 18 frozen packages
`Version24..Version41`. SPAARK `runmatches.py` (6 threads, "asap" mode disabling websocket,
indicators and replay saving, per-map Won/Lost table to `matches_summary.txt`), `copybot.py` to
freeze a version with its package renamed, and `changeseed.py` which rewrites the FlatBuffer
`randomSeed` of a `.map26` to get independent games on the same map; its `maps/` has 72 maps
including `[EDGE] microtest1-5` and `[SP] RushSimple` custom maps. food PM §Final Thoughts: build
"three to five maps to test a specific behavior", step through individual robot turns, "don't obsess
over win rate", and beware overfitting to an old version of yourself. GST PM §5.1/§7: scrim teams
above you and copy their macro ("much easier than reverse engineering micro"); "we spent far more
time reviewing scrims than actually writing code". *We* have `tools/gauntlet.sh`, `paired-check.sh`
and `replay-dump.sh` but a fixed 27-map set, no seed variation (both instruments are deterministic,
Iteration 193), no custom maps, and no live opponents (`RESEARCH.md` §2). UNderTheC's LLM-loop
write-up (different game) converged on the same discipline our log did -- one change per worker,
`beats_parent` with an explicit noise band ("38-40 or 41-37 is a tie"), analysts separated from
strategists -- and on the finding that rating rose linearly with tokens spent.

**Cheese taxi / courier.** GST PM §5.2: a rat with little cheese carries an ally holding >= 100 cheese
home (`BRScavengeRatPickup`, `ferrying`), dropping it within d^2 25 of the King; food `tryCheeseCourier`
(ally >= 80 cheese, adjacent, courier empty, no enemies or cats, > d^2 32 from the King). Both call
the impact small (carrying costs a 1.5x cooldown, "equivalent to carrying 50 cheese"). Never tried
by us; listed for completeness.

## 3. Ideas we tried but could not make work, that top teams made work

| ours (TRAINING_LOG) | verdict | what they did differently |
|---|---|---|
| Second / multiple Kings: 18, 50, 74, 75, 106, 107, 146, 173 | all REJECTED, direction closed | formation site is a *cheese mine* far from existing Kings, gated on standing-army size (GST) or `(k+2)*600` cheese plus known mines (food); rats self-initiate on a mine with 3+ allies; new King is the delivery point |
| Grab and throw: 17, 54, 71, 72, 143, 245; 246 "unfixable" | REJECTED | scored move+turn+throw enumeration with early exit (GST), ratnap from outside the victim's cone, HP management via cheese bites so `targetHealth < grabberHealth` holds, never throw toward own King |
| Squeaks: 109 (cheese sightings), 110 (dispersion), 166/237 (cat lure), 170 (danger) | REJECTED, "squeak steering CLOSED" | squeak *enemy* and *cat* positions with facing and HP every micro turn, typed header; suppress near cats and own King (SPAARK); lure cats *off the King*, not toward field traps |
| Cheese-mine memory: 232, 233 | REJECTED | per-mine cooldown / visit interval, ID-based assignment, threat level, mirrored twin, and a King that moves onto the mine |
| Batch deliveries: 115 | REJECTED | return threshold 20-40 scaled by distance to King, pickup cap 100-150, no detour for < 5 cheese |
| Bug-nav for collection: 36, 231; leash: 191, 225 | REJECTED | bug-nav with a per-tile state table and simulated rotation choice (food), or BFS+bug hybrid (GST), applied to every destination; no leash -- survival comes from micro, not from staying home |
| Retreat when hurt / outnumbered: 8, 9, 62, 104, 105, 149, 150, 188 | REJECTED | `inRetreat` / `bold` modes driven by ally counts pooled from squeaks and by raw cheese carried (GST); penalties scaled by local ratio (food, SPAARK) |
| Cat engagement gates on ally count: 63-67, 178, 179 | REJECTED / inert | the question is not how many allies but whether the tile is in the cat's cone or path; bite only with no enemies present |
| Face the unseen attacker: 123, 234 | REJECTED | defensive facing after every move, alternate L/R while walking, damage-signature ambush reaction (food), turn cadence (GST) |
| Trap the mines / field traps: 118, 190; speculative cat traps: 218 | REJECTED | rat traps on the enemy-facing side of a mine using the inferred symmetry (GST); cat traps in the cat's cone and *recycled* with `removeCatTrap` |
| Dirt walls: 171, 172 | REJECTED, "dirt direction closed" | dirt as a *tactical* block: on the cat's actionable tiles (GST rescue mode), two tiles toward a fleeing King's pursuer (SPAARK `RatKing.flee`), one-in-six toward a cat (food `evadeCat`) -- never a static wall |
| King relocation onto a mine: 126, 127 | REJECTED | continuous: King always heads for the nearest unclaimed mine and collects (food, GST, SPAARK) |
| Population raises: 28-31, 34, 37, 60, 111-114, 120, 125, 219, 228 | REJECTED (one accept, 147) | target the *live* count via `getCurrentRatCost()`; no build window |
| Deliberate late backstab: 221 | REJECTED | they are at war from the first contact; cooperation is a map property (walls that prevent contact, food PM Fig. 21), not a choice |

## 4. Ideas we had that top teams also had

- Rat traps around a threatened King (ours: static ring, Iteration 48/96/102; theirs: directional,
  toward the enemy -- food `evadeRats`, GST `RKFlee.attemptTrapInFrontOfEnemy`, SPAARK `RatKing.flee`).
- Reactive cat traps near the King when a cat approaches (ours Iteration 128; food `kingTryPlaceCatTrapInDirection`
  while cooperating; SPAARK `placeCatTrap` while fleeing).
- Cheese reserves and emergency thresholds (ours `RESERVE 150`, `8 * ratCost`; food `CHEESE_CRITICAL 150`,
  `CHEESE_EMERGENCY 600`; GST `BUILD_ANYTHING_THRESHOLD 80`).
- King builds before it fights, and bites the lowest-HP adjacent enemy (ours Iteration 99; food
  `kingFindBestTarget`; GST `attemptAttackAsRatKing`).
- King digs itself out when boxed in by dirt (ours `digTowardOpenSpace`; food `checkKingTrapped`;
  GST `attemptMineDirtAsRatKing`).
- Turn before stepping to avoid the strafe cooldown (ours Iteration 151 `stepTo`; food `executeMove`;
  food/GST strafe penalties in micro).
- Free bite on an adjacent cat when nothing else is going on (ours Iteration 5; food `doCatAttack`;
  GST `attemptAttackCat`).
- King flees cats (all), and the first King spawns toward the map centre (GST `addCenterBias`,
  food `myKingIndex == 0 -> directionTo(centerLocation)`, SPAARK `spawnTarget = mapCenter`; ours
  builds the nearest free tile -- same intent, not implemented).
- Per-robot RNG seeded from `rc.getID()`, no absolute-compass fallbacks (ours class comment; all three).
- Explore away from the spawn (ours ID-based heading; GST `exploreDirection = dirToFather.opposite()`).
- Bytecode-overrun detection by comparing round numbers (ours `reportBytecodeBudget`; food and
  SPAARK print on `getRoundNum() - prevRound != 1`).
- A ladder of frozen snapshots (ours `g_iter1..34`; GST `Version24..41`; SPAARK `TSPAARK*`).
- Cat waypoints are fixed and public (ours Iteration 197; GST PM §4.2.1 "keep Baby Rats from
  wandering into cat-visible tiles"; nobody read beyond that -- no bot predicts waypoints).

## 5. Sources

Post-mortems
- Generalized Stroke's Theorem (2nd): `Battlecode_2026_Postmortem.pdf` in https://github.com/uravt/Battlecode26 (same document as the blocked https://battlecode.org/assets/files/postmortem-2026-generalized-strokes-theorem.pdf, inferred from title/date).
- food (Top 12): `Battlecode_Postmortem_2026.pdf` in https://github.com/AlexT101/battlecode26.
- Index of 2026 post-mortems (blocked host, listing confirmed via WebSearch): https://battlecode.org/past.html -- Lorem Ipsum, 3MiceWalkIntoABar, Sloppy Joe Pirates NOT read.
- UNSW UnderTheC (different game): https://github.com/Battlecode-UnderTheC/battlecode-underthec.github.io, `_posts/2026-10-11-*.md`.

Code
- https://github.com/uravt/Battlecode26 -- `src/Version41/` (`RatKing.java`, `BabyRat.java`, `SymmetryManager.java`, `Reachability.java`, `Utilities.java`, `BabyRatStateUpdaters/DefectBRStateUpdater.java`, `RatKingBehaviors/{RKHeadquarter,RKFlee,RKExplore}.java`, `BabyRatBehaviors/{BRMicro,BRCatMicro,BRScavengeRatPickup,BRExplore,BRFormRatKing,BRThrownCarried}.java`, `Communication/Communicator.java`, `Pathfinding/Pathfinding.java`), `Tester.py`.
- https://github.com/AlexT101/battlecode26 -- `src/finalsbot/` (`RobotPlayer.java`, `Globals.java`, `Nav.java`, `Map.java`, `Helpers.java`, `Comms.java`).
- https://github.com/erikji/battlecode26 -- `src/SPAARK/` (`RobotPlayer.java`, `G.java`, `RatKing.java`, `RatKingMicro.java`, `Sustain.java`, `BabyRat.java`, `BabyRatMicro.java`, `Motion.java`, `Comms.java`), `runmatches.py`, `copybot.py`, `changeseed.py`, `maps/`.

Ours (https://github.com/terryvanbelle/battlecode26-vibe)
- `src/bot/RobotPlayer.java` (g_iter34), `BENCHMARK.md`, `RULES.md`, `RESEARCH.md`, `TRAINING_ALGORITHM.md` ("Closed directions"), `TRAINING_LOG.md` (Iterations cited inline; last entries 238-252), `tools/`.
