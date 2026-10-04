---
name: sync-wiki
description: >
  Full rebuild of an Obsidian knowledge vault from a codebase. Reads every wiki page's `source:`
  frontmatter as a dependency manifest, re-reads those source files, and regenerates each page as a
  source-verified mirror of the current code. Rebuilds the index and appends a log entry.
  Triggers on: "/sync-wiki", "sync codebase to obsidian", "rebuild the wiki", "refresh wiki from code",
  "sync knowledge to obsidian".
allowed-tools: Read Write Edit Glob Grep Bash
---

# sync-wiki: Codebase → Obsidian, full rebuild

Keep an Obsidian knowledge vault a **source-verified mirror** of a codebase. Each wiki page declares
the codebase files it was derived from in its `source:` frontmatter — that list is the page's
dependency manifest. This skill re-reads those files and regenerates every page from scratch so the
wiki never silently drifts from the code.

## Core idea

The curated page set defines coverage — **not** the whole codebase. A vault of ~20 pages may map to a
few dozen `source:` files even when the repo has thousands. Only read what the pages declare. Adding
coverage = adding a page with a `source:` list; that is a manual editorial act, not this skill's job.

## Inputs

Parse the command args:

- **codebase-path** — 1st positional arg. Default: `git rev-parse --show-toplevel` from cwd; if that
  fails, fall back to `~/Develop/meri/meri-pos`.
- **vault-path** — 2nd positional arg. Default: `~/Develop/meri/BA/factsblend`.
- **`--dry-run`** — report planned changes, write nothing.

Validate both directories exist. If the vault is missing, stop and say: *"Vault not found at <path>.
Run /wiki to create one first."* — do not scaffold here.

## Workflow

### 1. Build the coverage map

Glob the vault for page files: `**/*.md`, excluding `00-index.md`, `wiki/log.md`, and anything under
`.obsidian/`. For each page, read its YAML frontmatter and split `source:` on commas into individual
entries. Resolve each entry against `codebase-path`.

Record per page: its path, title (first `# H1`), `source:` entries, and `ingested:` date. A page with
**no** `source:` frontmatter can't be verified — leave it untouched and list it in the final report.

### 2. Full-rebuild loop (one page at a time)

For every page that has a `source:` list:

1. **Read the sources.** Read each file. For directory entries (trailing `/`), list the directory and
   read the key files inside (enough to ground the page's claims). Use Grep/Glob to confirm specific
   classes, enums, or symbols the page references still exist.
2. **Flag dead sources.** If a `source:` entry no longer resolves (moved / renamed / deleted), note it
   for the log and drop it from the refreshed `source:` list. Don't invent a replacement.
3. **Re-derive the body** in the vault's house style (see Conventions). Every table row and claim must
   trace to a file you actually read in this run. No speculation, no carried-over claims you couldn't
   re-verify — if a previous claim no longer holds in the code, correct or remove it.
4. **Preserve the heading skeleton** where the structure still fits the code; restructure only when the
   code changed shape.
5. **Write** (skip if `--dry-run`): overwrite the body, refresh `source:` (dead entries removed), set
   `ingested:` to today's date.

### 3. Rebuild `00-index.md`

Regenerate the navigation from the actual page set. Title each entry from the page's first `# H1`.
Group by top-level folder (`architecture`, `business-flows`, `conventions`, `dev-workflow`, `modules`,
…) matching the existing index's section order. Use wikilinks: `[[meri-pos/business-flows/order-flow|Order Lifecycle]]`.

### 4. Append `wiki/log.md`

Newest entry at the **TOP** (never edit past entries). Match the existing format:

```markdown
## <YYYY-MM-DD> | Full rebuild

- Codebase: `<codebase-path>` @ <short git sha if available>
- Pages rebuilt: <n> (changed: <n>, unchanged: <n>)
- Sources read: <n> files across <n> pages
- Deltas:
  - [[meri-pos/business-flows/printer-flow]]: <what changed vs previous>
- Dead sources flagged:
  - `<path>` (referenced by [[page]]) — moved/deleted
```

### 5. Report to the user

Concise summary: pages rebuilt, changed vs unchanged counts, dead sources flagged, pages skipped (no
`source:`). In `--dry-run`, present this as "would change" and confirm nothing was written.

## Conventions (match the existing vault exactly)

- **Frontmatter** — keep the shape:
  ```yaml
  ---
  source: lib/application/domain/order.dart, lib/enum/order_state_type.dart
  ingested: 2026-05-29
  ---
  ```
- **Source-verified callout** near the top:
  ```markdown
  > [!key-insight] Source-verified
  > Derived from `order.dart`, `order_state_type.dart`, `open_order_usecase.dart`.
  ```
- **Attributions** — introduce facts with their origin: ``From `lib/enum/printer.dart`:`` then the table.
- **Wikilinks** — full vault-relative path with display alias: `[[meri-pos/.../page|Display]]`.
- **No invented claims.** Source-verified means every statement is grounded in a file read this run.
- **Never touch `.obsidian/`.** Don't reformat unrelated pages or the log's past entries.

## Idempotency

On an unchanged codebase a second run should report mostly "unchanged" — bodies regenerate to
equivalent content and only `ingested:` advances. Treat large unexplained diffs on a static codebase as
a signal you drifted from the source, not the code.
