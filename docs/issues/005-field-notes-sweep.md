# Issue 005 — Field Notes sweep + card generation

**Type:** AFK
**Blocked by:** #001
**Label:** ready-for-agent

## What to build

Verify and harden the Field Notes collection behavior in `SKILL.md` step 6. After every module write, the skill must ask the end-of-session sweep prompt and — if the user provides entries — write correctly structured tagged cards into the `#field-notes-grid` container.

Each card must use the correct chip class for its category:

| Category | Chip class     |
|----------|----------------|
| Language | `chip green`   |
| Pattern  | `chip violet`  |
| Library  | `chip blue`    |
| Other    | `chip amber`   |

The sweep prompt must fire even if the user says "no" — the ask itself is required. If the user declines, no card is written and the session proceeds to step 7.

The empty-state placeholder (`<div class="empty-state">`) in the Field Notes grid must be removed when the first card is written.

## Acceptance criteria

- [ ] End-of-session sweep prompt fires after every module write, without exception
- [ ] Each accepted entry produces a card in `#field-notes-grid` with correct chip class matching category
- [ ] Empty-state placeholder removed on first Field Notes write
- [ ] Multiple entries in one sweep each produce a separate card
- [ ] Declining the sweep produces no card and does not error
- [ ] Cards are appended inside `<div class="grid-3" id="field-notes-grid">`, not outside it

## Blocked by

- #001 scaffold-template
