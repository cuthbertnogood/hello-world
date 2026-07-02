# hello-world

Personal business card — a static site consisting of `index.html` + `styles.css`. No build system, package manager, or dependencies.

## Cursor Cloud specific instructions

- This is a purely static site; there is nothing to install or build. Runtime tooling (`python3`, `node`) is preinstalled.
- To run in development, serve the repo root with any static server and open in a browser, e.g. `python3 -m http.server 8000` then visit `http://localhost:8000/`. Opening `index.html` via `file://` also works but relative assets are best served over HTTP.
- There is no lint or test tooling configured in this repo; there are no automated tests to run.
- The avatar image and some links load from external URLs (GitHub avatars, `cutnogood.xyz`); these require network access to display but do not affect local rendering of the layout.
- Production hosting is GitHub Pages (`master` branch, `/root`), per `README.md`.
