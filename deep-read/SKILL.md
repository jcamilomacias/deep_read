---
name: deep-read
description: Codebase exploration skill with three commands — /deep-read explores and documents a module into the learning ledger, /distill captures insights from conversation into the ledger with user control over what lands, and /capture processes tagged notebook cells into the ledger. Use when the user wants to explore code, trace flows, distill conversation insights, or capture notebook experiments.
---

# Deep Read

Three commands, one ledger.

```
/deep-read @src/auth/
/distill
/capture @analysis.ipynb
```

See [REFERENCE.md](REFERENCE.md) for the HTML ledger structure, slug rules, chip vocabulary, and section templates.

---

## `/deep-read`

### Quick start

```
/deep-read @src/auth/
/deep-read @analysis.ipynb
/deep-read @src/services/payment.py
/deep-read "how does the event loop work in this app?"
```

### Session workflow

#### 1. Parse the argument

Apply these rules in order:

1. **No `@` prefix** → free text. Stop. Ask the user which files to read. Do not guess or read anything until they answer.
2. **`@` prefix + path ends in `.ipynb`** → notebook. Go to notebook read rules in step 2.
3. **`@` prefix + path resolves to a directory** → directory. Go to directory read rules in step 2.
4. **`@` prefix + path resolves to a file** → file. Read that file.
5. **`@` prefix + path does not exist** → tell the user the path was not found and ask them to correct it. Do not proceed.

Edge cases:
- A path with no extension and no trailing `/` — run a quick check (`ls` or `find`) to determine whether it is a file or directory before deciding.
- Multiple `@` arguments — treat each as a separate file read; combine findings into a single entry named after the shared parent path or the concept they collectively represent.

#### 2. Read silently

Read the target without narrating. Build an understanding of:
- What this module/file/notebook does and why
- Its dependencies and interfaces
- Notable patterns, invariants, or surprises
- Data or execution flows

**Directory read rules:**
List the directory contents first. Then read in this priority order:
1. Language entry points: `main.py`, `__init__.py`, `index.ts`, `index.js`, `main.rs`, `lib.rs`, `app.py`, `server.py`, `README.md`
2. Files whose name matches the directory name
3. The largest remaining files by line count (cap at 5 files total for large directories)

The section covers the directory as a whole — not file by file. Do not enumerate every file in Key Findings.

**Notebook read rules:**
- Markdown cells → read as background context. Do not quote them verbatim.
- Code cells with outputs → extract the conceptual insight from the output. Do not dump raw `print()` results or tensor values.
- Code cells with no output → read the code itself; treat as implementation detail unless it encodes a key design decision.
- Code cells with error output → note the error in Open Tasks: `[ ] Cell N raised <ErrorType> — investigate`.
- The MODULE name is the notebook's top-level heading (first `# Heading` in any markdown cell), or the filename if no heading exists.

#### 3. Write to the ledger

Write immediately — no terminal draft, no confirmation prompt for new modules.

Target: `./docs/learning-ledger.html`

Create `docs/` and the file if they don't exist.

**Tone:** Write conversationally — a knowledgeable friend explaining the code, not formal API documentation. The Overview says what this code does and why it exists, in language a teammate would use. Key Findings name what is surprising, non-obvious, or load-bearing — not a catalogue of every class and method.

**Chip vocabulary is open-ended.** Key Findings, Flow Traces, and Open Tasks are defaults, not constraints. Add custom chip sections when the module has a category of information that doesn't fit the defaults — "Gotchas", "Dependencies", "Configuration Surface", "Performance Notes", or anything else warranted. Pick a chip color that fits the tone: amber for warnings, blue for external things, violet for patterns. Omit any section that has nothing to say.

**First run** — copy `scaffold.html` from this skill directory to `./docs/learning-ledger.html` (create `docs/` if needed), then replace `{{PROJECT}}` in the `<title>` with the project name. See [REFERENCE.md](REFERENCE.md) for details.

**New module** — two writes, in this order:

1. **Nav link**: inside `<aside>`, locate the comment `<!-- MODULE NAV LINKS GO HERE -->` and insert immediately before it:
   ```html
   <a href="#module-SLUG">Display Name</a>
   ```
   Display Name = the human-readable module name (e.g. `auth`, `Payment Service`). SLUG = slug derived per rules in REFERENCE.md.

2. **Section**: inside `<main>`, locate the comment `<!-- MODULE SECTIONS GO HERE -->` and insert immediately before it. Use the module section template from REFERENCE.md exactly — correct `id`, chip colors, and layout. **Omit any sub-block that has no content**: if there are no flow traces, omit the Flow Traces card entirely; if there are no open tasks, omit the Open Tasks card. Never write an empty `<ul class="list"></ul>` or an empty timeline.

After writing, confirm the file opens in a browser without errors (check for unclosed tags by scanning the written block).

**Existing module** — four steps, all required:

**Step 3a — Detect.** Before any write, search the ledger for `<section id="module-SLUG">`. If found, this is an existing module. If not found, treat as a new module (see above).

**Step 3b — Read existing.** Extract the current content of the existing section: the Overview paragraph, each Key Finding bullet, each Flow Trace, and each Open Task. Do this by reading the HTML — not from memory.

**Step 3c — Propose.** Compare the new read (from step 2) against the existing section sub-block by sub-block. Then show a merge proposal in this format:

```
MERGE PROPOSAL for <module name>:

Overview:    [replace]  "<first ~15 words of old>" → "<first ~15 words of new>"
Key Findings: [+N new]  [~M updated]  [-K removed]
  + "<new finding text>"
  ~ "<old>" → "<new>" (if a finding changed rather than being wholly new)
Flow Traces: [+N new]  [replace: <FlowName>]  [no change: <FlowName>]
Open Tasks:  [+N new]  [no removals]

OK to apply?
```

If the new read is identical to the existing content, show:
```
MERGE PROPOSAL for <module name>: no changes — section is already current. Nothing to write.
```
and skip step 3d.

**Step 3d — Write only after explicit confirmation.** When the user confirms, replace the entire `<section id="module-SLUG">…</section>` block (from opening `<section` tag to closing `</section>`) with the newly merged content. Do not append below the old section. The result must be a single clean section — not two stacked versions. Apply the same empty sub-block omission rule as for new modules.

#### 4. Log the session

**Step 4a — Determine session number.** Read `<div class="timeline" id="session-log-timeline">` in the ledger. Find the highest badge number among existing `<div class="badge">N</div>` entries. N for this entry = highest + 1. If the timeline is empty (only the empty-state placeholder), N = 1. Remove the empty-state placeholder before inserting the first entry.

**Step 4b — Build the entry.** Use today's date (`YYYY-MM-DD`). The summary must name what was specifically explored — not generic boilerplate. Bad: "Explored a module." Good: "Traced the chooser interaction and copy-button flows in html-playbook.html." The Touched list links to every module slug written or merged in this session (collected from step 3 and 3d).

**Step 4c — Prepend.** Insert the new entry as the **first child** inside `<div class="timeline" id="session-log-timeline">` — not appended at the bottom. Use the HTML template from REFERENCE.md:

```html
<div class="step">
  <div class="badge">N</div>
  <div class="card">
    <h3>YYYY-MM-DD</h3>
    <p>One-line summary of what was explored.</p>
    <p>Touched: <a href="#module-SLUG">Display Name</a>, <a href="#module-SLUG2">Display Name 2</a></p>
  </div>
</div>
```

After inserting, confirm no unclosed tags were introduced.

---

## `/distill`

### Quick start

```
/distill
```

Run any time during a session to surface insights from the conversation and route them into the learning ledger with explicit control over what lands.

### Session workflow

#### 1. Scan the conversation for candidate insights

Look back through the conversation from the last `/distill` invocation (or session start for the first run) to now. Extract distinct, non-trivial insights — things a teammate would want to find in the ledger later.

**What counts as an insight:**
- A non-obvious behavior or constraint that was discovered
- A design decision and its reason
- A data or execution flow that was traced
- A gotcha, edge case, or failure mode that came up
- A general language or library pattern that was demonstrated or explained

**What does not count:**
- Questions or hypotheses that were not confirmed
- Trivial facts derivable by reading the code directly
- Boilerplate or setup steps

For each insight, infer its routing:
- **Module section** — if the insight is specifically about a module identifiable by path or class name. Derive the slug using the slug rules in REFERENCE.md. Assign the chip type that fits best: Key Findings, Flow Traces, Open Tasks, or a custom type (Gotchas, Dependencies, Configuration Surface, Performance Notes, Data Contracts).
- **Field Notes** — if the insight is a general concept, language feature, or pattern not tied to a specific module. Assign the chip type: Language, Pattern, Library, or Other.

If no insights are found, output: "No distillable insights found in this conversation window." Stop. Do not write anything.

#### 2. Present the candidate list

Output all candidates as a numbered list. Each item on one line:

```
Distill candidates:

1. [→ auth / Key Findings] Token refresh happens on every request — there is no background refresh loop.
2. [→ auth / Gotchas] Passing an empty `scopes` list silently grants all scopes instead of raising.
3. [→ services-payment / Flow Traces] Payment flow: `initiate()` → `stripe.charge()` → webhook confirmation → `mark_paid()`.
4. [→ field-notes / Language] Python's `functools.lru_cache` does not respect TTL — entries live forever until the process restarts.
5. [→ auth / Open Tasks] [ ] Investigate why `refresh_token()` is called on GET requests — `src/auth/tokens.py:88`

Keep which? (e.g. `keep 1 3 5`, `keep all`, `keep none`)
```

Stop here. Wait for the user's response before doing anything else.

#### 3. Parse the user's selection

- `keep all` → accept every candidate.
- `keep none` → nothing is written. Stop. No session log entry.
- `keep N [M P …]` → keep only the numbered items listed. Discard the rest.
- Any other response → ask the user to clarify using the format above.

#### 4. Confirm routing targets

Group the kept items by target. Show one line per group, then wait:

```
Routing:
  • Items 1, 2, 5 → `auth` module
  • Item 3 → `services-payment` module
  • Item 4 → Field Notes

OK? (or correct with e.g. "item 3 → auth")
```

Wait for confirmation or correction before writing.

Apply any corrections: reassign the named item to the specified target. Re-derive the slug if the user provides a path (using slug rules from REFERENCE.md).

#### 5. Deduplicate, handle Overview updates, and write

Read `./docs/learning-ledger.html`.

**Deduplication:** For each kept item, compare its insight text against existing content in the target section. If a semantically equivalent entry already exists (same fact, even if differently worded), skip that item silently. Do not write it.

**Overview update detection:** If any kept item describes what a module fundamentally does (i.e., it would replace or significantly extend the Overview paragraph), treat it as an Overview update candidate. Show it separately before writing:

```
Overview update for `auth`:
  Old: "Handles token issuance and validation for all API requests…"
  New: "Manages token issuance, refresh, and scope enforcement for all API requests…"

Replace? (yes/no)
```

Wait for the user's answer. Only overwrite the Overview paragraph on "yes." Skip the update on "no" — discard the item entirely.

**Write the remaining approved items:**

For each non-Overview, non-duplicate item:

1. **Existing module section** — append the item as a new `<li>` under the matching chip section card. If the chip section (e.g. Gotchas) does not yet exist in the module, add the card using the custom chip section template from REFERENCE.md. Place it after the last existing card in the `<div class="grid-2">` or as a new card after it.

2. **Module not yet in the ledger** — create the full section: nav link + `<section>` block using the module section template from REFERENCE.md. Write an Overview paragraph inferred from the conversation context. Collect all approved items for that module into appropriate chip section cards. Omit any chip section that has no items. Follow the same two-write sequence as `/deep-read` step 3 (nav link first, then section).

3. **Field Notes item** — append a new card to `<div class="grid-3">` in the Field Notes section using the Field Notes card template from REFERENCE.md.

After all writes, confirm no unclosed tags were introduced by scanning the written blocks.

#### 6. Log the session

Only append a session log entry if at least one item was written to the ledger.

Follow the same steps as `/deep-read` step 4:
- Determine N (highest existing badge + 1, or 1 if the timeline is empty).
- Summary names what was distilled. Example: "Distilled 3 insights into `auth` and 1 to Field Notes."
- Touched list links to every section written (module slugs and/or Field Notes).
- Prepend as the first child of `<div class="timeline" id="session-log-timeline">`.

---

## `/capture`

### Quick start

```
/capture @analysis.ipynb
/capture @notebooks/experiment.ipynb
```

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
