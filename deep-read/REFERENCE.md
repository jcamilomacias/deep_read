# Deep Read — Reference

## Ledger structure

```
docs/learning-ledger.html
├── Sidebar nav (sticky)
│   ├── Session Log       → #session-log
│   ├── Module Map        → #module-map  (+ per-module anchors)
│   └── Field Notes       → #field-notes
│
├── Session Log           (#session-log)
│   └── Dated timeline entries, newest first
│       Each entry links to modules touched that session
│
├── Module Map            (#module-map)
│   └── One <section> per module  (#module-<slug>)
│       ├── Overview       (paragraph — updated/merged each visit)
│       ├── Key Findings   (bullet list)
│       ├── Flow Traces    (named flows: entry → steps → exit)
│       └── Open Tasks     (follow-ups with file:line links)
│
└── Field Notes           (#field-notes)
    └── Tagged cards grid  (chip: Language / Pattern / Library / Other)
```

---

## Module section template

```html
<section id="module-SLUG">
  <div class="section-heading">
    <div class="section-number">src/path/</div>
    <div>
      <h2>Module Name</h2>
      <p class="lead">Overview paragraph here. Updated on each revisit via merge.</p>
    </div>
  </div>

  <div class="grid-2">
    <div class="card">
      <span class="chip teal">Key Findings</span>
      <ul class="list">
        <li>First insight or invariant.</li>
        <li>Second surprise or non-obvious behavior.</li>
      </ul>
    </div>
    <div class="card">
      <span class="chip amber">Open Tasks</span>
      <ul class="list">
        <li>[ ] Follow-up item — <code>src/file.py:42</code></li>
      </ul>
    </div>
  </div>

  <div class="card" style="margin-top:14px">
    <span class="chip violet">Flow Traces</span>
    <div class="timeline">
      <div class="step">
        <div class="badge">→</div>
        <div>
          <h3>FlowName</h3>
          <p><code>entry_point()</code> → <code>Step.process()</code> → result returned</p>
        </div>
      </div>
    </div>
  </div>
</section>
```

**Slug rules** — apply in order:
1. Strip trailing `/` and file extension (`.py`, `.ipynb`, `.ts`, `.js`, etc.)
2. Replace underscores and spaces with hyphens
3. Lowercase everything
4. Keep only the last **two** path components, joined with a hyphen (prevents collisions in deep trees)
5. If the result is `.` or empty (user passed current dir), use the working directory's folder name

Examples:

| Argument | Slug |
|---|---|
| `src/auth/` | `auth` |
| `src/services/payment.py` | `services-payment` |
| `src/models/user_profile.py` | `models-user-profile` |
| `analysis.ipynb` | `analysis` |
| `notebooks/01_design_principles.ipynb` | `notebooks-01-design-principles` |
| `@.` (current dir named `my_project`) | `my-project` |
| `docs/issues/` | `docs-issues` |

---

## Session Log entry template

Entries go inside a `<div class="timeline">` container. Add newest entries at the top.

```html
<div class="step">
  <div class="badge">N</div>
  <div class="card">
    <h3>YYYY-MM-DD</h3>
    <p>One-line summary of what was explored this session.</p>
    <p>Touched: <a href="#module-auth">Auth</a>, <a href="#module-services-payment">Payment</a></p>
  </div>
</div>
```

N = session number (increment from last entry).

---

## Field Notes card template

Cards go inside a `<div class="grid-3">` container in the Field Notes section.

```html
<div class="card">
  <span class="chip green">Language</span>
  <h3>Concept Name</h3>
  <p>2–4 sentence explanation of the concept and why it matters in context.</p>
</div>
```

Chip color by category:

| Category | Chip class     | Color  |
|----------|----------------|--------|
| Language | `chip green`   | green  |
| Pattern  | `chip violet`  | violet |
| Library  | `chip blue`    | blue   |
| Other    | `chip amber`   | amber  |

---

## CSS variables (visual system)

Key variables for reference:

```css
--bg: #0a0b0d;
--ink: #f3f2ed;
--muted: #a9adb4;
--dim: #747b86;
--line: #2a3039;
--panel: #14171c;
--panel-2: #191d23;
--green: #8bd17c;
--teal: #52c7bd;
--amber: #f0bc58;
--red: #ff746c;
--blue: #83aaff;
--violet: #c4a1ff;
```

Layout: two-column grid (sticky sidebar + scrollable main). Responsive breakpoint at 960px collapses to single column.

---

## First-run scaffold

When `docs/learning-ledger.html` does not exist:

1. Copy `scaffold.html` from this skill directory (`~/.claude/skills/deep-read/scaffold.html`) to `./docs/learning-ledger.html`. Create the `docs/` directory first if needed.
2. Replace `{{PROJECT}}` in the `<title>` tag with the name of the current project (derive from the working directory name).
3. Immediately write the first module section into the file — do not leave the ledger empty.

The scaffold is self-contained: full inline CSS and correct empty-state HTML for all three sections. It does not depend on any file in the target project.
