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

## Sync across phones and iPads
Tokens sync live through Firebase Firestore (free plan). The config goes in `FIREBASE` near the top of the saving section in `index.html`; the database rules are in `firestore.rules`.

Each device joins the family once by opening the family link (`…/token-time/#family=CODE`). The link is kept in `family-link.txt`, which is git-ignored because anyone with the code can see and change the tokens. Never commit it.

On iPhone or iPad: open the family link in Safari first, then Share → Add to Home Screen, so the home screen app keeps the code. If a home screen app ever shows "Only on this device · tap to link", tap that label and paste the link.

Until `FIREBASE` is filled in, data is saved per browser only (localStorage).

## Run locally
```bash
python -m http.server 8000
```
Then open http://localhost:8000.
