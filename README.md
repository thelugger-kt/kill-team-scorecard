# Kill Team Scorecard

A static two-player Kill Team scorecard for horizontal iPad play.

## Run locally

Because the scorecard loads JSON data files, serve the folder through a local web server:

```bash
python -m http.server 8000
```

Open `http://localhost:8000` in your browser.

## GitHub Pages

1. Create a new repository named `Kill Team Scorecard` under `thelugger-kt`.
2. Upload this folder to the repository.
3. In **Settings > Pages**, choose **GitHub Actions** as the source.
4. The included workflow publishes the app automatically.

The expected URL is `https://thelugger-kt.github.io/kill-team-scorecard/`.

## URL state and OBS

Opening the base URL creates a fresh game ID automatically. The first browser becomes the host, and other browsers receive score changes live only when they use the exact same `?game=...` URL through a direct peer connection. The current scorecard is also encoded into the URL as query parameters, so each connected browser and OBS can expose the latest snapshot.

The main parameters are `leftCp`, `rightCp`, `leftCrit`, `rightCrit`, `leftTac`, `rightTac`, `leftRemaining`, `rightRemaining`, `leftKill`, `rightKill`, `leftTeam`, `rightTeam`, `leftTacOp`, `rightTacOp`, `leftPrimary`, `rightPrimary`, `tp`, and `initiative`.

Example game: `https://thelugger-kt.github.io/kill-team-scorecard/?game=match-1`

OBS uses the same game URL: `https://thelugger-kt.github.io/kill-team-scorecard/?game=match-1`

Use the **New game** button to generate another isolated game URL. Share the generated URL to connect another browser or OBS to that game. Do not reuse the same `game` value for separate games.

Example encoded state: `https://thelugger-kt.github.io/kill-team-scorecard/?game=match-1&leftCp=2&rightCp=1&leftRemaining=5&rightRemaining=4&leftKill=1&rightKill=2&tp=3&initiative=left`
