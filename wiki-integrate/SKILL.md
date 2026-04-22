---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight; does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
---

<!-- version: 3.3 -->

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime. Pages this skill writes to follow the structure in `wiki-schema.md` - both files need to be present.

1. **Identify scope**: Determine filesystem scope root (allowedDirectories for MCP, CWD for Code, equivalent for other surfaces)

2. **Scope check**: If scope is bare drive root (`C:\`, `D:\`, `/`), OS root, or user home (`C:\Users\X`, `/home/X`, `/Users/X`) → skip to step 5 (do not search for privacy)

3. **Scan `<available_skills>` for `wiki-config`.** Note whether it's available - this shapes the recommendations below. The bundled `references/setup-help.md` is also available; read it if the user needs orientation or if you get stuck.

4. **Search and assess**: Search recursively for `wiki-config.md` (first-match, max 5 levels). If found, read it (`blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`) and look for `wiki-schema.md` in the same directory. Then:

   - **Both present and parse cleanly** → read the schema (`mandatory_fields`, `conditional_fields`, `enums`) and proceed with skill operation.
   
   - **One or both missing** → recommend `/wiki-config` as the easiest path; it deploys both files and walks through setup in a minute. If wiki-config isn't available, or the user wants to press on without it, offer to deploy the missing file(s) from `references/wiki-config-template.md` and/or `references/wiki-schema.md`, then remind them to run `/wiki-config` later for fine-tuning (especially the `blacklist` placeholders in `wiki-config.md`).
   
   - **One or both malformed** → recommend `/wiki-config` - it has a guided repair flow and is safer here than a blind overwrite. If wiki-config isn't available, point the user at `references/setup-help.md` for manual repair guidance. If the user insists on a reset anyway, warn that resetting overwrites any customizations they've made, then deploy the default from references on their explicit OK.

5. **Config not found at all**: Ask the user for their wiki root path, search there (bounded, max 5 levels). If still nothing, they don't have a wiki yet - same cascade as "missing" above.


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


