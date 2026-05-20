# Issue 007 — GitHub Pages publish setup for `docs/learning-ledger.html`

**Type:** HITL
**Blocked by:** #001
**Label:** ready-for-agent

## What to build

Configure GitHub Pages to serve `docs/learning-ledger.html` from the project repository, independently of any MkDocs or other static site generator in the same repo.

This is a HITL slice: it requires the human to create or configure the GitHub repository and enable Pages in the repo settings. The agent's role is to verify the configuration is correct and the ledger is accessible at the resulting URL.

Steps:
1. Ensure `docs/learning-ledger.html` is committed to the `main` branch
2. In GitHub repo settings → Pages → set source to "Deploy from a branch", branch `main`, folder `/docs`
3. Verify the site is live at `https://<org>.github.io/<repo>/learning-ledger.html`
4. Add the public URL to the ledger's `<footer>` so it is self-referential

The ledger must be kept separate from any MkDocs Pages deployment. If MkDocs is also publishing from `docs/`, resolve the conflict by moving the ledger to a dedicated repo or configuring MkDocs to pass through the HTML file unchanged.

## Acceptance criteria

- [ ] `docs/learning-ledger.html` is committed to `main`
- [ ] GitHub Pages configured to serve from `main` branch, `/docs` folder
- [ ] Ledger accessible at a public HTTPS URL without authentication
- [ ] Ledger renders correctly in a browser (sidebar nav, cards, dark theme)
- [ ] No conflict with existing MkDocs or other Pages deployment
- [ ] Public URL recorded in the ledger footer

## Blocked by

- #001 scaffold-template
