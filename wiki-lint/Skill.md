---
name: wiki-lint
description: Health-check the wiki. Scans all pages for broken wikilinks, orphaned pages, stale index entries, missing connections between related pages, and orphaned binary assets. Produces a dated lint report in archive/. Use when you say /wiki-lint or periodically to maintain knowledge graph health. Never auto-fixes anything — report only. Requires filesystem read access and write access to archive/.
compatibility: Works with any markdown knowledge base supporting [[wikilinks]] — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "2.1"
---

# Wiki Lint

Health-checks the wiki and produces a report. Never modifies wiki content.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `vault_root` — absolute path to the wiki root
- `blacklist` — paths to skip entirely during the scan
- `index_excludes` — paths excluded from index.md
- `log_format` — format string for log entries

**If `wiki-config.md` is not found — run init:**
1. Ask the user: *"I couldn't find wiki-config.md. Please provide the path to your knowledge base root — the top-level folder containing your notes."*
2. Write `wiki-config.md` at that path using the template below
3. If `index.md` does not exist, create it:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Updated by wiki-ingest and wiki-integrate.*
   ```
4. If `log.md` does not exist, create it:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
5. Confirm what was created and proceed

**Config template — write frontmatter block first, then body:**

```
---
vault_root: /absolute/path/to/your/knowledge-base

blacklist:
  - Projects\
  - Archive\

index_excludes:
  - raw\
  - archive\

raw_subdirs:
  - clippings
  - documentation
  - papers
  - articles
  - data
  - notes

log_format: "## [YYYY-MM-DD] {type} | {subject}"
---
```

Then write this body after the closing `---`:

```markdown
## Configuration Guide

**vault_root** — Absolute path to your knowledge base root.
Windows example: `C:\Users\yourname\Documents\MyWiki`
macOS example: `/Users/yourname/Documents/MyWiki`

**blacklist** — Paths wiki skills must never write to (relative to vault_root).
Add Git repos, source code folders, or any area that should never receive agent writes.

**index_excludes** — Paths excluded from index.md tracking.
`raw\` is always excluded — source material is indexed only after wiki-ingest processes it.
`archive\` keeps deprecated pages out of the live catalogue.

**raw_subdirs** — Subdirectories inside raw\ for organising source material.
Wiki-ingest reads from here. Adapt freely — these are suggestions:
- `clippings` — web saves and browser clips (point browser plugins like Obsidian Web Clipper here)
- `documentation` — product docs, API references, technical specifications
- `papers` — academic papers, PDFs, research material
- `articles` — blog posts, news, long-form reading
- `data` — CSV, JSON, structured datasets
- `notes` — freeform drafts, quick capture, anything else
Other common additions: `source` for code snippets or reference implementations

**log_format** — Do not change without updating all wiki skills.
```


---

## Capability Requirements

This skill requires **filesystem read access** (to scan all pages) and **write access to `archive/`** (to file the lint report). If read access is unavailable, this skill cannot proceed.

---

## Workflow

### Step 1 — Build the page inventory

List all `.md` files in the vault recursively. Exclude paths in `blacklist` and `index_excludes`. This is the **in-scope page set**. Also list all non-markdown files outside blacklisted paths, raw\, archive\, and Projects\ as potential orphaned binary assets.

### Step 2 — Read index.md

Read `index.md`. Parse all wikilink references and file paths. Build a set of pages listed in index.md and their referenced paths.

### Step 3 — Check for broken wikilinks

For each in-scope wiki page, extract all `[[wikilinks]]`. Attempt to resolve each to an actual file. If no matching file is found: flag as a **broken wikilink** with the page where found, the broken link text, and a suggested correction if obvious.

### Step 4 — Check for orphan pages

For each page in the in-scope set, check:
- Is it listed in `index.md`?
- Is it referenced by any wikilink in any other in-scope page?

If neither: flag as an **orphan page**. New pages may be legitimate orphans — flag anyway, the human decides.

### Step 5 — Check for stale index entries

For each page listed in `index.md`, check whether the referenced file exists. If not: flag as a **stale index entry**.

### Step 6 — Qualitative and structural passes

**Conceptual issues:** Read `Overview.md` and scan for obvious contradictions or staleness — claims that appear to contradict specific wiki pages, or pages that seem significantly out of date. Qualitative pass only, not a full content review.

**Missing connections:** Scan `index.md` descriptions for significant term overlap between page pairs that do not link to each other. Pages that share multiple key concepts in their index descriptions but have no mutual wikilinks are candidates for missing connections. Flag pairs with meaningful overlap — do not flag tenuous keyword coincidences. This is a lightweight heuristic pass; wiki-integrate handles the actual linking if the user agrees a connection is genuine.

### Step 7 — Check for orphaned binary assets

For each non-markdown file outside blacklisted paths, raw\, archive\, and Projects\: search all in-scope pages for any reference to this filename. If none found: flag as an **orphaned binary asset**. A file that is contextually placed alongside related content (e.g. a PDF in a domain subfolder referenced by domain pages) is not an orphan — only flag files with no wiki references anywhere.

### Step 8 — Write the lint report

Create `<vault_root>/archive/lint-YYYY-MM-DD.md`:

```markdown
---
title: Lint Report YYYY-MM-DD
date: YYYY-MM-DD
---

# Lint Report — YYYY-MM-DD

## Summary
- Broken wikilinks: N
- Orphan pages: N
- Stale index entries: N
- Missing connections (candidates): N
- Orphaned binary assets: N
- Conceptual flags: N

## Broken Wikilinks
[page where found, broken link, suggested fix]

## Orphan Pages
[page path, reason it may be orphaned]

## Stale Index Entries
[index entry, the path that no longer resolves]

## Missing Connections
[page pair, overlapping terms, suggested action: run wiki-integrate]

## Orphaned Binary Assets
[file path, reason it has no wiki references]

## Conceptual Flags
[page, nature of potential issue]
```

### Step 9 — Append to log.md

```
## [YYYY-MM-DD] lint | Full wiki pass
Summary: N broken links, N orphans, N stale entries, N missing connections, N orphaned assets.
Report: archive/lint-YYYY-MM-DD.md
```

### Step 10 — Present findings

Report the summary. Do not offer to auto-fix anything. Suggest follow-up:
- Broken wikilinks → fix manually or run wiki-integrate on affected pages
- Orphan pages → run wiki-integrate, or move to archive/ if deprecated
- Stale index entries → remove from index.md or update the path
- Missing connections → run wiki-integrate on the flagged page pairs
- Orphaned assets → move to an appropriate location or delete if unwanted

---

## Key Rules

1. **Never modify wiki pages** — read-only except for writing the lint report to archive/
2. **Never delete files** — flag only; the human decides
3. **Do not flag contextually-placed files** — a file with wiki references in its domain is not an orphan
4. **Blacklisted paths are skipped entirely**
5. **Report is filed in archive/, not the wiki root**

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
