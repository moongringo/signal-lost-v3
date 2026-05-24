# Team System Refactor — Dynamic Team Names

## Goal
Replace hardcoded North/South team system with player-defined team names. Host picks a team on setup, joiners pick their team. Keep competitive team-vs-team scoring + AI threats. Teams compete for objectives while AI patrols the map.

## Design

### Team Assignment Flow
1. **Lobby screen** — Host and Join both get a "Team Name" input with a preset dropdown + free text field
2. **Host** — picks/enters their team name, clicks Host Mission → their team is set
3. **Joiner** — enters a team name (same presets + free text), clicks Join → joins with that team
4. **Bots** — assigned to the host's team (they're squad fillers, not independent)
5. **Team chat** — filters by team name, squad members only see their team's chat

### Faction (flavor layer, separate from team)
- Optional allegiance with presets + free text
- Shows in profile card, no gameplay effect
- Presets: "Phoenix Initiative", "Ghost Protocol", "Crimson Vanguard", "Iron Pact", "Shadow Collective"

### Team Colors
- Generate from a hash of the team name → picks from a palette
- Same team name always gets the same color
- Palette: ['#4fc3f7','#ff8a65','#81c784','#ba68c8','#ffd54f','#e57373','#4dd0e1','#aed581']

## Changes Needed

### index.html
- Lobby host card: add team name input (datalist + free text field) after callsign
- Lobby join card: same team name input after callsign
- Legend: replace "Squad North" / "Squad South" with "Your Squad" / "Other Squad"

### game-v2.js — All North/South replacements
1. `state.scores` init: `{ North: 0, South: 0 }` → `{}` (filled dynamically via teamName)
2. All `team: 'North'` fallback → `team: ''` or `team: state.teamName`
3. All `localTeam = state.isHost ? 'North' : 'South'` → `localTeam = state.teamName`
4. Squad color logic: `a.team === 'North' ? '#4fc3f7' : '#ff8a65'` → hash-based color lookup
5. Bot team assignment: `i % 2 === 0 ? 'North' : 'South'` → host's team name
6. TeamBalance: `{ North: [], South: [] }` → `{}` (dynamically keyed by team name)
7. Scores: `state.scores[local.team]` → works with any string key already
8. Chat team filter: uses team name string comparison
9. Roster/briefing team badge: `localTeam.toLowerCase()` → `getTeamColor(localTeam)`

### New utility functions to add:
- `getTeamColor(teamName)` — returns color from hash-based palette
- `teamPresets` — array of preset team names for the datalist
- `factionPresets` — array of preset faction names

### server.js (socket.io)
- No changes needed for scores — they're already keyed by team name string
- Team assignment uses whatever teamName the player sends

### styles-v2.css
- `.team-badge.north` / `.team-badge.south` → need dynamic badges (no fixed class names)
- Replace with `.team-badge` that uses inline style for color or add a generic `.team-badge-1` through `.team-badge-8` pattern

## Execution Order
1. Add team name inputs to index.html (lobby host + join cards, faction input optional)
2. Add getTeamColor() and presets to game-v2.js
3. Replace all North/South hardcoded references in game-v2.js
4. Update styles-v2.css dynamic badges
5. Update server.js if needed
6. Commit
