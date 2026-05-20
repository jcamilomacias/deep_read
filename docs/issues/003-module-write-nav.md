# Issue 003 — Module write + sidebar nav update

**Type:** AFK
**Blocked by:** #001
**Label:** ready-for-agent

## What to build

Verify and harden the module section write behavior in `SKILL.md` step 5. When the user confirms a draft, the skill must write a correctly structured `<section>` to `docs/learning-ledger.html` and add a matching anchor link to the sidebar nav.

The written section must match the REFERENCE.md module section template exactly:
- `id="module-<slug>"` on the `<section>` tag (slug derived from path: `src/auth/` → `auth`, `src/services/payment.py` → `services-payment`)
- Two-column grid with Key Findings card (teal chip) and Open Tasks card (amber chip)
- Flow Traces card below the grid (violet chip) with timeline layout
- Overview as a `<p class="lead">` inside the section heading

The sidebar nav `<aside>` must gain a new `<a href="#module-<slug>">` entry under the "Modules" nav group after every new module write.

## Acceptance criteria

- [ ] New module section written with correct `id`, chip colors, and layout matching `REFERENCE.md` template
- [ ] Slug derived correctly from the argument path (no spaces, lowercase, hyphens only)
- [ ] Sidebar nav updated with a link to the new module's anchor
- [ ] Sections with nothing to say (no flows, no tasks) omit those sub-blocks entirely rather than rendering empty containers
- [ ] HTML file remains valid and openable in a browser after the write

## Blocked by

- #001 scaffold-template
