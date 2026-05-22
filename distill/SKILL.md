---
name: distill
description: Read the current conversation and update the living documentation ledger. Sections are created or updated dynamically from the conversation's actual content — not reinterpreted through a template. User questions are preserved hidden inside each section; the document reads as clean technical documentation.
---

# /distill

Reads the conversation since the last `/distill` invocation (or session start) and updates `./docs/learning-ledger.html` with documentation sections that emerge from the conversation's content.

See [REFERENCE.md](~/.claude/skills/deep-read/REFERENCE.md) for HTML patterns and the section template.

---

### Workflow

#### 1. Read the conversation window

Scan from the last `/distill` call (or session start) to now. Identify **topics** — cohesive subjects that came up: a code flow, a concept explained, a gotcha discovered, a decision made, a pattern demonstrated, a question answered in depth.

A topic is distillable if it produced a real explanation — something a teammate would want to find and read later. Skip meta-conversation, unanswered questions, and trivial exchanges.

If nothing distillable exists: output `Nothing to distill from this conversation window.` Stop.

#### 2. Plan the distill

Read `./docs/learning-ledger.html` to check which sections already exist.

For each topic, decide:
- **[new]** — no matching section exists yet; will create one.
- **[update → Section Title]** — a matching section exists; will add or revise content.

Show the plan before writing anything:

```
Distill plan:

1. [new] "Token Refresh Flow"  —  src/auth/tokens.py
   Will cover: middleware triggers refresh synchronously, 15-min TTL, concurrent-refresh race condition

2. [update → Auth Module]  "Scope validation gotcha"
   Will add: empty scopes list silently grants all scopes instead of raising

3. [new] "Python lru_cache — No TTL"
   Will cover: cache entries live for the process lifetime, no built-in expiry mechanism

Proceed? (yes / edit / skip N)
```

Wait for the user's response before writing anything. Options:
- `yes` — proceed as shown.
- `edit` — apply corrections the user describes (reassign, rename, drop, merge items).
- `skip N` — drop item N from the plan.

#### 3. Write

Read `html-playbook.html` in the project root as your visual reference — it shows every CSS component available (cards, grids, timelines, callouts, matrix tables, chips, diagrams, choosers). Use whichever of those components fit the content. There is no prescribed structure.

For each approved topic:

**[new] section:**
Invent the section structure that best serves the content. Ask: what does a teammate need to *see* here — a prose explanation, a flow diagram, a comparison table, a code walkthrough, a warning callout? Build that. Add a nav link in `<aside>` pointing to the section's `id`.

Every section should end with a `<details class="source-q">` containing the user questions from the conversation that prompted it (collapsed by default — this is reference context, not primary content).

**[update] section:**
Read the existing section. Add what's new — extend the prose, add a callout for the new gotcha, insert a table if the new content is comparative, append to the question list in `.source-q`. Don't duplicate what's already there.

**Writing style:**
- Write as a knowledgeable teammate explaining what they found — conversational, not API docs.
- Use prose paragraphs as the base. Reach for visual components (callouts, grids, timelines) when they make the content clearer, not for decoration.
- Include code snippets when they illustrate a non-obvious point — short, focused excerpts, not full file dumps.
- Capture the *why* and the *gotcha*, not just the *what*.

#### 4. Log the session

Only if at least one section was written or updated.

If the ledger has a session log (a timeline section tracking exploration history), add an entry: date, one-line summary of what was distilled, and links to every section touched. If no session log exists yet, decide whether adding one would be useful given the document's current shape — it is not required.
