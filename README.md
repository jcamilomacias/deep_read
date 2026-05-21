# deep-read

A Claude Code skill for interactive codebase exploration. Three commands, one ledger — point them at code, notebooks, or conversations to build a self-contained HTML learning document that grows to reflect your personal understanding of a repo.

The ledger accumulates knowledge across sessions: module maps, distilled insights, notebook experiments, and concept notes — all in one navigable file you can publish to GitHub Pages.

## Install

Each command needs its own skill directory. Copy all three:

```bash
cp -r deep-read ~/.claude/skills/deep-read
cp -r distill   ~/.claude/skills/distill
cp -r capture   ~/.claude/skills/capture
```

All three commands are immediately available in any Claude Code session.

## The three commands

### `/deep-read @path`

Reads a file, directory, or notebook and writes a skeleton to the ledger immediately — no confirmation prompt.

```
/deep-read @src/auth/
/deep-read @analysis.ipynb
/deep-read @src/services/payment.py
```

Writes an Overview + Key Findings in a conversational tone (a knowledgeable friend explaining the code, not API docs). Claude can add custom chip sections beyond the defaults when the module warrants it. On revisit, proposes an Overview merge rather than silently overwriting.

### `/distill`

Surfaces insights from the current conversation and proposes them as a numbered list. You keep or drop each one — nothing lands in the ledger without your approval.

```
/distill
```

Run it as many times as you want during a session. Each run covers the conversation since the last `/distill`. Claude infers the target module from context, confirms the routing before writing, and deduplicates against existing ledger content. Can create a full module section from scratch if none exists yet.

Candidate list format:
```
1. [→ auth / Key Findings] Token refresh happens on every request — no background loop.
2. [→ auth / Gotchas] Empty `scopes` list silently grants all scopes.
3. [→ field-notes / Language] `lru_cache` entries live forever — no TTL support.

Keep which? (e.g. keep 1 3, keep all, keep none)
```

### `/capture @notebook.ipynb`

Scans a notebook for cells tagged `# CAPTURE`, writes each as a code block + interpretation to the ledger, then removes the tags from the notebook.

```
/capture @notebooks/experiment.ipynb
```

Tag any cell you want captured while you work:

```python
# CAPTURE
result = model.fit(X_train, y_train)
print(result.score(X_test, y_test))
```

After `/capture` runs, the tag is removed and the cell's code + a conversational explanation appear in the ledger. Reproducible — re-run any time, nothing duplicates.

## The learning ledger

`docs/learning-ledger.html` is a self-contained dark-theme HTML file — no build step, no external dependencies. Open it locally or publish to GitHub Pages.

Three sections:

- **Session Log** — dated entries, newest first, each linking to modules touched
- **Module Map** — one section per module; skeleton from `/deep-read`, enriched by `/distill` and `/capture`
- **Field Notes** — tagged concept cards (Language / Pattern / Library / Other)

### Publish to GitHub Pages

1. Commit `docs/learning-ledger.html` and `docs/.nojekyll` to `main`
2. Go to repo **Settings → Pages → Deploy from branch → `main` → `/docs`**
3. Your ledger is live at `https://<username>.github.io/<repo>/learning-ledger.html`

## Typical workflow

```
# Start exploring a module
/deep-read @src/pipeline/

# Discuss with Claude, ask questions, trace flows...

# Capture what matters from the conversation
/distill

# Keep experimenting in notebooks, tag cells as you go
# CAPTURE  ← add this to any cell worth documenting

# Flush tagged cells to the ledger
/capture @notebooks/pipeline_experiments.ipynb

# Keep discussing, distill again
/distill
```

The ledger grows to reflect your understanding — not Claude's first pass.

## Project structure

```
deep_read/
├── README.md
├── html-playbook.html        ← visual style reference
├── client_brief.md           ← original design brief
├── deep-read/                ← /deep-read skill (install to ~/.claude/skills/deep-read)
│   ├── SKILL.md              ← /deep-read command
│   ├── REFERENCE.md          ← HTML templates, slug rules, chip vocabulary
│   └── scaffold.html         ← first-run ledger template
├── distill/                  ← /distill skill (install to ~/.claude/skills/distill)
│   └── SKILL.md              ← /distill command
├── capture/                  ← /capture skill (install to ~/.claude/skills/capture)
│   └── SKILL.md              ← /capture command
├── ralph/                    ← AFK agent runner
│   ├── prompt.md             ← agent instructions
│   ├── once.sh               ← single iteration (no Docker)
│   └── afk.sh                ← full autonomous loop
└── docs/
    ├── learning-ledger.html  ← the ledger (grows across sessions)
    ├── prd-deep-read.md      ← product requirements document
    ├── adr/                  ← architecture decision records
    │   ├── 0001-deep-read-writes-without-confirmation.md
    │   └── 0002-reading-and-documenting-separated.md
    ├── issues/               ← implementation issues
    │   └── done/             ← completed issues
    └── .nojekyll             ← disables Jekyll for GitHub Pages
```

## Design decisions

- **`/deep-read` writes immediately** — no confirmation loop. The correction path is `/distill`, not a pre-flight review gate. See [ADR 0001](docs/adr/0001-deep-read-writes-without-confirmation.md).
- **Reading and documenting are separated** — `/deep-read` produces a structural anchor; `/distill` produces a personalized record. See [ADR 0002](docs/adr/0002-reading-and-documenting-separated.md).
- **One ledger per project** — `docs/learning-ledger.html` lives in the project root, scoped to that codebase
- **Merge, not append** — revisiting a module produces a single clean current picture, not accumulating noise
- **Self-contained HTML** — the ledger has no build step and no external dependencies
