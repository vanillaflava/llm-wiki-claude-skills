---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight — does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
compatibility: Works with any markdown knowledge base supporting [[wikilinks]] — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "2.1"
---

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `vault_root` — absolute path to the wiki root
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

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
