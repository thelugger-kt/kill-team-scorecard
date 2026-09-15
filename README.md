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

Scores are intentionally held in memory only and are cleared when the page is refreshed or reset.
