---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight — does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
---

<!-- version: 2.6 -->

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `wiki_root` — absolute path to the wiki root (may be a subfolder of a larger system)
- `blacklist` — paths never to write to
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
Add Git repos, source code folders, or any area that should never receive wiki writes.

**index_excludes** — Paths excluded from index.md tracking.
`raw\` and `ingested\` are always excluded — source files are not wiki pages.
`archive\` keeps deprecated pages out of the live catalogue.

**ingested_folder / ingested_subdirs** — Archive configuration for wiki-ingest.
Carried here so the config is valid regardless of which skill runs first and triggers init.

**log_format** — Do not change without updating all wiki skills.
```


---

## Capability Requirements

This skill requires **filesystem read and write access**. If unavailable, inform the user and stop.

---

## When to Use This Skill

Run wiki-integrate when:
- A new wiki page has been created directly in a chat (not via wiki-ingest)
- An existing page has been significantly revised and may have new connection opportunities
- Pages exist in the vault that are orphaned from the knowledge graph

**wiki-ingest** processes raw source material into wiki pages and links them automatically. **wiki-integrate** links pages that already exist as wiki content. Do not use one as a substitute for the other.

---

## Input

Wiki-integrate operates on a **target page**. Provide the path to the newly created or updated page. If no path is provided, ask the user which page to integrate before proceeding.

---

## Workflow

### Step 1 — Read the target page

Read the full content of the target page. Identify the main topic, key concepts, existing `[[wikilinks]]`, and the frontmatter title.

### Step 2 — Check the blacklist

Confirm the target page is not in a blacklisted path. If it is, stop — wiki skills cannot modify blacklisted paths.

### Step 3 — Check and update index.md

Read `index.md`. If the target page is not listed, add an entry in the correct section:
`- [[Page Title]] — One sentence description. \`relative/path/to/page.md\``

If already listed, verify the description is still accurate. Update if significantly stale.

### Step 4 — Find related pages

From the index catalogue, select 3–8 candidate pages with genuine topic overlap. Connections should be meaningful, not just keyword matches.

### Step 5 — Read candidate pages

Read each candidate in full. Confirm a genuine connection exists — shared concepts, one builds on the other, or a reader would benefit from knowing both. Discard candidates where no real connection exists.

### Step 6 — Add backlinks

For each confirmed pair, add links in both directions:
- In the target page: add `[[Candidate Page Title]]` where it fits naturally
- In the candidate page: add `[[Target Page Title]]` where it fits naturally

Write only the new wikilink — do not restructure or rewrite candidate page content. Check each candidate is not blacklisted before writing.

### Step 7 — Append to log.md

```
## [YYYY-MM-DD] integrate | <Target Page Title>
Added to index.md. Linked from: [[Page A]], [[Page B]]. Linked to: [[Page C]].
```

### Step 8 — Summarise

Report: whether the target was added to index.md, which pages now link to it, which pages it now links to, and any connections ruled out with brief reasoning.

---

## Key Rules

1. **Content is unchanged** — only adds wikilinks and index entries; never rewrites or restructures
2. **Never write to blacklisted paths**
3. **Links must be genuine** — only add links where a real connection exists
4. **Minimal footprint in candidate pages** — add only the new wikilink
5. **One target per run**
6. **Do not integrate archive/ or assets/ pages**


