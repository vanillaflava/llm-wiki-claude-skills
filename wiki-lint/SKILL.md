---
name: wiki-lint
description: Health-check the wiki. Scans all pages for broken wikilinks, orphaned pages, stale index entries, missing connections between related pages, orphaned binary assets, and orphaned sources in ingested/. Produces a dated lint report in archive/. Use when you say /wiki-lint or periodically to maintain knowledge graph health. Never auto-fixes anything — report only. Requires filesystem read access and write access to archive/.
---

<!-- version: 2.7 -->

# Wiki Lint

Health-checks the wiki and produces a report. Never modifies wiki content.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `wiki_root` — absolute path to the wiki root (may be a subfolder of a larger system)
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
wiki_root: /absolute/path/to/your/wiki-root

blacklist:
  - Repositories\

index_excludes:
  - raw\
  - archive\
  - ingested\

ingested_folder: ingested

ingested_subdirs:
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

**wiki_root** — Absolute path to the root of your wiki — the folder containing your notes.
This may be a subfolder of a larger system (a notes app, a sync folder, etc.). Set it to
wherever your .md files live, not necessarily the root of the enclosing application.
Windows example: `C:\Users\yourname\Documents\MyWiki`
macOS example: `/Users/yourname/Documents/MyWiki`

**blacklist** — Paths where wiki page creation is forbidden (relative to vault_root).
For wiki-lint: blacklisted paths are skipped entirely during scanning — lint will not
read content inside them. Add Git repos or any path that should be invisible to agents.

**index_excludes** — Paths excluded from index.md tracking.
`raw\` and `ingested\` are excluded — source files are not wiki pages.
`archive\` keeps deprecated pages out of the live catalogue.
Important: wiki-lint reads `archive\` and `ingested\` for link target validation even
though they are in index_excludes. A live page may link to archived or ingested content —
those links must resolve to real files to pass the check.

**ingested_folder / ingested_subdirs** — Archive configuration for wiki-ingest.
Carried here so the config is valid regardless of which skill runs first and triggers init.

**log_format** — Do not change without updating all wiki skills.
```


---

## Capability Requirements

This skill requires **filesystem read access** (to scan all pages) and **write access to `archive/`** (to file the lint report). If read access is unavailable, this skill cannot proceed.

---

## Workflow

### Step 1 — Build the page inventory

List all `.md` files in the vault recursively. Exclude paths in `blacklist` entirely — do not scan content inside them. Paths in `index_excludes` (`raw\`, `archive\`, `ingested\`) are excluded from being treated as wiki pages, but remain readable for link target validation: if a live wiki page links to a file in `archive/` or `ingested/`, that link must resolve to a real file. Build the **in-scope page set** from files outside both blacklist and index_excludes. Also list all non-markdown files outside blacklisted paths, raw\, archive\, and ingested\ as potential orphaned binary assets.

### Step 2 — Read index.md

Read `index.md`. Parse all wikilink references and file paths. Build a set of pages listed in index.md and their referenced paths.

### Step 3 — Check for broken wikilinks

For each in-scope wiki page, extract all `[[wikilinks]]`. Attempt to resolve each to an actual file. If no matching file is found: flag as a **broken wikilink** with the page where found, the broken link text, and a suggested correction if obvious.

Also flag: any wiki page containing a link that points into `raw/`. Once wiki-ingest moves files to `ingested/`, raw/ source links will break. Flag these as **future breakage warnings** — "[[Page]] links to raw/<filename>: will break after ingestion. Update to ingested/<subdir>/<filename> once ingest is complete." These are not yet broken but are guaranteed to become so.

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

For each non-markdown file outside blacklisted paths, raw\, archive\, and ingested\: search all in-scope pages for any reference to this filename. If none found: flag as an **orphaned binary asset**. A file contextually placed alongside related content (e.g. a PDF in a domain subfolder referenced by domain pages) is not an orphan — only flag files with no wiki references anywhere.

### Step 7a — Check for orphaned sources in ingested/

Every file in `ingested/` should have at least one wiki page that references it — via a Sources section in the page body containing the `ingested/` path. A source with no wiki reference has been processed but left no trace in the knowledge graph.

For each file in `ingested/` (all subdirs, including assets/):
1. Search all in-scope wiki pages for the file's relative path (e.g. `ingested/documentation/foo.md`). Check the full page content — `## Sources` section body, `changes:` frontmatter field, or anywhere else a path reference might appear. Treat any match as a valid reference regardless of where it appears.
2. If no match found: flag as an **orphaned source** — "ingested/[subdir]/filename has no wiki page referencing it"

A source in `ingested/assets/` with no reference is expected (it was unreadable at ingest time) — flag it at lower severity as a **note** rather than a warning, so the user knows it exists and can re-attempt ingestion if capabilities have improved.

### Step 8 — Write the lint report

Create `[wiki_root]/archive/lint-YYYY-MM-DD.md`:

```markdown
---
title: Lint Report YYYY-MM-DD
date: YYYY-MM-DD
---

# Lint Report — YYYY-MM-DD

## Summary
- Broken wikilinks: N
- Future breakage warnings (raw/ links): N
- Orphan pages: N
- Stale index entries: N
- Missing connections (candidates): N
- Orphaned binary assets: N
- Orphaned sources in ingested/: N (+ N notes in assets/)
- Conceptual flags: N

## Broken Wikilinks
[page where found, broken link, suggested fix]

## Future Breakage Warnings
[page where found, raw/ link, suggested post-ingest destination]

## Orphan Pages
[page path, reason it may be orphaned]

## Stale Index Entries
[index entry, the path that no longer resolves]

## Missing Connections
[page pair, overlapping terms, suggested action: run wiki-integrate]

## Orphaned Binary Assets
[file path, reason it has no wiki references]

## Orphaned Sources in ingested/
[file path, no wiki page references this source — consider re-ingesting or archiving]

## Notes: ingested/assets/
[file path, was unreadable at ingest time — re-attempt if capabilities have improved]

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
- Future breakage warnings → update raw/ links to ingested/ paths after the next ingest run
- Orphan pages → run wiki-integrate, or move to archive/ if deprecated
- Stale index entries → remove from index.md or update the path
- Missing connections → run wiki-integrate on the flagged page pairs
- Orphaned binary assets → move to an appropriate location or delete if unwanted
- Orphaned sources in ingested/ → re-ingest the source file (drop back into raw/ and run wiki-ingest), or investigate why no wiki page was created
- Notes in ingested/assets/ → retry ingestion if new tools or capabilities are available

---

## Key Rules

1. **Never modify wiki pages** — read-only except for writing the lint report to archive/
2. **Never delete files** — flag only; the human decides
3. **Do not flag contextually-placed files** — a file with wiki references in its domain is not an orphan
4. **Blacklisted paths are skipped entirely** — lint does not scan content inside blacklisted directories
5. **index_excludes are not wiki pages but are valid link targets** — archive/ and ingested/ are readable for link resolution even though they are excluded from the page inventory
6. **Report is filed in archive/, not the wiki root**

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
