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

See [REFERENCE.md](~/.claude/skills/deep-read/REFERENCE.md) for the HTML ledger structure, slug rules, chip vocabulary, and section templates.

---

### Session workflow

#### 1. Parse the argument

The argument must be a single `@`-prefixed path to a `.ipynb` file.

- **No argument** → tell the user a path is required. Example: `/capture @notebook.ipynb`. Do not proceed.
- **Argument is not a `.ipynb` path** → tell the user this command only works with Jupyter notebooks. Do not proceed.
- **Path not found** → tell the user the path was not found and ask them to correct it. Do not proceed.

Do not scan directories or infer paths. Read only the file the user named.

#### 2. Find tagged cells

Read the notebook. Scan every cell for a line containing `# CAPTURE` (case-sensitive, exact string match; may appear anywhere in the cell — first line, last line, or inline comment).

Collect all matching cells in the order they appear in the notebook.

If no cells contain `# CAPTURE`: output "Nothing to capture in `<path>` — no `# CAPTURE` cells found." Make no changes to the ledger or the notebook. Stop here.

#### 3. Route and write each tagged cell

Process each tagged cell in order.

**Step 3a — Strip the tag.** Remove the `# CAPTURE` line from the cell source. If `# CAPTURE` appears as an inline comment (e.g. `x = 1  # CAPTURE`), remove only the comment portion and keep the rest of the line. Keep all other cell content unchanged. This stripped version is what gets written to the ledger.

**Step 3b — Infer routing.** Decide where this cell belongs:
- **Module section** — if the cell imports from, calls into, or clearly demonstrates behavior belonging to a specific module identifiable by path or class name.
- **Field Notes** — if the cell demonstrates a general concept, language feature, or pattern not tied to a specific module.

Infer the module slug using the same slug rules as `/deep-read` (see REFERENCE.md).

**Step 3c — Write to the ledger.** The ledger is at `./docs/learning-ledger.html`.

For a **module section** entry:
- If the section (`<section id="module-SLUG">`) exists, append a new `<li>` under Key Findings. If the module has a natural "Experiments" category that doesn't fit Key Findings, add a custom chip section instead.
- If the section does not exist, create a full section: write an Overview paragraph inferred from the cell's context, then add the cell as a Key Findings entry.

For a **Field Notes** entry:
- Append a new card to `<div class="grid-3">` in the Field Notes section.
- Use the Field Notes card template from REFERENCE.md.
- Pick the most specific applicable chip: Language, Pattern, Library, or Other.

Each entry written to the ledger (whether module or field notes) contains two parts:
1. The stripped cell source in a `<pre><code class="language-python">…</code></pre>` block.
2. A `<p>` paragraph: conversational explanation of what the cell demonstrates and why a teammate should care. Same tone as `/deep-read` — not API docs, not a description of what the code does line by line.

#### 4. Edit the notebook

After all ledger writes are complete, edit the notebook file: remove the `# CAPTURE` line (or inline comment portion) from every tagged cell. Write the notebook back to disk in valid JSON.

Do not modify untagged cells. Do not reformat, reorder, or re-number cells. The only change is removal of `# CAPTURE` text.

#### 5. Log the session

Only append a session log entry if at least one cell was written to the ledger.

Follow the same logging steps as `/deep-read` step 4:
- Determine N (highest existing badge + 1, or 1 if empty).
- Summary names the notebook file and how many cells were captured. Example: "Captured 3 cells from `notebooks/experiment.ipynb` — routed 2 to `auth` module, 1 to Field Notes."
- Touched list links to every section written (module slugs and/or Field Notes).
- Prepend as the first child of `<div class="timeline" id="session-log-timeline">`.
