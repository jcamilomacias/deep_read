---
name: distill-to-docs
description: Read the current conversation and write it as full Markdown documentation to a specified MkDocs file. The complete content is displayed in the terminal as Markdown and written to the target file — not a summary, the full answer. Usage: /distill_to_docs @path/to/document.md
---

# /distill_to_docs

Reads the current conversation and writes the full content as MkDocs-compatible Markdown — displayed in the terminal and written to the target file.

```
/distill_to_docs @docs/auth.md
/distill_to_docs @docs/concepts/token-refresh.md
```

---

### Workflow

#### 1. Parse the argument

The argument must be a single `@`-prefixed path to a `.md` file.

- No argument → ask the user for a target file path. Do not proceed without one.
- Not a `.md` path → tell the user this command writes to Markdown files only.
- Path does not exist → create the file (and any missing parent directories) when writing.

#### 2. Read the last turn

Read only the most recent exchange: the user's last message and your response to it. Nothing earlier.

Write the full content of that response — the explanation, code, flow, or walkthrough exactly as it was given. Do not shorten, filter, or reframe it.

#### 3. Compose the Markdown

Write as MkDocs-compatible Markdown. Structure:

- `#` title at the top — the subject of this document
- `##` / `###` headings to divide topics as they came up
- Fenced code blocks with language identifiers (` ```python `, ` ```bash `, etc.)
- MkDocs admonitions for callouts:
  ```
  !!! note "Title"
      Body text here.

  !!! warning
      Something important to flag.

  !!! tip
      A useful shortcut or pattern.
  ```
- Collapsible block for source questions — the actual questions from the conversation, for reference:
  ```
  ??? question "Source questions"
      - "The user's question as asked."
      - "A follow-up that prompted more depth."
  ```
- Standard Markdown tables for comparisons or matrices
- Prose paragraphs as the base — headings and visual elements to organize, not decorate

Do not summarize or compress. Write the full explanation as it emerged. If the conversation included a code example, include the full code example. If an edge case was worked through step by step, include that walk-through.

#### 4. Display in terminal

Output the complete Markdown to the terminal. The user reads it before the file is written.

#### 5. Write to file

**File does not exist:** create it with the composed Markdown. Create any missing parent directories first.

**File exists and has content:** read the existing content. Check whether the new content covers topics already documented in the file.
- No overlap → append the new content after a `---` horizontal rule separator.
- Overlap detected → show which sections would overlap and ask: append anyway, merge into existing sections, or cancel.

After writing, confirm the file path.
