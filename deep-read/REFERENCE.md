# Deep Read — Reference

## Visual theme

The ledger uses the same visual language as `html-playbook.html` (in the project's root). Read that file to see every CSS component in action. The scaffold provides the full CSS; use any component from it.

**Available components:**

| Component | Class(es) | When to use |
|---|---|---|
| Hero header | `.hero`, `.kicker`, `h1`, `.dek` | Document title and intro |
| Section heading | `.section-heading`, `.section-number`, `h2`, `.lead` | Start of each section |
| Cards | `.card`, `.grid-2 / 3 / 4` | Group related information |
| Hero stat cards | `.hero-cards`, `.hero-card` | Key numbers or at-a-glance stats |
| Chips | `.chip.green / teal / amber / red / blue / violet` | Section type labels |
| Callout | `.callout` | Important asides, warnings, key insights |
| Timeline | `.timeline`, `.step`, `.badge` | Sequences, flows, session logs |
| Matrix table | `.matrix` | Comparison tables, decision matrices |
| Diagram | `.diagram` + inline SVG | Architecture diagrams, flow charts |
| Chooser | `.chooser`, `.option-list`, `.option`, `.play-panel` | Interactive option panels |
| Code blocks | `pre`, `code` | Code snippets |
| Prompt card | `.prompt-card`, `.copy` | Copyable text with copy button |
| List | `.list` | Styled bullet items |
| Source questions | `details.source-q` | Hidden question context (collapsed by default) |

## Source questions pattern

Every section written by `/distill` or `/deep-read` should include the questions that prompted it, hidden by default:

```html
<details class="source-q">
  <summary>Source questions</summary>
  <ul>
    <li>"The user question or agent inquiry that produced this section."</li>
  </ul>
</details>
```

## Scaffold placeholders

When initializing `docs/learning-ledger.html` from `scaffold.html`, replace:

| Placeholder | Replace with |
|---|---|
| `{{TITLE}}` | Document title (e.g. "Learning Ledger — my-project") |
| `{{MARK}}` | Short 2–3 char sidebar mark (e.g. "LL") |
| `{{NAV_TITLE}}` | Sidebar heading |
| `{{NAV_COPY}}` | Sidebar subtitle |
| `{{FOOTER}}` | Footer text |

## Slug rules (for `id` attributes and nav anchors)

1. Strip file extension and trailing `/`
2. Replace underscores and spaces with hyphens
3. Lowercase
4. For file paths: keep last two path components joined with hyphen
5. For invented titles: slugify the full title

Examples: `src/auth/` → `auth`, `Token Refresh Flow` → `token-refresh-flow`
