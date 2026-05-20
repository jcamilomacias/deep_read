# Issue 004 — Merge logic on module revisit

**Type:** AFK
**Blocked by:** #003
**Label:** ready-for-agent

## What to build

Verify and harden the merge behavior described in `SKILL.md` step 5. When `/deep-read` is run a second time on a target whose module section already exists in the ledger, the skill must surface a diff proposal before writing anything, and must not write until the user confirms.

The diff proposal must be explicit and human-readable:
> "Adding 2 findings, updating Overview (old version becomes stale), replacing the Login flow trace. OK?"

Three failure modes to verify do not occur:
- **Silent overwrite** — the section is replaced without any proposal shown
- **Append** — new findings are added as a sub-block below existing content rather than merged in
- **No-op** — the skill finds an existing section and stops without offering anything

If any failure mode is observed, update `SKILL.md` step 5 merge instructions to prevent it.

## Acceptance criteria

- [ ] Running `/deep-read` twice on the same target produces a diff proposal on the second run, not a silent write
- [ ] The proposal names specific changes (additions, removals, replacements) by sub-block
- [ ] No HTML is written until the user explicitly confirms the proposal
- [ ] After confirmation, the resulting module section is a single clean current picture (not two stacked versions)
- [ ] The Overview paragraph is updated/replaced, not appended to
- [ ] `SKILL.md` updated to close any ambiguities found during verification

## Blocked by

- #003 module-write-nav
