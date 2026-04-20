---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight; does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
---

<!-- version: 3.0 -->

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Config

Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime.

**Step 1 - Identify filesystem scope.** Determine the root directory the filesystem tool has access to (for filesystem MCP: `allowedDirectories`; for Claude Code: session CWD; for other surfaces: the equivalent).

**Step 2 - Check scope sensibility.** If the scope root is a bare drive root (`C:\`, `D:\`, `/`), an OS root, or a user home (`C:\Users\X`, `/home/X`, `/Users/X`), do NOT search. Go to Step 4.

**Step 3 - Bounded search.** If scope is sensible, search recursively for `wiki-config.md`. First-match early termination, max 5 directory levels deep. If found: the directory containing it is the wiki root. Read `blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`. Proceed with the skill's operation.

**Step 4 - Ask once.** If not found, or scope was insensible, ask the user:

*"I couldn't find `wiki-config.md` in my accessible scope. Where is your wiki root - the folder containing your notes and `wiki-config.md`? It must be inside my filesystem scope."*

If the user provides a specific path: look for `wiki-config.md` at that path or within it (bounded search, max 5 levels deep). If found, use the containing directory as wiki root, read config, and proceed. If not found at the provided path, go to Step 5.

If the user is unclear, unsure, or declines to answer: go to Step 5.

**Step 5 - Unable to proceed.** The skill cannot meaningfully operate without a configured wiki. State the situation clearly, recommend the proper setup path, and point at the bundled references:

*"I can't operate without a configured wiki. The recommended setup path is the `/wiki-config` skill - install it (if not already available) and run it for interactive setup. If you'd rather set up manually, I have two bundled references in this skill: `references/setup-help.md` (manual setup walkthrough) and `references/wiki-config-template.md` (the default config content). I can read those and walk you through it if you want."*

After this recommendation, let the user drive. If they want to proceed with wiki-config, confirm and stop (they invoke it themselves). If they want manual setup, read the references and assist them - including writing files on their explicit direction. Do not automatically create the config or scaffolding without the user's clear instruction.


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


