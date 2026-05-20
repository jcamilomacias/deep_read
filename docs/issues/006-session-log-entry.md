# Issue 006 — Session Log entry

**Type:** AFK
**Blocked by:** #003, #005
**Label:** ready-for-agent

## What to build

Verify and harden the Session Log write behavior in `SKILL.md` step 7. After the module section is written and the Field Notes sweep is complete, the skill must prepend a dated entry to the Session Log timeline.

The entry must:
- Use today's date in `YYYY-MM-DD` format
- Include a one-line summary of what was explored
- Link to every module section touched in this session using its anchor (`#module-<slug>`)
- Be prepended (newest first), not appended

The session number badge (`<div class="badge">N</div>`) must increment from the previous highest entry. On the first session it is `1`.

If multiple modules were touched in a single session (e.g. the user ran `/deep-read` on two directories back to back before ending), the single log entry must link to all of them.

## Acceptance criteria

- [ ] Session Log entry prepended (not appended) after every completed session
- [ ] Entry date is correct (`YYYY-MM-DD`)
- [ ] Entry links to all modules touched that session using correct anchor hrefs
- [ ] Session number badge increments correctly from the previous entry
- [ ] One-line summary accurately describes what was explored (not generic boilerplate)
- [ ] HTML file remains valid after the write

## Blocked by

- #003 module-write-nav
- #005 field-notes-sweep
