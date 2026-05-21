# 005 — Implement /capture command

**Blocked by:** 001
**GitHub:** https://github.com/jcamilomacias/deep_read/issues/6

## What to build

Implement the `/capture @notebook.ipynb` command. This command scans a single specified notebook for cells tagged with `# CAPTURE`, documents each one in the learning ledger, and removes the tag from the notebook.

**Flow:**
1. Read the specified notebook only — no directory sweeps
2. Find all cells containing a `# CAPTURE` comment
3. For each tagged cell: infer which module section it belongs to from the cell's content
4. Write to the ledger: a code block with the `# CAPTURE` comment stripped, followed by Claude's conversational interpretation of what the cell demonstrates
5. Edit the notebook to remove the `# CAPTURE` tag from each processed cell
6. If no tagged cells are found, report "nothing to capture" and make no ledger changes
7. Append a session log entry

Routing follows the same rules as `/distill`: general concepts go to Field Notes; module-specific behavior goes to the relevant module section.

## Acceptance criteria

- [ ] Invoking `/capture @notebook.ipynb` with two `# CAPTURE` cells writes exactly two entries to the ledger
- [ ] Each written entry contains a code block (tag stripped) followed by a conversational interpretation paragraph
- [ ] After the command completes, zero `# CAPTURE` strings remain in the notebook file
- [ ] Untagged cells are not read or written
- [ ] Invoking `/capture` on a notebook with no tagged cells produces a "nothing to capture" message and no ledger changes
- [ ] The command does not scan any notebook other than the one specified
- [ ] A session log entry is appended after each invocation with at least one write
