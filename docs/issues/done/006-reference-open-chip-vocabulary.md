# 006 — Update REFERENCE.md: open-ended chip vocabulary

**Blocked by:** 003
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/7

## What to build

Update `REFERENCE.md` to document that the chip vocabulary is open-ended. Add guidance on when to invent a new section versus reuse an existing one.

**Guidance to add:**
- The standard chip sections (Key Findings, Flow Traces, Open Tasks) are defaults, not constraints
- Claude should add a custom section when a module has a category of information that doesn't fit the defaults (e.g. "Gotchas", "Dependencies", "Configuration Surface", "Performance Notes")
- Custom sections follow the same card template as standard ones — pick a chip color that fits the tone (amber for warnings, blue for external dependencies, violet for patterns)
- Don't invent a custom section for content that fits naturally into an existing standard section

No changes to `SKILL.md` are needed in this slice.

## Acceptance criteria

- [ ] `REFERENCE.md` contains a section explaining that the chip vocabulary is open-ended
- [ ] At least three examples of custom chip sections are listed with appropriate chip colors
- [ ] The guidance clarifies when to use a custom section vs. fitting content into a standard one
- [ ] No changes are made to `SKILL.md` in this slice
