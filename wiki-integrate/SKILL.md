---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight; does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
---

<!-- version: 2.7 -->

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Configuration

**Wiki root is the directory containing `wiki-config.md`.** The skill derives it at runtime from the config file's location; it is not stored in the config.

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. The directory containing `wiki-config.md` is the wiki root for this session. Read the config and extract:
- `blacklist` - paths never to write to, relative to wiki root
- `index_excludes` - paths excluded from index.md
- `log_format` - format string for log entries

**Sanity-check the wiki root before writing anything:**
- If the resolved wiki root is a bare drive root, OS root, or user home (`C:\`, `D:\`, `/`, `/home/`, `/Users/`, `C:\Users\`), stop and ask the user to confirm. A wiki root at one of these is almost always a misplaced config file.
- If the resolved wiki root is outside the scope the MCP can actually reach, stop and tell the user. Do not attempt to widen scope.

**If `wiki-config.md` is not found - run init:**
1. Ask the user: *"I couldn't find wiki-config.md. Please provide the absolute path to your wiki root; the folder containing your notes. This folder must be inside your MCP scope, and this is where I will write wiki-config.md."*
2. Write `wiki-config.md` at that path using the template below. The directory you were given is the wiki root.
3. If `index.md` does not exist at the wiki root, create it:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Updated by wiki-ingest and wiki-integrate.*
   ```
4. If `log.md` does not exist at the wiki root, create it:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
5. Confirm what was created and proceed

**Config template - write frontmatter block first, then body:**

```
---
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

**Wiki root** - The directory containing this `wiki-config.md` file is the wiki root. The skills derive it from the config file's location; you do not need to write the path anywhere. If you move the wiki, move this file with it. Wiki root must be inside your MCP scope or the skills cannot reach your notes.

**blacklist** - Paths where wiki page creation is forbidden, relative to wiki root.
Add Git repos, source code folders, or any area that should never receive wiki writes.

**index_excludes** - Paths excluded from index.md tracking.
`raw\` and `ingested\` are always excluded; source files are not wiki pages.
`archive\` keeps deprecated pages out of the live catalogue.

**ingested_folder / ingested_subdirs** - Archive configuration for wiki-ingest.
Carried here so the config is valid regardless of which skill runs first and triggers init.

**log_format** - Do not change without updating all wiki skills.
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

### Step 1 - Read the target page

Read the full content of the target page. Identify the main topic, key concepts, existing `[[wikilinks]]`, and the frontmatter title.

### Step 2 - Check the blacklist

Confirm the target page is not in a blacklisted path. If it is, stop; wiki skills cannot modify blacklisted paths.

### Step 3 - Check and update index.md

Read `index.md`. If the target page is not listed, add an entry in the correct section:
`- [[Page Title]]: One sentence description. \`relative/path/to/page.md\``

If already listed, verify the description is still accurate. Update if significantly stale.

### Step 4 - Find related pages

From the index catalogue, select 3-8 candidate pages with genuine topic overlap. Connections should be meaningful, not just keyword matches.

### Step 5 - Read candidate pages

Read each candidate in full. Confirm a genuine connection exists: shared concepts, one builds on the other, or a reader would benefit from knowing both. Discard candidates where no real connection exists.

### Step 6 - Add backlinks

For each confirmed pair, add links in both directions:
- In the target page: add `[[Candidate Page Title]]` where it fits naturally
- In the candidate page: add `[[Target Page Title]]` where it fits naturally

Write only the new wikilink; do not restructure or rewrite candidate page content. Check each candidate is not blacklisted before writing.

### Step 7 - Append to log.md

```
## [YYYY-MM-DD] integrate | <Target Page Title>
Added to index.md. Linked from: [[Page A]], [[Page B]]. Linked to: [[Page C]].
```

### Step 8 - Summarise

Report: whether the target was added to index.md, which pages now link to it, which pages it now links to, and any connections ruled out with brief reasoning.

---

## Key Rules

1. **Content is unchanged:** only adds wikilinks and index entries; never rewrites or restructures
2. **Never write to blacklisted paths**
3. **Links must be genuine:** only add links where a real connection exists
4. **Minimal footprint in candidate pages:** add only the new wikilink
5. **One target per run**
6. **Do not integrate archive/ or assets/ pages**

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.


