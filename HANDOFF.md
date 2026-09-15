# Developer Handoff & Technical Guide

## Project Overview

**Kill Team Scorecard** is a lightweight, zero-build static web application designed to track games of Kill Team. It is tailored specifically for:
- Live horizontal playback on an **iPad 13" Pro** (1366 × 1024) in landscape mode.
- Real-time streaming scoreboards in **OBS Studio** via Browser Sources.
- Peer-to-peer multiplayer state synchronization across connected browser sessions.

---

## File Structure

```text
Kill Team Scorecard/
├── index.html                     # Monolithic app (HTML markup, CSS styles, JavaScript logic)
├── README.md                      # Project documentation and OBS guide
├── HANDOFF.md                     # Technical architecture & developer handoff
├── data/
│   ├── kill-teams.json            # Kill Team names and assigned archetypes
│   ├── starting-kill-grade.json   # Starting non-expendable operative counts per team
│   ├── kill-grade.json            # Threshold matrix for calculating Kill Op points
│   └── tac-ops.json               # Tac Op cards, archetypes, and descriptions
└── .github/workflows/
    └── pages.yml                  # GitHub Actions workflow for GitHub Pages deployment
```

---

## Core Architecture & State Model

### 1. In-Memory State (`state`)
```javascript
const state = {
  turningPoint: 0,       // Integer: 0 to 4
  battleEnded: false,    // Boolean: true after "END" button is clicked at TP 4
  initiative: 'left',    // 'left' | 'right'
  left: {
    team: '',            // Selected team name string
    startingOperatives: '', // Selected starting non-expendable operative count (e.g. "10", "12")
    operativesRemaining: 0, // Current remaining operatives
    modifiers: { reroll: false, one: false, two: false, three: false }, // Checkbox flags
    scores: { cp: 3, crit: 0, tac: 0 }, // Score steppers (CP defaults to 3, max 12)
    tacOp: '',           // Selected Tac Op name
    primary: ''          // 'crit' | 'kill' | 'tac' (Primary Op revealed at battle end)
  },
  right: { ... }         // Mirrored structure for right player
};
```

### 2. URL State Encoding (`writeUrlState` / `readUrlState`)
The app automatically synchronizes score and match state to URL query parameters on every mutation.

- **Included in URL**: `game`, `tp`, `battleEnded`, `initiative`, `leftTeam`, `rightTeam`, `leftRemaining`, `rightRemaining`, `leftKill`, `rightKill`, `leftCp`, `rightCp`, `leftCrit`, `rightCrit`, `leftTac`, `rightTac`, `leftTotal`, `rightTotal`, `leftTacOp`, `rightTacOp`, `leftPrimary`, `rightPrimary`.
- **Intentionally excluded from URL**: `startingOperatives` and `modifiers` (only synced live via PeerJS to keep URL clean and focused on stream/score overlays).
- **Backward Compatibility**: Handles legacy URLs where `tp >= 5` by interpreting them as `tp = 4` with `battleEnded = true`.

### 3. P2P Live Synchronization (PeerJS)
- The app uses PeerJS with room ID convention `kt-scorecard-${gameName}`.
- The first client to connect on a `game` ID claims the host role (`sync.host = true`).
- Subsequent clients connect to the host and receive state updates automatically on any change.

---

## Game Rules & Business Logic

### Starting Non-Expendable Operatives
- Loaded from `data/starting-kill-grade.json`.
- If a Kill Team has **only 1 option** (e.g. 10): the secondary dropdown is hidden (`operatives-hidden` / `visibility: hidden`) and the starting operative count is set automatically.
- If a Kill Team has **multiple options** (e.g. Brood Brothers with [13, 11, 10, 12]): the secondary dropdown becomes visible and defaults to the placeholder `"Starting Non-Expendable Operatives"`. Operatives remaining remains 0 until a selection is made.

### Kill Op Calculation
1. Opposing killed operatives = `opponent.startingOperatives - opponent.operativesRemaining`.
2. Thresholds are looked up in `data/kill-grade.json` for the opponent's starting operative count.
3. Raw Kill Op score is the number of thresholds met (0 to 6).
4. **End Battle Bonus**: If `state.battleEnded === true` and a player's raw Kill Op strictly exceeds the opponent's raw Kill Op, +1 bonus Kill Op point is awarded (capped at 6).

### Total Victory Points (VP) Formula
$$\text{Total VP} = \text{Crit Op} + \text{Kill Op} + \text{Tac Op} + \left\lceil \frac{\text{Primary Op Score}}{2} \right\rceil$$

### Turning Point & End Battle Flow
- **Turning Point 0–3**: Stepper changes TP normally.
- **Turning Point 4**: The `+` button transforms into the **`END`** button.
- **Clicking `END`**:
  - Sets `state.battleEnded = true`.
  - Disables the `END` button.
  - Unlocks the **Primary Op** dropdowns for left and right players.
  - Calculates the final Kill Op comparison bonus.
  - Highlights the winner in the Total VP display.

### Modifier Controls Progressive Availability
Modifier checkboxes are displayed below the Starting Operatives row:
- **Left order**: `Re-Roll`, `-/+ 1`, `-/+ 2`, `-/+ 3` (aligned left).
- **Right order**: `-/+ 3`, `-/+ 2`, `-/+ 1`, `Re-Roll` (aligned right).
- **Unlocking by TP**:
  - TP 0: Only `Re-Roll` is interactive.
  - TP 1: `-/+ 1` unlocks.
  - TP 2: `-/+ 2` unlocks.
  - TP 3+: `-/+ 3` unlocks (all active through TP 4).
  - When locked, modifiers have `opacity: 0.45` and `disabled: true`.

---

## UI Layout & CSS Architecture

### Grid Columns System
CSS custom property `--layout-columns` governs alignment across `.setup-grid`, `.layout-row`, and `.totals`:
```css
.score-layout {
  --layout-columns: minmax(0, 1.5fr) 56px minmax(85px, .85fr) minmax(70px, .75fr) minmax(118px, 1fr) minmax(70px, .75fr) minmax(85px, .85fr) 56px minmax(0, 1.5fr);
}
```
- Column 1: Left team select / Left dropdowns / Left credit text.
- Column 2–3: Left stepper buttons & score outputs.
- Column 4–6: Center rail (Initiative arrows, Turning Point, Center labels, Total VP label).
- Column 7–8: Right stepper buttons & score outputs.
- Column 9: Right team select / Right dropdowns / Reset & New Game buttons.

### Tac Op Archetype Color Coding
Options in the Tac Op select element are styled with background colors based on their archetype:
- `Security`: `--tac-op-security` (`#142f5c` - Dark Blue)
- `Seek and Destroy`: `--tac-op-seek-and-destroy` (`#5b1c24` - Dark Red)
- `Recon`: `--tac-op-recon` (`#704018` - Dark Orange)
- `Infiltration`: `--tac-op-infiltration` (`#292d30` - Charcoal)

### Action Buttons & Credit
- "Made by Shawn the Lugger" and "New game" / "Reset game" buttons are housed directly inside the `Total VP` row (`.layout-row.totals`), precisely aligned with the left and right team dropdown bounds.

---

## Key Development Guidelines for Future Changes

1. **Keep Single-File Simplicity**: All runtime styles and scripts are consolidated in `index.html` to avoid build steps and maintain instant deployment.
2. **Preserve Viewport Constraints**: The app is tuned for iPad 13" landscape (1366 × 1024). Keep layout heights and paddings balanced when modifying grid row heights.
3. **URL State Minimalism**: When adding new fields, determine whether they belong in the URL query string (for OBS overlays) or should only live in runtime/PeerJS state.
4. **Mobile Breakpoint**: Any changes to `.setup-grid`, `.layout-row`, or control sizes must have corresponding overrides inside `@media (max-width: 760px)`.
