---
name: wiki-query
description: Answer a question using the compiled wiki knowledge base. Reads index.md to identify relevant pages, synthesises an answer with [[wikilink]] citations, and optionally files valuable answers as new wiki pages. Use when you say /wiki-query or when the user wants to draw on accumulated wiki knowledge rather than general model knowledge. Requires filesystem read access.
---

<!-- version: 3.0 -->

# Wiki Query

Answers questions by reading and synthesising knowledge from the compiled wiki.

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


