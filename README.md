# /deep-read

A Claude Code skill for interactive codebase exploration. Point it at a file, directory, or notebook — it reads silently, drafts a structured summary for you to confirm, then writes findings into a self-contained HTML learning ledger (`docs/learning-ledger.html`).

The ledger accumulates knowledge across sessions: module maps, flow traces, open questions, and personal concept notes — all in one navigable file you can publish to GitHub Pages.

## Install

Copy the `deep-read/` folder into your Claude Code skills directory:

```bash
cp -r deep-read ~/.claude/skills/deep-read
```

The skill is immediately available as `/deep-read` in any Claude Code session.

## Usage

```
/deep-read @src/auth/
/deep-read @analysis.ipynb
/deep-read @src/services/payment.py
/deep-read "how does the event loop work in this app?"
```

### What happens each session

1. **Reads** the target silently (file, directory, notebook, or concept)
2. **Drafts** a structured summary in the terminal — Overview, Key Findings, Flow Traces, Open Tasks
3. **You confirm or correct** the draft
4. **Writes** to `./docs/learning-ledger.html` — creating it on first run, merging on revisit
5. **Sweeps** for personal concept notes to add to Field Notes (Language / Pattern / Library)
6. **Logs** a dated session entry linking to all modules touched

## The learning ledger

`docs/learning-ledger.html` is a self-contained dark-theme HTML file — no build step, no external dependencies. Open it locally or publish it to GitHub Pages.

It has three sections:

- **Session Log** — dated entries, newest first, each linking to modules touched
- **Module Map** — one section per module, accumulating knowledge across sessions via merge
- **Field Notes** — tagged concept cards (language features, architectural patterns, library behavior)

### Publish to GitHub Pages

1. Commit `docs/learning-ledger.html` and `docs/.nojekyll` to `main`
2. Go to repo **Settings → Pages → Deploy from branch → `main` → `/docs`**
3. Your ledger is live at `https://<username>.github.io/<repo>/learning-ledger.html`

## Argument types

| Argument | Behavior |
|---|---|
| `@src/auth.py` | Reads that file |
| `@src/auth/` | Reads directory — entry points first, capped at 5 files |
| `@analysis.ipynb` | Reads all cells; maps observations to Key Findings, pipeline steps to Flow Traces |
| `"free text"` | Asks you to locate relevant files before reading anything |

## Merge behavior

Revisiting a module you've already documented shows a diff proposal before writing anything:

```
MERGE PROPOSAL for auth:
Overview:    [replace]  "Handles JWT issuance..." → "Validates tokens and..."
Key Findings: [+1 new]  [no removals]
  + "Refresh tokens are stored in httpOnly cookies, not localStorage"
Flow Traces: [no change: Login flow]
Open Tasks:  [+1 new]
OK to apply?
```

Nothing is written until you confirm.

## Project structure

```
Documet_skill/
├── README.md
├── html-playbook.html        ← visual style reference and first documented module
├── client_brief.md           ← original design brief
├── deep-read/                ← skill source (install to ~/.claude/skills/)
│   ├── SKILL.md              ← 7-step interactive workflow
│   ├── REFERENCE.md          ← HTML templates, slug rules, CSS variables
│   └── scaffold.html         ← first-run ledger template
└── docs/
    ├── learning-ledger.html  ← the ledger (grows with each /deep-read session)
    ├── prd-deep-read.md      ← product requirements document
    ├── .nojekyll             ← disables Jekyll for GitHub Pages
    └── issues/               ← implementation issues (001–007)
```

## Design decisions

- **Read-first**: the skill reads code before asking questions — drafts are grounded in the actual source, not your memory
- **One ledger per project**: `docs/learning-ledger.html` lives in the project root, scoped to that codebase
- **Merge, not append**: revisiting a module produces a single clean current picture, not accumulating noise
- **Self-contained HTML**: the ledger has no build step and no external dependencies — open it anywhere
