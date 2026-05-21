# 004 — Implement /distill command

**Blocked by:** 001
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/5

## What to build

Implement the `/distill` command. This command captures insights from the current conversation and routes them into the learning ledger with explicit user control over what lands.

**Flow:**
1. Extract candidate insights from the conversation since the last `/distill` invocation (or session start for the first run)
2. Present a numbered proposal list. Each item format: `1. [→ module-slug / Chip Type] Insight text.`
3. User responds with which numbers to keep (e.g. `keep 1 3 5`)
4. Claude states the inferred target module in one line and waits for confirmation or correction
5. Write approved items: module-specific findings go to the relevant module section; general concepts go to Field Notes
6. If a proposed item would update the Overview, show it separately and require explicit confirmation before overwriting
7. If the target module has no existing section, create a full section: Overview inferred from conversation context plus the distilled findings
8. Deduplicate by comparing proposed items against existing ledger content — skip anything already captured
9. Append a session log entry

## Acceptance criteria

- [ ] Running `/distill` produces a numbered candidate list with routing tags before any write occurs
- [ ] Each candidate shows destination module slug and chip type
- [ ] User can keep a subset by number; unkept items are not written
- [ ] Claude confirms the inferred target module before writing; user can correct it
- [ ] Overview update proposals appear separately and require their own confirmation
- [ ] Running `/distill` twice with identical conversation content produces no new ledger entries on the second run
- [ ] Running `/distill` with no prior `/deep-read` on the target module creates a complete new section (Overview + findings)
- [ ] General concepts route to Field Notes; module-specific findings route to the module section
- [ ] A session log entry is appended after each invocation
