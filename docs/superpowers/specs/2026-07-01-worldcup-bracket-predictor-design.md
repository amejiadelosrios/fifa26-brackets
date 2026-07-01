# World Cup 2026 Bracket Predictor — Design

## Purpose

A local, no-build, single-user web page for predicting the knockout stage of the
2026 FIFA World Cup by dragging country flags forward through the bracket. Inspired
by the visual bracket on FIFA's official standings page
(https://www.fifa.com/en/tournaments/mens/worldcup/canadamexicousa2026/standings).

## Scope

Knockout stage only (Round of 32 through the Final, plus the third-place match).
The group stage is out of scope: the 32 Round-of-32 participants and their bracket
positions are supplied as fixed input data, not predicted by the user.

## Architecture

Static site, no build step, no server. Opened directly as a local HTML file.

```
fifa26-brackets/
├── index.html   # bracket DOM structure (rounds, match slots, connectors)
├── style.css    # bracket layout, flag sizing, drag/drop visual feedback
├── app.js       # state, rendering, drag & drop, localStorage persistence
├── teams.js     # editable data: 32 teams in fixed Round-of-32 bracket order
└── flags/       # locally downloaded SVG flags for the team pool (~48 countries)
```

### teams.js

An array of 32 entries, one per Round-of-32 bracket slot, in bracket order (so
adjacent pairs form the 16 Round-of-32 matches in the correct on-screen position):

```js
const TEAMS = [
  { name: "Mexico", code: "mx" },
  { name: "Poland", code: "pl" },
  // ... 32 total
];
```

`code` is the ISO 3166-1 alpha-2 code used to resolve the flag file at
`flags/{code}.svg`. Pre-populated with a best-effort bracket of the teams expected
to qualify for the 2026 World Cup (as known as of this session); the user can edit
this file directly to correct or rearrange teams — no UI is provided for editing
the initial 32.

### Bracket structure

Rounds, generated in `app.js` from `TEAMS`:

- Round of 32: 16 matches (fixed from `teams.js`)
- Round of 16: 8 matches (empty until predicted)
- Quarterfinals: 4 matches
- Semifinals: 2 matches
- Third-place match: 1 match (fed by the two semifinal losers' slots — populated
  only once both semifinal picks are made; user still must drag the winner into
  place same as any other match)
- Final: 1 match
- Champion: 1 slot, highlighted with a gold border

Every match slot beyond Round of 32 starts empty with a dimmed "Winner of match N"
placeholder until a flag is dropped there.

## Interaction

- **Drag a flag forward**: drag either team's flag from a decided match into the
  empty slot of the match it feeds into, to record that team as your predicted
  winner.
- **Change a pick**: drag the other flag from the same source match onto an
  already-filled next-round slot to overwrite the previous pick. Overwriting a
  pick clears any picks that were already made downstream of it (since those
  predictions assumed the old team).
- **Invalid drops** (e.g., dropping a flag onto a slot it doesn't feed into) are
  rejected with no state change.
- **Reset button**: a single "Reiniciar predicciones" control clears all
  downstream picks back to the fixed Round-of-32 state.

## Persistence

All picks (which team was dragged into each post-R32 slot) are saved to
`localStorage` on every change and restored on page load, so predictions survive
closing and reopening the file. No backend, no accounts — single browser/profile
only.

## Visual design

- Columns laid out left-to-right: Round of 32 → Round of 16 → Quarterfinals →
  Semifinals → Final, with classic bracket connector lines between matches.
  Third-place match rendered separately, near the Final.
- Each flag slot: ~32×24px flag image + country name label.
- Drag feedback: valid drop targets highlight on drag-over; the champion slot
  gets a distinct gold border once the Final is decided.

## Data sourcing

Flags are downloaded once during setup from flagcdn.com (SVG format, e.g.
`https://flagcdn.com/{code}.svg`) into the local `flags/` folder, covering the
~48 countries expected to be involved, so the page works fully offline afterward.

## Out of scope (this iteration)

- Group stage predictions.
- Editing the initial 32 teams via UI (edit `teams.js` directly instead).
- Multi-user / sharing / export of predictions.
- Scoring predictions against real results.
