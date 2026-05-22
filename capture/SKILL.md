---
name: capture
description: Scan a Jupyter notebook for cells tagged `# CAPTURE`, write each as a code block + interpretation to the deep-read learning ledger, then remove the tags from the notebook. Use when the user wants to flush tagged notebook cells into docs/learning-ledger.html.
---

# /capture

Scan a notebook for `# CAPTURE` tagged cells, write each to the learning ledger, then remove the tags.

```
/capture @analysis.ipynb
/capture @notebooks/experiment.ipynb
```

See [REFERENCE.md](~/.claude/skills/deep-read/REFERENCE.md) for available HTML components. Use `html-playbook.html` in the project root as your visual reference.

---

### Workflow

#### 1. Parse the argument

The argument must be a single `@`-prefixed path to a `.ipynb` file.

- **No argument** → tell the user a path is required. Example: `/capture @notebook.ipynb`. Do not proceed.
- **Argument is not a `.ipynb` path** → tell the user this command only works with Jupyter notebooks. Do not proceed.
- **Path not found** → tell the user the path was not found and ask them to correct it. Do not proceed.

Do not scan directories or infer paths. Read only the file the user named.

#### 2. Find tagged cells

Read the notebook. Scan every cell for a line containing `# CAPTURE` (case-sensitive, exact string match; may appear anywhere in the cell — first line, last line, or inline comment).

Collect all matching cells in the order they appear in the notebook.

If no cells contain `# CAPTURE`: output `Nothing to capture in <path> — no # CAPTURE cells found.` Make no changes. Stop here.

#### 3. Write each tagged cell to the ledger

Process each tagged cell in order.

**Step 3a — Strip the tag.** Remove the `# CAPTURE` line from the cell source. If `# CAPTURE` appears as an inline comment (e.g. `x = 1  # CAPTURE`), remove only the comment portion and keep the rest of the line. Keep all other content unchanged. This stripped version is what gets written.

**Step 3b — Decide the section.** Determine which section this cell belongs in:
- If the cell clearly relates to a specific module or component: find or create a section for that topic.
- Otherwise: create or update a general concept or experiments section appropriate to the cell's content.

**Step 3c — Write.** In `./docs/learning-ledger.html`:
- If a matching section exists: append the cell content. Use the same visual approach already established in that section — code block + explanation prose, or adapt to whatever fits best.
- If no matching section exists: create a new section whose structure fits the cell's content. Use `html-playbook.html` as your visual reference for available components. No template — invent the structure that serves the content.

Include a `<details class="source-q">` noting this came from the notebook, collapsed by default.

#### 4. Edit the notebook

After all ledger writes are complete, remove the `# CAPTURE` line (or inline comment portion) from every tagged cell. Write the notebook back to disk as valid JSON.

Do not modify untagged cells. Do not reformat, reorder, or re-number cells.

#### 5. Log the session

Only if at least one cell was written.

If the ledger has a session log, add an entry: date, specific summary of what was captured and where it landed (e.g. "Captured 3 cells from `notebooks/experiment.ipynb` — auth token behavior and pandas merge gotcha"), links to sections touched.
