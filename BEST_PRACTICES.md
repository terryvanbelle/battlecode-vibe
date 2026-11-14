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
`Cfg` constants, helpers for a strategy that was dropped. It also hides bugs that the season's measurements never
isolate, because each one costs only a few games. Both get worse with time. Schedule a cleanup and a bug audit at
milestones (after a big feature lands, before a long tuning phase, on a plateau), not only when something breaks.

- **Trim first, behaviour-exact.** Delete code that is dead under the current constants and merge duplicated logic
  into shared helpers. Change no behaviour. Verify it with identical replays: the trimmed bot must play the same games
  as the old one, turn for turn, on a set of maps, seeds and sides. In one season the main bot went from 10,403 to
  5,948 lines: 7 whole files and 227 unused constants were removed, and it replayed the old bot 60/60 identically.
  The smaller code base made the audit that followed much easier to do well.
- **Then audit for bugs, area by area.**
  - Give each reviewer one area (movement, combat, economy, comms, king/HQ logic, and so on).
  - Ask each reviewer for concrete bugs with a file and line, and for the game situation in which the bug triggers.
  - Verify every suspect adversarially before fixing it; plausible-looking bugs are often intended behaviour.
  - In the same season, six area reviewers found 21 suspects, and 16 survived verification.
- **Fix behind flags, and measure.** Put each fix behind its own flag, whose off position reproduces the old
  behaviour exactly, so fixes can be bisected. Check that all flags off still replays the old bot. Then measure all
  fixes on together against the old bot and the field. The 16 fixes together scored 40/64 against the bot they came
  from, with no loss against the field. No single one of them would have shown up in a 64-game test.
- **Keep the two passes separate.** Mixing cleanup with behaviour changes makes a regression impossible to place.
  Exact cleanup first, verified by replay; fixes second, verified by games.
