---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight; does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
---

<!-- version: 3.2 -->

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime.

1. **Identify scope**: Determine filesystem scope root (allowedDirectories for MCP, CWD for Code, equivalent for other surfaces)

2. **Scope check**: If scope is bare drive root (`C:\`, `D:\`, `/`), OS root, or user home (`C:\Users\X`, `/home/X`, `/Users/X`) → skip to step 4 (do not search for privacy)

3. **Bounded search**: Search recursively for `wiki-config.md` (first-match, max 5 levels). If found → read config (`blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`), proceed with skill operation

4. **Config not found** (scope was insensible OR search returned empty):
   - **Get oriented first**: Check if wiki-config skill is available (scan `<available_skills>` section for `wiki-config` entry), then read `references/setup-help.md` to load setup context
   - **Then ask**: "I couldn't find `wiki-config.md`. Where is your wiki root - the folder containing your notes and `wiki-config.md`?"
   - If user provides path: search there (bounded, max 5 levels). If found → read config, proceed. If not found or user is unsure → offer setup paths:
     - If wiki-config available: "Run `/wiki-config` for guided setup"
     - If not: "Install wiki-config skill for guided setup, or I can help with manual setup using the references I've loaded"

5. **Manual setup** (if user chooses): Follow setup-help.md guidance, assist with file creation on explicit user direction only. Never create config or scaffolding without clear instruction.


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


