# MISSED_IDEAS_2023 — what the top Battlecode 2023 teams did that we never figured out

Year: **Battlecode 2023, "Tempest"**. Written 2026-10-21 after the ban on reading other teams' code and 2023
post-mortems was lifted. Every claim about THEIR bots cites a file/function or a post-mortem section; every claim
about OUR bot cites `src/c_gd1`, LEARNINGS.md, HANDOFF.md, TRAINING_LOG.md or progress/ATTEMPTS.md. Anything not
read directly is marked **(inferred)**.

## 0. Sources used and where we stand

**Post-mortems**
- 4 Musketeers (3rd place): read in full (`4_Musk_Post_Mortem.pdf` in github.com/maxwelljones14/Battlecode2023).
- no thoughts head empty (Newbie 2nd): read in full (the README of github.com/reeceyang/battlecode-2023 is the post-mortem).
- Gone Fishin' (2nd; our ladder #1 as carlguo866.submit26_final): **search snippets only**. Both copies
  (battlecode.org and carlguo.com/pdfs/battlecode_report.pdf) are blocked by the network policy (403). About 12
  WebSearch queries recovered: section headings (5.2 "Micro: Kiting, or moving in and out of enemy vision range"),
  the shared-array layout (821 of 1024 bits, 5 nearest wells per resource, 14 bits per island, congestion and
  carrier-count bits), the mana-first reasoning, "always kite back after an attack (corroborated by Super Cow Powers,
  the sprint winner)", the Sprint-1 rotational-symmetry mistake that led to MapRecorder, "amplifiers and
  destabilizers did not win games decisively", "booster is never worth it; destabilizer maybe in groups of four",
  and the finals loss to Producing Perfection ("captured a long island near their base early and let launchers heal;
  found our wells earlier").
- Don't @ Me (7th): **search snippets only** (blocked PDF). Recovered: table of contents (Control Flow, Micro,
  Economy, Pathfinding, Communications, Bytecode Hacking, Version Control), "launcher rushing was highly effective",
  "squadrons" of launchers, damaged carriers throw cargo and retreat, early island capture + launcher fallback to
  heal on large maps, "elixir far too slow in the rush meta", a well-assigning system that balanced harvesting.
- Producing Perfection (1st): no post-mortem exists on battlecode.org; no public repo found (10 ls-remote guesses,
  3 searches). Known only second-hand: GitHub-Actions match runner (4 Musketeers 2.4.3), early healing island (Gone
  Fishin' finals snippet).

**Code read** (final packages; line counts are `wc -l` of the package):
- carlguo866/battlecode23-gonefishin `src/submit26_final` (3,229 lines incl. util): every file read in full.
- awesomelemonade/Battlecode2023 `src/finalBot` (21,632 lines, of which 14,509 is the generated
  FastQueueWithRemove): Launcher, LauncherMicro, Carrier, Headquarters, Communication, EnemyHqGuesser, WellTracker,
  IslandTracker, TryAttackCloud, Explorer, Pathfinding, Constants, RobotPlayer read in full; Nav/BFSVision/Util in part.
  Team name not found in the repo or searches. Also `run_matches.py`, `rust/` (file names only).
- IvanGeffner/battlecode23 `fortytwo` (27,934 lines, of which 21,019 is generated IfSorting): MicroAttacker,
  MicroCarrier, Launcher, Attacker, Carrier, Headquarters, Spawner, Explore, Robot, HQTracker, Destabilizer, Booster,
  Amplifier, AdjacentTiles, VisibleTilesManager read in full; OtherComm, MapData, BugPath, BFSManager in part.
- maxwelljones14/Battlecode2023 `src/MPWorking` (23,913 lines, ~10k generated BFS/Comms/FastSort): Launcher, Carrier,
  Headquarters, Util read in part (the post-mortem covers the rest); gen_*.py, run_matches.py, get_record.py, snap.py.
- vrangr1/BattleCode2023 `src/AFinalsBot` (32,264 lines, 24k generated per-unit pathing): BotLauncher, BotCarrier,
  BuilderWrapper, Comms, Symmetry, ElixirProducer in part; CHANGELOG.md, TODO.md, autoscript.sh, scrim notebook.
- georgezhang02/battlecode23 `src/FB_ZZZ` (7,501 lines): Launcher, HQ in full; Carrier, Comms, Database in part.
- jmerle/battlecode-2023 `src/camel_case_v30_final` (1,514 lines): all read; `scripts/compare_fast.py`.
- AnOvercookedFork/battlecode23 `src/quals`: Micro.java in full; optimizer/optimizer.py.
- NotLLeon/battlecode23 `src/v7`: Launcher, Headquarters skimmed.
- The-4-Musketeers/UniC-stable is not a Battlecode repo (a college-app web app); ignored.

**Where our final bot stands** (progress/ladder.md 2026-10-20, progress/ELO.md, HANDOFF.md): team vibe23 is
**#11 of 88** on the galaxy replica (displayed rating 1695.6; carlguo866 1879, awesomelemonade 1844, IvanGeffner
1835, maxwelljones14 1796, vrangr1 1743, AnOvercookedFork 1743, pranayagra 1724/1701, georgezhang02 1708, jmerle
1704). Bradley-Terry: c_gd1 2046 +- 43, rank 11 of 140 players, 5th among field bots behind awesomelemonade 2127,
carlguo866 2123, maxwell 2117, Ivan 2101; field score 92.2%, 39.9% against the four above. c_gd1 = src/c_gd1
(4,981 lines): mana-first swarm, parity-hold micro, siege ring, island mending, bitboard distance fields, no
amplifiers, no elixir, no blind fire. HANDOFF "Why we lose": the mid-game production race (carriers 3-5 per 100
rounds vs their 10-12), home carriers dying to enemy launchers by r150, launcher wars lost at parity, late islands.

---

## 1. Ideas we never figured out

### 1.1 Macro strategy and win condition

- **Launchers go first to the ENEMY's home mana well, not to the enemy HQ.** fortytwo `Attacker.getTarget()`: while
  `!manaChecked` the target is `explore.mapdata.getBestEnemyWell()` = the symmetric image of the mana well our own
  carriers reported for the nearest HQ (`MapData.getBestEnemyWell`, ENEMY_MANA_CHANNELS, one per HQ); `checkMana()`
  clears it only once the launcher stands within d2 4 of it. Then: remembered enemy launcher (10 rounds) > broadcast
  enemy launcher within d2 100 > nearest enemy carrier (lowest HP) > enemy island > enemy HQ > explore. MPWorking
  `Launcher.updateSymManaSectors`/`chooseSymmetricLoc` (MAX_SYM_MANA_SECTORS 2) and 4 Musketeers 2.5.5 ("the enemy
  can't get any mana, and they won't be able to build launchers"). vrangr1 `BotLauncher.runLauncher`: after r30 a
  launcher that cannot move sets `currentDestination = findNearestEnemyWell()`.
  *We did*: `Launcher.pickObjective` targets sightings, enemy islands, then raid wells only within RAID_R2 100 of the
  SIEGE HQ and only once symmetry is decided; c_raid1 (enemy mana wells at any distance) dropped at 19/40 study
  cells, c_hunt1 rejected (panel net 0), c_cp1 OPEN_CAMP built as a sweep build and never trialled (ATTEMPTS).
  *Why it mattered*: diag-top2 M1: by r100 our carriers died 105 times to their 17, 65 of them at home mana wells,
  and we built 210 launchers to their 407 in r101-250. Their opening army is a carrier-killing raid on the well we
  will mine from, timed by the same symmetry guess we already compute.

- **The first anchor is a healing station, placed early and near home; conquest comes later.** Gone Fishin'
  `Headquarter.tryBuildAnchor`: "allow for a single early anchor for healing" when `getClosestFriendlyIslandIndex()==0`,
  r>150, hqid 0 and robots/HQ > 15; later anchors when (robots-carriers)/HQ > 18 and carriers/HQ > 10, with interval
  `6500/(islands+30)*sqrt(numHQ)` rounds (191 at 4 islands, 98 at 36). Their `Carrier.anchor()` scores islands:
  +1000 neutral, -2000 on the enemy side, -5*|dist to our HQ - dist to theirs| ("centerness"), -dist to me, ignore
  penalties. awesomelemonade `Headquarters.action`: anchors as soon as 50-round mean mana income >= 5.99 (first),
  7.5 (second), 8.99 (up to 5), else from robotCount > 0.25*area or r1800. The Gone Fishin' finals snippet names this
  as how Producing Perfection beat them.
  *We did*: ANCHOR_START 400 after 20 launchers, blind anchors from r400, gate on island deficit until r800
  (C.java); MEND only after ANCHOR_START. diag-top2 V8: they hold a first island at median r300, we at r470; diag-AL:
  AL heals 27.4% of damage taken vs the S4 bots' 9.9%, and our damage per kill rises 197 -> 289 after AL's first anchor.

- **Heal thresholds are high and healing is a group decision.** Gone Fishin' `Launcher.tryIslandStuff`:
  HEALING_CUTOFF 121 HP (heal below 121, 6 hits short of full), leave only at full HP AND (`ourTeamStrength >= 3` or
  50 rounds since the last attack); a failed heal (island lost) is blacklisted 100 rounds. awesomelemonade: retreat at
  100 HP, heal to 200, keep micro'ing from the island tile (`tryMoveToHealAtIsland` runs `executeMicro()` while
  standing on it). MPWorking `shouldHeal`: <= 60 HP, or <= 100 HP when ANY visible ally launcher is <= 60 ("stick
  together and come back to fight later", 4 Musketeers 2.5.2); heal spot scored +1000 cloud, -1000 occupied,
  lattice parity, -distance (`Util.getBestHealLoc`).
  *We did*: MEND_HP 80, MEND_FULL 180, a lone decision (C.java c_mend1 block); c_lx12 MEND_HP 120 "within noise"
  in self-play (ATTEMPTS), never on the panel.

- **Endgame banking for the r2000 tiebreak.** Gone Fishin' `Headquarter.run`: no launchers after r1900 once
  robots/HQ > 30 ("accumulate mana for tiebreaker"), and no launchers at all while no enemy was seen for 5 rounds
  and robots/HQ > 100; `tryBuildLauncher` only spawns if `maxSpawn*200 + strength >= 0` (do not feed). jmerle
  `Headquarters.run`: no launchers after r1750. awesomelemonade: after r1800 (or robotCount > 0.25*area) anchors and
  carriers first, launchers only with fewer than 10 attackers nearby, "save mana for anchors and tiebreakers".
  *We did*: c_bk1 BANK_FROM parked ("inert in self-play", HANDOFF). Tiebreak order is islands > anchors > Ex > Mn > Ad
  (RULES 8.4); 4 of the 31 diag-top2 losses were tiebreaks (class C).

- **HQ build decisions use income rates and spawn-space, not resource thresholds.** awesomelemonade `Headquarters`:
  FastIntTracker 50-round income per resource and per-miner yield; `getNewTask` targets 0.9 mana share by projected
  income; `tryBuildCarrier` refuses when carriers fill half the passable squares within r2 5 (`numPassableSquaresNearby`)
  or no deposit square exists. Gone Fishin' `Headquarter.sense` congestion: friendlies > passable/2 (and >10), or
  spawn tiles > half occupied, or robots/HQ > 80 -> stop carriers, broadcast a congestion bit; "seeing an enemy
  immediately decongests".
  *We did*: carrier cap by census (CAP_PER_MN_WELL 8, CAP_MAX 45), spend-everything loop (HQ.build); our audit found
  MassiveL gridlock and Potions corridor plugs (2026-10-20-audit-c_gd1.md) that a congestion sensor would have caught.

### 1.2 Economy

- **The HQ hands every new carrier its well at spawn.** fortytwo `Headquarters.constructCarrier` picks
  `MapData.getBestAdamantium(8)`/`getBestMana(8)` (free slots, carriers already sent), spawns toward it
  (`Spawner.greedyToTarget`, up to 2 carriers/turn) and writes `(x,y,type)` in its own channel
  (`HeadquarterComm.broadcastTarget`); the carrier reads `targetOnSpawn` on its first turn. awesomelemonade
  `Communication.CarrierTask`: 16 slots of (7-bit offset from HQ, 2-bit HQ id, 4-bit task) rewritten every turn for
  empty carriers in vision; `Headquarters.assignTasks` keeps per-resource miner trackers. georgezhang02
  `Comms.writeHQCommand`; 4 Musketeers 2.4.1 ("one carrier per round so the flag is unambiguous").
  *We did*: `Carrier.swarmRole` by `id % AD_EARLY_EVERY` plus an HQ "wants Ad" bit; `gather()` takes the nearest known
  well, with probes (c_well3-9) and hot-well marks bolted on. Why it mattered: TRAINING_LOG 2026-10-16 05:52 "carriers
  walk empty 0.36 of turns vs 0.10"; MassiveL's (2,0) well never mined in 6/10 games (C.java c_well3 block).

- **Per-HQ nearest-well slots instead of a global first-come list.** awesomelemonade `WellTracker`: 12 shared slots =
  4 HQs x {Ad, Mn, Ex}, each the well NEAREST THAT HQ, with a pending->known promotion when a nearer one is seen; a
  carrier reads only its own HQ's slots (`lastHqIndex`). Gone Fishin' `Comm.reportWells`: 5 wells per resource,
  replacing the one farthest from any friendly HQ. jmerle `SharedArray`: 5 mana wells per HQ.
  *We did*: 16 global slots, first free slot claims (`Comms.registerWell`), so a far well published first stays the
  "nearest known" (the (0,39) vs (2,0) failure that motivated three probe builds).

- **Mana/Ad ratio chosen from map area, round and enemy distance.** Gone Fishin' `Carrier.resumeWork`: area <= 1000:
  mana only to r150 then 1 in 5 Ad; <= 1700: 1/5; <= 2600: 1/4; else 1/3 (by a shared carrier counter). MPWorking
  `getNextCarrierType`: first 4 carriers 4/0 on small maps, 3/1 semi-small, 2/2 only if the estimated enemy intercept
  time (distance + 15) exceeds a carrier cycle (40 + 1.5*well distance), then 0.7/0.3 by a 75-round live tracker.
  vrangr1 `BuilderWrapper.updatePrioritizationRatio`: 4:1 before r240 on small maps, 3:1/1:1/2:1 phases on large.
  fortytwo `shouldSendToAdamantium`: `manaSent - 3*adamantSent` with map-type cases. no-thoughts: 4:1 small / 1:1 big
  before r60, 9:1 after.
  *We did*: AD_EARLY_EVERY 8, AD_LATE_EVERY 4 from r200, the same on every map (C.java); abl3 found the knobs "neutral"
  in self-play, and diag-top2 concluded "Ad is not a loss root". The map-size switch is what we lacked, not the ratio.

- **Carriers are weapons: throw when the group is weaker, or when the throw lands.** Gone Fishin' `Carrier.senseEnemy`:
  strength = sum of ally launcher HP within r2 16 of the enemy minus enemy HP; if negative, attack if in range and run.
  `runaway()` scores the 6 back directions: -1000 within r2 20 of the enemy, -1000 within 16 of where it was last
  sensed, +3000 if it can still hit with >= 16 Ad, +3000 if adjacent to the HQ with >= 20 Mn (deliver before dying),
  then deposits or attacks after the move. fortytwo `Carrier.shouldAttack`: throw when damage >= 10 and HP < 40, or
  when carrying <= 5 Mn and damage >= 20; `MicroCarrier` picks the 9-tile step minimising launchers within r2 20.
  awesomelemonade `Carrier.willDie`/`shouldAttack`: throw when summed launcher damage >= HP (and < 20 allies) or >= 8
  enemies in view. Don't @ Me and 4 Musketeers 2.3.1 describe the same rule.
  *We did*: throw only at HP <= 60, d2 <= 9, load >= 8 (`Carrier.survive`); c_gren1/2 parked. A 40 kg throw deals 50
  (RULES 5.4): three throws kill a carrier, four a launcher.

- **Deposit a partial load when threatened and the HQ is closer than the enemy.** awesomelemonade
  `Carrier.tryMoveToTransferWhenNotFull`: >= 15 Mn, an attacker seen within 30 rounds, HQ nearer than the enemy or
  not in the enemy's direction -> walk to the HQ and transfer. vrangr1 DOING_EARLY_MANA_DEPOSITION (threshold 15).
  *We did*: `survive()` walks home but `deliver()` runs only when `returning` (30 kg) or after r1900.

- **Box out enemy carriers at a shared well.** MPWorking `Carrier.doMine`: when friendlies outnumber enemy carriers
  and an enemy carrier is "searching for a spot", move to the collect square nearest it (`Util.getCollectLocClosestTo`);
  4 Musketeers 2.5.4. *We did*: nothing; c_mid1's launcher squad at the site was rejected.

- **Mine in a pattern that lets others in; heavy carriers only step toward home.** Gone Fishin' `Carrier.collect`:
  move among `MapRecorder.getMinableSquares` (excludes squares a current blows off, and Ad squares adjacent to a Mn
  well), yield to an adjacent teammate under 39 kg, carriers over 30 kg move only on round%5 or toward the HQ; fortytwo
  `shuffleMine`: < 10 kg move AWAY from base, <= 30 stay, > 30 move toward base. awesomelemonade carriers travel on a
  checkerboard: `Pathfinding.predicate = (x+y)%2==0` going to the HQ, `==1` going to a well (lanes, no counterflow).
  *We did*: c_pack1 PACK (nearest free collect square) accepted; nothing about lanes or load-ordered positions.

### 1.3 Combat micro

- **Kite out of VISION, not out of threat count.** Gone Fishin' `Launcher.kite`/`chase`: pick the direction minimising
  the number of enemy launchers that can SEE the destination (r2 20, or 4 when either side is in a cloud); ties: a
  non-cloud tile, then an orthogonal move ("from Cow: prefer non-diagonal moves to preserve formation"); always kite
  after attacking (post-mortem 5.2: "always kiting back after an attack is better, corroborated by Super Cow Powers").
  awesomelemonade `getScoreForKiting`: -2,000,000 per attacker within vision, +10,000 x distance to the closest,
  -100 x distance to the nearest ally, +50 diagonal; `getScoreWithActionSingleEnemyAttacker`: +20M not entering a
  cloud, +10M can attack, -5M in HQ aura, +2M stand still, +30k x distance from enemy, +10k straight. fortytwo
  `MicroAttacker.MicroInfo.isBetter` is lexicographic: can move > not cloud > fewest launchers in ATTACK range minus
  canLandHit > fewest in VISION range minus canLandHit > canLandHit > nearest ally > (in range ? farther : nearer).
  4 Musketeers 2.5.5: "go directly outside the vision radius of the closest enemy".
  *We did*: `Launcher.fight` scores 9 tiles by threat = fighters that can hit NEXT turn (r2 26), + edge-of-range, and
  the parity hold (C.java MICRO2); kited launchers end at d2 17-26 where they are still seen. Why: vision 20 vs attack
  16 means a tile at d2 17-20 is the zone where you see and are seen but nobody shoots; d2 >= 21 breaks contact.

- **Engage on allies that can SEE the enemy vs unique enemies seen in the last 6 turns, discounted by kill shots.**
  awesomelemonade `LauncherMicro`: a StringBuilder of enemy ids with per-turn markers (ENEMY_HISTORY_LENGTH 6);
  `getMicroDirection` advances when `allies within r2 20 of the enemy + 1 >= uniqueEnemies - (1 if any enemy is one
  shot)`, otherwise stands still if `allowedToStandStill` (not in attack range and every tile the enemy could step to
  toward us is covered by an ally) else kites; with no enemy in sight but history > 0 it moves toward the nearest ally
  only if that also closes on the last enemy location. fortytwo keeps `closestEnemyLauncher` for TURNS_MEMORY_TARGET 10
  and stands still (TURNS_STATIC 6) after losing sight, moving only on odd rounds while the memory lives; a broadcast
  enemy launcher within d2 52 that it cannot see still counts as a threat in `MicroAttacker.doMicro`.
  *We did*: ahead = allies in OUR vision + 1 - enemies in our vision >= 2 (`fight()`); c_gh2 (2-round memory) failed
  the screen, c_gx1 GHOST pending, c_ks1/c_ks2 kill-step dropped. diag-AL: AL's step-in hits kill 0.157 vs our 0.080.

- **Focus fire by hits-required-for-the-group, cloud-aware.** Gone Fishin' `getImmediatelyAttackableTarget`: for each
  enemy in range, count allies that can also hit it (within r2 16, or 4 if either is in a cloud), hitRequired =
  ceil(health / (20 x count)); lowest wins, ties nearest; a one-shot target returns at once. awesomelemonade
  `getImmediateAttackScore`: launcher +1M, killable +100k + health x 100 (the HEALTHIEST killable, so the kill is
  not wasted on an already-dead one), else -health x 100. *We did*: lowest HP then lowest id (`Launcher.shoot`).

- **Chase visible-but-out-of-range enemies when stronger, re-sense and re-micro after the move.** Gone Fishin'
  `Launcher.run`: sense -> micro -> macro -> if action still ready: sense, micro again -> tryGuessAttack; `micro()`
  chases when HP > target HP or `ourTeamStrength > 2` or the target is not a launcher, else kites. georgezhang02
  `findMovementCombat`: step in only if the target is killable, an ally already covers it, or (at full HP) allies that
  can see the target >= enemies or >= 2. 4 Musketeers 2.5.5: "after moving we may see a better target, switch".
  *We did*: no chase; `shoot(); fight(); shoot()` re-senses only after a move.

- **Conditional grouping on the healthiest launcher, only right after combat.** Gone Fishin' `Launcher.macro`: with
  fewer than 3 friends within r2 8 and an attack in the last 10 rounds, follow `groupingTarget` = the visible launcher
  with the most HP (ties: nearer the enemy HQ), cached 6 rounds; a weaker launcher adjacent to it stops to let the
  healthier one lead. fortytwo `GreedyStep.isBetterThan`: when a step does not get closer to the target, prefer the
  step nearest an allied launcher. *We did*: C.ARMY followed the lowest-id leader with GROUP_MIN 4 at all times; it was
  the largest drag we carried (c_loose1 +20 net; c_lb11/12 "much worse", LEARNINGS "Ablate what we carry").

- **Spawn launchers only when they will not be fed, and on the far side.** Gone Fishin' `tryBuildLauncher`: spawn
  up to 5 per turn only if `5*200 + (ally HP - enemy HP in vision) >= 0`; with an enemy in view, spawn 4 tiles on the
  opposite side; later launchers cluster at the centroid of this turn's spawns. fortytwo `Spawner.constructDanger`:
  with enemy launchers in view, only tiles farther than r2 26 from every one of them. *We did*: SPAWN_SAFETY picks the
  tile fewest fighters reach, but still spawns one launcher at a time whenever Mn >= 45 (HQ.build); diag-top2 V5: 532 of
  948 launchers in lost games were born besieged with a median life of 4 rounds; c_siege1 (batching) net -3.

### 1.4 Navigation

- **Pick the bug-nav hand by simulating both.** Gone Fishin' `Unit.getTurnDir`: on hitting an obstacle, simulate up to
  15 wall-following steps turning left and 15 turning right on the recorded map (3,000-bytecode cutoff), choose the side
  with fewer steps + Chebyshev remainder; the choice is frozen 40 rounds after combat and 100 after a map-edge flip.
  4 Musketeers 2.1.2: obstacle "points of interest" by a double BFS spread over turns (15,000 bytecodes for a 20-wall
  obstacle). *We did*: hand from the robot id, flipped after 24 stalled moves or a 2(W+H) budget (`Nav`), plus the
  distance field (c_nav3/4). Our Target/dead-end problems (HANDOFF "Navigation") were solved by the field, so this is
  a cheaper alternative rather than a gap.

- **Mirror the whole map memory once symmetry is known.** fortytwo `OtherComm.fillSymmetry` writes the mirror code
  (currents transformed) into every unseen tile of `MapData.mapInfo` incrementally at end of turn, so BFS and bug nav
  know walls the robot never saw. *We did*: `Field.think` ORs mirrored wall rows into a field (C.java FIELD_*), but
  `MapMem.tile` itself is never filled, so greedy steps and `Nav.blockedByWall` stay blind.

- **Danger zones around PREDICTED enemy HQs before they are seen; currents applied to the end square.** fortytwo
  `AdjacentTiles.computeSymDanger` marks the 29-tile aura of each symmetric HQ image as impassable once a symmetry is
  known, and `DirInfo.update` applies the current push (twice) before testing danger; Gone Fishin'
  `MapRecorder.reportEnemyHQ` writes WITHIN_HQ_RANGE into the map so pathing treats the aura as wall but blind fire
  still targets it; jmerle `Unit` adds RANGE16 around enemy HQs to `dangerousLocations`. *We did*: `Nav.aura` only for
  SEEN HQs, and the audit confirmed AURA_PUSH ignores the current push (audit-c_gd1).

- **Current rule: a current may be entered only if it points within 45 degrees of the move, unless a carrier under
  13 kg.** Gone Fishin' `MapRecorder.check`; no-thoughts post-mortem (Navigation); jmerle `Unit.tryMoveTo` prefers
  current tiles that carry toward the target. *We did*: `Nav.score` at the push destination and `pushesBack`; close.

### 1.5 Communication

- **Buffer pool: read the 64 ints at turn start, write only dirty ints at the end.** 4 Musketeers 2.2.2 and
  `gen_buffer_pool.py` (64 unrolled reads, dirty flags); Gone Fishin' `Comm.turn_starts`/`commit_write` with
  `buffered_share_array` + a FastIterableIntSet of changed indexes, and `writeBits(bit, len, value)` at arbitrary bit
  offsets (821 bits used); vrangr1 `Comms` keeps up to 20 unwritten locations with priority overwrite. *We did*:
  direct `readSharedArray` per field and ad-hoc writes guarded by `canWrite` (`Comms.put`); writes cost 75-100 each.

- **Enemy sightings expire in 4-12 rounds, are erased when a robot sees the spot empty, and are one slot per HQ.**
  Gone Fishin' `Comm.updateEnemy`: 10 slots with a 2-bit round stamp; HQ 0 clears the stamp two rounds back; any
  writer that can sense a slot's location erases it. fortytwo `OtherComm.checkEnemyComms`: one channel per HQ, 4-bit
  round, cleared when stale or seen empty. awesomelemonade: 10 slots, 4-bit turn, 12-round expiry, carriers write at
  most 3 per turn. *We did*: 4 slots with 16-round stamps, expiry after 3 stamps (48-63 rounds), cluster-merge within
  r2 20 (`Comms.reportEnemy`); launchers chase stale sightings for a minute.

- **Amplifier liveness heartbeat and amplifiers as moving writers for the army.** Gone Fishin' `Comm.amplifierUpdate`
  writes `4 + round%4` per amp; HQ 0 resets a slot whose stamp lags by 2; one amp per even HQ after r300, every 200
  rounds, following the launcher centroid at >= r2 20 from enemies and reporting islands (`Amplifier.java`). fortytwo
  builds an amplifier when `16 + 24*amps <= launchers`. (4 Musketeers 2.5.3 could not make amplifiers pay; Gone Fishin'
  snippet: "did not win games decisively".) *We did*: no amplifier is ever built (HQ.build); c_amp2/c_amp3 failed the
  screen, c_guard2 sentry held. A wash across the field, so low priority.

### 1.6 Symmetry and exploration

- **Spiral exploration from the HQ over UNSEEN tiles, and disintegrate when done.** Gone Fishin' `Carrier.setScoutTarget`:
  targets on the spiral r = theta (step pi/6, direction by id parity) that the MapRecorder has not seen; a carrier that
  finishes the spiral with no well calls `rc.disintegrate()` to free the HQ's space. fortytwo `VisibleTilesManager`:
  per-tile last-seen round, random targets least recently seen; `Explore.getEmergencyTargetClose` within 12 tiles of
  the Ad well before r100. *We did*: random points 8-15 tiles from home, a centre-first leg for mana (`Carrier.explore`);
  c_expl1 frontier grid rejected (-3), SoundWave explore share 0.71 (diag-top2 V10).

- **Prefer the rotational guess only on small 1-HQ maps; invalidate whole symmetry classes from one empty prediction.**
  awesomelemonade `EnemyHqGuesser`: 3 predictions per HQ in a bitfield, `isSymmetryPossible` drops a class when any of
  its predictions is invalid or a confirmed HQ is outside it; `getClosestPredictionPreferRotationalSymmetry` halves the
  score of rotational candidates only when area <= 625 and 1 HQ. 4 Musketeers 2.4.2 same invalidation. *We did*: the
  same elimination on tiles and HQ sightings (`MapMem`); comparable (see 3).

### 1.7 Bytecode and engineering

- **Generated code everywhere.** 4 Musketeers: `gen_comms.py` from a schema (adapted from smite 2022), `gen_pathing.py`
  / `gen_cooldown_pathing.py` (unrolled BFS at radii 34/20/10 chosen by bytecodes left, cooldown-aware),
  `gen_sorting.py`, `gen_buffer_pool.py`; fortytwo `IfSorting` (21k lines of sorting networks) and unrolled BFSGreedy;
  awesomelemonade `BFSVision` ("AUTOGENERATED") and Rust tools (`bc23-fastsort`, `bc23-mapconvert`); vrangr1 3.6-5.6k
  line per-unit pathing classes; no-thoughts generated Bellman-Ford; georgezhang02 `BFPathing20` (2,200 lines).
  *We did*: hand-written loops with bytecode guards; the bitboard field is our one "clever" structure. Our launcher
  spent a mean 7,200 bytecodes a turn (TRAINING_LOG 2026-10-09) and overran until c_bc1's cuts.

- **Zero-cost big arrays: a 3,600-char constant String turned into char[].** Gone Fishin' `Constants.MAP_LEN_STRING`
  -> `MapRecorder.vals`; awesomelemonade `PassabilityCache` as a StringBuilder and `LauncherMicro`/`Nav` using
  `StringBuilder.indexOf` as an O(1)-bytecode set. *We did*: `MapMem.alloc()` pays W*H bytecodes lazily.

- **Offline micro trainer (inferred).** awesomelemonade `rust/microtrainer/` contains `arena.rs`, `simulated_annealing.rs`,
  `visualizer.rs`, `micro/`: a standalone launcher-duel simulator used to tune micro weights by annealing (inferred
  from file names; not read). *We did*: c_sim1 (in-game simulation) failed the screen; our knob races ran full games.

- **Local resign to shorten hopeless test games.** fortytwo `Robot.initTurn`: `rc.resign()` at r>400 with <= 4 robots;
  MPWorking `Headquarters.localResign` (r>=400, robots <= HQs). *We did*: none; TRAINING_LOG 2026-10-19 noted "9 of
  c_eco3's 11 wins were IvanGeffner resigning at r401" distorting an h2h, which is the flip side.

### 1.8 Tooling, testing and process

- **GitHub Actions as a match farm** (Producing Perfection, copied by 4 Musketeers 2.4.3): both sides x every map,
  summary table per run (`run_matches.py`, `matches-summary*.txt`, `get_record.py` for scrimmage records into a
  spreadsheet). awesomelemonade `run_matches.py` greps `FLAG{...}` prints so each game reports which code paths fired.
  jmerle `scripts/compare_fast.py`: asyncio-parallel engines, websocket to the server, outcome from flatbuffer replays,
  rich table. vrangr1 `autoscript.sh` (parallel A/B per map, greps "Warning", "Birth" freezes) and a notebook that
  auto-requests scrimmages against ladder teams by Elo through the official API. AnOvercookedFork `optimizer.py`: a
  genetic algorithm over constants (5 individuals, 100 rounds).
  *We did*: more than any of them (galaxy replica, gauntlet.sh, screen.py, swiss.py, race.py, param.py, paired.py,
  telemetry; 72 files in tools/). No gap here except the FLAG idea and the GA; no-thoughts' "Takeaways" names fast
  parallel A/B testing as the thing top teams had.

---

## 2. Ideas we tried but could not make work, that top teams made work

- **Blind fire into clouds and at remembered positions.** Every top bot does it: Gone Fishin' `tryGuessAttack`
  (cached enemy +-8 rounds and its 8 neighbours, then 10 random r2 5-16 offsets nearest the enemy HQ skipping recorded
  walls, then cloud tiles nearest the enemy HQ; 800 bytecodes); awesomelemonade `TryAttackCloud` (from a cloud, walk
  the shot toward the last enemy to the range edge); fortytwo `desperateAttack` (broadcast location, last seen <= 6
  rounds, cloud spiral); MPWorking `attackClouds` with a cooldown check so a cloud shot never costs next turn's action;
  jmerle `tryBlindAttack` RANGE_5_TO_16 away from home; NotLLeon `takePotshot`. Gone Fishin' also attacks
  `Comm.updateEnemy()`'s nearest reported enemy when it cannot sense it. *Ours*: c_blind1, c_blind2, c_cld2 rejected;
  c_ms1 (SHOOT_KNOWN + MEMSHOT) net +4 over two blocks but not accepted alone (HANDOFF); LEARNINGS 1.1 estimated ~1% of
  damage. Theirs fires at memory FIRST and clouds last, and MPWorking's cooldown guard avoids the cost we feared.

- **Fighter memory in the engagement count.** c_gh2 failed the screen (9/20), c_gx1 pending; awesomelemonade's 6-turn
  unique-id history and fortytwo's 10-round memory are the working forms (1.3).

- **Launcher cohesion.** c_army1/C.ARMY, c_grp1, c_lb11/12, JOIN_MIN all lost; Gone Fishin' grouping is conditional
  (fewer than 3 within r2 8 AND combat in the last 10 rounds) and follows the healthiest, not the lowest id (1.3).

- **Carrier survival.** c_flee1/4/9 failed the screen on overruns and 8/20 h2h; fortytwo `MicroCarrier` is a 9-tile
  lexicographic pick with a 2,000-bytecode guard, run in a loop while movement is ready; Gone Fishin' `runaway` is a
  6-direction score with deposit-if-adjacent (1.2).

- **Holding or raiding contested mana wells.** c_mid1 (-1), c_raid1, c_hunt1 (0), c_front1 failed; fortytwo and
  MPWorking make the enemy's well the launchers' FIRST target instead of a side squad (1.1).

- **Exploration for wells.** c_expl1, c_seam1/2, c_explore1 rejected; Gone Fishin' spiral over unseen tiles and
  fortytwo's least-recently-seen grid (1.6).

- **Siege-spawn discipline.** c_batch1, c_siege1 failed; Gone Fishin' spawns only when strength allows and away from
  the enemy, fortytwo only on tiles beyond r2 26 of every enemy launcher (1.3).

- **Amplifiers.** c_amp2/3, c_guard2 failed; Gone Fishin' and fortytwo field them, but 4 Musketeers and Gone Fishin's
  own report say they were not decisive, so this one is a draw.

- **Kill-step / focus.** c_ks1 dropped, c_ks2 not positive; Gone Fishin' hits-required and awesomelemonade's
  healthiest-killable scoring are the working forms (1.3).

---

## 3. Ideas we had that top teams also had (brief)

- Mana-first economy with a small Ad share (all); 30 kg loads (jmerle cargoTarget 30; Gone Fishin' optimal_mining.py).
- Symmetry elimination from tile features incl. transformed currents, and from HQ sightings (Gone Fishin'
  `MapRecorder.recordSym`, fortytwo `checkSymmetry`, vrangr1 `Symmetry`, awesomelemonade `EnemyHqGuesser`); our MapMem.
- HQ-aura avoidance and a siege ring outside r2 9; rotating the siege target between enemy HQs (Gone Fishin'
  `macro` advances `enemyHQID` when within d2 16 and nothing is seen; awesomelemonade blacklists an HQ that is not
  getting closer in 35 turns or has >= 5 allies within r2 20); our RING_MIN/MAX and SIEGE_ROTATE (c_hr1).
- Heal on owned islands (all); our c_mend2 (thresholds differ, 1.1).
- Spawn toward wells for carriers and toward the enemy/centre for launchers (jmerle, Gone Fishin', fortytwo); our
  `HQ.tryBuild`. Gone Fishin' `MapRecorder.hqInit` also excludes sealed spawn tiles ("get us out of jail"), our
  SPAWN_POCKET.
- Shoot - move - shoot and carriers moving twice (awesomelemonade `tryMultiAction/tryMultiMove`, fortytwo
  `while(attack())`); our `Launcher.run`, `Carrier.moveTwice`.
- Live carrier census in the shared array (Gone Fishin' 100-round counter; ours 64-round two-slot census).
- HQ raid alarm recalling nearby launchers (awesomelemonade broadcasts the enemy's direction in the HQ slot,
  launchers defend within d2 257; ours ALARM + RECALL_R2 225).
- Switch carriers to mana when the HQ is pressed (MPWorking `shouldSwitchToMana`, awesomelemonade `getNewTask`;
  our AD_PRESS_MN).
- Predicted islands/wells by symmetry, anchor couriers that flee, neutral-only anchor targets (all).
- Per-robot RNG and bytecode guards on every loop; 9-tile micro scoring (fortytwo/AnOvercookedFork MicroInfo; our fight()).
- Well packing / stepping onto the well tile (ours c_pack1; theirs 1.2).

---

## 4. Sources

Post-mortems: https://github.com/maxwelljones14/Battlecode2023 (`4_Musk_Post_Mortem.pdf`);
https://github.com/reeceyang/battlecode-2023 (README);
https://battlecode.org/assets/files/postmortem-2023-gone-fishin.pdf and https://www.carlguo.com/pdfs/battlecode_report.pdf
(blocked; WebSearch snippets); https://battlecode.org/assets/files/postmortem-2023-dont-at-me.pdf (blocked; snippets);
https://battlecode.org/assets/files/postmortem-2023-no-thoughts.pdf; https://battlecode.org/past.html (placings).

Code (each `owner_repo` below is https://github.com/owner/repo):
carlguo866_battlecode23-gonefishin/src/submit26_final/{Launcher,Carrier,Headquarter,Comm,MapRecorder,Unit,Amplifier}.java,
src/scripts/optimal_mining.py; awesomelemonade_Battlecode2023/src/finalBot/{robots/Launcher,robots/LauncherMicro,
robots/Carrier,robots/Headquarters,util/Communication,util/EnemyHqGuesser,util/WellTracker,util/IslandTracker,
util/TryAttackCloud,util/Explorer,pathfinder/Pathfinding,RobotPlayer}.java, run_matches.py, rust/microtrainer/;
IvanGeffner_battlecode23/fortytwo/{MicroAttacker,MicroCarrier,Launcher,Attacker,Carrier,Headquarters,Spawner,Explore,
Robot,HQTracker,OtherComm,MapData,AdjacentTiles,VisibleTilesManager,BugPath}.java;
maxwelljones14_Battlecode2023/src/MPWorking/{Launcher,Carrier,Headquarters,Util}.java, gen_*.py, run_matches.py;
vrangr1_BattleCode2023/src/AFinalsBot/{BotLauncher,BotCarrier,BuilderWrapper,Comms,Symmetry,ElixirProducer}.java,
autoscript.sh, BlueDragonScrimRequestScript.ipynb, TODO.md, CHANGELOG.md;
georgezhang02_battlecode23/src/FB_ZZZ/{Launcher,HQ,Carrier,Comms,Database}.java;
jmerle_battlecode-2023/src/camel_case_v30_final/**, scripts/compare_fast.py;
AnOvercookedFork_battlecode23/src/quals/Micro.java, optimizer/optimizer.py; NotLLeon_battlecode23/src/v7/{Launcher,Headquarters}.java.

Ours (https://github.com/terryvanbelle/battlecode23-vibe): src/c_gd1/*.java (C, HQ, Launcher, Carrier, Comms, Nav, Field,
MapMem, Mid, Amplifier, G, RobotPlayer); HANDOFF.md ("Standing", "Why we lose", "In flight"); LEARNINGS.md (1.1-1.3,
"Process and gates"); progress/ATTEMPTS.md; progress/ladder.md; progress/ELO.md; TRAINING_LOG.md (2026-10-09 to
2026-10-20 entries cited inline); research/diagnosis/2026-10-19-top2.md, 2026-10-19-awesomelemonade.md,
2026-10-20-audit-c_gd1.md; RULES.md sections 1, 2, 5.4, 8.4.
