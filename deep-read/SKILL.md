---
name: deep-read
description: Interactive codebase exploration skill that documents learning in a structured HTML learning ledger at docs/learning-ledger.html. Reads files, directories, or notebooks first, drafts findings in the terminal for user confirmation, then writes to the ledger. Use when the user wants to explore a codebase, trace a data or execution flow, understand a module, or capture insights from a Jupyter notebook experiment.
---

# Deep Read

## Quick start

```
/deep-read @src/auth/
/deep-read @analysis.ipynb
/deep-read @src/services/payment.py
/deep-read "how does the event loop work in this app?"
```

## Session workflow

### 1. Parse the argument

Apply these rules in order:

1. **No `@` prefix** → free text. Stop. Ask the user which files to read. Do not guess or read anything until they answer.
2. **`@` prefix + path ends in `.ipynb`** → notebook. Go to notebook read rules in step 2.
3. **`@` prefix + path resolves to a directory** → directory. Go to directory read rules in step 2.
4. **`@` prefix + path resolves to a file** → file. Read that file.
5. **`@` prefix + path does not exist** → tell the user the path was not found and ask them to correct it. Do not proceed.

Edge cases:
- A path with no extension and no trailing `/` — run a quick check (`ls` or `find`) to determine whether it is a file or directory before deciding.
- Multiple `@` arguments — treat each as a separate file read; combine findings into a single draft under one MODULE entry named after the shared parent path or the concept they collectively represent.

### 2. Read silently

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

The draft covers the directory as a whole — not file by file. Do not enumerate every file in Key Findings.

**Notebook read rules:**
- Markdown cells → read as background context. Do not quote them verbatim in the draft.
- Code cells with outputs → extract the conceptual insight from the output. Do not dump raw `print()` results or tensor values.
- Code cells with no output → read the code itself; treat as implementation detail unless it encodes a key design decision.
- Code cells with error output → note the error in Open Tasks: `[ ] Cell N raised <ErrorType> — investigate`.
- The MODULE name is the notebook's top-level heading (first `# Heading` in any markdown cell), or the filename if no heading exists.

### 3. Draft in terminal

Print the draft for user review. Do NOT write to the HTML file yet.

Use this exact format:

```
MODULE: <name or path>

OVERVIEW
<one paragraph — what it does and why it exists>

KEY FINDINGS
• <insight, invariant, or non-obvious behavior>
• <dependency worth noting>
• <anything that would surprise a first-time reader>

FLOW TRACES
<FlowName>: <entry point> → <step> → <step> → <exit/output>

OPEN TASKS
[ ] <follow-up to investigate — include file:line if known>
```

For notebooks: Key Findings = experiment observations; Flow Traces = analysis pipeline steps.

Omit sections that have nothing to say (e.g., no flows in a pure config file).

### 4. Confirm and refine

Ask: "Does this look right? Anything to add, correct, or cut?"

Incorporate feedback. Repeat until the user confirms the draft.

### 5. Write to the ledger

Target: `./docs/learning-ledger.html`

Create `docs/` and the file if they don't exist.

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

**Step 5a — Detect.** Before any write, search the ledger for `<section id="module-SLUG">`. If found, this is an existing module. If not found, treat as a new module (see above).

**Step 5b — Read existing.** Extract the current content of the existing section: the Overview paragraph, each Key Finding bullet, each Flow Trace, and each Open Task. Do this by reading the HTML — not from memory.

**Step 5c — Propose.** Compare the new draft (from step 3) against the existing section sub-block by sub-block. Then show a merge proposal in this format:

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

If the new draft is identical to the existing content, show:
```
MERGE PROPOSAL for <module name>: no changes — section is already current. Nothing to write.
```
and skip step 5d.

**Step 5d — Write only after explicit confirmation.** When the user confirms, replace the entire `<section id="module-SLUG">…</section>` block (from opening `<section` tag to closing `</section>`) with the newly merged content. Do not append below the old section. The result must be a single clean section — not two stacked versions. Apply the same empty sub-block omission rule as for new modules.

### 6. Field Notes sweep

Always ask this after step 5 completes — even if the merge was a no-op:
> "Anything to add to Field Notes? Any Python concepts, architectural patterns, or library behavior you want to document from this session?"

**If the user declines** — proceed to step 7. Write nothing.

**If the user provides entries** — collect all of them first, then write:

For each entry, ask:
- Name (short concept title)
- Explanation (2–4 sentences)
- Category: `Language` / `Pattern` / `Library` / `Other`

Then write one card per entry inside `<div class="grid-3" id="field-notes-grid">`:

```html
<div class="card">
  <span class="chip CHIP-CLASS">CATEGORY</span>
  <h3>Name</h3>
  <p>Explanation.</p>
</div>
```

Chip class by category:
| Category | Chip class     |
|----------|----------------|
| Language | `chip green`   |
| Pattern  | `chip violet`  |
| Library  | `chip blue`    |
| Other    | `chip amber`   |

**First card only**: before inserting, remove the `<div class="empty-state">…</div>` placeholder inside `#field-notes-grid`. Subsequent cards just append.

Cards are appended as the last children inside `<div class="grid-3" id="field-notes-grid">` — never outside it.

### 7. Log the session

**Step 7a — Determine session number.** Read `<div class="timeline" id="session-log-timeline">` in the ledger. Find the highest badge number among existing `<div class="badge">N</div>` entries. N for this entry = highest + 1. If the timeline is empty (only the empty-state placeholder), N = 1. Remove the empty-state placeholder before inserting the first entry.

**Step 7b — Build the entry.** Use today's date (`YYYY-MM-DD`). The summary must name what was specifically explored — not generic boilerplate. Bad: "Explored a module." Good: "Traced the chooser interaction and copy-button flows in html-playbook.html." The Touched list links to every module slug written or merged in this session (collect these from steps 5 and 5d).

**Step 7c — Prepend.** Insert the new entry as the **first child** inside `<div class="timeline" id="session-log-timeline">` — not appended at the bottom. Use the HTML template from REFERENCE.md:

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

See [REFERENCE.md](REFERENCE.md) for full HTML structure, section templates, and CSS variables.
