# Best practices

Practices that pay off in every Battlecode year, whatever the rules. Add to this list as new ones prove themselves.

## 1. Have a thorough, complete replay dumper, and use it effectively

The replay file is the only complete record of a game. For online scrimmages it is your only view of how opponents
play. Build the replay dumper early, and treat it as core infrastructure rather than a side tool.

- **Thorough and complete.** Extract everything the replay format carries:
  - map and initial state;
  - per-round team data;
  - per-turn unit state;
  - every action and event, with its actor, target and amount;
  - indicator strings, dots and lines;
  - timeline markers;
  - profiler data.

  Check completeness field by field against the year's replay schema and the engine code that writes it. Also check
  it against what the official client shows: anything the viewer displays is something you can extract too.
- **Know what the format does not record.** Some events are never written to the replay, or are written only in some
  cases. Document those gaps so that analysis tools fail loudly instead of quietly reporting zero.
- **Smuggle debug information through the bytecode count.** Tournament and scrimmage replays often strip debug output
  such as indicator strings, but they keep each robot's bytecodes used per turn. To recover per-robot information,
  pad the end of every turn with no-op steps, so that the low-order bits of the turn's bytecode count encode a small
  value: a role, a decision code, or a target index.
  - Pad with straight-line steps of known cost (for example, a fall-through switch of one-instruction increments),
    so the count is exact rather than approximate.
  - Pad only when the remaining bytecodes leave a safe margin.
  - Decode the value in the replay dumper.
  - Verify the channel on a local game, where indicator strings survive, by checking that the decoded value matches
    the indicator string on every turn.
- **Verified.** Cross-check each dump against the official result: winner, round count, win reason and team totals.
  Unit-test the dumper on a real replay.
- **Used effectively.**
  - Dump each replay once, cache the result, and share it across every analysis tool.
  - Aggregate over many replays (all games against the field, or the top opponents), not just one game.
  - Point the reports at why games are lost: how each unit died, where resources flowed, combat exchanges,
    and each component of the scoring rules.
- **Audit it regularly.** When a question about a game can't be answered from the dump, the dumper is incomplete.
  Fix the dumper, not the question.

## 2. Spend diagnostic time on losses against opponents in the 25-75% band

A candidate improves the bot only by flipping results, and results flip only where games are close. Against an
opponent you beat 10% of the time, every build near yours loses the same games. A fix aimed at that opponent rarely
shows in any measurement, and the time spent on it is lost.

- **Filter before you tally.** A loss tally over all games is dominated by the strongest opponents, because they
  supply most of the losses. In one season, two bots beaten 3 of 24 supplied 42 of 68 lost games, and five
  candidates in a row were built against them.
- **Where to look:** losses against opponents you beat 25-75% of the time, games whose result differs by side or
  between builds, and maps where results are close.
- **Measure the band by win rate, not rating.** Ladder ratings are compressed and misplace opponents: a bot rated just
  above us won 21 of 24 games. Run a calibration sweep of the current bot against the ladder to find the band.
- **Stronger bots are a source of ideas, not targets.** Watch what they do that you don't, but don't build fixes
  against them. They move into the band as the bot improves.

## 3. Periodically trim dead code and audit for bugs

A bot that grows by trying idea after idea piles up code that no longer runs: switched-off experiments, unused
constants, helpers for a strategy that was dropped. It also hides bugs that measurements never isolate, because each
one costs only a few games. Both get worse with time. Schedule a cleanup and a bug audit at milestones (after a big
feature lands, before a long tuning phase, on a plateau), not only when something breaks.

- **Trim first, behaviour-exact.** Delete code that is dead under the current settings and merge duplicated logic
  into shared helpers, changing no behaviour. Verify it with identical replays: the trimmed bot must play the same
  games as the old one, turn for turn, across maps, seeds and sides. A smaller code base makes the audit that follows
  much easier to do well.
- **Then audit for bugs, area by area.** Give each reviewer one area (movement, combat, economy, comms, HQ logic, and
  so on), and ask for concrete bugs with a location and the game situation that triggers them. Verify every suspect
  adversarially before fixing it; plausible-looking bugs are often intended behaviour.
- **Fix behind flags, and measure.** Put each fix behind its own flag, whose off position reproduces the old
  behaviour exactly, so fixes can be bisected. Check that all flags off still replays the old bot, then measure the
  fixes together against the old bot and the field. Many small fixes that are each invisible in a test can add up to
  a clear gain.
- **Keep the two passes separate.** Mixing cleanup with behaviour changes makes a regression impossible to place.
  Exact cleanup first, verified by replay; fixes second, verified by games.
- **Update the test suite afterwards.** Remove or rewrite tests for code that was deleted, and point tests at the
  shared helpers that replaced duplicated logic. Add a regression test for each verified bug that recreates the game
  situation that triggered it. Then refresh any saved replays or baselines to the cleaned, fixed bot, so the next
  cleanup starts from a suite that matches the code.
- **Finally, remove the fix flags.** The flags exist only to bisect and measure the fixes. Once the fixes are
  accepted, delete each flag together with its old code path, keeping only the fixed behaviour, and drop any test that
  exercises the flags-off build. A flag left in place is a switched-off branch, which is exactly the dead code the
  next cleanup would have to remove. A fix that measured badly is reverted outright, not left behind an off flag.

## 4. Try an aggressive rush early, and keep a rush archetype to train against

Year after year, bots that open with an aggressive rush have done well, and top teams have had to build defences aimed
at one. A rush is also simple to write, so it makes a good first strategy. Whether or not it becomes the main plan, a
frozen rush bot is the sparring partner that tests your defence.

- **The evidence, year by year.**
  - 2019: "rushes beat aggressive expansion".
  - 2020: "rush had no easy defence"; even the winner's rush defence lost to one stacked rush.
  - 2021: a top team wrote a rush bot and an economy bot in week one and tested them against each other.
  - 2022: finalists sent soldiers from turn 5 on small maps.
  - 2023: "launcher rushing was highly effective", in what one team called "the rush meta".
  - 2024: a top team's micro had a special mode for one opponent's rush.
  - 2025: soldier rushes toward the mirrored enemy start.
  - 2026: "the payoff for early aggression ... pretty much always outweighs the uncertain outcomes of cooperation".
- **Try it as an early strategy.** Write a plain rush in the first days: find the enemy base from map symmetry, send
  the first units straight there, and attack whatever decides the game. Measure it against the field. Where it
  loses, find out when and why: map size, path distance, or the defender's response.
- **Choose per map from measurements.** Rushes tend to win on small, close maps and lose on large ones. Measure the
  rush distance as path cost, not Euclidean distance.
- **Keep a rush archetype frozen, and train against it.**
  - Put it in every evaluation basket next to turtle, expander and harasser archetypes. Self-play alone cycles, and
    may never find the rush.
  - For a learned bot, give the rush a fixed share of the training opponents, so the policy has to learn to survive
    it. Hold out a variant with different timing or targets to check that the defence generalises.
  - Strengthen it once the main bot beats it nearly every game: an opponent beaten every time no longer measures
    anything.
- **Defend without freezing the economy.** Standing defences placed before the rush arrives beat reactive ones; bots
  that wait for evidence of a rush lose more. Time-box and cap emergency responses. In rush against rush, the side
  that does not defend usually wins.
- **Grade it on the whole field.** Results between bots aren't transitive: an internal rusher can beat the incumbent
  and still rate well below it on the field. Judge a rush, and the defence against it, by the score against the whole
  field, not by one head-to-head.
