# Token Time

A single-file web app for the family token system (Matilda and Brooks). No build step: open `index.html` or serve the folder as static files.

See [HANDOFF.md](HANDOFF.md) for the rules, data model and known issues.

## Files
- `index.html`: the whole app.
- `manifest.webmanifest` and `icons/`: lets the app install to a home screen and open full-screen.

## Hosting on GitHub Pages
1. Push this folder to a GitHub repo.
2. Repo Settings → Pages → Source: "Deploy from a branch", branch `main`, folder `/ (root)`.
3. Open the Pages URL on the iPad in Safari, then Share → Add to Home Screen.

Data is saved per browser (localStorage), so each iPad keeps its own tokens until a sync backend is added.

## Run locally
```bash
python -m http.server 8000
```
Then open http://localhost:8000.
