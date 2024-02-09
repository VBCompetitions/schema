# VBCompetitions/schema

A schema definition for a general volleyball competition

## Basic description

The schema defines a format for representing a competition of some kind.  This may be a single tournament, a league or all of the leagues run by an organisation.

The competition itself can be thought of as:

- a list of teams, where a team contains:
  - a team name
  - a list of players (optional)
  - a list of contacts (optional)
- a list of stages, where a stage is a grouping of matches.  Stages happen in order, i.e. a pool stage happens before a knockout stage.  A stage contains:
  - a list of groups, where a group is a grouping of matches.  Groups can happen in parallel, i.e. all of the groups in a pool stage happen independently of each other, A group contains:
    - some configuration for the matches, e.g. is this a league and how are the league points defined, are matches played to sets or continuously, are draws allowed etc
    - a list of matches, where a match contains:
      - a match ID
      - a home team
      - an away team
      - lots of other optional data about the match

There are some examples of competition definitions in `json/examples`

## Team References

Any field in a match that can refer to a team can either use that team's ID directly or use a team reference, as defined below.  This allows us to define matches where the teams depend on previous results, i.e. a knockout stage that contains the winners from multiple groups in a pool stage

A team reference is defined as follows:

```
team reference = {STAGE-ID:GROUP-ID:TYPE-INDICATOR:ENTITY-INDICATOR}

STAGE-ID - the ID for a stage

GROUP-ID - the ID for a group

TYPE-INDICATOR - an indicator of what to look up in the group
               - one of:
                 - MATCH-ID - to look  up a match result
                 - "league" - to look up a league position

ENTITY-INDICATOR - an indicator of what to look up within the type
                 - For TYPE-INDICATOR = MATCH-ID, one of:
                   - "winner" - the match winner
                   - "loser" - the match loser
                 - For TYPE-INDICATOR = "league", this is the league
                  position as an integer, e.g. "1" will return the
                  league winner
```

For example:
- `{STG1:GRP1:M1:winner}` refers to the winner of match `M1` in group `GRP1` in stage `STG1`.
- `{STG1:GRP1:league:2}` refers to the team that is in 1st place in the league in group `GRP1` in stage `STG1`.

### Ternaries

Sometimes you may want to make logic decisions on which team to choose for a team reference.  For example, imaging you have a tournament where teams referee each other, and that competition consists of multiple stages.  You may find that there are matches where it is possible that a team may referee the last game in one stage and then the first game in the next stage.

To avoid this scenario, you can use a ternary statement.  This lets you say "if 'this' team is the same as 'that' team then resolve this reference to 'another' team, otherwise use 'this' team".  The syntax is as follows:

```
{TEAM-A}=={TEAM-B}?{TRUTHY-TEAM}:{FALSY-TEAM}
```

For example:
- `{S1:G1:league:1}==TM4?{S1:G1:league:2}:TM4` if the league winner in stage `S1`, group `G1` is `TM4` then resolve to the team in 2nd place, otherwise resolve to `TM4`
- `{S1:G2:M1:winner}=={S1:G2:M3:winner}?{S1:G2:M1:loser}:{S1:G2:M1:winner}` if the winner of match `M1` in stage `S1`, group `G1` is the same as teh winner of match `M3` then resolve to the loser of match `M1`, otherwise resolve to the winner

## Player References

The player list in the home/away teams in a match may contain a list of player names, or player references (or a mix of both).

A player reference is defined as follows:

```
player reference = PLAYER-ID
                 | {PLAYER-ID}
                 | {TEAM-ID:PLAYER-ID}

TEAM-ID - the ID for the team the player is in
        - when this is not present, it is assumed that the player is defined as part of the
          home/away team that is playing that match
        - when this is present, this may refer to any team defined in the competition.  This
          allows players to "play up"/"play down" for a team other than their own.  Any
          restriction on whether the player is in a team from the same club is not part of
          the specification

PLAYER-ID - the id for the player
```

The `TEAM-ID` and `PLAYER-ID` can be any ASCII character except the following: `"` `:` `{` `}` `?` `=`

Note that the schema does not verify that the referenced player is defined, but implementations MUST
