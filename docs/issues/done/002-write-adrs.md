# 002 — Write ADRs: one-shot write and command separation

**Blocked by:** None — can start immediately
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/3

## What to build

Create two Architecture Decision Records documenting the key design decisions made during the skill redesign.

**ADR 1: `/deep-read` writes without user confirmation**
Record the decision to remove the draft-confirm loop. The prior loop was the root cause of the autonomy problem: it forced coarse block-level approval. The replacement mechanism is `/distill`, which provides fine-grained per-item control after the fact.

**ADR 2: Reading and documenting are separated across two commands**
Record the decision to split the original fused workflow. `/deep-read` produces a structural anchor; `/distill` produces a personalized record. The ledger grows to reflect the user's understanding rather than Claude's first pass.

ADRs live at `docs/adr/` with sections: status, context, decision, consequences.

## Acceptance criteria

- [ ] `docs/adr/0001-deep-read-writes-without-confirmation.md` exists with status, context, decision, and consequences sections
- [ ] `docs/adr/0002-reading-and-documenting-separated.md` exists with status, context, decision, and consequences sections
- [ ] Both ADRs are marked status: `Accepted`
- [ ] Each ADR names the alternative that was rejected and why
