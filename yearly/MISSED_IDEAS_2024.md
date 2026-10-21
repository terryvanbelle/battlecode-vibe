# MISSED_IDEAS_2024 — what the top 2024 teams did that we never figured out

**Year / game.** Battlecode 2024, "Breadwars" (ducks, 3 flags a side, 200-round setup behind a dam, traps, crumbs,
level-sum tiebreak). Written 2026-10-21 after the source/post-mortem ban was lifted.

**Post-mortems.**
- *cout for clout* (HS 1st, 2nd on the combined ladder), `battlecode.org/assets/files/postmortem-2024-cout-for-clout.pdf`:
  **search snippets only** (host blocked). Snippets gave: table of contents (Attack Micro, Healer Micro, Chicken Micro,
  Placing Traps, Filling Water, Flag Sniping, HotlineBling comms), the builder strategy tried twice and dropped after
  seeing Gone Sharkin' did not use it, "very micro-heavy year", flag passing moving a flag 2*sqrt(2) in two turns,
  canSpawn() used to detect enemies on the spawn pad, distress = mark the spawn nearest the stolen flag so jailed ducks
  respawn there. Their code (jonters/battlecode24, package `waxingmoon`, 4,325 hand-written lines + 23k generated BFS)
  was skimmed (Robot.java, ChickenMicro.java head, log.txt).
- *muskellunge* (13th): listed on battlecode.org/past.html; **PDF not found** by search. The dteather.com / Towards Data
  Science write-up "Battlecode 2024 finalist, 13th" is probably the same team (inferred): snippets only.
- *zhugeduck* (dmtrung.com): snippets only (role split front/back line).
- *Honey Ducklings* (noahzemlin/Honey-Ducklings README): **read in full** (in the repo).
- *Gone Sharkin'* (1st): no write-up exists; code read instead.

**Code read** (shallow clones of the public repos):

| team (ladder name) | repo / package | lines | how much |
|---|---|---|---|
| Gone Sharkin' (chenyx512) | chenyx512/battlecode24 `bot1` (README: "the main bot is bot1") | 5,685 (+1,747 generated BFS20) | full: Robot, Micro, FlagManager, SetupManager, SpecialtyManager, RoleAssigner, KillRecorder, Cache, Explorer, MapRecorder, Constants, CommsTemplate, gen_comms.py schema; PathFinder head |
| don't eat my nonorientable shapes (Strequals) | Strequals/bc24 `ineedtodomypdehw` (README: final) | 3,390 | full: RobotPlayer (1,724), Micro, Communications, MapSymmetry, Explore; BugNavigation 1/3 |
| camel_case (jmerle, 2nd) | jmerle/battlecode-2024 `camel_case_v21_final` | 5,343 (4,333 generated Bellman-Ford) | full: every hand-written file |
| kuma (IvanGeffner) | IvanGeffner/BTC24 (root package `kuma`) | 2,054 | full except BFS/BugPath/Util |
| andli28 | andli28/bc2024 `v9_USQuals_angle` | 5,308 (1,994 generated Bfs) | RobotPlayer ~1,950 of 2,260, Comms.update, Info, Pathfinder head |
| NotLLeon, andrewgopher, hsmalladi, Gymhgy, CyrilSharma | ladder packages | 1.5-4.2k each | skim: SetupPhase/MainPhase, Traps, BotSetupFlagDuck, StartPhase, TrapMicro/Heist |

**Two ladder caveats found on the way.** (1) Our ladder's `chenyx512.flagbot_final` (3,079 lines) is an older package;
the winning bot is `bot1`, which differs in every file and adds BFS20 and EnemyTracker (every claim below about chenyx512
is about `bot1`). (2) Our `Strequals.duck0127v5` is also older than the README's final `ineedtodomypdehw` (3,390 vs
3,103 lines). The real finals are therefore (inferred) somewhat stronger than the 2283/2284 we measured.

**Where we stand** (progress/ELO.md, HANDOFF.md): final bot `src/g_iter7`, 2113 +- 25, rank 6 of 88, field score 88.6%,
30.5% against the five bots above it: Strequals 2284, chenyx512 2283, jmerle 2278, kuma 2258, andli28 2187 (g_iter7
15-33, 9-39, 15-33, 14-34 and 1739-4109 against them). LEARNINGS S4/S6/S8/S9 record the open gaps: heal-XP level gap,
failed offensive-presence arms, saved robots not becoming captures, and the kill-reward economy against the upper tier.

---

## 1. Ideas we never figured out

### Macro strategy / win condition

**Need-driven role market instead of fixed roles (chenyx512 RoleAssigner).** Six roles: defend own flag 0-2, attack enemy
flag 0-2. `getNeedLevel`: an own flag in distress returns a probability `disScore * assignedScore` with `disScore = max(0.02,
1 - (dist - 8)/(W+H)*5)` and `assignedScore = 1 - assigned/15` (MAX_ASSIGNMENT 15); a carried enemy flag near home needs 5
escorts, far from home 15; an uncarried enemy flag scores `-0.1 - assigned/100`, which spreads the rest evenly over the
three enemy flags. Each robot reassigns itself with probability = need (`FastMath.fakefloat() < need`), and counters per
role live in the shared array (`myflags.assigned`, `oppflags.assigned`, 6 bits). Offensive roles spawn at a random HQ
("to cover the map / find flanking angle / not get clogged"), defensive ones at the HQ minimising distance to mission +
distance to the enemy, skipping HQs flagged congested (>8 allies adjacent → `HqCongestRound` 15). Evidence: bot1/RoleAssigner.java
(all), Robot.java:78-85. **We**: roles by creation index (3 defenders, 47 army, Duck.isDefender), everyone marches on the
nearest registered enemy flag (Duck.fieldTargetFrom), alerts answered within ALERT_RADIUS2 100; the flank-raid squad
(TACTICS T12, RUSHERS/RUSH_FLANK) and ALERT_NEAREST (g4alert) never shipped. **Why it mattered**: LEARNINGS S6 found
"presence at enemy flags is an outcome of the fight"; the winner never had 47 ducks on one flag, so each of its three
pressure points met a third of our army. CAPABILITIES C12 (force division) was below par all week.

**Three-way army split by id (andli28).** `attackSquadOne/Two/Three` = shortId <16 / <32 / >=32, each squad targets "its"
enemy flag (comms slot, else broadcast slot) and falls back to the others when its flag is carried (RobotPlayer.java:198-200,
457-501). Four attack specialists per squad. Same mechanism, deterministic. We rejected the flank raid as a dose arm
(TACTICS T12: 6/10/15 ducks); nobody tried a permanent even split.

**Setup flag carry with a destination choice, a connectivity test and a hill-climb (chenyx512 SetupManager).** Ducks
seq 4-6 carry the flags; ducks 7-12 are "route setters" that walk between each pair of spawn centres in r1-150 and write
`HQConnected[src*3+dst]` when within dist2 25. At r20 each flag heads for the spawn centre farthest from the enemy
centres; at r70, if the route is not yet confirmed connected, it reverts to its own centre; from r150 (or on arrival) it
hill-climbs on `score = sqrt(dist to nearest enemy centre) - sqrt(dist to own nearest HQ)` (after arrival the HQ term is
averaged with the distance to the chosen destination), plus noise 2/1/0 in the last 20/8 rounds "so the flags converge
and do not block each other", honouring MIN_FLAG_SPACING; drop at r198; at r200 the carrier builds a WATER trap toward the
enemy and a STUN on the flag tile (SetupManager.java:316-399). This is why chenyx512's flags cluster (TACTICS T2 census:
median 14 tiles apart, nearest pair 6, "it beats us 94%"). **We**: RELOC_CLIMB maximises distance to the nearest live
enemy centre only, within 20 tiles of the spawn centre and 8 from other flags (Duck.relocClimb, C.CLIMB_R2 400, CLIMB_SEP2
64). **Why it mattered**: our own g7spawn result (LEARNINGS S8: a flag far from spawn "lost its reinforcements") is the
missing `- dist to own HQ` term; and their flags stay mutually defensible.

**Flag placement scored on terrain pockets (hsmalladi, dteather).** hsmalladi BotSetupFlagDuck.getScore: + sqrt(dist) to
every enemy spawn, -10000*d for another own flag within dist2 63, and **x2 for a tile with 7 wall neighbours**. The
13th-place write-up (snippet) says rivals "scored map spots on nearby walls and distance from spawns". We never scored walls.

**Map-size and phase conditional aggression (chenyx512 Micro).** HP bars lowBar 450 / highBar 700 drop to 350 / 550 when
`H*W < 1600 && round < 500` or when an own flag is urgent, with the comment "to deal with camel_case style rush so that we
get to late game safely" (Micro.java:102-109). Healers hold (never offensive) unless an own flag is urgent and they are
farther than (W+H)/12 from home. **We** had no map-size term anywhere (C.java) and one RETREAT_HP 300.

**Distress bookkeeping that ages and expires (chenyx512 FlagManager.initTurn).** The master duck advances a distressed
own flag's believed location one tile toward the nearest enemy base every 2 rounds, caps at 80 unseen rounds then marks it
`exists = 0` ("prevents our ducks from clogging up and trying to recover a flag that's long gone"); a sighting resets the
counter; false distress is cleared when a duck stands within dist2 4 of the reported spot with no enemy in view
(FlagManager.java:61-90, 176-189). **We**: C.FLAG_LOST after 60 unseen rounds (g_iter4, +16) and CARRY_PREDICT/CONTACT
(both off after g4pred/g4contact failed). Theirs is the same idea in 30 lines with no miss counting.

**Alive-count estimation (chenyx512 KillRecorder; andli28 Comms respawnTimer).** Each kill decrements the shared enemy
count and is re-added JAILED_ROUNDS later (`killCnt[round % 25]`; andli28 keeps a queue of (round+25, kills)). chenyx512
uses `cntDiff < -4` to let everyone build stuns; their comment says "actually not too useful". We never knew the score.

### Economy

**Exactly three builders, and they are the trap economy.** Every top bot pins a tiny fixed set of ducks to build XP and
gives them the traps:
- chenyx512: `NUM_BUILDER 3` by sequence id; builders dig a parity lattice (`(x+y)%2 == masterID%2`) when `buildLevel < 4`
  and >dist2 25 from a spawn centre (SpecialtyManager.act); `tryDropTrap` lets a non-builder build **only** when a flag is
  urgent or the team is 5 units down with 5+ enemies and 4+ allies in view; builders use radius 16 (others 8) and may
  build on up to four tiles a turn (self, toward, rotateLeft, rotateRight) (Micro.java:187-239).
- kuma: `isTrapper` = internal ids 17, 19, 35; `digLikeCrazy` from round 5 until build level 6 (skipping tiles that
  `Util.shouldFill` would refill); non-trappers trap only as flag defenders or above 3,000 crumbs, and only if their action
  cooldown is < 5 so they can still act (TrapManager.java).
- Strequals: `NUM_BUILDERS 3` claimed through a shared counter; `tryDigAndBuild` digs `(x+y)%2==1` tiles until level 6, then
  the builder walks to the stun tile nearest the enemy and lays diagonals (RobotPlayer.java:1448-1562).
- andli28: shortIds 2, 29, 47 `trainToSixByDigging` at home during setup (30 XP), refusing tiles with water on two sides or
  a dam/wall neighbour (RobotPlayer.java:183, 1657-1686); `role == BUILDING` while the action cooldown + build cooldown < 10.
**We**: C.BUILDERS one-in-ten plus "non-builders still build when no builder in view" (g4builder, g4builder2: parked, 1/7
at 5(a) twice; LEARNINGS S4, TACTICS T14); setup digs for every duck (a3dig5/10, b3own: rejected). The top bots' version
costs 3 x 30 digs = 1,800 crumbs once and halves every later trap for the whole game.

**Trap volume scales with the bank, not a fixed enemy count.** kuma stun minimum enemies 4 / 3 / 2 at crumbs 400 / 1000;
explosive only above 1,250 crumbs with 3-5 enemies; trap spacing 5 / 4 / 2 at 500 / 800 crumbs; enemy distance 5, or 8
above 1,500 (TrapManager.getMinEnemiesStun, placeStun). Cyril TrapMicro: build if blast-count score >= 5 enemies always,
3 above 500 crumbs, 2 above 1,500, 1 above 5,000, and only if an enemy can reach the trigger tile in two steps (bitmask
dilation over passable tiles). jmerle: an adjacent tile with >3 enemies within dist2 8, or >2 when crumbs > 50 x 100.
andli28: stun spacing 2 → 1 above 200 crumbs; explosives only above 3,000, spaced 16. andrewgopher Traps.chooseTrapLoc:
spacing 4 / 2 / 1 by crumbs and enemy count, trap value = enemies whose last two positions head toward the tile with line
of sight. **We**: STUN_ENEMIES_MIN 3 and TRAP_RESERVE 200 fixed; ct1/ct2 (2 or 1 enemies, no reserve) read 20-22 and
16-18 and were rejected (TRAINING_LOG "Iteration 9"). LEARNINGS S9: they built 359 traps to our 216 on equal map crumbs.

**Water traps as an area weapon on enemy territory.** chenyx512 builders, once on enemy territory after r200, build WATER
on every tile with `x%4 == masterID%4 && y%4 == masterID%4` (a 4x4 lattice of invisible moat triggers; SpecialtyManager.act,
Micro.tryDropTrap). kuma `placeWater` only on tiles whose territory is the opponent's, spaced dist2 9. Strequals builders
put a WATER trap **on the own flag tile** and STUNs on its diagonals (tryBuildBuilderDefense); Gymhgy sentries and
chenyx512 carriers build WATER toward the enemy at r200. **We**: WATER_WHEN_WEAK off (t7water 85/110, "lean positive,
never adopted", TACTICS T7); flag rings of STUN/EXPLOSIVE only.

**Crumbs outrank fighting when nothing threatens.** chenyx512 `Robot.play` runs `findCrumb()` before `Micro.act()`
(with a 10-stuck counter after r200); kuma ObjectiveManager goes crumb → flag within vision → enemy → flag → explore;
Strequals `tryStunForCrumbs` builds a STUN on an adjacent tile holding >= 25 crumbs when two such tiles are near an enemy
(the enemy stepping onto the crumb fires it) and lowers crumb priority if a trap already sits there; jmerle's attack-move
scores the step tile by its crumbs minus enemies within 10. **We**: POST_SETUP_CRUMBS only for idle ducks, CRUMB_STEP +40
(g4crumb, half of g_iter5). Map crumbs were at parity against the upper tier (LEARNINGS S9), so the gap is spending, above.

### Combat micro

**Plan the step-attack one turn ahead (chenyx512, andli28).** chenyx512 `shouldPlanStepAttack = actionCooldown < 20`;
in OFFENSIVE state the tile order is: fewest enemies in range minus a kill, canAttack, canKill, then if planning: more
`canAttackNext` (reach an enemy next turn through a free adjacent tile, `checkCanAttack`), fewer `canBeAttackedNext`,
fewest `numAttackRangeNext` but at least 1; if not planning: fewest `canBeAttackedNext`. andli28 `findOptimalCombatDir`
retreats (maximise mean dist2 to enemies) when `!isActionReady()` **unless** the cooldown is in its last turn and the
nearest enemy is >= dist2 17, and also retreats when a target is already in range (shoot, then leave), else advances to
the tile with the smallest mean dist2 > 3 (RobotPlayer.java:1913-1933). **We**: kite tile = -1000 per threat + 50 if minD
in 11..20 (Micro.fight:412); the recharge-tempo arms g7kite/g7kiterc (TACTICS T19) were parked/untested at shutdown. The
upper tier stayed in reach after a strike 0.111 of the time to our 0.252 (research/upper-tier-study-2026-10-07).

**Lethality check, not an HP constant (andli28).** Retreat if `health <= sum over enemies within dist2 10 of the advance
tile of (150 + their level bonus)` or `numHostiles - 2 >= numFriendlies` (RobotPlayer.java:1735-1745, 1913). kuma
`gonnaDie = (150 or 250 after r600) * enemiesInMoveRange >= health`. **We**: RETREAT_HP 300; g7ehp (700) was +28 against
andli28 but band-neutral (TACTICS T18). The sum-of-damage form adapts to the group size we face.

**Do not count a killable enemy as a threat.** Strequals `enemiesAttacking - jailableAttackers` and `enemiesTargeting -
jailableAttackers` (Micro.java:163-198); chenyx512 `numAttackRange - canKill` in DEFENSIVE state. **We** counted every
enemy within dist2 10 (Micro.threat). This is one mechanism behind our 22.5% vs 43.8% step-in kill conversion
(research/andli28-study synthesis).

**Track who is stunned and ignore them.** kuma StunManager: at turn end remember own STUN trap tiles in view; at turn
start a vanished trap marks every enemy within dist2 13 as stunned for the round (`StunRounds[id % 4096]`), and the micro
counts `enemiesInRangeNoStun` separately. Strequals `checkDetonated` does the same with id strings `stunned0..2` rolled
each round (and uses the previous positions string to catch enemies that moved into radius); stunned enemies do not add
to `enemiesAttacking` and `shouldEnterRange` alternates by round parity. **We** had no stun tracking at all (STUN_WARY was
only about our own risk).

**Target value by level and time-to-kill (chenyx512).** `getAttackTargetScore`: +1e9 if killable now, +1e8 carrier, then
`getRobotScore / ceil(hp / damage)` where robot score is a DPS table by attack level (1.05/0.95-1 ... 1.6/0.4-1), build
level x4, heal level; `tryAttack` recurses so a level-6 attacker fires twice. Heal target: same value, +1e5 for
non-healers, never heal >= 870 HP, and `canHeal()` is false one XP short of heal level 4 for builders/attackers whose main
skill is <= 3 (Micro.java:267-337, SpecialtyManager.canHeal). Strequals score `1000/(1+allies within 10 of target) +
hp/10 - 2000 kill - 4000 carrier` (focus on the enemy most allies cover). jmerle: priority = attack+heal+build level,
carrier +1000, tie lowest HP. **We**: carrier, then lowest HP, then attack level (Micro.bestTarget); kill-first existed only
in the dead `bestTargetV2`.

**Do not block a hurt teammate's way out (chenyx512 MicroDirection.updateAlly).** A step adjacent to an ally with HP <
min(mine, 700) (or carrying a flag) is rejected if it would occupy the ally's escape direction (toward-enemy opposite,
+-45 deg) and no other exit tile is free (`blockTeammate`, second key of `isBetterThan`). Nothing like it in Micro.java.

**Someone has to go in (chenyx512 OFFENSIVE).** When `myTotalStrength > 900 && myTotalStrength - oppTotalStrength > 500`
(sums of HP in view) `canAttack` outranks safety, with an ASCII diagram of the choke-point equilibrium it breaks
(Micro.java:560-573). Our ADVANCE_MARGIN 3 was a head count, not HP.

**Heal gates stricter than ours.** jmerle `healFriendly` returns if any enemy is in vision (dist2 20); kuma heals only if
own HP <= 500 or no attack in the last 5 rounds; chenyx512 heals before moving only if no attack/high-value heal is
possible after moving. **We** shipped HOLD_R2 10 (g_iter7, +144). LEARNINGS S4: the level gap is heal XP (-27.6 of -35.3
levels) — their heals land on targets below 870 HP when safe, and their 20-25 designated healers collect the XP.

### Flags, carriers, escorts

**Escorts keep their distance and clear the water (chenyx512 PathFinder.escort, FlagManager).** An escort targets
`carrier + 3 * protectDir` with `protectDir = DIRS[seqID % 8]` (a ring 3 tiles out), steps away if adjacent, reports the
first enemy seen as `escort_loc` (a distress the RoleAssigner answers with up to 15 ducks), and any duck within dist2 8 of
an allied carrier fills water in its way; a carrier with < 2 friends steps away from the nearest enemy before pathing.
Strequals: allies within dist2 2 of the carrier `tryMoveAway`, within 8 fill checkerboard tiles beside it, beyond 8 close
in. NotLLeon: `ESCORT_SIZE 5` request counter per flag within range 100. **We**: ESCORT_* off after g3escrg/g3escrg2
(carrier deaths 15.8 vs 8.8; LEARNINGS S6); nothing moved our ducks *out* of the carrier's way.

**Drop the flag toward home when about to die (kuma Robot.dropFlag).** If `gonnaDie`, drop on the adjacent tile with the
best BFS distance home (else nearest to the spawn zone) so the next duck carries on; also the published "flag passing"
of cout for clout. We adopted only pick-after-move (T13) and rejected relays (T3).

**Carrier never paths through water, everyone else fills (jmerle, chenyx512).** jmerle's generated Bellman-Ford treats
water as weight 3-4 and visitable only when `!hasFlag`; chenyx512 `getPassible` treats remembered water as impassable
when carrying a flag or below 30 crumbs (water memory stamped by round/10, forgotten after 50 rounds). We had FILL_SMART
off and fills gated by stall only (Nav.fillToward).

**Return along remembered waypoints (andli28).** `prevWaypoints[20]` spaced 5 tiles on the way out; the carrier backtracks
through them (RobotPlayer.java:115-118, 1031-1053). Our carriers bug-navigate home from scratch.

### Navigation

**Generated local shortest paths.** jmerle: `generate_bellman_ford.py` emits a 4,333-line relaxation over the 20-dist2
vision disc, weights `1 + (water ? 2-3 : 0) + enemies within dist2 10 of the tile` (enemies counted only when some are in
view), falls back to bug after 3 rounds without progress (Navigator.java, BellmanFordNavigator.java:73-135). andli28
`Bfs.getBestDir` (1,994 lines) when > 5,000 bytecodes remain, else "buhg" with a boundary-scan heuristic for the first
turn direction (Pathfinder.java). chenyx512 BFS20 (1,747 lines, 2,500 bytecodes) is used in micro to ignore enemies that
are unreachable within vision (`canIgnoreEnemy`) and to refuse trapping unreachable enemies. cout for clout: eight 2.8k-line
BFS classes by heading. **We**: greedy + bug with per-robot handedness and a 20-turn stall flip (Nav.java, 115 lines),
plus a 3-move reachability BFS (REACH_FIX). Honey Ducklings' README names exactly our failure: bug nav "paths very poorly
in maps with small corridors".

### Communication

**Generated, bit-packed, pull-once/push-dirty comms (chenyx512 gen_comms.py).** Schema: sync id 13b; HQ loc 12b +
congestRound 5b x3; symmetry 3b; myflags {id 12, exists 1, loc 12, original_loc 12, assigned 6, distress 1, not_seen_cnt 7}
x3; oppflags {id, exists, loc, original_loc, escort_loc, confirmed, carried, assigned} x3; team counts 6b x2; enemy loc
x3; wall loc x6 (walls are reported so every duck's map agrees); HQConnected x9. All 64 words are read once in
`Comms.pull()` and written back only if dirty in `push()` (CommsTemplate.java). **We**: hand-laid 64 slots (Comms.java
schema), per-call readSharedArray.

**A decaying heat map of fights (Strequals Communications.updateEnemies).** 8 report slots, each `round(11b) | score(9b) |
x,y`; score = 2 per enemy - 1 per ally + 60 if an ally/self holds a flag + 200 if an enemy carries ours + 60/(1+d) per
enemy near an own flag; decays x0.7 per round; a report within vision of a stronger one is merged; `readEnemies` picks the
best score / (1 + sqrt(dist)) and otherwise the least-recently-seen point within radius 10 of a broadcast. Spawning,
seeking (`seekEnemy`) and respawn placement all read it. kuma keeps a 15-entry ring buffer of enemy sightings with 4-bit
round stamps (Comm.java). **We** shared only flag alerts and carrier positions; "where is the fight" never existed.

**Broadcast every duck's cooldowns (andli28 Comms.writeSelfCDs / getAllyCDs).** 4 bits per robot (action cd/10, move
cd/10) in slots 28-40 so allies know who can strike this turn. Not in our schema.

### Symmetry & exploration

**Least-recently-seen exploration grid (Strequals Explore).** `roundSeen[x][y]` updated every turn; explore target = best
of 10 random samples by oldest sighting; the broadcast disc is searched with `getLeastRecentlyVisitedWithinRadius(bfl,
10)` on a 3-step lattice. This is our HINT_SWEEP (block 2), which stayed off and untested (C.java:153).

### Bytecode & engineering

**Cheap big arrays via strings (chenyx512 Constants).** "it takes n bytecode to declare an array of length n, but 1
bytecode to convert a string of length n to a char array": `STRING_LEN_4200.toCharArray()` backs the 60x60 map record and
the id → sequence table. Per-phase bytecode tags (`Debug.bytecodeDebug += " micro=" + Clock.getBytecodeNum()`), an
overrun printer that can `rc.resign()` under DEBUG_FAIL_FAST; jmerle prints any turn above 95%; cout for clout's log.txt
holds per-phase bytecode profiles at rounds 100-1000. **We** had maxBc/overrun counters, but MEAS7 (LEARNINGS M10) shows
our near-miss check was saturated by Sym.update's fill-to-limit.

**Code generators for the hot paths** (gen_comms.py, gen_bfs.py, generate_bellman_ford.py, Cyril's bitmask tile
loaders). We generated nothing.

### Tooling, testing & process

**Tournament map lists and win-condition tallies.** chenyx512 `compare_bots.py` (copied from jmerle's 2023 script) runs
both sides on the 77 official maps grouped early/sprint1/sprint2/qual/HS over 6 cores; jmerle `scripts/compare.py` tallies
wins by condition ("T Unit Levels", "T Flags Captured", "T Crumbs", "Resigned") in a rich table. Cyril `modal_tester.py`
runs a full match set on Modal cloud containers in < 5 min with replays on a network volume. Gymhgy keeps ablation variants
as packages (v10o1-o4, v10off1-5). **We** had a seeded VM queue, Bradley-Terry ladder and censuses (LEARNINGS T1-T5),
i.e. more measurement than any of them; what we lacked was the per-condition tally as a first-class output (our level-sum
loss share was discovered late, in the andli28 study).

---

## 2. Ideas we tried but could not make work, that top teams made work

| ours (verdict) | theirs | the difference |
|---|---|---|
| Specialisation sp3/sp5/sp7: 30-70% of ducks never heal (REJECT 12-30; TACTICS T11, LEARNINGS S5) | chenyx512 3 builders / 20 healers / 27 attackers; Strequals 3 builders / 25 healers; andli28 12 attack specialists | attackers still heal up to one XP short of heal level 4 (`canHeal`, `shouldHeal`), so they keep level-3 healing; healers are the majority, counted in comms |
| Builders g4builder/g4builder2 (parked); setup digs a3dig5/10, b2dig5, b3own, g7dig (rejected) | kuma ids 17/19/35, andli28 2/29/47, Strequals/chenyx512 3 by seq: dig to build 6 (chenyx512 to 4) during setup; others never build routinely | exactly three, all the way to level 6, and non-builders are *forbidden* to trap except in emergencies; dig tiles filtered for water/dam adjacency |
| End-game dig g7fc (passed 5(a), failed delivery), g7bank (failed) (TACTICS T17) | Strequals `DIG_ROUND 1950`: any unthreatened ready duck digs a checkerboard tile (RobotPlayer.java:653, 1648) | unconditional (no "level game" gate), costs nothing to gate |
| Flag relay a2relay/g4relay/g4relay2 (rejected; T3) | kuma `dropFlag` toward home when `gonnaDie`; cout for clout flag passing | relay only at the moment of death, placed by BFS distance home |
| Escorts g3escrg/g3escrg2, ESCORT_TIGHT, ESCORT_BEHIND (failed; S6) | chenyx512 escorts 3 tiles out on a seq-id ring, fill water for the carrier, step off if adjacent; distress calls escorts | escorts never crowd the carrier and never fight from inside its path |
| Carrier prediction g4pred, g4contact (failed, R3/R4) | chenyx512 master moves the believed flag 1 tile / 2 rounds toward the nearest enemy base, 80-round expiry | one shared estimate, no miss counting, feeds a probabilistic role market |
| Front stuns g4front (crumb-bound FAIL), STUN_FRONT off (T14) | Cyril TrapMicro bitmask reach x blast count with bank-tiered thresholds 5/3/2/1; kuma/andli28 thresholds by crumbs | the trigger count required falls as the bank grows, and only builders pay |
| Water trap when weak t7water (85/110, not adopted; T7) | chenyx512 4x4 water lattice on enemy territory; kuma enemy-territory only; Strequals/Gymhgy water on/at the flag | territory-gated, not odds-gated |
| Flag-tile stun n1flagstun (20-25, rejected; T1) | chenyx512 r200: STUN on the flag tile **and** WATER toward the enemy; Strequals WATER on the flag tile | water on the approach, which also digs the raider's escape |
| Recharge tempo g7kite (parked), g7kiterc (untested; T19) | andli28 retreat unless cd in last turn and dist2 >= 17; chenyx512 shouldPlanStepAttack | they also leave after every strike when a target is still in range |
| Step-in gate g7ehp ENGAGE_HP 700 (band-neutral; T18) | andli28 `health <= incoming damage sum`; chenyx512 states at 450/700, 350/550 on small maps | damage-sum, not a constant; map-size aware |
| Hint sweep HINT_SWEEP (never shipped; block 2) | Strequals least-recently-visited point within radius 10 of the broadcast | a persistent roundSeen grid instead of random re-draws |
| Banking c5bank/c6pair/b1v2 (neutral; T5) | jmerle, kuma, chenyx512: zero setup traps, trap in fights by bank tier | the bank is spent by rules that scale with it (section 1), not by one reserve constant |

## 3. Ideas we had that top teams also had

- ATTACK at r600 > HEALING at r1200 > CAPTURING at r1800 (chenyx512, Strequals, jmerle, andli28, hsmalladi, cout for
  clout; kuma/Gymhgy/Cyril buy `ACTION` first, the same slot) — our S5 finding.
- Heal hold near enemies (jmerle: any enemy in vision; kuma: 5 rounds after a strike; chenyx512: only if nothing better) —
  our g_iter7 lever, in a weaker form (dist2 10).
- Observed symmetry from wall/spawn-zone memory with early elimination (chenyx512 MapRecorder; Strequals MapSymmetry) —
  our Sym after audit A3; chenyx512 additionally shares wall sightings through 6 comms slots.
- Flag id = location index of its start (chenyx512 getOppFlagIndex "engine hack") — our audit A2 (Sym.observeEnemyCentre).
- Pick up / attack after the move when that is better (jmerle attackOpponent move-then-attack scoring; andli28 attackMove
  chooses the order by which target has less HP) — our pick-after-move (BOT5, g_iter5).
- Combat stun with a minimum enemy count near the tile (jmerle >3 within dist2 8 of the tile; ours STUN_ENEMIES_MIN 3).
- Crumb sweep in setup, exploration toward unseen random points, dam wait at r>180 (all bots; Strequals/jmerle/kuma wait
  rather than bug around the dam).
- Flag relocation away from enemy spawns in setup (hsmalladi, Gymhgy sentries, chenyx512) — our RELOC_V2/RELOC_CLIMB.
- Flag distress → respawn at the threatened flag (kuma emergency bit, Strequals defender claim, chenyx512 distress,
  cout for clout HotlineBling) — our alerts + trySpawn.
- A per-robot sequence id claimed through a shared counter (chenyx512 sync slot, andli28 shortId) — our Comms.claimIndex.
- Carrier-first targeting and chasing enemy carriers (all).
- Version packages kept for every submission, a copy-bot script (jmerle duplicate.py, chenyx512 copybot.py, NotLLeon
  copy_bot.py) — our src/* packages and tools.
- A reachability test before engaging an enemy behind terrain (chenyx512 canIgnoreEnemy via BFS20) — our REACH_FIX (A4).

## 4. Sources

Post-mortems and write-ups (search snippets unless noted):
- https://battlecode.org/assets/files/postmortem-2024-cout-for-clout.pdf (blocked; also linked from jonters/battlecode24 README with a Google Doc copy)
- https://battlecode.org/past.html (lists "cout for clout" HS 1st and "muskellunge" 13th post-mortems; muskellunge PDF not found)
- https://dteather.com/blogs/battlecode24/ and https://towardsdatascience.com/battlecode-2024-finalist-ad3166c1acd5/ (13th place)
- https://dmtrung.com/blogs/battlecode_writeups.html (zhugeduck)
- https://battlecode.org/assets/files/postmortem-2025-spaark.pdf (2025 team recalling 2024 cout for clout's flag passing; snippet)
- noahzemlin/Honey-Ducklings README.md (read in full)

Code:
- https://github.com/chenyx512/battlecode24 — src/bot1 (Micro.java, RoleAssigner.java, SetupManager.java, FlagManager.java, SpecialtyManager.java, KillRecorder.java, MapRecorder.java, PathFinder.java, Constants.java, gen_comms.py, gen_bfs.py), compare_bots.py
- https://github.com/Strequals/bc24 — src/ineedtodomypdehw (RobotPlayer.java, Micro.java, Communications.java, Explore.java, MapSymmetry.java, BugNavigation.java)
- https://github.com/jmerle/battlecode-2024 — src/camel_case_v21_final (Unit.java, Navigator.java, BellmanFordNavigator.java, BugNavigator.java, SharedArray.java), scripts/compare.py, scripts/generate_bellman_ford.py
- https://github.com/IvanGeffner/BTC24 — kuma (Robot.java, MicroManager.java, TrapManager.java, HealManager.java, AttackManager.java, StunManager.java, SpawnManager.java, ObjectiveManager.java, Comm.java, MovementManager.java)
- https://github.com/andli28/bc2024 — src/v9_USQuals_angle (RobotPlayer.java, Comms.java, Pathfinder.java, Info.java)
- https://github.com/CyrilSharma/Battlecode2024 — src/finalBot/TrapMicro.java, Heist.java; modal_tester.py
- https://github.com/Gymhgy/battlecode-24 — src/v10official/StartPhase.java
- https://github.com/hsmalladi/Battlecode2024 — src/finalbot/BotSetupFlagDuck.java
- https://github.com/NotLLeon/battlecode24 — src/v3/SetupPhase.java, MainPhase.java
- https://github.com/andrewgopher/battlecode24 — src/player22/Traps.java
- https://github.com/jonters/battlecode24 (cout for clout) — src/waxingmoon/Robot.java, ChickenMicro.java, log.txt

Ours (https://github.com/terryvanbelle/battlecode24-vibe): README.md, LEARNINGS.md (S4-S9, M10, R3-R6), TACTICS.md (T1-T19),
HANDOFF.md, RULES.md, progress/ELO.md, BENCHMARK.md (roster), TRAINING_LOG.md (iterations 9-10, ablations),
research/andli28-study-2026-10-06/synthesis.md, research/upper-tier-study-2026-10-07, research/CAPABILITIES.md,
src/g_iter7/{C,Duck,Micro,Nav,Sym,Comms,G}.java.

Not reached: battlecode.org PDFs (403), web.archive.org, dmtrung.com, dteather.com, Google Docs (network policy);
repos carlguo866/battlecode24, Tim-gubski/BattleCode2024, muskellunge/battlecode24 do not exist.
