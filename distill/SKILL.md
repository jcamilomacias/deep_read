---
name: distill
description: Surface insights from the current conversation and propose them as a numbered list for the deep-read learning ledger. You keep or drop each one — nothing lands in the ledger without approval. Use when the user wants to capture what was learned in a conversation into docs/learning-ledger.html.
---

# /distill

Run any time during a session to surface insights from the conversation and route them into the learning ledger with explicit control over what lands.

See [REFERENCE.md](~/.claude/skills/deep-read/REFERENCE.md) for the HTML ledger structure, slug rules, chip vocabulary, and section templates.

---

### Session workflow

#### 1. Scan the conversation for candidate insights

Look back through the conversation from the last `/distill` invocation (or session start for the first run) to now. Extract distinct, non-trivial insights — things a teammate would want to find in the ledger later.

**What counts as an insight:**
- A non-obvious behavior or constraint that was discovered
- A design decision and its reason
- A data or execution flow that was traced
- A gotcha, edge case, or failure mode that came up
- A general language or library pattern that was demonstrated or explained

**What does not count:**
- Questions or hypotheses that were not confirmed
- Trivial facts derivable by reading the code directly
- Boilerplate or setup steps

For each insight, infer its routing:
- **Module section** — if the insight is specifically about a module identifiable by path or class name. Derive the slug using the slug rules in REFERENCE.md. Assign the chip type that fits best: Key Findings, Flow Traces, Open Tasks, or a custom type (Gotchas, Dependencies, Configuration Surface, Performance Notes, Data Contracts).
- **Field Notes** — if the insight is a general concept, language feature, or pattern not tied to a specific module. Assign the chip type: Language, Pattern, Library, or Other.

If no insights are found, output: "No distillable insights found in this conversation window." Stop. Do not write anything.

#### 2. Present the candidate list

Output all candidates as a numbered list. Each item on one line:

```
Distill candidates:

1. [→ auth / Key Findings] Token refresh happens on every request — there is no background refresh loop.
2. [→ auth / Gotchas] Passing an empty `scopes` list silently grants all scopes instead of raising.
3. [→ services-payment / Flow Traces] Payment flow: `initiate()` → `stripe.charge()` → webhook confirmation → `mark_paid()`.
4. [→ field-notes / Language] Python's `functools.lru_cache` does not respect TTL — entries live forever until the process restarts.
5. [→ auth / Open Tasks] [ ] Investigate why `refresh_token()` is called on GET requests — `src/auth/tokens.py:88`

Keep which? (e.g. `keep 1 3 5`, `keep all`, `keep none`)
```

Stop here. Wait for the user's response before doing anything else.

#### 3. Parse the user's selection

- `keep all` → accept every candidate.
- `keep none` → nothing is written. Stop. No session log entry.
- `keep N [M P …]` → keep only the numbered items listed. Discard the rest.
- Any other response → ask the user to clarify using the format above.

#### 4. Confirm routing targets

Group the kept items by target. Show one line per group, then wait:

```
Routing:
  • Items 1, 2, 5 → `auth` module
  • Item 3 → `services-payment` module
  • Item 4 → Field Notes

OK? (or correct with e.g. "item 3 → auth")
```

Wait for confirmation or correction before writing.

Apply any corrections: reassign the named item to the specified target. Re-derive the slug if the user provides a path (using slug rules from REFERENCE.md).

#### 5. Deduplicate, handle Overview updates, and write

Read `./docs/learning-ledger.html`.

**Deduplication:** For each kept item, compare its insight text against existing content in the target section. If a semantically equivalent entry already exists (same fact, even if differently worded), skip that item silently. Do not write it.

**Overview update detection:** If any kept item describes what a module fundamentally does (i.e., it would replace or significantly extend the Overview paragraph), treat it as an Overview update candidate. Show it separately before writing:

```
Overview update for `auth`:
  Old: "Handles token issuance and validation for all API requests…"
  New: "Manages token issuance, refresh, and scope enforcement for all API requests…"

Replace? (yes/no)
```

Wait for the user's answer. Only overwrite the Overview paragraph on "yes." Skip the update on "no" — discard the item entirely.

**Write the remaining approved items:**

For each non-Overview, non-duplicate item:

1. **Existing module section** — append the item as a new `<li>` under the matching chip section card. If the chip section (e.g. Gotchas) does not yet exist in the module, add the card using the custom chip section template from REFERENCE.md. Place it after the last existing card in the `<div class="grid-2">` or as a new card after it.

2. **Module not yet in the ledger** — create the full section: nav link + `<section>` block using the module section template from REFERENCE.md. Write an Overview paragraph inferred from the conversation context. Collect all approved items for that module into appropriate chip section cards. Omit any chip section that has no items. Follow the same two-write sequence as `/deep-read` step 3 (nav link first, then section).

3. **Field Notes item** — append a new card to `<div class="grid-3">` in the Field Notes section using the Field Notes card template from REFERENCE.md.

After all writes, confirm no unclosed tags were introduced by scanning the written blocks.

#### 6. Log the session

Only append a session log entry if at least one item was written to the ledger.

Follow the same steps as `/deep-read` step 4:
- Determine N (highest existing badge + 1, or 1 if the timeline is empty).
- Summary names what was distilled. Example: "Distilled 3 insights into `auth` and 1 to Field Notes."
- Touched list links to every section written (module slugs and/or Field Notes).
- Prepend as the first child of `<div class="timeline" id="session-log-timeline">`.
