# Issue 001 — Add `scaffold.html` template to the deep-read skill

**Type:** AFK
**Blocked by:** None — can start immediately
**Label:** ready-for-agent

## What to build

Add a `scaffold.html` file to `~/.claude/skills/deep-read/`. This file is the canonical first-run template the skill uses when `docs/learning-ledger.html` does not yet exist in a project.

The scaffold must be self-contained: full inline CSS (dark theme, CSS custom properties, two-column grid layout) and the correct empty-state HTML for all three top-level sections — Session Log, Module Map, and Field Notes. It must not depend on `html-playbook.html` being present in the target project.

On first run, the skill copies this scaffold, sets the `<title>` to match the project name, and immediately writes the first module section into it.

Update `REFERENCE.md` to point to `scaffold.html` as the first-run source instead of instructing Claude to copy from `html-playbook.html`.

## Acceptance criteria

- [ ] `~/.claude/skills/deep-read/scaffold.html` exists and is self-contained (no external deps, full inline CSS)
- [ ] Scaffold contains correct empty-state structure: Session Log (empty timeline), Module Map (heading only), Field Notes (empty grid with placeholder card)
- [ ] `REFERENCE.md` updated to reference scaffold.html as the first-run source
- [ ] Running `/deep-read` on a project with no existing `docs/learning-ledger.html` produces a valid, openable HTML file without any manual setup

## Blocked by

None — can start immediately
