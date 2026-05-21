# ADR 0001 — `/deep-read` writes without user confirmation

**Status:** Accepted

## Context

The original `/deep-read` skill had a 7-step workflow: read the target, draft the findings in the terminal, ask the user to confirm or correct the draft, then write to the ledger. The confirmation step was intended to keep the user in control of what landed in the ledger.

In practice the loop created friction without providing meaningful control. The confirmation was coarse: approve the whole draft or reject it and start over. Users could not accept three of the five Key Findings and reject two — they had to accept the entire block or go back. This forced Claude to produce a draft good enough that the user would accept it wholesale, which reinforced Claude's curatorial role over the ledger rather than the user's.

The deeper problem: every session began with a round-trip that delayed getting into the actual discussion. The confirmation prompt inserted a pause precisely at the moment when the user wanted to start exploring.

## Decision

Remove the confirmation loop entirely. `/deep-read` reads the target and writes a skeleton to the ledger immediately — no draft-in-terminal step, no confirmation prompt.

The rejected alternative was keeping a lighter confirmation: showing the draft but defaulting to "yes" after a short delay, or showing a summary line instead of the full draft. This was rejected because any confirmation prompt — however lightweight — trains the user to pause and review before continuing. The goal is zero interruption on the write path.

The correction path is `/distill`, not confirmation. After `/deep-read` writes the skeleton, the user can run `/distill` at any point to review what Claude inferred, propose changes, and approve exactly which items to update. `/distill` provides the fine-grained per-item control that the confirmation loop tried to approximate coarsely.

## Consequences

- `/deep-read` sessions start faster: the user moves into discussion immediately after the write.
- The ledger may contain Claude's first-pass interpretations that the user would have corrected at confirmation time. This is acceptable because `/distill` provides a cleaner correction mechanism after the fact.
- `/distill` becomes load-bearing for quality control. If `/distill` is not implemented or not used, errors from the initial write persist in the ledger without a correction path.
- The one exception to immediate writing is revisits: when `/deep-read` is called on a path that already has a section in the ledger, it proposes an Overview merge before overwriting, to prevent silent regression of existing understanding.
