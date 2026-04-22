---
name: wiki-query
description: Answer a question using the compiled wiki knowledge base. Reads index.md to identify relevant pages, synthesises an answer with [[wikilink]] citations, and optionally files valuable answers as new wiki pages. Use when you say /wiki-query or when the user wants to draw on accumulated wiki knowledge rather than general model knowledge. Requires filesystem read access.
---

<!-- version: 3.3 -->

# Wiki Query

Answers questions by reading and synthesising knowledge from the compiled wiki.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime. Pages this skill writes (if you choose to file an answer) follow the structure in `wiki-schema.md` - both files need to be present.

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

This skill requires **filesystem read access**. Write access is only needed if the user chooses to file the answer as a new wiki page.

If running on a surface without filesystem tools (web, mobile), inform the user that this skill cannot access the local knowledge base. Offer to answer from general model knowledge instead, with the caveat that it will not draw on wiki-specific context.

---

## Workflow

### Step 1 - Read the index for orientation

Read `index.md` from the wiki root. Scan the full catalogue to identify pages likely to be relevant to the query. Do not read every page; use the one-line descriptions to target your search.

Also read `Overview.md` if the query is broad or cross-domain; it provides a fast orientation to the current state of knowledge.

### Step 2 - Identify target pages

Select the 2-6 pages most likely to contain information relevant to the query. Consider:
- Direct topic match (page title or description mentions the subject)
- Domain relevance (if the query is about FASD, read FASD pages)
- Cross-domain connections (the query may span multiple sections)

If the query is highly specific, prefer targeted pages. If broad, start with Overview.md and follow backlinks.

### Step 3 - Read target pages

Read each identified page in full. Note: direct answers to the query, related context, wikilinks to other relevant pages, and gaps or open questions flagged in the notes.

### Step 4 - Follow backlinks if needed

If initial pages reference others that seem directly relevant, read those too. Limit follow-up to 2-3 additional pages. Stop when new pages are not adding information relevant to the query.

### Step 5 - Synthesise the answer

Write a clear answer to the query:
- Ground every claim in specific wiki pages using `[[wikilink]]` citations
- Distinguish between: well-evidenced claims, claims requiring nuance, open questions or gaps
- Use the language and framing of the wiki
- Be honest about what the wiki does and does not contain; do not fill gaps with general model knowledge without flagging it

Format: prose for narrative answers; tables or lists for comparative or structured information.

### Step 6 - Offer to file the answer

After delivering the answer, offer: *"This answer could be filed as a new wiki page for future reference. Would you like me to create a page for it?"*

If the user agrees:
- Create a new wiki page in the appropriate section
- Include frontmatter (title, version, date, changes: "Created by wiki-query")
- Add `[[backlinks]]` to the source pages cited
- Run wiki-integrate to weave it into the knowledge graph
- Append to log.md: `## [YYYY-MM-DD] query | <query summary>`

If the user declines, do not modify any files.

---

## Query Scope

**In scope:** All wiki pages in the wiki root except paths in the blacklist.
**Out of scope:** `raw/` source files; the wiki is the synthesised view, not the raw sources.

If a query requires information only available in raw source files, note this and suggest running wiki-ingest first if the source hasn't been processed.

---

## Key Rules

1. **Cite the wiki, not general knowledge:** every factual claim should trace to a specific wiki page
2. **Flag gaps honestly:** if the wiki doesn't contain the answer, say so clearly
3. **Do not modify files unless the user explicitly asks:** reading is always safe; writing requires consent
4. **Do not access blacklisted paths:** even for reading in query context

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.


