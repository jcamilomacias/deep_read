# ADR 0002 — Reading and documenting are separated across two commands

**Status:** Accepted

## Context

The original `/deep-read` skill fused two distinct activities into a single workflow: understanding a codebase (reading files, tracing flows, forming a mental model) and documenting that understanding (curating findings, deciding what belongs in the ledger, writing the permanent record).

Fusing them meant every exploration session ended with a documentation step driven by Claude's read of the code. The ledger grew to reflect Claude's structural summary of the module rather than the user's evolving personal understanding. Items that mattered to the user but weren't prominent in the code were under-represented; boilerplate structural observations that Claude found significant were over-represented.

The confirmation loop (see ADR 0001) was an attempt to fix this by giving the user a veto over Claude's curation. It failed because the veto was too coarse to express preferences — it could only accept or reject the whole draft.

## Decision

Split the fused workflow into two separate commands with distinct responsibilities:

- **`/deep-read`** is responsible for producing a structural anchor: an Overview paragraph and Key Findings that reflect an honest reading of the code. It writes immediately and does not filter for what the user cares about. Its output is a starting point, not a finished record.
- **`/distill`** is responsible for producing a personalized record: it surfaces insights from the conversation (not from re-reading the code), lets the user keep or drop each one, and routes approved items to the right section. Its output reflects what the user actually found significant during exploration.

The rejected alternative was a single command with two modes — a fast mode that skips confirmation and a careful mode that proposes items for approval. This was rejected because mode-switching inside a single command creates ambiguity about which behavior the user gets. Separate commands make the contract explicit: `/deep-read` always writes immediately; `/distill` always requires approval.

A secondary alternative was making `/distill` optional — keeping the fused command but making confirmation opt-in. This was rejected because optional correction paths are rarely used. Separating the commands makes the pattern the default workflow rather than a feature the user has to remember.

## Consequences

- The ledger has two distinct layers of content: Claude's structural read (from `/deep-read`) and the user's curated understanding (from `/distill`). These can diverge, which is intentional.
- The workflow requires two commands to fully document a module. Users who only run `/deep-read` get a skeleton; the ledger only reflects their understanding once they also run `/distill`.
- `/distill` must handle the case where no `/deep-read` has been run for a module — it cannot assume a section exists. It creates a full section (Overview + findings) from conversation context when needed.
- The session log entry runs after any command that writes to the ledger, so partial workflows (only `/deep-read`, only `/distill`) are still recorded.
