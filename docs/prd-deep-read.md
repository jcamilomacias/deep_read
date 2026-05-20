# PRD: `/deep-read` — Interactive Codebase Exploration Skill

## Problem Statement

When exploring an unfamiliar codebase, a developer accumulates understanding through reading files, tracing execution flows, and running experiments in notebooks. This learning is ephemeral — it lives in the developer's head, in scattered notes, or not at all. When returning to the same codebase days or weeks later, the exploration starts over from scratch. There is no structured, persistent, and visually navigable record of what was learned, what questions remain open, and what personal knowledge gaps were encountered along the way.

## Solution

A Claude Code skill (`/deep-read`) that acts as an interactive exploration companion. When invoked with a file, directory, notebook, or free-text question, it reads the target silently, drafts a structured summary in the terminal for the developer to confirm, then writes or merges the findings into a per-project HTML learning ledger (`docs/learning-ledger.html`). At the end of each session it sweeps for personal concept notes (Field Notes) and logs a dated session entry. Over time, the ledger becomes a navigable, self-contained map of accumulated codebase knowledge.

## User Stories

1. As a developer, I want to point `/deep-read` at a file and get a structured draft of what it does, so that I can confirm the summary before it is written to the ledger.
2. As a developer, I want to point `/deep-read` at a directory, so that it reads entry points and key modules and drafts a module overview automatically.
3. As a developer, I want to point `/deep-read` at a Jupyter notebook, so that experiment observations and analysis pipelines are captured as Key Findings and Flow Traces.
4. As a developer, I want to ask `/deep-read` a free-text question, so that it asks me to locate relevant files before reading anything.
5. As a developer, I want the skill to read files silently before asking questions, so that its draft is grounded in actual code rather than my possibly-faulty memory.
6. As a developer, I want to review the draft in the terminal and correct it before it is written, so that the ledger reflects my actual understanding.
7. As a developer, I want the skill to write to the ledger only after I confirm the draft, so that the HTML file is never touched with content I haven't validated.
8. As a developer, I want the ledger organized by module/topic, so that I can look up "what do I know about auth?" rather than "what did I learn on Tuesday?".
9. As a developer, I want each module section to have four fixed sub-blocks (Overview, Key Findings, Flow Traces, Open Tasks), so that the structure is predictable across all modules.
10. As a developer, I want to add personal concept notes (Field Notes) at end of each session, so that gaps in my knowledge of Python, patterns, or libraries are documented alongside codebase knowledge.
11. As a developer, I want Field Notes as tagged cards in a grid with color-coded chips (Language / Pattern / Library / Other), so that I can scan by category at a glance.
12. As a developer, I want each session to produce a dated log entry linking to modules touched, so that I have a chronological trail alongside the topic-organized map.
13. As a developer, I want the skill to merge new findings into an existing module section rather than appending or replacing, so that each module stays a single clean current picture.
14. As a developer, I want the skill to state proposed merge changes explicitly before writing, so that I can catch bad interpretations before they are committed.
15. As a developer, I want the ledger created automatically on first run, so that there is no separate setup step.
16. As a developer, I want each project to have its own `docs/learning-ledger.html`, so that the ledger is scoped to the codebase being explored.
17. As a developer, I want the ledger to be a self-contained HTML file with no external dependencies, so that I can open it locally or publish it to GitHub Pages without a build step.
18. As a developer, I want the ledger to use the same visual system as `html-playbook.html`, so that the output is consistently readable.
19. As a developer, I want the sidebar nav to list all module sections as links, so that I can jump directly to any module without scrolling.
20. As a developer, I want Open Tasks to include `file:line` references where possible, so that I can navigate directly to the code when following up.
21. As a developer, I want Flow Traces to name the flow and show entry → steps → exit, so that execution paths are scannable without re-reading source.
22. As a developer, I want the ledger publishable to GitHub Pages independently of any MkDocs setup, so that it can be shared as a standalone documentation site.

## Implementation Decisions

**Skill structure** — Two files at `~/.claude/skills/deep-read/`: `SKILL.md` (7-step interactive workflow) and `REFERENCE.md` (HTML templates, chip color map, CSS variables, first-run scaffold). No executable scripts — entirely prompt-driven.

**Argument dispatch** — File path → read file. Directory → read entry points and index files. `.ipynb` → read all cells and outputs. Free text → ask user to locate files first.

**Read-first model** — All target files are read silently before any output. Draft appears in terminal in fixed plaintext format. HTML file is not touched until user confirms.

**Ledger structure** — Three top-level areas: Session Log (dated timeline, newest first), Module Map (one `<section>` per module), Field Notes (tagged cards grid, three columns).

**Module section anatomy** — Overview (paragraph, merged on revisit), Key Findings (bullet list), Flow Traces (named flows with entry → steps → exit), Open Tasks (`file:line` references where available). For notebooks: Key Findings = observations, Flow Traces = analysis pipeline.

**Merge on revisit** — Proposes specific additions and contradictions explicitly before writing. Never appends (noise) or replaces (destroys history).

**Field Notes** — End-of-session sweep only. Categories and chip colors: Language (green), Pattern (violet), Library (blue), Other (amber).

**Output path** — `./docs/learning-ledger.html`, independent of any MkDocs setup. Created on first run.

**Visual system** — Copies `<style>` block from `html-playbook.html` verbatim on first run. No separate design system to maintain.

## Testing Decisions

A good test verifies observable behavior: given a known input, does the skill produce the correct draft? Does confirmed content produce correct HTML structure? Does revisiting a module produce a diff proposal rather than a silent overwrite?

**Modules to test:**
- Argument dispatch — each input type routes to the correct read behavior
- Draft format — correct plaintext structure, empty sections omitted
- First-run scaffold — `docs/learning-ledger.html` created with correct top-level structure
- Merge logic — revisiting a module produces a diff proposal, not a silent overwrite or append
- Field Notes chip color — category maps to correct chip class
- Session log entry — dated entry appended with correct module anchor links

**Testing approach** — Integration-level invocations against fixture files, with snapshot assertions on generated HTML sections. No prior art for skill integration tests in this repo — this will establish the pattern.

## Out of Scope

- Automatic detection of personal knowledge gaps
- Multi-file diff view or side-by-side comparison in the ledger
- Full-text search across ledger entries
- Export to markdown or PDF
- GitHub Wiki integration
- MkDocs integration
- Any backend, database, or server component

## Further Notes

- `/deep-read` was chosen over `/ledger`, `/document-learning`, and `/explore` for typing ergonomics and clarity.
- `html-playbook.html` serves as both the visual style source and the first documented module in the ledger — making the system self-documenting from its first invocation.
- The "reader surface vs source of truth" principle applies to the ledger itself: open tasks and decisions surfaced in the ledger should be distilled back into source files and issues, not left only in the HTML.
