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
- **Verified.** Cross-check each dump against the official result: winner, round count, win reason and team totals.
  Unit-test the dumper on a real replay.
- **Used effectively.**
  - Dump each replay once, cache the result, and share it across every analysis tool.
  - Aggregate over many replays (all games against the field, or the top opponents), not just one game.
  - Point the reports at why games are lost: how each unit died, where resources flowed, combat exchanges,
    and each component of the scoring rules.
- **Audit it regularly.** When a question about a game can't be answered from the dump, the dumper is incomplete.
  Fix the dumper, not the question.
