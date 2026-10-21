# MISSED_IDEAS_2022 — Battlecode 2022 ("Mutation") — ideas the top teams used that we never figured out

Written 2026-10-21, post-hoc comparison. Our bot: https://github.com/terryvanbelle/battlecode22-vibe (practice run
2021-12 to 2026-09, first practice year, 128 logged iterations, 62 accepted snapshots).

## 0. Header: what was read, and where we stand

**Post-mortems**
- *5 Musketeers — Battlecode 2022 Strategy Guide* (David Lyons, 7th-8th at finals,
  `battlecode.org/assets/files/postmortem-2022-5-musketeers.pdf`): **read only via WebSearch
  snippets** (battlecode.org is blocked from this sandbox; the local copy our project once had
  is gone) plus the two verbatim passages our own `TRAINING_LOG.md` Iteration 89 quoted when it
  still had the file (anomalies "nothing you could do about"; sages "assassinate the soldier from
  the shadows"). Also the author's project page summary (maxwelljon.es/projects/battlecode):
  "stack to store states", "distributed k-clustering algorithm for troop movements ... at most
  three main enemy clusters", soldier micro weighing "troop counts, health, cooldown, and land
  passability".
- No other 2022 post-mortem exists on battlecode.org. smite's README says "Write-up Coming Soon!"
  (never published); TestSubjector's says "Postmortem may or may not happen". Both have in-repo
  notes/CHANGELOG that were read in full.

**Bot code read (shallow clones of the public repos)**

| Team / repo | Package read | Lines | How much |
|---|---|---|---|
| wololo (Isaac Liao, **1st**), `iliao2345/Battlecode2022` | `src/fury_fix_20` (highest-numbered = final) | 4,704 (of which `Bfs.java` 605 generated) | all 19 files read in full |
| camel_case (jmerle, 13th-16th; our `sample_camelcase`), `jmerle/battlecode-2022` | `src/camel_case_v25_final` | 1,400 hand-written + 13k generated Dijkstra | all hand-written files read |
| Do Lonely Laboratories Dream of Golden Anomalies (TestSubjector, finalist, "Most Adaptive Strategy"; our `sample_afinals`), `TestSubjector/BattleCode2022` | `battlecode22-scaffold/src/AFinalsBot` | 7,210 (incl. 1,493 generated BFS) | Archon/Miner/Soldier/Sage/Builder/Lab/CombatUtil/Globals full; Comms/Movement/Explore/Watchtower skimmed; NOTES.md, CHANGELOG.md, SCRIMS.md, autoscript.sh |
| smite (mvpatel2000 et al., finalist), `mvpatel2000/Battlecode2022` | `src/athena` (README: "finals submission") | 23,871 (of which ~15k generated pathing + 2.6k generated comms) | Archon/Robot/Miner/Builder/Lab/Soldier/Sage/Watchtower full; CommsHandler schema; `notes/micro.txt`, `notes/jan 16 game analysis`; `scripts/*.py` |
| 5 Musketeers (BSreenivas0713, 7th-8th), `BSreenivas0713/Battlecode2022` | `src/MPTempName` (teamA in gradle.properties; largest/latest package) | 7,385 + `bfs/BFSUnrolled*.java` (~5k) + `fast/*` | Archon/Soldier/Sage/Miner/Robot/Builder/Laboratory full; Comms/Explore/Nav skimmed |
| `awesomelemonade/Battlecode2022` (team name not recoverable; README is stock) | `src/bot` | 8,335 (of which 6.2k generated BFS) | all hand-written files read; `run_matches.py`, `generate.py` |

Probed and absent: `iliao2345/battlecode22`, `maxwelljones14/Battlecode2022`, `The-4-Musketeers/Battlecode2022`,
any Producing-Perfection 2022 repo under the obvious names.

**Where our final bot stands** (`TRAINING_LOG.md` Iterations 126-128; `progress/vs_old_bots_history.csv`)
- Final accepted snapshot: `src/g_iter62/` (Iteration 127, 438/720 = 60.8% on the 36-peer Gauntlet).
- Benchmark tally at the last check (Iteration 126 shipped, re-confirmed through 127):
  **`sample_camelcase` 0/20 (0%)**, **`sample_afinals` 3/20 (15%)** — camelcase has been 0/20 for
  the whole project; afinals plateaued at 3/20 over five consecutive checks (one 1/20 reading on a
  rejected variant was judged noise).
- Last rows of `vs_old_bots_history.csv` (g_iter58 vs its own history): 80% vs g_iter11, 55% vs
  g_iter21, 60% vs g_iter31, 50% vs g_iter41, 50% vs g_iter51 — i.e. the bot stopped improving
  against its own lineage about 20 snapshots before the end.
- Code: one 1,772-line `RobotPlayer.java` plus a vendored 2,388-line `Dijkstra20.java` (camel_case's
  generated unrolled Dijkstra, radius² 20, used by *every* unit type).

---

## 1. Ideas we never figured out

### 1.1 Macro strategy / win condition

**Lead farming by disintegrating cheap units onto empty tiles (the dominant late-game economy).**
In 2022 a dead robot drops lead on its tile, and any tile holding ≥1 lead regenerates +5 lead every
20 rounds forever. Top teams turned this into an infinite income source: build a 40-lead Builder,
walk it to a tile with 0 lead, `rc.disintegrate()` — the tile now holds lead and regenerates for
the rest of the game. Evidence: wololo `fury_fix_20/Archon.java` (`farmer_rate`,
`desired_farmer`, `Action.buildFarmer`) and `Builder.java::act()` — a farmer builder walks a random
ray to the wall (`Math2.extend_to_wall`) and `if (rc.senseLead(Info.loc) == 0) rc.disintegrate()`;
smite `athena/Builder.java::determinePurpose()` ("first builder is the main one ... everyone else is
a sacrifice"), `shouldDisintegrate()`, `move()` ("Lead farmers move to adjacent tile without lead
and disintegrate", preferring low rubble), and `Archon.build()` phase `"midgame farming"` which
keeps producing Builders while `numNearbyBuilders <= 5 && round <= 1800`; smite README: "demeter:
farming + improved build order (quals submission)", "refined farming w.r.t. lead bursts";
awesomelemonade `Builder.tryMoveToSeppuku()` (disintegrate on a 0-lead tile in an ally chunk after
40 idle moves); AFinals `BotMiner.lowHealthStrategy()` (a dying Miner walks to
`getClosestNonLeadLocation` near its Archon and disintegrates) and `toDieOrNotToDie()`; Musketeers
`Archon.trySacrifice()` (whole Archon disintegrates when ≥3 Archons). What we did: nothing — the
word `disintegrate` never appears in `src/bot/RobotPlayer.java`, and the only "regen" in our docs is
the single rule "leave 1 lead" (Iteration 1). Why it mattered: our LEARNINGS "Open questions" say
`sample_camelcase` out-produces us and that team lead "balloons to 6748+ unspent" late; a farm turns
that dead lead into permanent 5-lead/20-round income on the tiles *we* choose (next to the Archon,
on low rubble), which is exactly what the sage-spam late game needs.

**Archon "sacrifice" when far from the action (wololo, Musketeers).** wololo `Archon.java`
`enable_sacrifice = far_archon && health_to_heal==0 && !lacking_miners && robots+lead/50 < 10`; a
far Archon with nearby Miners and no Labs first demands a Builder and then `rc.disintegrate()`s —
dropping a 600-HP Archon's worth of lead onto farmable tiles and removing a liability (the win
condition is losing *all* Archons, so a 3rd Archon far from the fight is pure upside to cash in).
Musketeers `Archon.trySacrifice()` disintegrates the Archon farthest from the enemy cluster when
`count > 2` and `numMinersBuilt > 0` (present but commented out of `takeTurn()` in the final
package). What we did: Archon count was treated as sacred; Archon relocation (Iteration 34) only
walks *toward* the objective after round 500. (inferred) Our multi-Archon "shared-lead contention"
thread (Iterations 39-61, 23 iterations) was fighting the symptom of having too many producers.

**Lead-reservation / turn-queue across Archons instead of fairness-yield heuristics.** wololo
`Comms.java` slots 15-19: each Archon publishes `build queue max urgency`, a `did build` bit and the
`required amount` (40/50/75); `lead_needed_before_building` sums the requests of Archons more urgent
than itself (`Comms.update()`, the `if (Info.type == ARCHON)` block) so lead is *reserved* for the
most urgent Archon. smite `Archon.build()` writes `reservedLead/reservedGold` into shared slots
(`LEAD_RESERVE_SCALE=5`) and "passes" its build turn with probability `1/(archonsAlive −
archonsBehindMe)` ("Starvation solution: randomized passing"); Musketeers `Comms.canBuildPrioritized`
/ `canBuildPrioritizedEven` / `getArchonOrderGivenClusters` (the Archon closest to the enemy cluster
is "important" and gets priority on lead); AFinals `BotArchon.waitQuota()`
(`archonWhoShouldBuild = (round-1) % archonCount`); awesomelemonade `Communication.reserve()` with a
heartbeat bit. What we did: `SA_LAST_SOLDIER_BUILDER`/`SA_SOLDIER_HUNGRY` 6-round fairness-yield
(Iteration 61) after 10 rejected variants (`TRAINING_LOG.md` 39-61). Why it mattered: our
LEARNINGS call shared-lead contention a "turn-order dominance" problem; every top team solved it
with an explicit reservation protocol, not a cooldown.

**Explicit build-order *states* (INIT / CHILLING / OBESITY / UNDER_ATTACK / BUILDING_LAB) and
"obesity" = spend hoarded lead on buildings.** Musketeers `Archon.java` `enum State` with a
`stateStack`; `checkForObesity()` = `lead > 180 + 75*(archons+1−archonNumber)` switches to
`OBESITY` which runs `SoldierBuilderRatio(11, ...)` (one Builder per 10 Soldiers) and triggers
`BUILDING_LAB`; `enoughLeadForLab()` thresholds 200/300. smite: a 9-phase build order in
`Archon.build()` (first miners → first builder → *pause until first lab* → rest of miners →
miners by `resourcesOnMap` → wait for extra lab if `resourceRateEMA > 4` → soldiers →
farming builders) plus overrides. AFinals: weighted round-robin (`currentWeights[i] += 1/aBUWeights[i]`,
pick the smallest) with map-size-specific weights. What we did: a priority chain
`wantSage ? SAGE : needBuilder ? BUILDER : needMiners ? MINER : SOLDIER` (RobotPlayer.java
~l.690) with threshold gates that our own LEARNINGS admit repeatedly "never actually clear in a real
game". Why it mattered: LEARNINGS "Strategic insights" record 9,750 and 62,785 lead unspent in
single games; an obesity state is the generic fix.

**Income-rate estimation drives Lab count.** smite `Archon.updateResourceRate()` keeps an EMA
(`RESOURCE_ALPHA=0.04`) of the per-round lead delta that Miners accumulate into a shared slot
(`commsHandler.writeLeadDelta`, `Miner.mineNearbySquares`) and Labs subtract from
(`Laboratory.transmute`); `waitForLab = resourceRateEMA > 4 && (labs < 2 || turnsSinceLastLab > 40)`.
wololo `Info.update()` fits a decaying least-squares line to income (`income_didt`,
`income_estimate`) and sets `not_enough_laboratories = mining_miners*3 >= labs*4 || lead > 250`
(roughly one Lab per 1.33 working Miners). What we did: fixed surplus bars 1,500/3,000 lead for Labs
2/3 and `MAX_LABS=3` (Iterations 118/122); Iteration 121 concluded "the real bottleneck is gold
income rate itself" but never measured income.

**Spend gold the instant it exists, and gate Soldiers on gold (sage-first doctrine).** AFinals
`shouldBuildSoldier()` returns false when `gold > 19`; `shouldBuildSage()` = `gold >= 20`; Sage
weight 2.5-10 vs Soldier 0.5-5.5; `shouldTransmute()` keeps transmuting while
`minerCount > sageCount/2`. smite: "Final override: if we have gold, just make a sage" and halts gold
production (`ProductionControl.HALT`) only when `minerCount < min(4, 2+resourcesOnMap/100)` or gold
≥ 60 banked. Musketeers CHILLING/UNDER_ATTACK both start with `if (gold >= 20 && isPrioritizedArchon
|| gold >= 40) build SAGE`. wololo `try_to_build()`: Sage first whenever `gold >= 20 &&
most_urgent_archon`. What we did: Sage became top priority only at Iteration 115 (after a
"no contact for 60 rounds" gate was found to be permanently closed), and gold income stayed ≤64 per
game (Iteration 121) because of Lab count — see 1.2.

**Transmutation rate depends on how many friendly robots are near the Lab — place Labs where
nothing else is.** `rc.getTransmutationRate()` rises with nearby allies (GameConstants
`ALCHEMIST_LONELINESS_*`). wololo `Laboratory.update()`: `transmute_cost = floor(20 −
18·exp(−0.02·n_friendly))` and transmutes only when `transmute_cost < 6`; `Builder.java` places
Labs on the lowest-rubble tile not already within the vision radius of 2 existing Labs, keeps a
`lab_crowded` slot (shared_arr[31]) and the Builder/Miners *walk away* from crowded Labs. AFinals
`BotBuilder.findLaboratoryBuildLocation()` = nearest **map corner**; `BotLaboratory.moveIfNeeded()`
relocates to the lowest-rubble tile in vision and reads `getTransmutationRate()` every turn. smite
`Laboratory.shouldRequestUpgrade()` asks a Builder for Lab level 2/3 when `transmutationRate >= 6`
(mutate reduces the loneliness penalty). Musketeers `Laboratory.findNewSpot()` moves when a tile
≥20 rubble better exists. What we did: Builder walks 7 tiles straight out from the Archon then
builds (Iteration 64) — into the middle of our own unit traffic; `runLaboratory()` is
`if (rc.canTransmute()) rc.transmute()`; `getTransmutationRate` is never called. Why it mattered:
LEARNINGS "Open questions" blame Lab *throughput*; the rate was the hidden multiplier.

**Watchtowers as the lead sink, on a lattice, and relocating with the army.** camel_case
`Builder.run()` builds Watchtowers only on tiles with `|dx|%2==0 && |dy|%2==0` from the Archon (a
checkerboard lattice that never blocks movement) and `Archon.spawnOrder` spends 1 Builder per 3
Soldiers + 1 Miner whenever `lead >= 300`; `Watchtower.run()` transforms to PORTABLE when no target
is in range and *walks* toward danger targets / enemy Archons, transforming back when a target
appears. awesomelemonade `Builder.tryBuild` uses `(x+y)%2==0 && x%2==0`. Musketeers
`Builder.makeWatchtowerIfPossible()` builds on `Util.makePattern(archonLoc)` up to 13 towers on
small maps. What we did: Watchtowers only adjacent to the Archon (`me.isWithinDistanceSquared(home, 8)`),
one relocation per game after round 500, and LEARNINGS record that this self-blocks the Archon's
build ring. Why it mattered: our own metrics show camelcase reaching 143 Watchtowers in one game
(Iteration 109 note) vs our 3.

**Decide rush vs. economy from map size / enemy distance.** AFinals `Globals.SMALL_MAP = area <
799`, `MEDIUM_MAP < 3602`, `MAX_LABORATORY_COUNT = SMALL ? 1 : 4`, soldiers from turn 5 on small maps,
turn 15 otherwise; Musketeers `isSmallMap() = area <= 625`, `isSemiSmallMap() <= 1000`,
`numINITminers = 12/archons` on big maps else `8/archons`; wololo `send_initial_soldiers` fires
when the enemy is seen or there isn't lead for 2 Soldiers + 1 Miner; smite `initialMiners =
max(4, area/240 + 3)`, `maxMiners = area/36`. What we did: `richHome` (lead in Archon vision > 600)
is our only map-doctrine branch (LEARNINGS "Architecture patterns"). Enemy distance never enters any
decision.

### 1.2 Economy

**Miner "employment" as the Miner-production signal.** wololo: each Miner reports whether it is
`employed` (`self_lead_excess > 0`, i.e. visible lead minus 120 per competing Miner) into a per-Archon
slot (shared_arr[38:46]); Archons keep a 20-round rolling average (`employment[20]`) and build
Miners only while `summary_employment_rate > 0.7 || see_extra_lead`, and build **farmers** when
`see_extra_miners` (lead < 30·(miners−1) and employment < 0.5). What we did: quota =
`max(lead tiles adjacent, 5)` on turn 1, then a round-number floor `min(6+round/100, 25)`
(Iterations 16/20/23/101/110) — "bring Miners back to a target count" never knows whether the
existing Miners have anything to mine.

**Lead/rubble trade-off scoring for choosing which tile to mine.** AFinals
`findOptimalLocationForMiningLead`: `(lead+1)/(rubble+1)`; Musketeers
`getLeadDistTradeoffScore = lead − 5·dist − amountMinedBeforeWeGetThere` (subtracts what nearer
Miners will mine first); camel_case `Miner.tryMine` assigns 2 Miners per lead tile by
simulating every visible Miner's best choice (`minersRequiredByOption`); smite Miners **stand on
the lowest-rubble tile adjacent to the deposit** (`rubble*100 + dist`), wololo likewise
(`best_adjacent_loc`). What we did: `lead > bestLead` with a distance tiebreak, and a sticky
beacon (Iteration 35). We never choose the standing tile by rubble, so Miners mine from
high-rubble squares (action cooldown ∝ 1+rubble/10).

**Deplete lead to 0 only in enemy territory / when losing.** wololo `mine_until = (enemy attackers >
ours && archons equal) || enemy archons > ours ? 0 : 1`; smite `determineRequiredLead()`: `onOurSide ? 2 : 1`,
and 1 when an enemy Archon is visible, 2 within 49 of our own Archon; AFinals `checkIfToDepleteMine()`
(deplete if closer to an enemy Archon than 1/1.2× of ours); Musketeers `shouldDepleteUnitLead()`.
What we did: always leave 1 everywhere (`rc.senseLead(t) > 1`), which farms lead *for the enemy*
in their half of the map.

**Miners (and everyone) flee out of enemy *action* radius, not just on sight.** AFinals
`isSafeToMine()` tests `enemy.type.actionRadiusSquared >= dist`; `tryToBackUpToMaintainMaxRangeMiner`
moves to the tile maximizing distance to the nearest attacker without increasing rubble. smite Miners
keep a `fleeingCounter = 6` and kite 5 tiles away. wololo Miners use a *team-wide* warzone estimate
(`Comms.closest_attacker_loc`, drifting toward the enemy at `WARZONE_SPEED`) with
`WARZONE_BUFFER = 6` tiles. What we did: flee from any visible combat unit by one step
(`moveToward(me.add(threat.directionTo(me)))`), plus the Sage-specific `SA_SAGE_SEEN` patch
(Iterations 96/97).

**Mutation was a shared *request* queue, not an idle-Builder afterthought.** smite
`CommsHandler.builder_request` (type + x,y) written by Labs that want level 2/3, served by the
"main builder"; `Builder.buildOrHealOrUpgrade()` mutates anything when `lead >= 500`; Musketeers
`Comms.setArchonNeedsHeal` with Builders in `GOING_TO_HEAL`; AFinals `healArchon()` assigns a Builder
to each damaged Archon via comms flags. What we did: mutate only what happens to be in range of a
Builder that has finished its own two builds (Iterations 82/83/85); Archon level 3 (80 gold) never
fired (Iteration 121).

### 1.3 Combat micro

**Score every adjacent tile by enemies-that-can-shoot-it, rubble cooldown and heal benefit.**
This is the single most consistent technique across the top bots and the one we never built.
wololo `Micro.battle_tile_values_sparser()`: for each of 9 tiles, `n_enemy_shootable` (counting
enemy soldiers already "preoccupied" by a friendly as 0.9), `+2000·floor((1+rubble/10)·actionCooldown)`,
a small term toward the attacker centroid, `+100` if a healing Soldier would leave Archon heal range;
the min-value tile set becomes `illegal_tiles[][]` for `Pathing.approach`. smite
`Robot.combatKiteMove()`: `score = archonHealRate·10/(10+rubble)` if damaged, `− dmg·10/(10+enemyRubble)`
per enemy that can hit the tile, `− GAMMA(0.9)·dmg` per enemy that can *see* it, `+ myDamage·10/(10+rubble)`
if we can attack from it, 1v1 chase/flee term `(myHP·rubbleFactor − enemyHP)·(−dist)/1e4`,
`+ 10·rubbleFactor`; `notes/micro.txt` lists the factors. Musketeers `Soldier.chooseForwardDirection`
/ `chooseBackupDirection` (fewest enemies that can hit the tile, then rubble, never >10 more rubble
than now); awesomelemonade `tryMoveAttackingSquare` (`dist<=13 ? 0.5+dist/26 : 0` plus
`10/(1+rubble/10)`). What we did: `repositionForRubble()` (step to a lower-rubble tile still in
action range, Iteration 21) and `moveToward(target)` — distance only. LEARNINGS "Combat trade
efficiency" notes we never found a per-engagement lever; this is it.

**Attack-then-move vs move-then-attack ordering keyed to cooldown state.** wololo
`Soldier.act()`: `try_to_shoot(delay_shot = move_ready)` first — if a move is still available and the
only targets are non-combat units, *delay* the shot, move to the best tile, then shoot; smite
`Soldier.runUnit()`: `attack(); combatMove(); attack();`; Musketeers `moveAndAttack(dirs, attackFirst)`
with `attackFirst = shouldRunAway()`; camel_case `tryAttack()` repositions to lower rubble *before*
firing and `tryMoveToSafety()` *after* firing if damaged ("attack then kite"). What we did: attack
then `moveToward`; the kite-back-after-shooting step exists only for Sages (Iteration 116).

**Retreat decisions are local force comparisons, not HP thresholds.** AFinals
`retreatIfOutnumbered()` (TheDuck314 lineage): count hostiles that can hit me vs allies that can hit
the closest hostile; retreat to the vector sum of "away from each hostile" using
`tryHardMoveInDirection` (only onto lower rubble); `tryMoveToEngageOutnumberedEnemy` advances when
`allies >= hostiles || (1 hostile && myHP > itsHP)`; `tryMoveToHelpAlly` joins fights an ally is
already in. Musketeers `shouldRunAway()` = `HP<=6 || someone can hit me now || friendlies+1 < enemies
|| equal fight and my HP <= enemy HP−15 || fewer sages than they have`; wololo `Micro.update()`
`retreat_timer = 2` when `enemy attackers > friendly attackers + 1` or losing a 1v1, `winning_1v1_soldier`
chases. What we did: `retreatHp` 10 (Soldier) / 30 (Sage) and the Iteration-9 "mass ≥3 before
advancing" gate; LEARNINGS "Anti-patterns" record four rejected "outnumbered" signals that fired
"far more promiscuously than the diagnosis suggested" because they counted units, not *who can hit
whom this turn*.

**Heal-queue aware retreat: pick the Archon with the shortest expected wait, and refuse when the
hospital is full.** wololo: Archons publish `health_to_heal` (slot 25-28); a Soldier computes
`expected_healing_delay = travel + travel-from-attacker + archon_healing_health/2` for each Archon and
sets `low_hp_overwhelm` when `archon_healing_health/50 > 8`; overwhelmed low-HP Soldiers go
`find_suicide_spot` and disintegrate on a 0-lead tile (feeding the farm). smite `Archon.repair()`
sets `NOT_ACCEPTING_PATIENTS` when `amountToRepair >= 160·rubbleFactor·(archons+1)`; `Robot.baseRetreat()`
only paths to Archons accepting patients and sets `hospitalsAllFull` so the unit keeps fighting.
Musketeers `Soldier.estimateWaitTimeAt()` / `Comms.isAtHealingCap` (`MAX_TROOPS_HEALING=4`) and
`HealTimeout=200`. What we did: `nearestHomeArchon()` always; nothing stops 30 wounded Soldiers from
queueing at one Archon (LEARNINGS "Over-solicitude" bug, Iteration 91, was the mirror image).

**Archon repair policy: weakest when enemies are near, *strongest* when not (to get units back to
the front).** wololo `Archon.try_to_heal()` (two branches on `closest_attacker_dist < 10`; Miners
only if `miners_inactive`); smite `Archon.repair()` ("If there's enemies, repair the weakest ... If
there aren't, repair the strongest so we can send it back out", Soldiers/Sages ≤90 HP prioritized,
Miners only ≤9 HP); awesomelemonade `tryRepair()` identical rule; Musketeers `tryRepairMaxHealth()`.
What we did: most-wounded first, combat before Miners (Iteration 14), gated on `leadAbundant`.

**Sage usage: charge/fury by expected damage, 22%/10% math, kiting outside Soldier vision.**
wololo `Micro.sage_micro()`: score targets by `((hp−1)/45+1)·45 − hp%45` (prefer targets a 45-damage
shot finishes cleanly), `+4000·rubble` of the firing tile, CHARGE when ≥7 enemies in action range or
target HP ≤ 22% max, FURY on turret buildings when we own no buildings nearby, retreat along a
perpendicular to the home–enemy axis while on cooldown, and `advance` only when 60% of the shot
cooldown has elapsed. Soldiers near many Sages switch to `sage_assist_shot()` (pseudohealth `(hp−1)%45·45
+ (hp−1)/45`) to set up one-shots. smite `Robot.attack()` sums CHARGE damage (`maxHP·22/100` per
droid) and FURY damage over all targets *and friendly Archons*, envisions only if kills/damage beat a
plain attack. Musketeers `Sage.scanEnemies()`/`tryAttack()` compute `predictedDamage` for charge
(min(11,hp) per soldier etc.) vs best single target vs `totalBuildingHealth` for fury, and
`checkAvoidCharge()` moves away from friendlies 20 rounds before a scheduled global CHARGE.
AFinals `BotSage.mostEnemyDroidsAdjacentLocation()` steps into a tile that puts ≥4 soldiers in range
then charges. What we did: Sage runs `runSoldier` with overkill avoidance (Iteration 81) and a 1-step
standoff (Iteration 116); `envision` is never called (TRAINING_LOG 5539 "set aside"; Iteration 89
closed the anomaly thread on the post-mortem's remark about *global* anomalies — but every top bot
used the *Sage-local* CHARGE/FURY).

**Target scoring = damage/(hp·(10+rubble)) and instant-kill bonus; prototypes near completion.**
AFinals `getEnemyScore()` = `220·dmg/(hp·(10+rubble))` for combat units, `100000` for any unit with
hp ≤ 3 (instakill), Labs 1, Archons 1e-5; smite `attack()`: +1,000,000 for a kill (highest HP
killable first), +100,000 for a PROTOTYPE within 10 HP of completion, +19k/18k/17k Sage/Watchtower/
Soldier, Archon +20k after round 500; camel_case priorities Soldier 7 > Lab 6 > Sage 5 > Watchtower
4 > Builder 3 > Archon 2 > Miner 1. What we did: `targetPriority` {combat 3, Lab 2, Archon 1, else 0}
then lowest HP (Iterations 28/32/126). We never valued a kill, rubble, or an unfinished building.

### 1.4 Navigation

**Pathfinding generated per vision radius, with a bytecode budget switch.** Musketeers
`Nav.getBestDir()` picks `BFSUnrolled29/20/18/13` by `Clock.getBytecodesLeft()` and falls back to
greedy; falls back to `directionTo` after 3 turns without getting closer. smite generates one
`<Unit>Pathing.java` per unit type (`scripts/generate_pathing.py`, radii 34/20/13 with "smaller"
variants), uses `fuzzyMove` on turn 1 and after revisiting a tile (`tracker[113]`). wololo's
`write_pathfinding_algorithm_18.py` produces `query_cheap`/`query_expensive` over radius² 20 that
encode *distance, first direction and end tile in one double* (`dist*16%16` = direction) and add
`wait_cost` for occupied tiles so friendly units are soft obstacles; units pass
`(round − last_move_turn)/10 + 1` as the wait cost. camel_case uses `Dijkstra20/34/53` per type.
What we did: the single vendored `Dijkstra20` for all types (Archons/Watchtowers have vision 34),
no bytecode-aware fallback, friendlies as hard obstacles with a side-step (Iteration 29). Iteration
127 found overruns "in the longest, highest-unit-count games".

**"Approach the target's approach": chase a moving enemy toward where it will be.** wololo
`Pathing.get_closer_target()` walks the target 2 steps toward us along its cheapest-rubble path
before computing our move. Not seen elsewhere, not in ours.

**Warzone estimate instead of fixed objective.** wololo units keep a local `closest_enemy_dx/dy`
updated by their own moves (`Action.move`), by reservoir-sampled enemy sightings (4 per round
parity, slots 2-10) and by a drift of `WARZONE_SPEED/(1+rubble/10)` toward the enemy each round;
soldiers march to `Comms.closest_attacker_loc` and reset it when they arrive and see nothing. What we
did: `armyObjective()` = home threat > Miner cry > known enemy Archon > rotational mirror of our first
Archon, plus `SA_FOCUS` (one shared tile). LEARNINGS Iteration 29/33 traces show 64% of
Soldier-turns marching to a stale point.

### 1.5 Communication

**Sector / cluster maps.** smite divides the map into 5×5/6×6 clusters (`setupClusters`,
`computeClusterSizes`), stores 3 bits resource + 3 bits control per cluster in 100 shared slots via
generated accessors (`scripts/generate_comms_handler.py` SCHEMA), publishes short lists of 8 combat
/ 10 mine / 10 explore clusters that units claim (`getNearestExploreCluster` sets `CLAIMED`;
`writeMineClusterClaimStatus` = resourceCount/5+1 slots), and the first Archon re-adds regenerating
clusters with a `timeToRegen >= 30` timer ("ghost pings" comment). awesomelemonade `Communication`
packs 144 5×5 chunks at 4 bits each into 36 slots (ally/enemy/unexplored), buffered and flushed once
per turn. Musketeers keep up to 3 running-average enemy clusters (`LAST_ROUND_AVG_ENEMY_LOC_IDX_1..3`,
the "distributed k-clustering" from the project page). What we did: 8 lead beacons (`SA_LEAD_0..7`),
4 enemy Archon slots, one `SA_FOCUS`, one `SA_HOME_THREAT`. Our Miners could never know where
unexplored lead was; our Soldiers could never know a *second* fight existed.

**Read the whole array once, buffer writes, flush dirty slots (wololo `Comms.recv/set/send`,
awesomelemonade `buffer[]`, AFinals `readAllChannels`, smite codegen).** Our bot does ~30
`rc.readSharedArray` calls per unit per round scattered through `runSoldier`/`runArchon`.

**Alive-ness via round-parity stamps.** wololo slot 21-24 (`round_num` + portable bit, compared
against `Info.round_num − (archon_number < i ? 1 : 0)`), smite `STANDBY_ODD/EVEN`, camel_case
`getArchonTurnIndex` (`round*10 + counter`). Ours: `SA_OUR_ARCHON_0..3` are written once and never
invalidated (`publishStart`), so dead Archons remain "home" forever.

**Enemy Archon candidate lists from all three symmetries, invalidated on sight.** camel_case
`setPossibleEnemyArchonLocations()` writes 15 candidates (3 per Archon) and `getPossibleArchonTarget`
clears any it can sense; Musketeers `guessAndSortSymmetryLocs()`; AFinals `rememberedEnemyArchonLocations[3]`
+ `enemyArchonLocationGuessIsFalse`; wololo `possible_enemy_archon_locs` filtered by live symmetry
bits. Ours: a single rotational mirror (LEARNINGS "Open questions"; Iteration 76/88 attempts).

### 1.6 Symmetry and exploration

**Cheap symmetry elimination on round 2 from Archon positions, then rubble pairs.** wololo
`Comms.update()`: on round 2 each Archon checks whether the X/Y/XY mirror of every own Archon is
(a) an *enemy* Archon if visible or (b) not an own Archon — one `senseRobotAtLocation` each — and
afterwards *every* unit compares rubble at one pair of mirrored tiles per round, writing three
"impossible" bits into slot 0. smite `Robot.updateSymmetry()` tests rubble pairs across the midline
whenever a unit is within 5 (9 for Archons) tiles of it. Ours: Iteration 88's dedicated scout Miner
cost more than it gained (REJECTED) because detection was a separate *job*, not a free by-product
of every unit's turn.

**Exploration toward map extremities with persistence.** wololo `Miner.get_new_explore_loc()`
projects a random direction to the wall (12-way corner/centre seeding); AFinals `Explore.assignExplore3Dir`
(100-tile ray, rotate when 3 tiles from the edge); Musketeers `Explore` with `EXPLORE_BOREDOM=20` and a
`visited[64][64]` map; camel_case `tryWander()` cycles the four quadrants in random order. Ours:
`moveExplore()` random direction re-rolled 1/15 rounds with a stuck detector (Iteration 86).

### 1.7 Bytecode and engineering

- **TLE detection and recovery** is universal: wololo throws `"OUT OF BYTECODE!"` when
  `rc.getRoundNum() > Info.round_num`; smite `checkTLE()` re-runs `announceAlive()`; camel_case
  prints at ≥95%; awesomelemonade computes "over by N". We added equivalent monitoring only in
  Iteration 127 (the last accepted change), and never *acted* on it (no cheaper code path).
- **Bytecode-gated loops** (`Clock.getBytecodesLeft() < 2500 → break`, `Math.min(nearbyEnemies.length, 10)`,
  `getRestrictedFriendlies()` shrinking the sense radius when >15 results) appear in every top bot;
  ours has none.
- **Code generation** for pathfinding (all five), comms (smite), and spiral orders (`make_spiral_order.py`).
- **Custom data structures** (Musketeers `fast/FastIterableLocSet`, `FastMinQueue`, `FastMath.getProjection`).
- **Cached per-turn sensing** (wololo `Info.update()` sorts `senseNearbyRobots` once into typed arrays;
  awesomelemonade `Cache`). Ours re-senses in several helpers per turn.
- **Per-robot `Random(rc.getID())`** (wololo, AFinals) — the shared-seed `Random(6147)` pitfall in our
  LEARNINGS was avoided by them from day one.

### 1.8 Tooling, testing and process

- **Parallel local match runners with side-swap and "dominates both sides" reporting**: camel_case
  `scripts/run.py` (multiprocessing pool, prints ties and per-map domination), smite `test_bots.py`
  (6 threads, `--p1/--p2/--maps`), AFinals `autoscript.sh` (24 maps, both sides in parallel, greps
  `wins`), awesomelemonade `run_matches.py` (emoji table), Musketeers `get_record.py` (parses the
  *online scrimmage queue* into a spreadsheet). Ours: `tools/gauntlet.sh` on a remote GCE VM — the same
  idea, but every iteration cost a VM round-trip; smite's README lesson list is "scripts and thorough
  testing; replay analysis is key; start simple and iterate".
- **Versioned bot packages per tournament** (camel_case v01..v26, smite hermes→dionysus, Musketeers
  MP*, AFinals O*/A*) with `copy.py`/`copy_folder.py`/`prep_submission.py` to rename packages — we did
  the same (`g_iterN`, `tools/snapshot.sh`).
- **Game-analysis notes against named opponents** (`smite/notes/jan 16 game analysis`: "we make 6
  miners, they make 4 + 2 soldier ... we lose because we try to fight near their archons without
  critical mass") — the same discipline as our `--metrics` traces, but aimed at *other teams'* bots
  from the scrimmage ladder, which we never had.
- **Debug flags with profiles** (`Profile.ERROR_STATE`, `Debug.INDICATORS`, `DEBUG_RESIGN` to end
  games early at round 800) — cheap A/B iteration.

---

## 2. Ideas we tried but could not make work, that top teams made work

| Our attempt (TRAINING_LOG) | Why ours failed | How a top team made it work |
|---|---|---|
| Symmetry detection (Iterations 76, 87, 87v2, **88** REJECTED, "closes the whole symmetry thread") | a dedicated scout Miner cost economy; reactive detection came too late | wololo/smite: elimination by every unit as a side effect of its normal turn (one rubble pair per round, Archon-position check on round 2) — zero dedicated units |
| Sage standoff kiting (Iteration 89 REJECTED no-op; 116 accepted but marginal) | we had no gold, so no Sages | AFinals/Musketeers/smite/wololo made Sages the *primary* combat unit by gating Soldiers on gold and transmuting from round ~100 |
| Laboratory/Sage economy (Iteration 57 REJECTED "catastrophic", 58 REJECTED, 64 accepted at 1 Lab, 118/122 at 2-3 Labs) | Lab on the Archon ring blocked building; later, fixed lead bars 1,500/3,000 | wololo/smite/AFinals: Builder-placed Labs on low rubble away from units, count driven by Miner count / income EMA (`mining_miners·3 >= labs·4`, `resourceRateEMA > 4`), Labs relocate themselves |
| Archon repair-vs-build throttle (Iterations 63, 74, 75 REJECTED) | tuned a fixed/self-calibrating threshold on *whether* to heal | smite/wololo/awesomelemonade: *whom* to heal (weakest in combat, strongest otherwise) plus a hospital-capacity flag so the build queue is not starved |
| Miner-rescue / distress reinforcement (Iterations 69-73, 75 REJECTED except 73's throttle) | fixed distance thresholds fired promiscuously | camel_case `addDangerTarget(loc, expiry)` list of 20 with per-round expiry and `enemyCount > defenderCount` check before raising; Musketeers 3 running enemy clusters |
| Fairness-yield across Archons (Iterations 49-53, 60 REJECTED; 61 accepted) | cooldown heuristics | explicit lead reservation / turn queue (wololo urgencies, smite `reservedLead`, Musketeers `canBuildPrioritized`) |
| Active gold scouting (Iterations 84, 98 REJECTED) | 1-in-10 Miners wandered; cost > payoff | nobody scouted for gold tiles; gold came from Labs. (wololo Miners do mine adjacent gold opportunistically and prefer it when fleeing.) |
| Rally / consolidation gates for reinforcing Soldiers (Iterations 104, 107, 108, 124 REJECTED) | count-based gates mis-fired on `highway` | per-tile micro scores (1.3) make the decision at combat time, per unit, from who can hit whom |
| Removing fixed-compass tie-breaks (Iteration 126 two variants REJECTED, third shipped at a cost) | 60 iterations of tuning depended on the bias | top bots randomize ties (Musketeers `loadBuildDirections` reservoir-samples equal-rubble directions; AFinals `getBestSpawnDirection` random among best rubble) or key them to the enemy/centre from the start |
| Multi-Watchtower scaling (Iterations 54-56 REJECTED, 109/117/123/125 accepted) | towers only adjacent to the Archon | camel_case lattice placement anywhere the Builder wanders + mobile Watchtowers |
| Archon relocation (Iteration 34 accepted, marginal) | once per game, after r500, 6 steps | camel_case `findOptimalLocation` at r10/150 and after every VORTEX; wololo relocates toward the warzone to within 11 tiles on the lowest rubble; AFinals `transformAndMove` 20 steps toward combat with a 200-turn portable limit |

---

## 3. Ideas we had that top teams also had

- Rubble-weighted unrolled Dijkstra as the main pather (ours vendored from camel_case).
- Leave 1 lead on a tile so it regenerates (universal).
- Soldier before Archon in target priority; Laboratory high (camel_case list, Iteration 28/32).
- Step to lower rubble before/while attacking (camel_case `tryAttack`, Iteration 21).
- Low-HP retreat to an Archon; critical-HP-only thresholds (camel_case `<10`, smite `FLEE_HEALTH=8`).
- Builder flees combat units; Builder repairs PROTOTYPE buildings first.
- Live unit census via shared accumulators reset by the first/last Archon (wololo slots 36-45, AFinals
  `clearCommCounts`, smite `writeWorkerCountAll(0)`).
- Vortex-aware Archon transform (camel_case `vortexHappened`, AFinals `vortexMoveIfNeeded`) — we noted
  it but did not implement.
- Sage overkill avoidance and Sage-specific retreat HP (wololo `low_hp = HP <= 50`, Musketeers 45, ours 30).
- Bytecode overrun detection via round-number comparison (Iteration 127).
- Snapshotting every accepted bot version as a future opponent and running both sides on every map.

---

## 4. Sources

Post-mortem / write-ups
- 5 Musketeers, *Battlecode 2022 Strategy Guide* (D. Lyons, Feb 2022) —
  https://battlecode.org/assets/files/postmortem-2022-5-musketeers.pdf (search snippets only; two
  verbatim passages preserved in our `TRAINING_LOG.md` lines 6973-7010)
- Maxwell Jones project page — https://maxwelljon.es/projects/battlecode/ (search snippet)
- battlecode.org past years list (winner wololo / Isaac Liao) — https://battlecode.org/past.html
- smite README and `notes/` — https://github.com/mvpatel2000/Battlecode2022
- TestSubjector README, `CHANGELOG.md`, `NOTES.md`, `SCRIMS.md` — https://github.com/TestSubjector/BattleCode2022
- camel_case README — https://github.com/jmerle/battlecode-2022

Code (`owner_repo` = https://github.com/owner/repo)
- `iliao2345_Battlecode2022/src/fury_fix_20/{Archon,Miner,Soldier,Micro,Sage,Builder,Laboratory,Watchtower,Comms,Info,Action,Pathing,Exploration,Bfs}.java`, `write_pathfinding_algorithm_18.py`
- `jmerle_battlecode-2022/src/camel_case_v25_final/**`, `scripts/{run,dijkstra,copy}.py`
- `TestSubjector_BattleCode2022/battlecode22-scaffold/src/AFinalsBot/**`, `autoscript.sh`
- `mvpatel2000_Battlecode2022/src/athena/**`, `scripts/{test_bots,generate_pathing,generate_comms_handler}.py`, `notes/`
- `BSreenivas0713_Battlecode2022/src/MPTempName/**`, `get_record.py`
- `awesomelemonade_Battlecode2022/src/bot/**`, `run_matches.py`, `generate.py`

Our repo (https://github.com/terryvanbelle/battlecode22-vibe)
- `src/bot/RobotPlayer.java`, `src/bot/Dijkstra20.java`, `LEARNINGS.md`, `TRAINING_LOG.md`
  (Iterations 34, 57-64, 76, 82-91, 98, 109-128), `TRAINING_ALGORITHM.md`, `RESEARCH.md`, `ART_OF_WAR.md`,
  `progress/vs_old_bots_history.csv`, `tools/gauntlet.sh`.

Not reachable from this sandbox: the 5 Musketeers PDF itself (battlecode.org denied; no GitHub
mirror found; the local copy referenced by `TRAINING_LOG.md` 6979 no longer exists), the 2022 game
specs (battlecode.org), the final-tournament bracket, Producing Perfection / Polar Bears / Baby Ducks
2022 repos (no public repo found under the obvious names).
