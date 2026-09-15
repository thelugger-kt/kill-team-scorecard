# Kill Team Scorecard

A responsive two-player Kill Team scorecard optimized for horizontal iPad play and OBS streaming overlays.

## Features

- **Live Two-Player Scorecard**: Tracks CP, Crit Op, Kill Op (via operatives remaining thresholds), Tac Op, and Total VP for left and right players.
- **CSV/JSON-Driven Operatives**: Automatically resolves starting non-expendable operative counts per Kill Team from `data/starting-kill-grade.json`. Single-count teams auto-select; multi-option teams present a dropdown defaulting to placeholder.
- **Tac Op Archetype Styling**: Colors Tac Op dropdown options according to archetype (Dark Blue for Security, Dark Red for Seek and Destroy, Dark Orange for Recon, Charcoal for Infiltration).
- **Progressive Modifier Tracking**: Modifier checkboxes for Re-Roll and dice modifiers (-/+ 1, -/+ 2, -/+ 3) unlocking progressively by Turning Point (TP 0: Re-Roll only; TP 1: -/+ 1; TP 2: -/+ 2; TP 3+: -/+ 3).
- **Battle Flow & End Game**: Turning Point progresses from 0 to 4. At TP 4, the `+` button transitions to `END`, unlocking Primary Op selection, computing final kill op bonuses, and declaring the winner.
- **Live PeerJS Synchronization**: Real-time peer-to-peer state synchronization across multiple browsers connected to the same game ID.
- **URL-Encoded State for OBS**: Fully serializes game state to query parameters on every update, allowing streaming tools like OBS Browser Source to capture live scoreboard snapshots.
- **Integrated Action Controls**: "New game" (creates fresh isolated match) and "Reset game" buttons integrated into the Total VP row.

## Run Locally

Because the scorecard fetches local JSON data files, serve the directory through a local web server:

```bash
python -m http.server 8000
```

Open `http://localhost:8000` in your browser.

## GitHub Pages Deployment

The repository includes a GitHub Actions workflow (`.github/workflows/pages.yml`) that deploys directly to GitHub Pages.

1. Ensure the repository has GitHub Pages enabled under **Settings > Pages** with **GitHub Actions** as the source.
2. Pushes to `main` will automatically build and deploy.

Published URL: `https://thelugger-kt.github.io/kill-team-scorecard/`

## URL Query Parameters & OBS Integration

The scorecard automatically keeps the URL query string updated with the current game state.

### Query Parameters

| Parameter | Description |
|-----------|-------------|
| `game` | Unique room/match identifier used for PeerJS connection |
| `tp` | Current Turning Point (`0` - `4`) |
| `battleEnded` | Present and set to `'true'` when the game has ended |
| `initiative` | Current player with initiative (`left` or `right`) |
| `leftTeam` / `rightTeam` | Selected Kill Team name |
| `leftRemaining` / `rightRemaining` | Operatives remaining count |
| `leftKill` / `rightKill` | Calculated Kill Op score |
| `leftCp` / `rightCp` | Command Points (`0` - `12`, default `3`) |
| `leftCrit` / `rightCrit` | Crit Op score (`0` - `6`) |
| `leftTac` / `rightTac` | Tac Op score (`0` - `6`) |
| `leftTotal` / `rightTotal` | Total calculated Victory Points (VP) |
| `leftTacOp` / `rightTacOp` | Selected Tac Op name |
| `leftPrimary` / `rightPrimary` | Selected Primary Op category (`crit`, `kill`, `tac`) |

### OBS Browser Source

Add a Browser Source in OBS pointing to your match URL:
```text
https://thelugger-kt.github.io/kill-team-scorecard/?game=match-abc123
```
Any updates made in the host controller browser will update the URL state and sync live across connected clients.
