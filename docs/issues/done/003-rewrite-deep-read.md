# 003 — Rewrite /deep-read as one-shot writer

**Blocked by:** 001
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/4

## What to build

Replace the existing 7-step `/deep-read` workflow (read → draft → confirm → write → field notes → session log) with a 2-step one-shot flow: read, then write immediately to `docs/learning-ledger.html` with no confirmation prompt.

The skeleton written consists of an Overview paragraph plus Key Findings plus any additional chip sections Claude deems useful for the specific module. Claude is not limited to the standard chip vocabulary — if a module warrants a "Gotchas" or "Dependencies" section, Claude should create it. Tone throughout is conversational: a knowledgeable friend explaining the code.

On revisit (section already exists): Claude proposes an Overview merge and does not silently overwrite. The session log entry is still written after each invocation. The Field Notes sweep (old step 6) is removed from this command's flow.

## Acceptance criteria

- [ ] Invoking `/deep-read @path` writes a section to the ledger immediately — no confirmation prompt appears
- [ ] The written section contains an Overview paragraph and at least one Key Findings bullet
- [ ] Claude may add chip sections beyond Key Findings, Flow Traces, and Open Tasks when warranted
- [ ] The tone of the Overview and findings is conversational, not formal documentation language
- [ ] Invoking `/deep-read` on a path with an existing section proposes an Overview merge rather than silently overwriting
- [ ] A session log entry is appended after each invocation
- [ ] The Field Notes sweep (old step 6) is removed from this command's flow
