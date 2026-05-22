---
name: deep-read
description: Codebase exploration skill. /deep-read explores a file or directory and writes living documentation into the learning ledger — sections emerge from what the code actually does, not a template. /distill updates the ledger from the current conversation. /capture processes tagged notebook cells into the ledger. Use when the user wants to explore code, trace flows, distill conversation insights, or capture notebook experiments.
---

# Deep Read

Three commands, one ledger.

```
/deep-read @src/auth/
/distill
/capture @analysis.ipynb
```

See [REFERENCE.md](REFERENCE.md) for HTML patterns and the section template.

---

## `/deep-read`

### Usage

```
/deep-read @src/auth/
/deep-read @src/services/payment.py
/deep-read @analysis.ipynb
/deep-read "how does the event loop work?"
```

### Workflow

#### 1. Parse the argument

Apply in order:
1. No `@` prefix → free text. Ask the user which files to read. Do not proceed without an answer.
2. `@` + `.ipynb` path → notebook. Use notebook read rules in step 2.
3. `@` + directory path → directory. Use directory read rules in step 2.
4. `@` + file path → read that file.
5. Path not found → tell the user, ask to correct. Do not proceed.

For a path with no extension and no trailing `/`, check (`ls` or `find`) whether it is a file or directory before deciding.

#### 2. Read silently

Build a mental model without narrating. Understand:
- What this code does and why it exists
- How it's structured and what its entry points are
- Non-obvious behaviors, constraints, and design decisions
- How data or control flows through it

**Directory read rules:**
List contents first. Then read in priority order:
1. Entry points: `main.py`, `__init__.py`, `index.ts`, `index.js`, `app.py`, `server.py`, `README.md`
2. Files whose name matches the directory name
3. Largest remaining files by line count (cap at 5 total for large directories)

**Notebook read rules:**
- Markdown cells → read as background context; do not quote verbatim.
- Code cells with output → extract the conceptual insight, not raw print values.
- Code cells with no output → read the code; treat as implementation detail unless it encodes a key design decision.
- Code cells with error output → note as an open question to document.
- Section title = first `# Heading` in any markdown cell, or filename if none.

#### 3. Write to the ledger

Target: `./docs/learning-ledger.html`

**First run:** Copy `scaffold.html` from this skill directory to `./docs/learning-ledger.html`. Create `docs/` if needed. Fill the scaffold placeholders (`{{TITLE}}`, `{{MARK}}`, `{{NAV_TITLE}}`, `{{NAV_COPY}}`, `{{FOOTER}}`) based on the project. Write immediately — no confirmation needed.

**How to write:**
Read `html-playbook.html` in the project root as your visual reference. It shows every CSS component available and when each earns its keep: cards, grids, callouts, timelines, matrix tables, diagrams, choosers, chips. Use whichever components fit the content you found. There is no template to follow.

Ask yourself: what does a teammate need to *see* to understand this code? A flow diagram? A comparison table of two approaches? A callout warning about the gotcha? A prose walkthrough with code snippets? Build that. The structure comes from the content, not from a predefined mold.

Let the code's complexity drive how many sections you create. A tangled module with multiple interacting flows might produce several sections. A small focused file might produce one paragraph. Invent section titles a teammate would search for — not generic labels.

Every section should end with a `<details class="source-q">` containing the questions or angles that drove the exploration (collapsed by default — it's reference context, not primary content).

Add a nav link in `<aside>` for each section you create.

**For an existing section:** Read its current content. Show a brief merge proposal (what will change) and wait for confirmation before writing. Extend, don't duplicate.

**Writing style:** conversational teammate, not API docs. Prose as the base. Visual components when they add clarity. Code snippets for non-obvious points — excerpts, not full dumps. Focus on the *why*.

#### 4. Log the session

If the ledger has or would benefit from a session log (a timeline of exploration entries), add an entry: date, specific one-line summary of what was explored, links to sections touched. Be specific — "Traced the token refresh flow in src/auth/tokens.py and found the concurrent-refresh race condition" is useful; "Explored auth module" is not.

---

## `/distill`

See [distill SKILL.md](~/.claude/skills/distill/SKILL.md).

---

## `/capture`

### Usage

```
/capture @analysis.ipynb
/capture @notebooks/experiment.ipynb
```

### Workflow

#### 1. Parse the argument

Requires a single `@`-prefixed path to a `.ipynb` file.

- No argument → tell the user a path is required. Do not proceed.
- Not a `.ipynb` path → tell the user this command only works with Jupyter notebooks. Do not proceed.
- Path not found → tell the user, ask to correct. Do not proceed.

#### 2. Find tagged cells

Read the notebook. Scan every cell for a line containing `# CAPTURE` (case-sensitive, exact match; may appear anywhere in the cell).

Collect matching cells in order. If none found: output `Nothing to capture in <path> — no # CAPTURE cells found.` Stop.

#### 3. Write each tagged cell to the ledger

For each tagged cell:

**Step 3a — Strip the tag.** Remove the `# CAPTURE` line from the cell. If it's an inline comment (`x = 1  # CAPTURE`), remove only the comment portion. Keep everything else unchanged. This stripped version is what gets written.

**Step 3b — Decide the section.** Determine which section this cell belongs in:
- If the cell clearly relates to a specific module or component: find or create a section for that topic.
- Otherwise: create or update a general "Experiments" or concept section appropriate to the cell's content.

**Step 3c — Write.** In `./docs/learning-ledger.html`:
- If a matching section exists: append the cell content. Use the same visual approach already established in that section — code block + explanation prose, or adapt to whatever fits.
- If no matching section exists: create a new section whose structure fits the cell's content. Use `html-playbook.html` as your visual reference for available components.

Include a `<details class="source-q">` noting this came from the notebook, collapsed by default.

#### 4. Edit the notebook

After all ledger writes: remove `# CAPTURE` (or inline comment portion) from every tagged cell. Write back to disk as valid JSON. Do not reformat, reorder, or re-number other cells.

#### 5. Log the session

Only if at least one cell was written.

Determine N and prepend a session log entry as in `/deep-read` step 4. Summary example: "Captured 3 cells from `notebooks/experiment.ipynb` — documented auth token behavior and pandas merge gotcha."
