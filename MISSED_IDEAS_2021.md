# MISSED_IDEAS_2021 — Battlecode 2021 ("Campaign" / Enlightenment Centers): what the top teams did that we never figured out

*Written 2026-10-21. Post-hoc comparison written after the ban on reading other teams' 2021 code and
post-mortems was lifted. Every claim about THEIR bots comes from code or a post-mortem read as listed
below; every claim about OUR bot comes from https://github.com/terryvanbelle/battlecode21-vibe. Anything
inferred rather than read is marked (inferred).*

## 0. Sources read and where we stand

**Post-mortems**

| team (place) | how read |
|---|---|
| Baby Ducks (1st; Brunner, Grebe, Ye, Zhang) | **in full**: `postmortem/index.html` inside the `JasonYe4273/et-battlecode-2021` repo (same text as the blocked web.mit.edu page) |
| naalit (finalist, solo) | **in full**: README of `naalit/battlecode21` is the post-mortem |
| waffle (finalist, winkelmantanner) | **in full**: README of `winkelmantanner/battlecode2021_waffle` |
| California Roll / Stone Tao (9th) | README of `StoneT2000/Battlecode2021` in full; the blog post only via WebSearch snippets |
| wololo (7th-8th, Isaac Liao = `iliao2345`) | **search snippets only** (battlecode.org PDF blocked); code read in full instead |
| 3 Musketeers (9th-12th) | **search snippets only** (PDF blocked and not in repo); code skimmed |
| Malott Fat Cats (4th), Ivy Zhang (HS 3rd) | search snippets only; no code |
| wstan2001 | README (flag/opcode scheme) |

**Code read** (shallow clones; `owner_repo` = https://github.com/owner/repo):

| bot | package | lines | depth |
|---|---|---|---|
| Baby Ducks (winner) | `JasonYe4273_et-battlecode-2021/src/finals` | 8,135 (1,914 generated table, 4,236 unrolled nav; ~2,000 logic) | all logic files in full |
| awesomelemonade (our #1 ladder bot) | `sprint1bot` (2,969, the ladder build) and `combobot3` (4,762, their final, Jan 28) | both read in full except the 989-line slanderer table |
| rzhan11 (our #2 ladder bot) | `rzhan11_Battlecode2021/src/sprint2` | 6,111 | EC, Politician, Muckraker, Slanderer, Role, Comms, CommManager, HQTracker, Robot, Map in full; Bug/Nav skimmed |
| wololo (`iliao2345.attacker` is on our ladder, 48-90 for them) | `iliao2345_Battlecode2021/src/slands3` (final, "slanderer economy with guards") | 3,274 | EC, ECInfo, Muckraker, Politician, Slanderer, Guard, Burier, Targetter, RelayChain, Flag, CombatInfo in full |
| 3 Musketeers | `BSreenivas0713_Battlecode2021/src/musketeerplayerfinal` | 8,770 | EC state machine, Comms schema, LatticeProtector/DiagonalSlanderer/BuffProtector heads, bidding, Nav head |
| Stone Tao | `StoneT2000_Battlecode2021/src/maxecosushi` | 3,640 | EC build logic, Muckraker/Politician lattice and flank code, Comms constants |
| rqi3 (rank 4 on our ladder, 11-1 vs us) | `rqi3_battlecode-2021/src/qualification_bot` | ~3,500 | EC bidding and spawn logic |
| Ivan Geffner (`thirtyone`, 0% for us in 6 games) | `IvanGeffner_battlecode2021/thirtyone` | 13,220 (5,300 unrolled BFS) | EC in full, Pathfinding |

**Where our final bot stands** (`POSTMORTEM.md`, `progress/ELO.md`): submission `g_iter13`, `src/bot`
~1,480 lines in 13 files; 334/432 (77.3%) over nine ladder blocks; **33% vs awesomelemonade.sprint1bot,
46% vs rzhan11.sprint2**, 67-100% vs the other six; Elo 1742, rank 3 of 21 rated bots after 1,248
scrimmages. Recorded weaknesses (`POSTMORTEM.md` 7-8, `HANDOFF.md`): our slanderers are hunted and
theirs are not; fresh captures fall within 100 rounds to attackers sized to the target; the swarm forces
~100 guards by r250 (the whole economy); the bank sits idle in losses; the early vote race is won and
the game still lost.

---

## 1. Ideas we never figured out

### 1.1 Macro strategy and win condition

**Income-targeted build order, not unit-count caps (Baby Ducks).** The centre keeps a ledger
`slandererBuilds[round] = income/round of the slanderer built that round`, and every turn recomputes
`expectedCurrentIncome` (sum over the last 50 rounds) and `expectedTotalIncome` (remaining income of all
live slanderers). Build rules are thresholds on that number: below 1,500 total income it builds slanderers
(with the gate "4 small politicians once income > 500"); at 1,500-2,500 it builds politicians until 8
exist; at 2,500 until 12; above that it builds slanderers only while
`expectedTotalIncome < INCOME_TARGET`, where `INCOME_TARGET = 187 * (polyCount - 3) * 1.5` -- *the target
rises with the number of politicians alive*, which the post-mortem explains: "If we have more living
politicians, that indicates they aren't exploding ... so this map is probably more defensive and we adjust
our target income up". Once the target is met the mix is 25% big politician (`min(inf - target/10,
(inf + expectedTotalIncome)/20)`), 37.5% small politician, 12.5% small muckraker, 25% buffraker
(`Center.mainturn`, `finals/Center.java`). *Ours:* fixed counts -- `MAX_SLANDERERS=20`,
`SPEND_SLANDERER_CAP=40`, `GUARD_BASE + slanderers/2` guards, `SCOUT_BASE + round/40` scouts (`C.java`,
`EC.build`). We never measured our own income or set a target from it; three of our accepts were cap
removals (`LEARNINGS.md` "find the constraint that binds"). This is the general form of that lesson.

**Ratio-driven roles with a safety budget (rzhan11).** `Role` enum: MUCK 1.0, DEFENSE_POLI 1.0,
ATTACK_POLI 0.1, EXPLORE_POLI 1.0, SLAN 1.0; each turn `score = count / ratio` and the EC builds the
lowest-scoring role (`Role.updateRoleScores`, `EnlightenmentCenter.turn`). Modifiers: ATTACK score x2.5
when a neutral is targeted; MUCK score = inf when `MUCK_CAP=100` reached or influence > 10^7; SLAN capped
at 50. Everything is bought from `mySafetyBudget = influence - sum(max(0, enemyPolConviction*buff - 10))`
over enemy politicians in sensor range (`updateMaxBudget`). *Ours:* an ordered if/else chain; `reserve()`
keeps only `2*bid` (`EC.reserve`). Iteration 10 ("threat-sized guards, else bank") was rejected because it
*reacted* to a visible attacker; the safety budget is a standing deduction, not a reaction.

**Time-varying unit proportions (Ivan Geffner `thirtyone`).** `typeProportions[SLANDERER] = max(3, 3.5 -
t/200)`, `[POLITICIAN] = min(4.5, 2 + t/100)`, `[MUCKRAKER] = max(2.5, 4.5 - t/100)`, `[BIG_MUCKRAKER]
0.5`, `[CONQUEROR] 10000` (only when affordable); the EC builds whichever eligible type has the lowest
`currentProportions[i]` and adds `1/typeProportions[i]` on each build (`Enlightment.getTypeStandard`,
`build`). Politician and muckraker sizes follow the ruler sequence 1 2 1 3 1 2 1 4 ... over tier tables
`{16,24,36,54,81,121,181}` and `{1,13,33,43}` so sizes are mixed deterministically. *Ours:* no mixing of
sizes at all; guards are `min(max(20, inf/4), 60)` or 15.

**Buffrakers: muckrakers with real conviction, in every top bot.** Baby Ducks: 25% of post-target
builds are muckrakers of `(inf + expectedTotalIncome)/20` (post-mortem: "a raker of any size is capable
of destroying slanderers ... politician defenses assumed all rakers would be small"). rzhan11: a
`targetConviction = ceil((age+1)/25)` muckraker with probability 0.5, plus a BIG muckraker of 25% of the
safety budget (cap 1,000) every 50 rounds while the team buff is < 4 (`makeMuckraker`). awesomelemonade
combobot3: after r250, 10% of turns build a muckraker of `max(influence - 50, 150)`; the cheap muckraker
itself costs `max(1, 0.1*influence/(1500-round))`. Stone Tao: "scouting buff mucks that flank" at 25% of
allowance (50% if > 1,000), every 15 rounds, aimed first at directions where enemy slanderers were
reported (`EnlightmentCenter` lines 538-620). waffle: muckrakers up to 500 in proportion to slanderers.
Musketeers: every 20th muckraker after 50 is `max(50, inf/5)`. *Ours:* every muckraker is 1 influence
(`EC.build`, `Roles.SCOUT/HUNT`). Iteration 28 tried 150- and 50-influence hunters and rejected them at
41%/30% *in the mirror*, whose guards are already cheap -- the mirror could not show the point of a
buffraker, which is to walk through 15-20-conviction guards.

**Guards sized to the muckraker actually seen.** awesomelemonade `reactDefense`: cost = `min(5c + 11,
2c + 15, c + 20) + rand(4)` where `c` is the conviction of the nearest or the largest muckraker seen within
d^2 400 (`CentralUnitTracker.maxMuckrakerConviction`), and the number wanted is `numNearbyAllySlanderers
* 3/5` while a muckraker was seen in the last 200 rounds, 2/5 to 500, 1/3 after. wololo: guard cost =
`max(muckSize + 10 + 1, conviction/100)`, with `unhandled_muckraker_loc/size` taken from relayed
sightings and each enemy muckraker id (mod 127) assigned at most one guard
(`ECInfo.targetting_guard_ids`, `EnlightenmentCenter.act`). Stone Tao: `antiBuffMuckPoliSizeNeeded =
(size + 10) / 0.7` for the first reported big muckraker not yet covered by friendly firepower. Ivan
Geffner: `getConvictionToKillMuckraker = c + 13`, skipped if a politician that can already kill it is
adjacent (bitset `myPolis`). naalit's post-mortem names the counter: "notice when you see a buffraker,
then build some guard politicians with just enough conviction". *Ours:* `GUARD_MUCK_COST=15` flat under a
swarm, else 20-60, never read from the threat (`C.java`). A 50-influence muckraker walks through our
15-guards.

**One guard per muckraker, not every guard per muckraker.** Baby Ducks `Politician.movement`: a
politician moves toward the relayed muckraker only if no other friendly politician is closer to it
(`nearestPoliticanToRaker`). awesomelemonade `shouldChaseMuckraker`: chase only if at most one friendly
politician is already within d^2 5 of it, and only if the size ratio is sane (`2c+20 >= mine && 2*mine+20
>= c`, or it is within 9 tiles of a slanderer/EC and `10c+30 >= mine`). *Ours:* every guard within the
leash chases `nearestEnemyMuck` (`Politician.guard`), which is how 108 politicians died killing 83
influence of muckrakers against `arch_hunt` (`TRAINING_LOG` 2026-09-22 15:00).

**Neutral-EC capture sized by distance and claimed by id (Baby Ducks).** `cost = min(neutral + 64, 500)
+ 11 + max(0, d^0.75 - 50)/2` (extra conviction for far targets, which pass enemy units on the way); one
politician per neutral tracked by `polIDToKillNeutral[i]` and re-issued only if that id has been unreadable
for 10 rounds; while saving, build 1-influence muckrakers as long as `expectedTotalIncome > 3 * cost`
(`Center.mainturn`). rzhan11: `cost = 25 + neutralInf + 10`, re-sent only after `hqAttackDelays =
2*sqrt(dist) + initialCooldown` rounds, and the EC *stops bidding* when the shortfall will be covered by
income within 5 rounds (`STOP_BID`). Stone Tao: `+120` margin and a gate `incomePerTurn * factor >= cost`
with factor 8/15/30 by round. *Ours:* `neutralInf + 14`, cheapest first, one capturer per target
(`captureAffordable`); no distance term, which is one reason 34 of 60 captured centres fell within 100
rounds (`POSTMORTEM` 6) -- a politician that arrives at a contested neutral with +14 flips it with ~4 to
spare.

**Capturers add up their convictions on arrival (Baby Ducks, awesomelemonade).** `Politician.attackHQ`
sums `(conviction - 10) * buff` over all friendly politicians in sight and subtracts enemy politicians'
before deciding to speak; `shouldAttackHQ` does the same when choosing to approach. awesomelemonade
`tryEmpowerAtEC` computes `convictionBalance` over friendly politicians within d^2 16 of the EC and fires
if `balance > ecConviction` (`+20` if an enemy politician is present), standing on a *cardinal-adjacent*
tile and waiting otherwise. *Ours:* a capturer speaks only if its own share flips the centre
(`captureShare`); otherwise it "chips" and the next one finishes. Iteration 44 (chip captures) was rejected
because the chips were uncoordinated and drained the bank; the top bots' version is coordinated and
cheap.

**The attack on enemy centres is sized to the target and aimed at the weakest (wololo, awesomelemonade).**
wololo's buriers (muckrakers sitting on the enemy EC) report the EC's influence in their flag (6-bit log
scale); the EC picks `weakest_ec_loc` and builds a targetter of `max(max(50, weakest) + 11,
conviction/4)` when `max_safe_build_limit` allows, plus "monster" politicians of `max(300, conviction/2)`
once `target_all_ecs_flag` (`EnlightenmentCenter.act`, `ECInfo.update`). A burier prices an enemy EC at
`3 * (conviction + (sqrt(dist to spawn) + 10) * passiveIncome)` -- the influence the EC will gain while the
politician walks. awesomelemonade politicians with conviction >= 50 target, in order: the closest neutral
they can solo, the closest enemy EC within d^2 400, then the EC closest to the centroid of our own ECs
(`compareECs`). *Ours:* enemy centres were attacked by two branches aimed at `enemyEC[0]`, priced from a
single bucket or the whole bank; Iteration 53's target-aware attack never passed the gate and lives only
in `src/arch_lemon` (`HANDOFF.md`).

**Defenders that go to the source.** Baby Ducks: a politician that has not seen an enemy for 10 turns
stops defending and charges the nearest reported muckraker or the enemy base ("the spawn point for enemy
muckrakers, so they could be killed at the source"); a slanderer-turned-politician does the same. rzhan11:
an ATTACK politician without a target for 100 rounds turns on "extreme aggression" and empowers at any
enemy. *Ours:* guards hold a ring forever (`GUARD_LEASH_D2=80`); only camouflaged slanderers attack
(`CAMO_ATTACKS`). Iteration 13 (the army) was rejected as a replacement for guards, not as a timeout.

**Spawn blocking as a role ("bury"), done with spacing (wololo, rzhan11, awesomelemonade).** wololo's
`Burier` sticks muckrakers to the enemy EC's adjacent ring, then the ring at distance 2, then 3, reports
`need_support` while any adjacent tile is empty, and the EC stops building muckrakers once
`everything_buried` (`Burier.update`, `ECInfo.all_ecs_buried`); `Info.everything_buried` is also what
switches the whole bot from muckrakers to slanderers ("map_controlled"). rzhan11 muckrakers approach to
`MEDIUM_CLOSE (d^2 8)`, circle, and the lowest-id muckraker at the EC ("reporter") broadcasts
SURROUNDED when the adjacent ring is full or the medium ring 80% full, so other muckrakers retarget
(`Muckraker.checkLocalSurround`, `HQTracker`). awesomelemonade `tryECSpiral`: approach spaced apart, then
walk the tangent. *Ours:* one scout in four walks to the enemy EC and sits adjacent, no spacing, no
"full" signal (`Muckraker.turn`, `CAMP_ONE_IN=4`).

**Uncovering your own spawn tiles when buried (wololo, Musketeers).** wololo computes
`net_enemy_conviction` on each adjacent tile (enemy politicians count influence + conviction, muckrakers
conviction + 1, minus the damage friendly adjacent politicians would deal to their neighbours), finds the
cheapest enemy to remove and builds a politician of `max(26, 3*min_cost + 10)` on a tile adjacent to it
whenever `open_spawn_tiles < ceil(11 * passability / 2) + 1` (`EnlightenmentCenter.act`). Musketeers have a
`REMOVING_BLOCKAGE` state (trigger: >= 6 enemies adjacent). *Ours:* `blockedRounds` is a diagnostic
counter only (`EC.build`); a buried centre of ours simply does nothing.

**A reactive one-tile shield against a converter (wololo, waffle, Ivan Geffner).** wololo: when
`max_safe_build_limit < 0` (an enemy politician in range could flip the EC) every build goes on the tile
nearest the largest enemy politician, as a 1-influence body that dilutes the speech. waffle: "If an EC sees
an enemy politician, the EC builds small muckrakers in the four cardinal directions ... a shield because
of the way empowering works". Ivan Geffner: `attacked -> build 1-influence muckrakers toward the enemy
politician` unless the planned build still leaves `influence >= enemyCombinedAttack + 4`. *Ours:*
Iteration 3 (a permanent wall of 4-7 muckrakers) and Iteration 52 (four diagonal sentinels) were rejected
-- both were *standing* bodies that died to every speech and blocked spawns; the top bots build the shield
only while an attacker is in sensor range and only when the budget says the centre would otherwise fall.

**Heal the centre instead of hoarding (awesomelemonade).** A politician of >= 50 conviction speaks
*into* a friendly EC when `2 * sum(enemy politician conviction within d^2 64) > ec.conviction` and the EC is
not our spawn (`tryHealEC`, `shouldHeal`), standing on a cardinal-adjacent tile so the speech lands only
on the EC. *Ours:* Iteration 15/18 ("deposit": expired slanderers speak at the EC) was a near miss and
rejected as a surplus sink; the healing form -- triggered by a threat, not by a surplus -- was never
tried.

**Nobody bids early; everybody caps the bid as a fraction that rises late.** Baby Ducks `vote()`:
`voteProp` in [min, max] by round -- 0.001-0.004 from r50, 0.01-0.02 from r400, 0.02-0.03 from r1000,
0.03-0.05 from r1200, 0.05-0.1 from r1350, 0.1-0.25 from r1450; up by max/4 on a loss, down by max/16 on a
win, reset to 0 when a loss happens at the cap; stop when `opponentVotes - myVotes > rounds left`, where
opponentVotes counts rounds we bid and did not gain. rzhan11: **no bids before r250** and only when the
safety budget > 100; bid = last x1.5 on a loss, /1.1 on a win, capped at budget/10, floor budget/50 after
r500. wololo: `BID_START_TURN=200`, exponential `bid_power`, floor `3*conviction/(rounds_left -
bids_required + 5)`, bid 1 when poor. rqi3: percentage bound 0 until r50, 0.1 to r200, 0.15, 0.30 from
r1300, 0.50 from r1480. Ivan Geffner: nothing before r170, then bid 1 until r400 unless behind.
awesomelemonade: 1-3 or 1-2% of influence only when influence > 100, uniformly random. *Ours:* bid from r1,
adaptive, cap `inf/30` to r600 (`EC.doBid`); we placed 150 bids by r50 against rzhan11's zero
(`TRAINING_LOG` 2026-09-18 17:30). Stone Tao's post-mortem calls bidding "an unnecessary layer"; naalit's
says "most games are decided by annihilation ... all finalists did bid, but usually not that much".

**Endgame conversion.** Baby Ducks: after r1450 a politician with > 20 allies within sensor range and any
enemy in action range empowers at the max-kill radius; awesomelemonade: after r1490 with >= 10 robots per
remaining round and votes < 751, empower if anything would be drained; wololo: after r1490 every
politician empowers at its best radius. The tiebreak after votes is EC count then total unit influence,
and converting enemy politicians moves influence from their total to ours. *Ours:* nothing changes at
r1490.

**Map-adaptive aggression.** Stone Tao `ratiocap` (slanderers per politician) = 0.25 if the enemy EC is
within d^2 155, 0.3 to 200, 0.4 to 425, else 0.8; if the enemy EC is unknown, 0.25 when the first enemy
muckraker arrived before r100; `threatFactor` decays 10% every 10 rounds. rzhan11: `DEFENSE_POLI_ROLE.ratio
*= 1.5` in a corner, 1.25 on an edge (`MIN_WALL_DIST=3`). Baby Ducks: the living-politician proxy above.
*Ours:* no map-size, distance or wall term anywhere in `C.java`.

### 1.2 Economy

**Slanderers up to 949, and "stop when income no longer matters".** Baby Ducks' `Threshold` table and
wololo's `embezzle_floor(min(949, budget))` buy up to 949; naalit: "the optimal return comes from
slanderers with a starting conviction of 949 ... what you'll usually see in games between top players".
rzhan11 caps at 497, awesomelemonade at 463 like us (`buildSlanderer`), but adds two stops: no slanderer if
`conviction >= 300 * (1500 - round)` (the income cannot be spent) and none within 10 rounds of the last
unless there are zero (`shouldBuildSlanderer`). rzhan11 builds a slanderer only if `cost * 50 >=
mySlanTotalValue / 10` (a new slanderer must add at least a tenth of the remaining slanderer value) or it is
the 497. *Ours:* `MAX_SLANDERER_SIZE=463`, no late-game stop; in losses we hold 20-40k while the opponent
holds six centres (`POSTMORTEM` 8.3). (inferred) The 949 ceiling matters once a centre has thousands: a
949 returns ~1.6x in 50 rounds and uses one build slot where two 463s would use two.

**Per-slanderer income ledger** (Baby Ducks `slandererBuilds[1500]`, rzhan11 `mySlanSpawnRounds /
mySlandererEarns / mySlanTotalValue`, wololo `embezzler_ids/incomes[50]` and `total_income_per_build =
income * 2 / tile_cost`, awesomelemonade `perTurnProfit = influence - lastInfluence`). All four bots know
their income per round and per build slot and gate decisions on it (saving, bidding, attack timing).
*Ours:* `Econ.incomeOf` exists but the EC never sums it; `@econ` logs `inf` only.

**Save for a target with 1-influence muckrakers, not with idleness (Baby Ducks).** While `inf < cost`
for the chosen neutral and `expectedTotalIncome > 3*cost`, the centre builds a 1-muckraker every turn
(scout + blocker) instead of a slanderer, so the save costs build slots but keeps the map pressure. *Ours:*
save mode (Iteration 22) froze the economy at two slanderers and was switched off (Iteration 51).

### 1.3 Combat micro

**An empower metric that prices units and influence together.** Baby Ducks: speak when
`100*(kills - 2) + influenceDrained - convictionSpent > 0`, kills counting a muckraker as 2 and an EC as 10
(post-mortem; `Politician.checkEmpower` uses `(kills-2)*100 + infDrain - conviction`), with special cases:
near base or a slanderer, any double kill; adjacent enemy EC with a friendly politician adjacent; and a
"beefy raker" (> 30 conviction) hunted by any politician over 100. wololo `CombatInfo`: `gains = kills *
unit_price + costs`, where `unit_price` is broadcast by the EC as twice its income per build slot (i.e. the
price of a build *slot*, which the post-mortem snippets describe as the scarce resource: "a trade of about 1
unit count for about 12 conviction"); an enemy politician counts 2 kills (+1 per adjacent EC, +10 if it
blocks a targetter), an EC 1,000, a neutral EC 500. awesomelemonade `shouldEmpower`: nine ordered rules,
including `requiredRatio = 1/(0.03*(dist_to_own_EC + 5)) + 1` so a muckraker kill must return a larger
fraction of our conviction the farther from home we are, and `numKills * 15 >= conviction - 10` for
politicians >= 60. rzhan11: total score >= `0.5 * (conviction - 10)` with kill bonuses x2 and a
`killHungryTarget` override for the muckraker a defender was sent for. *Ours:* a flat bar of 12 with a
muckraker worth 25 and a slanderer 40 (`Politician.bestSpeech`, `SPEECH_MIN_VALUE`); Iteration 47 made
the bar flat because the proportional one (`conv/3`) made big guards fussy. None of the top bots uses a
constant.

**Moving to control someone else's speech (wololo `compute_move_gains`).** For the largest politician in
sight (friend or enemy) wololo evaluates the gain of that politician's best speech with and without our
unit inside each radius, for each of our 9 possible moves, and moves to the tile that maximises our gain
(step *into* a friendly speech to be healed or out of an enemy one; step *into* an enemy speech to dilute
it when that saves a bigger unit). Muckrakers do the same (`Muckraker.act`). awesomelemonade's
`kiteBigPNearNeutral`/`chaseBigPs`: small units step away from our own big politician near a neutral so the
capture is not diluted, and a spent politician (conviction <= 10) walks *into* enemy big politicians to
absorb damage. *Ours:* `planCapture` waits or flees if < 60% of the speech would land on hostiles -- the
capturer moves, never the bodies around it.

**Identify friendly slanderers from flags.** Politicians cannot distinguish friendly slanderers from
politicians (`RULES.md`), so Baby Ducks put a politician bit (`0x080000`) on every politician flag and
treat a friendly "politician" without it as a slanderer (`Politician.checkEmpower`, `defend`);
awesomelemonade encode direction 9 (`CURRENT_DIRECTION_CENTER_SLANDERER`) in the flag of a stationary
slanderer (`UnitCommunication.isPotentialSlanderer`); rzhan11 use status bit 3; Musketeers and Stone Tao
broadcast `ROBOT_TYPE`/`IMASLANDERERR`. The bit drives politician positioning (stay near slanderers,
never adjacent), guard counts and the "any kill near a slanderer is worth it" rule. *Ours:*
`Politician.threatens` tests `f.type == RobotType.SLANDERER` on sensed friendlies -- which a politician
never observes (inferred from the rules: politicians sense slanderers as politicians), so the slanderer
branch of Iteration 56's threat test cannot fire. We have no slanderer identity in `Comms`.

**Slanderers flee relayed muckraker sightings, with age.** Baby Ducks: every unit that sees a
muckraker sets flag `0x1xxxxx` with its location and `round mod 16` age; readers pick the sighting
minimising `age^2 + distance` and slanderers flee it up to d^2 400 even beyond sight, politicians chase it
(`Robot.findRakerFlags`, `Slanderer.movementS`). rzhan11: `REPORT_ENEMY_MUCKRAKER` to the EC every 3
rounds, `ECHO_ENEMY_MUCKRAKER` unit-to-unit, `alertEnemyMuckraker` expiring after 6 rounds; slanderers
build their lattice on the side away from it. awesomelemonade combobot3: the EC broadcasts, instead of
its rotation, the enemy muckraker within d^2 225 with **no defender within d^2 9**, scored by `distToEC -
sqrt(nearest defender distance)`, together with that distance (6 bits), so defenders go where no one is
and slanderers kite if within `(distance+3)^2` (`CentralUnitTracker.calculateBroadcastedEnemy`,
`setEnemyFlag`, `processFlagsFromECs`). waffle: the EC broadcasts the nearest enemy and all slanderers move
to the opposite side. *Ours:* slanderers flee only what they sense (`Slanderer.turn: nearestEnemy`), r^2
20 against an exposer moving on a 1.5 cooldown; Iteration 12 (EC relays the nearest enemy) was rejected
at +2 roster cells, measured against opponents whose hunters had already found us.

**Politician defence as force fields, not a ring (Baby Ducks `defend`, awesomelemonade `tryDefend`,
wololo `Guard`).** Baby Ducks: repulsion `1000/d^2` from every other politician, a `200/d` pull away from
home when within d^2 25, `1000/(d to each known wall)` push from edges; awesomelemonade: `2.25/d^3` repulsion
from small politicians, slanderers, ECs and walls. wololo guards keep a `defense_layer` (min over
neighbours' layers + 1, a distributed BFS depth from the slanderers), repel same-layer guards, and hold
`layer_separation^2` between 17 and sight of the inner layer. *Ours:* `spreadOut` picks the adjacent tile
with the fewest friends inside a ring 20-80 from home (`Politician.guard`, `Robot.spreadOut`).

### 1.4 Formations and navigation

**Lattices everywhere.** awesomelemonade: slanderers hide on `(x + y) % 2 == 0` tiles, nearest to an own
EC but never within d^2 4 (`Slanderer.hidingScore`, `LatticeUtil`); defenders score lattice tiles outside
d^2 16 of the EC not adjacent to a slanderer, by `5000*dist + dist to enemy EC`. rzhan11: slanderer lattice
`(x + y) % 2 == 0` built on the side *away* from the alert/enemy EC and never adjacent to the EC
(`Slanderer.makeSlanLattice`, `Nav.updateIsDirMoveable`), politicians on a quarter-density lattice `(x%4) +
4*(y%4) in {1, 11}` at least d^2 4 out and 40 from corners, yielding a slot to a "worse friend"
(`Politician.makePoliLattice`). Stone Tao: slanderers on `x%2 != y%2`, defenders on `x%3==0 && y%3==0`
(`Slanderer.locOnLattice`, `Politician.LATTICE_SIZE=3`). wololo: slanderers park on "storage sites"
`(x+y)%2 == 1` outside d^2 8 of the EC and leave along "outgoing lines" `(x+y)%2 == 0`
(`Slanderer.act`). Musketeers: `DiagonalSlanderer`, `LatticeProtector`. naalit's post-mortem gives the two
reasons: a dense slanderer lattice lets converted politicians and newborns walk out diagonally, a sparse
politician lattice sees and reaches every tile. Stone Tao's post-mortem says "several teams copied my
slanderer latticing". *Ours:* a ring `8 <= d^2 <= 45` with a crowding rule (`SLANDERER_RING_*`,
`CROWD_MAX`), which `POSTMORTEM` 8.2 lists as the unmeasured question "where our slanderers should live".
Our centres also ended games walled in by their own bodies (Iteration 40, 63 guards around one centre).

**Local pathfinding with a bytecode fallback, where it mattered.** Baby Ducks: Bellman-Ford over a 5x5
window, two relaxation sweeps, fully unrolled (`UnrolledNavigation`, 4,236 lines), used only when
`Clock.getBytecodesLeft() >= 9000`, else the 5-direction passability heuristic ("if the passability cost
of moving straight is more than twice that of 45 degrees, move 45 degrees"). Ivan Geffner: unrolled BFS
over 97 tiles (muckraker) / 81 (politician) with bug fallback. Musketeers: 5x5 Dijkstra with a custom
`FastMinQueue`. awesomelemonade combobot3: a two-step look-ahead heuristic `2*ln(passability) +
0.8*(distance gained)` with a bug mode. *Ours:* one-step greedy `cheb + 1/passability` with bug fallback
(`Nav.step`). We measured that our units move as much as awesomelemonade's and step onto swamp less
(`TRAINING_LOG` 2026-09-19 16:40) and dropped the Dijkstra unrun -- probably right for us, but note that
Baby Ducks' post-mortem says their "complete lack of pathfinding was starting to hurt" by quals.

**Move-along-lattice navigation is cheaper than general navigation (inferred).** With units parked on
lattices, most movement is "to the nearest free lattice tile", a one-step choice; the expensive planner is
needed only for capturers and scouts.

### 1.5 Communication

**Dead-reckoning children by their last move (awesomelemonade).** Every unit flag carries the direction
it moved this turn (4 bits); the EC keeps a linked list of up to 60-80 children with their *predicted*
location and advances it by that direction each round (`CentralCommunication.loop: current.location =
current.location.add(dir)`). Because the EC therefore knows where each child stands, a unit can report
an enemy as a 4-bit dx, 4-bit dy offset from itself (`UnitCommunication.setFlag`), leaving 10 bits for
conviction. *Ours:* 14 bits for an absolute mod-128 location in every message (`Comms.encode`), 6 bits
of payload, and the EC does not know where its children are.

**Rotation with a heartbeat, and a 5th slot for enemy slanderers.** awesomelemonade's EC flag cycles
heartbeat / ally EC / enemy EC / neutral EC / enemy slanderer location every 5 rounds with 7-bit dx/dy and
9 bits of conviction; the heartbeat (dx=dy=0) lets readers synchronise which slot they are reading
(`setRotationFlag`, `processFlagsFromECs`). The slanderer slot feeds `MapInfo.enemySlandererLocations`,
a queue with turn stamps that muckrakers walk to (`Muckraker.goToCommunicatedSlanderers`); the first
sighting is also *predicted* from the first enemy unit's bearing via symmetry (`CentralCommunication`
"Enemy Prediction", `Util.findSymmetry`). *Ours:* enemy *units* are reported (`ENEMY_UNIT`) but never
rebroadcast by the centre and never consumed by muckrakers (`EC.updateBroadcast`, `Robot.absorb`), so
our hunters never knew where their slanderers were -- the mirror image of our open question 8.2.

**Paired messages carrying robot ids (rzhan11).** `HQ_LOC_PAIRED` followed next round by `*_HQ_INFO`
with the EC's 14-bit id, so every unit can `canGetFlag(id)` to learn whether a target EC is alive or
converted and read its flag directly; `BROADCAST_MY_MASTER` lets a newborn tell siblings its EC's id and
`REPORT_NON_MASTER` adds unknown friendly ECs to the read list (`Comms`, `HQTracker`). Messages live in a
normal queue and a *repeat* queue (map bounds, symmetry, HQ list) that cycles forever
(`CommManager`). *Ours:* `OWN_EC_ID` is relayed only by scouts in a 9-slot fact rotation and captured
centres were "born deaf" until Iteration 48 (`HANDOFF.md`).

**Hop-count ("pressure") fields for spreading and relaying (wololo).** Relay-chain units carry a 3-bit
pressure = min over visible chain members + 1 (0 at a unit that sees an enemy muckraker or a target), repel
units of equal or higher pressure and the map edges, and so form a chain that fills the map and funnels
sightings back; the EC reads `min_pressure > 5` as "map controlled" and switches production to slanderers
(`RelayChain.update`, `ECInfo.map_controlled`). Search snippets of the post-mortem describe this as a
"distributed Bellman-Ford" for guard layers. *Ours:* repulsion among muckrakers only (`spreadOut`), no
relayed field.

**Role encoded in the flag header (wololo, Musketeers, wstan2001).** wololo's top bits say relay/guard/
burier/targetter/exterminator/slanderer and the EC classifies its children from them
(`ECInfo.update` over `unclassified_ids`); Musketeers newborns read the EC flag's `SubRobotType` (16
politician roles) in the constructor; wstan2001 uses 4-bit opcodes MOVE/SCOUT/TROOP/BOMB/SLAND. *Ours:*
`ORDER` with 3 bits of role held for two rounds, only for captures (`EC.build`, Iteration 48 dose 4).

### 1.6 Symmetry and exploration

**Passability-based symmetry elimination (rzhan11).** Politician scouts compare passability at
`HardCode.BOX_EDGES` offsets with the mirrored tile under each surviving hypothesis once the relevant
bounds are known (`Map.updateSymmetryByPassability`), and HQ-based checks mark a symmetry wrong when the
image of a known EC is sensed empty; the EC broadcasts the 3 "not" bits (`SYMMETRY_MSG`). *Ours:*
`DESIGN.md` claims passability contradiction pruning but `MapState` implements only
`pruneWithEmptyTile` and `pruneWithEnemyEC` (read from `MapState.java`); the passability test was never
written.

**Explored-area bitmap (awesomelemonade).** `Cache.explored = int[32]`, one bit per 4x4 chunk around
spawn; `smartExplore` picks a random unexplored chunk among the 3x3 around the unit and marks a chunk
explored after 10 failed moves (`Util.smartExplore`, ~300 bytecodes). rzhan11: explore tasks in priority
order -- master's direction, unknown bounds, unconfirmed symmetric EC location, then a default direction
rotated by 1 or 3 on arrival. *Ours:* random far waypoints plus symmetry candidates split by id
(`Muckraker.pickExplore`); coverage 46% to their 93% in one measured loss.

### 1.7 Bytecode and engineering

- **Generated switch tables** for slanderer breakpoints and income (Baby Ducks `thresholds.py` ->
  `Threshold.java`; Musketeers `precompuation.py`; awesomelemonade `SlandererBuild`), one `switch`
  lookup per call. *Ours:* a 60-entry array scanned linearly (`Econ.bestSize`), fine for us.
- **Resumable flag-reading loops** bounded by `Clock.getBytecodesLeft()` (Baby Ducks `readAllFlags(8000)`
  keeps `flagLoopIndex` across turns; rzhan11 `MIN_MESSAGE_BYTECODE=5000` per role list; wololo 30 ids per
  class per turn through `IntCycler` ring lists; waffle). *Ours:* 24 children round-robin and off-turn
  sibling reads after the Iteration 49 overrun (`EC.readChildren`) -- the same idea, found the hard way.
- **Strings as sets** (naalit `StringSet`: `StringBuilder` ops cost 1 bytecode) and `javap` to inspect
  hot loops.
- **Nolog builds for submission** (rzhan11 `scripts/createNoLogSrc.py` strips `log(...)` calls into a
  `_nolog` package; `checkKeywords.py`). *Ours:* `Debug.log` behind `C.DEBUG` but string concatenation
  still runs when `DEBUG=true` (`Debug.java`, inferred).
- **Deterministic RNG is ours alone**: every top bot uses `Math.random()`; we seed an LCG from the id
  for reproducible mirrors (`Robot.nextInt`). Keep.

### 1.8 Tooling, testing and process

- Baby Ducks: `test.sh` runs two packages on 35 maps both sides into one log; `a.sh` clones a package
  (our `snapshot.sh`); the post-mortem's core process lesson is **"we started out writing two separate
  bots -- one that focused on rushing with muckrakers, and one that focused on slanderer economy -- and
  tested both against each other extensively"**, i.e. the sparring-partner idea from week one.
- awesomelemonade: a plain-Java unit test of the politician conviction-transfer formula
  (`combobot3/test/PoliticianTest.java`, 26 asserts) -- the same habit as our `test/bot`.
- Stone Tao: `findTeamExceptions.js` greps a match log for one team's exceptions and prints 10 lines of
  context. rzhan11: `scripts/run.sh` plus the nolog pipeline. naalit: `botsnapshot.py`, `sanity.sh`.
- Process lessons stated in the post-mortems and absent from ours: "Read the spec! ... I didn't realize
  that ECs could see flags at any range until halfway through week 2" (naalit); "Tweaks are never going
  to make your bot that much better, so unless you're near the top already" spend the first half of each
  week on new code (naalit); "maps win games" -- Finals used low-EC-passability maps that favoured
  slanderer turtles (Baby Ducks, Stone Tao).

---

## 2. Ideas we tried but could not make work, that top teams made work

| ours (iteration, verdict) | theirs | the difference |
|---|---|---|
| Relayed muckraker threats for slanderers (12, rejected +2 roster; 24 re-test 42.5%) | Baby Ducks raker flags with age; rzhan11 alert/echo; awesomelemonade EC "undefended muckraker" broadcast | theirs pairs the flee with a *lattice to flee into* and a defender sent to the same spot; ours fled into a ring that the hunter was already patrolling |
| Hunters with conviction (28, rejected 41%/30%) | buffrakers in every top bot, 25-50% of budget, periodic | gated in a mirror whose guards were 15-60 and whose slanderers were ringed by them; the buffraker's target is a 15-guard screen, which our twin also had but which never had to be *passed* because our hunters sat on the EC tile |
| EC wall of muckrakers (3) and diagonal sentinels (52), rejected | wololo/waffle/Geffner reactive shield only while an enemy politician is in range and the safety budget is negative | permanent bodies die to every speech and block spawns; a one-turn shield built *toward* the attacker does neither |
| Chip captures (44, rejected) | Baby Ducks/awesomelemonade sum friendly convictions at the EC and the last arrival fires | ours chipped alone and drained the bank; theirs wait on cardinal tiles until the sum flips |
| Big standing guards (19, rejected), rich interceptors (54/57/59, null) | guards sized to the muckraker seen (`c+20`, `c+11`, `c+13`) | size from the threat, not from the bank |
| The army: 300+ attackers instead of guards (13, rejected) | Baby Ducks: income target first, then 25%/50%/25% big P / small / buffraker; defenders charge the base after 10 quiet turns | theirs keeps the guard count and converts *surplus* income; ours replaced the guards |
| Save mode for the first neutral (22 accepted, 51 reverted) | Baby Ducks save with 1-muckrakers while `expectedTotalIncome > 3*cost`; rzhan11 stop bidding only when income covers the gap in 5 rounds | theirs is bounded by measured income; ours by a round count |
| More scouts (14, 38, 63; coverage up, no wins) | explored-chunk bitmap; explore tasks ordered bounds -> symmetry -> default; corners | coverage was the symptom; the knowledge sinks (neutral capture, slanderer hunting via the enemy-slanderer queue) are what cash it |
| Economy pause release (39, 43, rejected) | safety budget `influence - enemy politician power` as a standing deduction | same intent, but a budget lets slanderers be built *from the remainder* instead of all-or-nothing |
| Deposit: expired slanderers speak at the EC (15, 18, near miss) | awesomelemonade heal when `2 * enemy conviction near the EC > its conviction` | triggered by a threat, from any big politician, not by expiry |
| Target-aware attack (53/55, null; lives in `arch_lemon`) | wololo weakest-EC targetter priced with travel-time income; awesomelemonade centroid targeting; Baby Ducks `d^0.75` distance term | we lacked the distance/income term and a safety budget to pay from |

## 3. Ideas we had that top teams also had

- Slanderer breakpoint tables (21, 41, 63 ... 949) -- universal (Baby Ducks, rzhan11, Musketeers, Geffner).
- Slanderer at r1 before anything else (awesomelemonade `getRoundNum()==1`, rzhan11, Baby Ducks `r<3`).
- A few 1-influence muckraker scouts early (rzhan11 `EARLY_EXPLORER_COUNT=4` in directions chosen from
  the visible walls; awesomelemonade first-30-units cycle).
- Mod-128 coordinates decoded relative to the reader (Baby Ducks `flagToLoc`, rzhan11 `bits2loc`,
  awesomelemonade, wstan2001 README) -- identical to `Comms.loc`.
- The EC reads every child's flag by id and keeps a child list; dead children detected by `canGetFlag`.
- Symmetry candidates for the enemy EC from the map edges, broadcast as targets (Baby Ducks
  `sendNonfriendlyHQ` guesses all three and marks them enemy).
- Camouflaged slanderers keep their controller and switch to politician code (Baby Ducks `Slanderer
  extends Politician`, waffle `SlanPol`, awesomelemonade).
- Building slanderers away from the enemy, scouts spread around the ring (awesomelemonade
  `enemyDirection` accumulator, `getDistributedDirection`; Geffner min `dist/passability`).
- Bytecode overrun detection in the main loop (awesomelemonade prints "out of bytecodes", rzhan11
  `endTurn`, wololo) -- ours logs `@bc over=`.
- Expose the highest-influence slanderer in range (everyone).
- No `java.util` collections in hot paths (Baby Ducks use `HashSet` only in dead code).
- Unit tests for pure logic (awesomelemonade `PoliticianTest`, ours `test/bot`).

## 4. Sources

Post-mortems
- Baby Ducks: https://github.com/JasonYe4273/et-battlecode-2021 `postmortem/index.html` (mirror of
  http://web.mit.edu/agrebe/www/battlecode/21/index.html, blocked)
- naalit: https://github.com/naalit/battlecode21 `README.md`
- waffle: https://github.com/winkelmantanner/battlecode2021_waffle `README.md`
- Stone Tao: https://github.com/StoneT2000/Battlecode2021 `README.md`; https://blog.stoneztao.com/posts/bc21/ (blocked; WebSearch snippets)
- wololo: https://battlecode.org/assets/files/postmortem-2021-wololo.pdf (blocked; WebSearch snippets)
- 3 Musketeers: https://battlecode.org/assets/files/postmortem-2021-musketeers.pdf (blocked; WebSearch snippets)
- Ivy Zhang: https://ivyzhang.me/bc21-postmortem (DNS blocked; WebSearch snippets)
- Malott Fat Cats (4th): listed at https://battlecode.org/past.html; PDF not reachable, only
  second-hand (Stone Tao: "never used the muck buff ... insanely good micro"; jmerle 2022 reused their
  Dijkstra)
- wstan2001: https://github.com/wstan2001/Battlecode2021 `README.md`

Code (`owner_repo` = https://github.com/owner/repo)
- `JasonYe4273_et-battlecode-2021/src/finals/{Center,Politician,Muckraker,Slanderer,Robot,Navigation,UnrolledNavigation,Threshold}.java`, `test.sh`, `a.sh`, `thresholds.py`
- `awesomelemonade_battlecode-2021/sprint1bot/**` and `combobot3/**` (`EnlightenmentCenter`, `Politician`, `Muckraker`, `Slanderer`, `util/{CentralCommunication,UnitCommunication,CentralUnitTracker,Pathfinder,LatticeUtil,MapInfo,SlandererQueue,Util}`, `test/PoliticianTest`)
- `rzhan11_Battlecode2021/src/sprint2/{EnlightenmentCenter,Politician,Muckraker,Slanderer,Role,Comms,CommManager,HQTracker,Robot,Map,Message}.java`, `scripts/`
- `iliao2345_Battlecode2021/src/slands3/{EnlightenmentCenter,ECInfo,Muckraker,Politician,Slanderer,Guard,Burier,Targetter,RelayChain,Flag,CombatInfo,Pathing,RobotPlayer}.java`
- `BSreenivas0713_Battlecode2021/src/musketeerplayerfinal/{EC,RobotPlayer,Comms,Util,Nav,LatticeProtector,DiagonalSlanderer,BuffProtectorPolitician,HunterMuckracker,SpawnKillPolitician}.java`, `precompuation.py`
- `StoneT2000_Battlecode2021/src/maxecosushi/{EnlightmentCenter,Muckraker,Politician,Slanderer,Comms,Constants}.java`, `findTeamExceptions.js`
- `rqi3_battlecode-2021/src/qualification_bot/EnlightenmentCenter.java`
- `IvanGeffner_battlecode2021/thirtyone/{Enlightment,Pathfinding,BFSMuckraker}.java`

Ours (https://github.com/terryvanbelle/battlecode21-vibe)
- `POSTMORTEM.md`, `LEARNINGS.md`, `DESIGN.md`, `HANDOFF.md`, `RESEARCH.md`, `RULES.md`, `BENCHMARK.md`,
  `progress/ELO.md`, `progress/ONSET.md`, `TRAINING_LOG.md` (Iterations 3, 10, 12-15, 19, 22, 24, 28,
  38-39, 43-44, 47-63 and the sections "Watching a strong opponent build", "What three opponents do
  that we do not", "Navigation: the hypothesis was checked and dropped", "A last diagnosis for the next
  competitor"), `src/bot/*.java`, `src/arch_lemon/C.java`, `tools/README.md`, `test/bot`.
