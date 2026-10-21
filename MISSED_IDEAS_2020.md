# MISSED_IDEAS_2020.md -- what the top 2020 teams did that we never figured out

**Year / game:** Battlecode 2020, "Soup" (miners, landscapers, delivery drones, design schools, fulfillment
centers, net guns, vaporators, refineries, HQ; the map floods on an exponential water curve). Written 2026-10-21,
after the project's ban on 2020 post-mortems and other teams' source was lifted.

## Sources read

| source | how read | notes |
|---|---|---|
| Java Best Waifu (1st) post-mortem, Ivan Geffner, 10 pp. | **in full** (`IvanGeffner/battlecode2020/BC2020_Postmortem.pdf`, pdftotext) | the official battlecode.org copy is network-blocked; the repo copy is identical in title/author |
| The High Ground (4th) post-mortem, Lifland/Ho/Hoganson, 12 pp. | **in full** (`uvafan/bc20/Postmortem.pdf`) | same document as `postmortem-2020-the-high-ground.pdf` |
| Bowl of Chowder (5th-6th) post-mortem, Stone Tao | **in full** (`StoneT2000/stonet2000.github.io`, branch `old`, `battlecode/2020/index.html`) | the live page is blocked; the built HTML is in the old branch of the blog repo |
| smite (2nd) README "Overview / Strategy / Communication / Resource Collection / Bytecode" | **in full** (`mvpatel2000/Battlecode2020/README.md`) | a short post-mortem in its own right |
| Battlegaode (3rd) post-mortem (web.mit.edu/agrebe) | **search snippets only** (WebSearch, 4 queries) | blocked host; the navigation section is quoted second-hand |
| confused (HS 2nd) post-mortem | **search snippets only** (2 queries) | thin: only the vaporator nerf and "rush had no easy defence" survived |
| winkelmantanner (the-levee-builders, finalist 5th-6th) README | in full | 6 bullet points, two of them quantitative |

Code read (final/submitted package; `wc -l` of the package):

| team | repo / package | lines | what was read |
|---|---|---|---|
| Java Best Waifu (1st) | `IvanGeffner/battlecode2020/finalbota` (= our ladder bot `IvanGeffner.finalbota`, 1958, 6th on our ladder; the identical code is `battlecode20-team4.finalbota` and `ronniesong0809.finalbota`, 3rd and 5th on our ladder) | 5,462 | HQ, HQWall, BuildingZone, BuildingManager, Comm, Constants, Landscaper, Miner, Drone, Danger, DangerDrone, NetGunManager, RushManager, BugPath, WaterManager, Design, Fullfillment in full; ExploreMiner/ExploreDrone/ExploreLandscaper/DroneBugPath skimmed |
| smite (2nd) | `mvpatel2000/Battlecode2020/src/qual` (= `mvpatel2000.qual`, 1879, 9th on our ladder) | 12,559 | HQ in full; Landscaper (construct, terraform, defense, wall phases, fill order), DeliveryDrone (constants, shell, poke, A-move, corner holder), Message, Robot comms header, Miner vaporator placement; the rest skimmed |
| The High Ground (4th) | `uvafan/bc20/src/v14_final_bot` (= `uvafan.v14_final_bot`, 2201, **1st on our ladder, 0-16 against us**) | 7,163 | MagicConstants, Comms, Strategy, EcoLattice, Rush, Turtle, HQ, Landscaper, DeliveryDrone, Miner, Nav, Unit, DesignSchool, FulfillmentCenter in full; Utils (2,583 lines of tables) skimmed |
| citricsky (awesomelemonade; 2051, 2nd on our ladder) | `awesomelemonade/Battlecode2020/citricsky` | 3,989 | LandscaperBot, HQBot, DroneBot, BuildOrder, LatticeUtil, Communication, Hash, CommunicationAttacks, EnemyHQGuesser, Pathfinding in full; MinerBot head |
| Bowl of Chowder (5th-6th) | `StoneT2000/Battlecode2020/src/FinalChowBot.zip` -> `Chow10` | 5,704 | Constants in full; HQ, Landscaper, DeliveryDrone skimmed around the island / swarm logic |
| Kryptonite (7th-8th) | `rzhan11/Battlecode2020/src/quals_bot` (= `rzhan11.quals_bot`, 1806, 10th on our ladder) | 6,758 | BotMinerRush, BotLandscaperRush (head), BotHQ (head), Communication (head), Wall, Constants, Globals (constants) |
| the-levee-builders | `winkelmantanner/battlecode2020_the-levee-builders` (= `winkelmantanner.tannerplayer`, 1890, 8th) | 3,258 | README; Landscaper/DeliveryDrone/Miner skimmed for the pit, levee and vaporator rules |
| AngusRitossa `newbot` (1951, 7th on our ladder) | `AngusRitossa/battlecode20/src/newbot` | 2,970 | RobotPlayer constants, Landscaper turtle logic, DeliveryDrone gap/rescue logic skimmed |

Not reached: Battlegaode's code (no GitHub repo found; the MIT page is blocked), Bagger288's and confused's code
(no repos found by name), Bruteforcer / Super Cow Powers (the original lattice bots; ineligible after Sprint, no repos
found), NP-ez, Steam Locomotive, Prasici.

## Where our final bot stands

`src/bot` = `src/g_iter19`, 1,800 lines in 15 files (`DESIGN.md`). `progress/ELO.md` (2026-09-30 refit, 15,383
scrimmages): **1752 +- 29, 11th-12th of the 65 external ladder bots** (rank 11 of 121 players counting our own
builds), field score 81.4%, 23.9% against the ten bots above us. Those ten, in order: uvafan.v14_final_bot (THG) 2201
(we are 0-16 against it), awesomelemonade.citricsky 2051, ronniesong0809.finalbota 1993, StoneT2000.FinalChowBotStable
1979, battlecode20-team4.finalbota 1970, IvanGeffner.finalbota 1958 (JBW), AngusRitossa.newbot 1951,
winkelmantanner.tannerplayer 1890, mvpatel2000.qual 1879 (smite), rzhan11.quals_bot 1806 (Kryptonite). `BENCHMARK.md`
roster: 285 packages from 68 repositories. Our architecture (DESIGN.md "The incumbent"): an eight-seat wall on the
HQ's own ring from r1 with eight Chebyshev-2 helpers, a poortho-style school rush by the second miner, 4 early miners
(max 8), one center from r400 with four "quarry" drones that hold flooded Chebyshev-2 tiles, a capped C3 drone shell
from r1000, a raid from r1400, zero net guns (`C.NETGUNS_MAX = 0`), and in practice 0-1 vaporators a game. The binding
weakness, measured (HANDOFF "Next lead", TRAINING_LOG 2026-09-30): the economy freezes when the base's ground floods
(r680-710 at elevation 3): **median 0 miners, 0 schools, 0 centers at r1000**, 550-1,100 soup unspent (up to 12,000
by r3000), landscapers 9/6/5 at r1500/2000/2500 against the field's 30/30/20.

---

## 1. Ideas we never figured out

### 1a. Macro strategy and win condition

**The lattice is the base, not an add-on: terraform first, wall late, and put EVERY building on raised parity tiles.**
What they did: JBW classifies every tile (BuildingZone.java) into INTERIOR / NEXT_TO_WALL / WALL / OUTER_WALL /
HOLE. Outside the 7x7 wall, a tile is OUTER_WALL iff `(x-hqx)%2==1 || (y-hqy)%2==1` and a HOLE iff both offsets are
even; buildings go only where both offsets are odd (`BuildingZone.canBuild`). Wall and outer-wall tiles are raised to
`WALL_HEIGHT = 8` (Constants.java), holes are dug to `MIN_DEPTH = -20`. THG copies it exactly (`isOnLatticeIntersection`,
pits where `loc.x%2==hqLoc.x%2 && loc.y%2==hqLoc.y%2`, `LATTICE_HEIGHT = 8`); citricsky's `LatticeUtil.isPit` /
`isBuildLocation` is the same parity rule; smite uses a 3-period lattice (`dxy[0]%3 + dxy[1]%3 == 0` are dig sites,
`Landscaper.getTerraformDigDirection`, `LATTICE_SIZE = 48` r2). Every team with a lattice builds its vaporators,
schools and centers on it from the start, not after some count is reached. Evidence: JBW PM sec. 3.2; THG PM sec. 6;
Chowder PM 2.2, 2.6.2; smite README "Qualifying Tournament"; code cited. What we did: DESIGN.md "R1 -- lattice first"
tried it (grid = even-dx or even-dy, sites = odd dx on even rows) but kept g_iter12's eight ring seats racing from
round 1 and let only landscapers past the 16th work the lattice (Landscaper.java R1 stage 11); the builder may leave
the lattice for the refinery and first school (`mayFallBack`), and in the field we got 0.5-0.9 vaporators a game
(C.java comment at ANY_BUILD_FROM). Why it mattered: their buildings outlive the r677 flood and production never
stops; ours all drown at r680-710 and nothing is built after (HANDOFF). The lattice IS the post-flood economy we kept
looking for in a pocket, a plateau, a perch, a keep and an enclosure.

**A 7x7 wall around the HQ with an open interior for buildings, chosen by BFS, not a 3x3 ring.** JBW's HQ spends its
spare bytecode in the first turns running a BFS (`HQWall.run`) over the 7x7 block (Chebyshev 3), excluding any tile
with elevation below -20, and broadcasts a 145-bit mask (`posToBit`, 5 ints) of which tiles are interior; the 24
tiles just outside the interior become WALL. The interior (Chebyshev <= 2) holds the school, center and 3-6
vaporators. THG hard-codes the same 24 offsets (`WALL_X_OFFSETS/WALL_Y_OFFSETS` at |d|=3) and the HQ announces
`WALL_COMPLETE` when all 24 reach LATTICE_HEIGHT 8. The wall is only ever raised to 8 at first; height comes later
(JBW "fortify after round 1400", sec. 3.6). What we did: our wall is the HQ's own 8 neighbours, raised to 1,600-2,600 by
r3000 (LEARNINGS "A wall grows at 0.5 elevation per landscaper per round"); our Iteration 6 citadel tried Chebyshev 2
(16 tiles) and was refuted because "16 tiles halve the per-tile rate" -- which is true for a wall race, and
irrelevant when the point of the wall is to keep an economy alive, not to out-climb the water. Why it mattered: the
HQ keeps its spawn tiles (LEARNINGS "The HQ stops spawning the moment the ring is seated"), the school keeps its doors,
and THG's drones have dry tiles at height 8 to drop stray units on (`getDropOffLoc`).

**Drones first; the early drone decides the rush and the harass decides the economy.** JBW's build order
(BuildingManager.getNextBuilding): fulfillment center and ONE drone before the first design school and before any
vaporator (`MINERS_BEFORE_FULFILLMENT = 5`); then one landscaper and one drone per vaporator until r1200
(`shouldBuildDrone/Landscaper` alternate on `drones <= landscapers`), then one landscaper per two drones. THG copies:
EcoLattice "5 miners, fc, miner then vap, ds", `LATTICE_ARMY_COMP = {0 NG, 1 FC, 1 DS, 20 drones, 20 vaps, 20
landscapers}`. citricsky: 2 landscapers, 3 vaporators, 2 more landscapers, 3 drones, vaporators to 15, then alternate
(BuildOrder.java). JBW PM 3.2: "this early drone stops incoming rush miners ... rush bots have a hard time". Chowder PM
2.6.1: "drones first was way better and easier" for rush defence; the harass "demolished my economy and lattice by
rendering my miners and landscapers useless". What we did: the center comes after the school and the first vaporator
or at r400 (`C.QFC_FROM`, `FC_EARLY_BANK` gated at 52%); our rush defence is five landscaper-based tries, 0 of 5
(TRAINING_LOG "Copying tactics"); the home-defence drones of Iteration 23 were "refuted as built" because they were
"helpers not built". Why it mattered: a 150-soup drone parked at home beats a 150-soup school rush (JBW 3.4: "if the
opponent builds a Net Gun to counter it, they spend 100 more than us"), and the same drone is the economic weapon that
breaks the field's economies (we lost 0-16 to THG mostly before our wall mattered).

**The harass: fly to the enemy HQ early, pick up anything, drop it in water; later half the drones go to the enemy's
soup.** THG DeliveryDrone objectives: `HARASS_ENEMY_HQ` from birth; from the 9th turn every second drone
(`HQ_SOUP_HARASS_RATIO = 2`) switches to `HARASS_SOUP_LOCATIONS` until r700, targeting the symmetry image of soup
clusters miners reported; `findAndPickUpEnemyUnit` takes the nearest non-drone, non-building; `dropUnitInWater` uses a
list of up to 30 broadcast water locations. citricsky's `getPriority`: on the enemy's half, miners before landscapers;
on ours, landscapers first. JBW's Drone: `getTarget` -> closest enemy landscaper, then miner, then enemy HQ guess.
What we did: Drone.java patrols a point between home and the enemy HQ guess and lifts what it meets; our drones
exist from r400-600 at the earliest and sit on quarry/shield slots. Why it mattered: every post-mortem (JBW 3.2, THG
6, Chowder 2.6.1) names the harass as the decisive mid-game lever; THG: "every top team, no matter what their
strategy, now had a drone harass".

**The crunch: a timed, massed drone+landscaper assault at ~r1500 with net guns ignored at the end.** JBW: after
`MIN_TURN_PREPARE_CLUTCH = 1400` every drone grabs a friendly landscaper and paths to the enemy HQ; after
`MIN_TURN_CLUTCH = 1550` the net-gun danger map is ignored (DroneBugPath) and landscapers are dropped adjacent to the
closest enemy building (`tryDropNextToBuilding`); a second attack after r2200 (PM 3.6). THG: `PICK_UP_LANDSCAPER_ROUND
1175`, `DONT_PICK_UP 1460`, `CRUNCH_ROUND 1500`; drones born in that window alternate landscaper/miner cargo
(`round % LANDSCAPER_MINER_RATIO(7)`), miners are dropped within r2 13 of the enemy HQ to build net guns
(`CRUNCH_ENEMY_HQ_DROP_MINER`). citricsky: the HQ raises `ATTACK_STATE_ENEMYHQ_IGNORE_NETGUNS` when 30 drones report
ready (or 15 after 250 turns), for 70 turns, and alternates drone-only and drone+landscaper waves. smite: `POKE_TURN
900` (a 50-round probe), `ATTACK_TURN 1875` with 200 rounds of "posture" (spiral at distance, then charge), a
`cornerHolder` parked at (3,3) from the enemy HQ -- r2 18, just outside the gun. What we did: Iteration 61/65/69 raids
(rally 5 out, charge at 20 drones, lift seats) -- "null on the ladder" because "our drones lift what the enemy leaves
in reach and the field's walls are shielded"; cand70's "drop our landscapers beside the bare HQ" never fired. The
difference is not the tactic but the count (THG 20 drones/20 landscapers from vaporator income; EmaPajic 136 drones)
and the "ignore net guns" phase that trades drones for the kill.

**Rush defence as a production rule, not a position rule.** JBW RushManager.rushBuild: under a rush (any enemy
miner/landscaper/school/net gun in the HQ's sight before r300, or on a "critical" tile later), build a center first;
if there is no enemy net gun, spawn one drone per enemy landscaper+miner in sight (`freeDrones <
theirLandscapers + theirMiners`); if a net gun is up (so drones die), spawn landscapers to `theirLandscapers + 1`;
drones only if the enemy spawned/built within the last 5 rounds (`turnEnemyBuilt`, so they cannot afford a gun);
`emergencyLandscaper` when the HQ carries > 25 dirt and a free tile adjacent to both the school and the HQ exists. THG:
`HQ_ATTACKED` / `HQ_OK` messages, `RUSH_DEFENSE_ARMY_COMP = {1 drone, 1 landscaper}`, return to the normal build order
at r400 regardless (the Kryptonite stand-off fix, PM sec. 8). citricsky: HQ state NO_HELP / NO_ADDITIONAL_HELP (4+
landscapers adjacent or all neighbours occupied) / NEEDS_HELP, broadcast when it changes. What we did: `rushSeen()`
(school or landscaper within r2 64 before r400) makes the builder put the school first and freezes miners; the center
"right after the school" (Iteration 54) "never found a site". Why it mattered: our fast-loss class (poortho 11 of
g_iter12's 129 losses; rzhan11 20 of 42 g_iter15 games before r210) stayed open all project.

**Self-destruct as a tactic.** THG: miners standing on the wall at CRUNCH_ROUND disintegrate; anyone within r2 8 of
the HQ after r1640 (when height 8 floods) disintegrates (`rc.disintegrate()` in Miner.takeTurn and
DeliveryDrone.takeTurn); landscapers within r2 25 of the enemy HQ disintegrate the round before the crunch
(`suicideOnCrunch`) to clear drop tiles. Chowder's drone-wall buster: drones drop landscapers holding 20+ dirt INTO
water 6 tiles from the enemy HQ -- a drowned landscaper leaves its dirt, so repeated drops build an island; miners are
then dropped there to build net guns that shoot the drone wall (PM A.3, Chow10 HQ.java `buildIsland`,
DeliveryDrone.java "check center, if center works, then disintegrate"). Bagger288 (per THG PM sec. 9): wall landscapers
self-destruct and are replaced by net guns when a crunch is imminent. What we did: nothing of the kind; the engine's
`disintegrate()` is listed in RULES.md and never used.

### 1b. Economy

**Vaporators as THE economy: 20-45 of them, on the lattice, within a round window.** JBW builds vaporators from
`MIN_TURN_BUILD_VAPORATORS = 500` to `MAX_TURN_BUILD_VAPORATORS = 1150` (250 rounds pays back before the lattice
floods) and sizes everything else to them: `10*fulfillment <= vaporators` -> another center, `10*design <= vaporators`
-> another school, miners = vaporators (`getMinerNumber`, 10 + 2/3 beyond ten), one landscaper and one drone per
vaporator. THG stops vaporators `TURNS_FOR_VAP_TO_PAY = 250` before `CRUNCH_ROUND`. citricsky: 3 vaporators after
the second landscaper, 15 before mass production, up to 120 while soup < 800. smite README: "most finalists built at
least 10 vaporators, and some more than 20"; its miners build one whenever soup > 500 + round/100 within r2 20 of the
HQ until r1700. winkelmantanner: "exponential vaporator growth on about 80% of maps" once miners went 3 -> 7, and
"vaporators must only be built on high ground" (elevation > water level 500 rounds ahead, Miner.java). Measured on
our ladder (TRAINING_LOG): team4 44 vaporators at r1000, winkelmantanner 45 at r1000 and 72 at r1500, uvafan 43. What
we did: `VAPORATOR_BANK 500`, `VAPORATORS_MAX 40` on paper; 0.1-0.4 a game (DESIGN.md "Why"); Iteration 72/84
(hold 500 for vaporators) rejected 3-20 and 2-8 because "the mirror's wall race punishes any soup taken before the
school drowns". Why it mattered: our mirror gate was measuring the wrong game -- against a wall-first incumbent a
vaporator is a loss; against the field it is the only thing that pays after r700.

**Miner count scales with soup seen and with vaporators; miners are retired, not capped at 8.** JBW
`shouldBuildMiner`: 4 at once, then `miners <= (maxSoup - 2*miners*round)/300` (`SOUP_PER_MINER = 300`, maxSoup from
the HQ's own scan plus the largest soup message), stop once the wall is finished or 2 landscapers exist, max 15; a
"builder" miner is spawned when the next building is affordable (`BUILDER_SOUP`) and told so by a BUILDER message.
THG: 6 non-build miners + 1 build miner; after the wall is complete one more every 150 rounds and at least
landscapers/7 (`LANDSCAPER_MINER_RATIO`). smite: 4-5 miners plus 3 "terraformer" miners at r150/210/270. What we did:
`MINERS_EARLY 4, MINERS_MAX 8, MINERS_TOTAL 16` fixed; Iteration 17's 8-24 miners was "blocked on the sink" because
there was nowhere to spend the soup.

**Refineries are built by soup amount and distance, not one by the HQ.** JBW: a refinery when > 1000 soup is visible
and the nearest refinery is > r2 50 away (Miner.tryBuildRefinery). THG: a soup-priority formula from
`distToClosestRefine * 120 + soupNearby * 5`, capped, with a 1000 penalty when the only drop-off is the HQ. What we
did: one refinery at Chebyshev 2 from the HQ, a second only when the walk home stalls (Miner.java 34b).

**Hold-production messages.** smite: a miner that needs a net gun posts HOLD_PRODUCTION; every producer stops for up
to 30 rounds or until 200 soup is spent (`checkIfContinueHold`). citricsky: `isSavingForNetgun` is the first branch of
`BuildOrder.getNextRobotType`. Kryptonite: the rush miner's soup is "stacked" (THG PM 6: "save up a ton of soup as they
rushed, then as soon as they arrived spit out a ton of units"). What we did: fixed per-role banks (`HELPER_BANK`,
`ATTACKER_BANK`, `QUARRY_BANK`) with no way for one unit to reserve soup for another; LEARNINGS "A reserve nobody
spends is bodies nobody has" was the failure of the fixed-reserve design.

### 1c. Combat micro

**Net guns, placed by rule, as part of the economy.** Our `NETGUNS_MAX = 0`; every top bot builds them. JBW
(Miner.tryBuildNetGun): when a drone has been seen and a friendly landscaper is in sight, on a NEXT_TO_WALL tile or a
wall tile at height 8, at least r2 8 from the nearest gun (r2 5 near the enemy HQ), and never while `vaporators <
2*netguns`; a drone's `Danger.minDist` makes the gun go toward the drone. THG (Unit.buildNetGunIfShould): on sight of
drones, on the four "inside corners" of the 7x7 or on a lattice tile at height 8, >= r2 8 apart (`MIN_NET_GUN_CLOSENESS`),
>= r2 8 from the HQ, `numNGs <= numVaps`, excited (closer spacing, any tile) when 3+ drones/buildings or near the enemy
HQ; costs more soup the farther from home (`DIST_SOUP_MULTIPLIER`). smite README: "attackers needed a drone : net gun
ratio of 10:1". JBW PM 3.4: net guns also "destroy the enemy Drones protecting their HQ from our crunch". What we did:
Iteration 4 (one gun at a bank of 350) rejected 24-40; Iteration 16/31/37/47/79 guns drowned with the base. The gun was
rejected because it had no dry ground to stand on -- the lattice again.

**Rank the drone you shoot.** citricsky HQ: priority 2 for a carrier when all our neighbours are occupied, else for
an empty drone (`getPriority`). We have a four-rank version (Robot.shootDrone, R1 stage 17) -- one of the few places
we match them. JBW shoots the nearest.

**Drones kite net guns by cooldown and treat unseen guns as terrain.** JBW DangerDrone: a per-drone `dangerMap[w][h]`
gets +1 on every tile within r2 15 of a reported gun (via a 45-step direction path, `dirPath`, resumable across turns)
and -1 when a gun is reported destroyed; drones treat `> 0` as impassable until the clutch. Visible guns are handled by
cooldown: `getMaxDist(cd)` lets a drone step to r2 13 of a gun with cooldown <= 2, r2 5 at cooldown 3-4, r2 1 at 5-6,
and never at 7+ -- i.e. drones dash past a gun that has just fired. THG `SafetyPolicyAvoidAllUnits`: `acceptableDist =
15 - (cooldown-3)*2`. citricsky: drones never move diagonally while avoiding guns (a diagonal step covers r2 2 and
can land at r2 13 unseen; cf. JBW PM footnote 5 "drones have blind spots"; Chowder `DroneBlindSpots = {4,3},{3,4},...`
-- the eight tiles at r2 25 from which a gun at r2 15 is one step away and invisible). What we did: `fleeFrom(gun)`
when a gun is within r2 24 and "approach onto tiles with r2 > 15 only" (LEARNINGS); no memory of guns out of sight, no
cooldown reading.

**Walkers avoid drones by distance field, not by fleeing.** JBW BugPath.checkArrayWithDrones: for each of the 9
move options compute the min r2 to any enemy drone (`Danger.minDist`), mark tiles within r2 5 of our HQ or a net gun as
safe, and forbid any move whose min distance is below the best available (capped at `MIN_DIST_FLEE = 25`; 13 while
attacking). THG `fleeIfShould`: score = 2 * Manhattan to nearest drone - Manhattan to HQ; never flee when a gun or the
HQ is within r2 8. citricsky `Util.tryKiteFromAdjacentDrones` is the first behaviour of every landscaper and miner.
What we did: `fleeFrom(nearestEnemy)` when a drone is within r2 8 (Miner.java), nothing for landscapers.

**Drop enemies in pits when water is far.** winkelmantanner: `pit_max_elevation = HQ elevation - 10`; a lifted enemy
is dropped on any tile at or below it (README; DeliveryDrone.java `is_valid_enemy_drop_loc`) -- a unit at -10 beside a
ring at 50 is as dead as a drowned one. What we did: drones carry to the nearest flooded tile (Drone.java), sometimes
across the map.

**Drone wall and shell drones.** smite drones become `shellDrone` after `DEFEND_TURN 1100` and hold tiles around the
HQ at Chebyshev 1-2 (`inShell`), `SHRINK_SHELL_ROUND 2605`; Chowder drones ring the X wall (8 ring + 3 per corner,
PM 2.4); AngusRitossa drones `tryMoveIntoTurtleGap` after r800. THG PM 6: "a full drone wall ... was impossible to
crunch on as drones could not move through other drones". What we did: Iteration 81's shield on FLOODED Chebyshev-2
tiles only, then the capped C3 shell (8 drones); "absent where needed -- in 14 of 15 raid-window losses no center or
drone existed". Same idea, no economy behind it.

**Landscapers flee the flood by table, 7 rounds ahead; and fix flooded interior tiles before anything.** JBW
WaterManager keeps the round-at-which-elevation-k-floods table (`wHeight`, 51 entries) and three water levels (now, +7
rounds, +60 rounds); BugPath marks a tile "flooded" if its elevation <= level(+7) AND it touches water, and a unit
whose own tile is at risk flees to `closestSafeCell` for at least 7 rounds. The Landscaper priority list (PM 3.2, code
`AdjacentSpot.computeScore`): FLOODED_INTERIOR 0 < FLOODED_NEXT_TO_WALL 1 < FLOODED_WALL 2 < FLOODED_OUTER_WALL 3 <
WALL_LOW 5 < BUILDING_AREA 6 < NEXT_TO_WALL 7 < HOLE 8 < WALL_SUPER_HIGH 9 < WALL_HIGH 10; deposit on the lowest-scored
adjacent tile if its score <= WALL_LOW, dig from the highest-scored. A tile counts as "flooding" if it is flooded or
touches water and is below level + 2. THG: `needsUrgentFixing` (floods within 50 rounds), `urgentTile`, and
`getRoundFlooded` precomputed by `precomputation/round_flooded.py`. What we did: `waterLevel(r)` evaluated with
`Math.exp` and `Math.sin` at every call, `floodDanger()` 10 rounds ahead with `climb()` to the highest neighbour; no
notion of repairing a breach from inside. JBW PM 3.2: the priority list is what "could complete the inner wall even in
maps like Hills".

### 1d. Navigation

**Bug navigation with a state set to break loops, soft resets, and the obstacle remembered.** JBW BugPath: when
greedy fails, pick the rotation side by which side gets closer (`checkRotate`), follow the obstacle (`lastObstacleFound`),
reset when closer than ever, reverse on the map edge, and hash `(myLoc, lastObstacle, rotation)` into a `HashSet`
to detect a repeated state and reset. `moveTo` soft-resets when the target moves by < 3. THG Nav is the classic
Battlecode bug (wall side, rotation count, `canEndBug` when rotation count returns to 0 and distance <= start). What we
did: greedy Chebyshev with a 6-tile recent-list and a per-robot handedness bug after 2-4 stuck turns (Nav.java, 89
lines); LEARNINGS records miners "circling an unreachable plateau" and stall rules in every role.

**Navigation as a safety policy object.** THG `Nav.goTo(dest, NavSafetyPolicy)`: `SafetyPolicyAvoidAllUnits`
(flooding-soon tiles, net-gun ranges with cooldown, drones adjacent), `SafetyPolicyAvoidAllUnitsAndLattice` (never
step off the lattice, never into a pit, never inside the HQ square), `SafetyPolicyCrunch` (only pollution > 8000 is
forbidden). citricsky `Pathfinding.naiveMove`: landscapers that cannot step (|dElev| > 3) terraform the step instead
("if toDifference > currentDifference deposit/dig from to, then current"), miners never step below water level + 1
after r180, nobody steps next to an enemy drone. What we did: `safeTile` (flooded check) and `avoidRing` flags on the
robot; no per-call policy.

**Battlegaode's navigation (snippets only).** Non-drones change elevation by at most 3, "so certain tiles are
accessible from some directions but not others" and a straight path "may not be optimal or even permissible";
inspecting every visible tile cost ~100 bytecode each, so they limited themselves to a 5x5 grid around the robot, and
"unrolling the loops and storing everything as individual variables rather than arrays cut the bytecode cost by more
than a factor of two" (search snippets of web.mit.edu/agrebe/www/battlecode/20). (inferred) their landscapers dig
through walls and water on the way to the enemy HQ ("landscapers can dig their way through high walls and flooded
tiles", snippet). What we did: nothing dug a path; our rusher gave up when blocked (`RUSH_GIVEUP`), and the rush census
found 47 of 104 games where it never planted (TRAINING_LOG).

**Drones ferry stuck friends onto the wall.** JBW ExploreDrone.stuckAlly + Drone.shouldGrabLandscaper: a landscaper
or miner not on the wall is lifted and dropped on an unfinished wall tile first, else a finished one at height 8, never
within r2 9 of an enemy drone (`MIN_DIST_SAFE`); JBW PM 3.2 "we also needed them to put our stray Miners and
Landscapers on top of the Wall". THG `HELP_FRIEND_UP`: friends within r2 9 of the HQ, dropped on a free wall tile at
height 8-50 (`getDropOffLoc`). smite `START_FERRY = 300`, drones "fill the wall" from `FILL_WALL_ROUND 530` and the
outer ring from `FILL_OUTER_ROUND 1000`. What we did: the citadel ferry (one drone lifting from F to the ring) and the
quarry drone's `trappedOurs` rescue -- both for our own geometry's traps, neither general.

### 1e. Communication

**Encrypt with a per-round key and bid above the enemy.** JBW: word 6 = `round ^ MASK` with `MASK = 4534653 +
team`; `getBidValue()` reads last round's block and, if it was full, bids one more than the highest non-ours cost.
THG: word 0 = `FAST_SECRET_NUM + team`, word 1 = a smeared xor of words 2-6, the round and `SLOW_SECRET_NUM`; "we made
sure to change our communication encryption constants right before we submitted" (PM 9) -- Prasici replayed Steam
Locomotive's seeding-bot messages in the finals and sent its landscapers to the map edge. smite: a 16-bit header
`floorMod(64556*(team+1)*H*W*round, 2^16-1)` plus a 3-bit schema, bit-packed payloads (`Message.writeToArray`).
Kryptonite: XOR of all seven ints with fixed keys plus an id/confirm word, dynamic cost = last block's 7th cost + 1, an
unsent-message queue, and important messages re-posted every 100 rounds (`REVIEW_SIGNAL`). citricsky: a murmur-hash
signature of a random-salted footer and a 262,144-bit Bloom filter to reject REPLAYED messages, and
`CommunicationAttacks.java` -- bit-flip and replay-DOS attacks on the enemy's own messages. What we did: `Comms.auth`
(a salted hash of words 0-5, the round and the team) in word 6, fee always 1, no replay defence, no bidding. Our auth
would have resisted Prasici's replay (round-salted) but not an adaptive bidder.

**Report every net gun (and its destruction), water, soup clusters, unit counts.** JBW Comm message types: ENEMY_HQ,
SOUP (max soup seen + location), BUILDER, BUILDING (type built -> every unit keeps `buildings[]` counts), WALL (the
145-bit mask), EMERGENCY, ENEMY_UNIT, WATER, GUN, GUN_DESTROYED, three symmetry-elimination messages, WALL_FINISHED,
RUSH/RUSH_END. THG: 16 types including UNIT_CREATED/UNIT_DEATH (so every bot knows the team composition the
`Strategy` ratios need) and NET_GUN_REMOVED broadcast by the landscaper that lands the 15th dirt. smite: the HQ
re-broadcasts the top soup "patches" (3x3 zones) every few rounds and the enemy HQ every 100. What we did: 7 types,
none for guns or counts; the school names lattice workers by id (`LATTICE_ORDER`), the center posts `CENTER_UP`.
LEARNINGS found the hard way that "a message posted once is seen by a third of the units".

**Read the whole chain on birth, within a bytecode budget.** JBW `Comm.readMessages` loops `while (turn < round &&
bytecodesLeft >= 300)` from round 1, so a newborn catches up over its first turns; THG `readMessages(upUntil)` with
`Clock.getBytecodesLeft() > 1000`; smite landscapers read a 6-round window for their birth message; Kryptonite
`readOldBlocks` with `READ_TRANSACTION_MIN_BYTECODE`. What we did: each role reads one block on one residue mod 3 and
newborns `readBack` 12-20 blocks; "newborns born inside a sealed area know nothing" (LEARNINGS).

### 1f. Symmetry and exploration

**Eliminate symmetries by sight and share the elimination.** JBW: `getEnemyHQLoc` checks each of the three images;
a visible image with no enemy HQ sends HORIZONTAL/VERTICAL/ROTATIONAL_SYMMETRY (= "not this one"); two eliminations
give the answer. citricsky `EnemyHQGuesser`: a 2-bit shared "mode" (which guess is known false), deduce the third.
Kryptonite's rush miner walks to the closest image first and switches when `isSymmetryHQLoc[i] == 2`. THG's rush
miner "would first run to the middle of the map in an attempt to determine which of 3 possible symmetries the map
was" (PM 4). What we did: MapState keeps the same three hypotheses and prunes on sight AND on remembered terrain
(`observe`) -- a stronger local test than theirs -- but the pruning is per robot; only a sighted HQ is shared.

**Precompute the visible-tile walk.** JBW PM 3.1 ("saved more than 2000 bytecodes per turn"): instead of X[]/Y[]
offset arrays, hard-code the sequence of Directions a unit follows to pass through every visible tile
(`ExploreMiner.dirPath[sensorRadius]`), since `loc.add(dir)` is 2-3 bytecodes against 15+ for `new MapLocation`. THG PM
6 names this trick as the reason JBW's landscaper scoring was more robust than theirs. Chowder generated BFS-ordered
delta tables (`Constants.BFSDeltas24/35`, PM A.1) so the first hit is the nearest. smite's `visible`/`SPIRAL_ORDER`
tables likewise. What we did: `observeTerrain` over Chebyshev 1 only, `rememberSoup` strided over `senseNearbySoup()`,
and double loops `for dx for dy` with `new MapLocation` everywhere (Landscaper.lattice, Drone.shellScan,
Miner.latticeBuild).

### 1g. Bytecode and engineering

**Build-order as a priority function, not an if-chain.** THG `Strategy.updateBasedOnDesiredComp`: for a desired
composition {type: count}, the type with the lowest `have/want` ratio gets `soupPriority = cost + 1`, every other type
`bestType.cost + type.cost + (ratio - bestRatio) * 50`; a unit builds when `teamSoup >= soupPriority[type]`. So every
spawner (HQ, school, center, miners) runs the same arithmetic on the same shared counts and the team converges on the
composition without a coordinator. JBW's `BuildingManager` is the if-chain version of the same idea but every spawner
runs it on the chain's `buildings[]` counts; miners carry a "desperation index" (PM 3.2: build the school on an
interior tile, after 8 wanting turns on next-to-wall, after 30 anywhere; code `MAX_BUILD_TURNS 6`, `DESPERATE_TURNS
30`). What we did: `DesignSchool.turn` and `Miner.build` are long if-chains over local counters (`builtVap`, `builtFC`)
that die with the miner; TRAINING_LOG stage 8: "RandomSoup1 banked 39,000 with no drone (the first center came after
the vaporators, i.e. never)".

**Custom data structures and tables.** Chowder PM A.2: own HashSet 65 bytecodes vs java.util 161 per add; a
JavaScript generator for the BFS delta table. citricsky `util/`: FastIntSet2D, FastIntCounter2D, BooleanArray,
FastMapLocationDeque, a murmur Hash. JBW: `HashSet` only in BugPath for loop states, everything else arrays. What we
did: arrays throughout (DESIGN.md), but `new MapLocation` in nested loops cost us bytecode overruns in Drone.shellScan
(1,500 a scan) and Miner (noted in C.java and TRAINING_LOG 83).

**Round-indexed constants instead of per-call transcendental math.** JBW `WaterManager.wHeight`, THG
`Utils.getRoundFlooded` (switch generated by `precomputation/round_flooded.py`), AngusRitossa `water_level_round[]`,
citricsky `Util.TURNS_TO_FLOODED`. Ours: `Robot.waterLevel(r)` = `Math.exp(...)` with two `Math.sin` at every call site
(Landscaper.help, wall, Drone...). (Not measured, but a `Math.exp`+`sin` call is tens of bytecodes against a table
lookup of ~3.)

**Spare HQ bytecode spent on map analysis.** JBW's HQ uses its 20k budget in rounds 1-3 to BFS the 7x7 and broadcast
the wall mask; smite's HQ scans all 145 visible tiles for soup, buckets it into 3x3 "tile numbers" and keeps
accessible/inaccessible lists it broadcasts as MinePatchMessages. What we did: the HQ's turn is ~3.3k of 20k (LEARNINGS
"Bytecode") and it computes nothing about the map.

### 1h. Tooling, testing, process

**Copy the best bot.** THG PM 11, first lesson: "Don't be afraid to copy ... the most tried and true strategy in
Battlecode is to copy whatever better teams are doing. We were able to temporarily have a significantly better bot
than Java by copying exactly what they were doing, then making some beneficial tweaks" -- from replays, in a few days,
in week 3. We were forbidden source and post-mortems by design, but we also had their replays from 2026-09-26 (PROMPTS
45) and only ever copied single tactics (the rush, +79; the raid, null; the vaporator hold, -17), never the whole
design; DESIGN.md's R1 lattice was reverse-engineered from winkelmantanner boards and kept our wall-first skeleton.

**Prioritise by win-rate delta; one or two changes made well.** THG PM 11: "it is not a good strategy to go through
a replay and note every single area of improvement, then go fix them all"; JBW PM 5: "every time I'd try to implement
a sophisticated strategy that requires a lot of coordination it would always flop". Our TRAINING_LOG has 87 numbered
iterations plus R1-R19 and 36 enclosure stages in eight days; most are mechanisms of the wall-first design.

**Rewrite when the strategy changes.** JBW PM 5: "if the code undergoes drastic changes (for instance because you
change the bot's main strategy) I would suggest to do a bot from scratch"; THG PM 8 regrets not having written the
drone class "in a more extensible fashion". Our R1 "rewrite" kept the seats, helpers, school pacing and quarry of
g_iter12 (DESIGN.md "What R1 keeps from us") and the lattice never became the base.

**Indicator dots/lines for location bugs** (THG PM 11: "save so much time compared to print statements"); every bot
read uses `setIndicatorLine/Dot` heavily. We used `@tag` logs into the replay and a replay dumper, which is stronger
for counting but we never looked at a game with the client.

**Mass testers.** THG `scripts/Alephaestus.py` (all maps, both sides, win counts), THG tuned nothing ("we ended up
not being able to tune any of our constants robustly, but encourage future Battlecode teams to tune better", PM 10);
Kryptonite `scripts/runAll.sh` + `checkKeywords.py` greps every map's log for "Exception", "BYTECODE LIMIT EXCEEDED";
smite `prep_submission.py` strips `System.out`/indicator calls for the submission zip. Our tooling (tools/README.md:
gauntlet, paired mirror, SPRT, Bradley-Terry ladder, replay dumper, puppet replays, census scripts) is far ahead of any
of theirs -- the process was not the gap.

---

## 2. Ideas we tried but could not make work, that top teams made work

- **A sealed interior with buildings in it (Iteration 6 citadel; the enclosure, 36 stages; the laurenschneider
  enclosure).** Ours: ring at Chebyshev 2 (16 tiles) or a shell at 2-3, buildings inside, "16 tiles halve the wall
  rate" -> refuted; enclosure "the bound below is reached", ladder probe 4-44. JBW/THG: the 7x7 wall at height 8 only
  (24 tiles, ~0.33 of a landscaper-round per tile), interior buildings, and the wall raised further only after r1400
  (JBW 3.6) -- the wall is a levee for an economy, not a race, and the economy (20-45 vaporators) pays for the bodies
  that raise it later. Our enclosure stage 36 "every hole closed" never had a vaporator income because the interior
  was quarry and the sites were four.
- **The lattice (Iteration 78 stages 1-14, R1 stages 1-19).** Ours grew to `LATTICE_R` 4-8 from the HQ with only the
  17th+ landscaper, kept pits (-17) beside sites, and the school at ground; stage 10 "the buildings do not survive",
  stage 14 "6 vaporators standing at r1250-1500". JBW: every landscaper works the lattice from the inner wall out
  (ExploreLandscaper.closestWallToBuild prefers OUTER_WALL tiles), buildings only on odd-odd tiles at height >= 8
  (`BuildingSpot.score` 2), the lattice built "towards the enemy HQ if possible" (PM 3.4).
- **Vaporator economy (Iterations 17, 72, 80, 83, 84).** All rejected or null in the paired mirror against our
  wall-first incumbent. Every top bot does it; the mirror could not price it (LEARNINGS "§8b: the mirror is not its
  judge" was written but the gates kept using it).
- **The late drone raid (Iterations 9, 61, 65, 69, 70; `arch_raider`, `arch_swarm`, `arch_wave`).** Ours: 20-70 drones
  from the late bank, rally 5 out, lift what is in reach -- null. JBW/THG/citricsky: drones carry landscapers, ignore
  guns for a window, drop miners to build guns on the spot, and the enemy's landscapers were already harassed for 1000
  rounds. Theirs is the same engine behaviour with three more pieces and a 5x economy.
- **Home-defence drones / the shield (Iterations 10, 23, 81; `arch_drone`).** Ours were "helpers not built" or
  "absent where needed". smite's shell drones and Chowder's drone wall exist because the center comes first and the
  cookie's vaporators keep paying.
- **Net guns (Iterations 4, 16, 31, 37, 47, 79).** Every attempt drowned or displaced a landscaper; theirs stand on
  lattice tiles at height 8 under the vaporator-count rule.
- **Rush defence (Iterations 29, 53, 54, 56, 57, 58, 62, 64, 66, 71; R8-R12).** 0 of 5 against a real rusher; JBW's
  RushManager (center first, drone per enemy unit, landscaper +1 under a gun) and THG's `HQ_ATTACKED` composition won
  most rush games (JBW PM 3.4) except against Kryptonite's stacked soup.
- **The perch / keep / plateau (Iterations 18-20, 24, 86-87).** Raised pads for buildings fed by dedicated
  landscapers -- claim races, drownings, "priced below the gate". The lattice is the generalised plateau: every
  parity tile is a pad and every landscaper raises the nearest low one (`shouldRenovate` in THG, `tryTerraform` in
  citricsky).
- **The rush (Iteration 63, ACCEPTED, +79).** Ours plants a school from r2 8 and gives up at r200; Kryptonite's plants
  on a cardinal tile adjacent to the HQ, builds a center after 3 moves without progress so a drone can carry the
  miner, builds a net gun beside the school when a drone or center is seen, protects the gun with 4 landscapers, and
  aborts on 3+ enemy landscapers (BotMinerRush.java, BotLandscaperRush.java). Battlegaode's dug through obstacles.

## 3. Ideas we had that top teams also had

- Three-hypothesis symmetry with pruning on sight (ours also prunes on remembered terrain; theirs share eliminations).
- Shoot-priority on the HQ/net gun (ours four ranks; citricsky two; JBW nearest).
- Dirt on our own buildings is dug off first ("heal"), including the HQ (JBW `buildingHurtMove`, THG `tryHealAdjacent`,
  smite "Healing HQ", ours `hqDigs`).
- Bury an adjacent enemy building with carried dirt, with a type priority (citricsky FC 6 > NG 5 > DS 4 > vaporator 3
  > refinery 2 > HQ 1; THG HQ 2 > vaporator/NG 1; ours HQ first, else nearest).
- A round-salted message authenticator (ours `Comms.auth`; THG/JBW/smite equivalents).
- Water-aware movement (`canMove` does not check flooding -- JBW BugPath.updateArray, THG SafetyPolicy, ours `safeTile`).
- The assumed (0,0) origin (`C.ASSUME_ORIGIN`): every bot read uses absolute coordinates and `w - x - 1` images.
- Don't dig under a friendly unit or building; don't raise a tile a miner stands on (AngusRitossa `hasMiner`, ours
  "never under our own units").
- A levee margin (ours `LEVEE_MARGIN`, water 300 rounds ahead + 2; JBW `MIN_SAFETY_WALL 2`, `SAFETY_WALL_TURNS 60`).
- An early school under a rush (ours `rushSeen`; THG `hqAttacked` DS priority; JBW "if we're being rushed ... Design
  School" second).
- Deterministic per-robot RNG (ours an LCG; theirs mostly `Math.random` or `Random(id)`).
- A fixed engine seed makes paired games (ours `paired.sh`); THG's Alephaestus plays both sides -- the same instinct.

## 4. Sources

Post-mortems:
- Java Best Waifu: `BC2020_Postmortem.pdf` in https://github.com/IvanGeffner/battlecode2020 (official copy
  https://battlecode.org/assets/files/postmortem-2020-java-best-waifu.pdf, blocked)
- The High Ground: `Postmortem.pdf` in https://github.com/uvafan/bc20 (official copy
  https://battlecode.org/assets/files/postmortem-2020-the-high-ground.pdf, blocked)
- Bowl of Chowder: `battlecode/2020/index.html` on branch `old` of
  https://github.com/StoneT2000/stonet2000.github.io ( live page https://stonet2000.github.io/battlecode/2020/, blocked)
- smite: README of https://github.com/mvpatel2000/Battlecode2020
- Battlegaode: http://web.mit.edu/agrebe/www/battlecode/20/index.html (blocked; WebSearch snippets)
- confused: https://battlecode.org/assets/files/postmortem-2020-confused.pdf (blocked; WebSearch snippets)
- the-levee-builders: README of https://github.com/winkelmantanner/battlecode2020_the-levee-builders

Code (local clone directory names, with their GitHub repos):
- `IvanGeffner_battlecode2020/finalbota/` (https://github.com/IvanGeffner/battlecode2020)
- `uvafan_bc20/src/v14_final_bot/`, `uvafan_bc20/scripts/Alephaestus.py`, `uvafan_bc20/precomputation/` (https://github.com/uvafan/bc20)
- `mvpatel2000_Battlecode2020/src/qual/`, `scripts/` (https://github.com/mvpatel2000/Battlecode2020)
- `awesomelemonade_Battlecode2020/citricsky/` (https://github.com/awesomelemonade/Battlecode2020)
- `chow_final/Chow10/` from `chowder/src/FinalChowBot.zip` (https://github.com/StoneT2000/Battlecode2020)
- `rzhan11_Battlecode2020/src/quals_bot/`, `scripts/` (https://github.com/rzhan11/Battlecode2020)
- `winkelmantanner_battlecode2020_the-levee-builders/` (https://github.com/winkelmantanner/battlecode2020_the-levee-builders)
- `AngusRitossa_battlecode20/src/newbot/` (https://github.com/AngusRitossa/battlecode20)
- `ronniesong0809_battlecode20-RADS/src/finalbota/` (identical to IvanGeffner's finalbota, same constants and file set)

Ours (https://github.com/terryvanbelle/battlecode20-vibe): `README.md`, `DESIGN.md`, `LEARNINGS.md`, `HANDOFF.md`,
`RESEARCH.md`, `RULES.md`, `BENCHMARK.md`, `progress/ELO.md`, `progress/METRICS.md`, `TRAINING_LOG.md` (sections
"What the field does that we do not", "The enclosure", "Copying tactics: scorecard", "The lattice", "Ledger (closed
directions)", "2026-09-30 ladder-position calibration"), `src/bot/*.java` (HQ, Robot, C, Landscaper, Miner, Drone, Nav,
Comms, MapState, NetGun, DesignSchool, FulfillmentCenter), `src/arch_*`, `tools/README.md`.
