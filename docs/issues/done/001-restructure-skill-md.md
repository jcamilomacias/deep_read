# 001 — Restructure SKILL.md into three-section skeleton

**Blocked by:** None — can start immediately
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/2

## What to build

Restructure the existing single-workflow `SKILL.md` into a three-section document — one section per command (`/deep-read`, `/distill`, `/capture`) — without yet changing any command behavior. The existing `/deep-read` workflow moves into its dedicated section unchanged. Two stub sections are added for `/distill` and `/capture` with placeholder content so the file structure is established and subsequent slices have a clear landing zone.

The shared `REFERENCE.md` (HTML structure, slug rules, chip vocabulary, session log template) remains unchanged by this slice.

## Acceptance criteria

- [ ] `SKILL.md` has three clearly delimited command sections with headings: `/deep-read`, `/distill`, `/capture`
- [ ] The existing `/deep-read` workflow content is preserved verbatim inside its section
- [ ] `/distill` and `/capture` sections exist with stub content
- [ ] The file is valid Markdown with no broken cross-references to `REFERENCE.md`
