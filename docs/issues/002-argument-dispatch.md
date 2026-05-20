# Issue 002 — Argument dispatch: verify all four input types

**Type:** AFK
**Blocked by:** #001
**Label:** ready-for-agent

## What to build

Verify and harden the argument dispatch logic in `SKILL.md` by running `/deep-read` against each of the four input types and confirming the correct read behavior fires in each case. Refine `SKILL.md` wording wherever edge cases reveal ambiguity.

The four input types and expected behavior:

- **File path** (e.g. `@src/auth.py`) → read that file directly
- **Directory** (e.g. `@src/auth/`) → list contents, read entry points and `__init__.py`/index files first, then key modules
- **Notebook** (e.g. `@analysis.ipynb`) → read all cells and their outputs; map observations to Key Findings, pipeline steps to Flow Traces
- **Free text** (e.g. `"how does the event loop work?"`) → do not read anything; ask the user to point to relevant files first

If any input type produces incorrect routing or an unclear prompt to the user, update `SKILL.md` step 1 to fix it.

## Acceptance criteria

- [ ] File path input → skill reads the specified file and produces a draft scoped to that file
- [ ] Directory input → skill reads entry points first; draft covers the module as a whole, not individual files
- [ ] Notebook input → Key Findings contains experiment observations; Flow Traces contains analysis pipeline steps; raw cell outputs are not dumped verbatim
- [ ] Free text input → skill asks the user to locate relevant files before reading anything (does not attempt to guess or read blindly)
- [ ] `SKILL.md` updated to resolve any ambiguities found during verification

## Blocked by

- #001 scaffold-template
