---
name: wiki-query
description: "Answer a question using the compiled wiki knowledge base, synthesising a response with [[wikilink]] citations. Always use this skill when the user says /wiki-query, 'what does my wiki say about', 'what do I know about', 'check my notes on', 'search my wiki for', or 'what's the current state of [topic] in my notes'. Also use when the user asks a question and signals they want to draw on their own accumulated knowledge rather than general knowledge — phrases like 'what have we worked on regarding', 'based on my notes', 'what's my thinking on', or any question where the answer should come from personal knowledge rather than the model. Optionally files valuable answers as new wiki pages. When in doubt whether personal knowledge retrieval is being requested, use this skill. Requires filesystem read access."
metadata:
  version: "3.8"
---

# Wiki Query

Answers questions by reading and synthesising knowledge from the compiled wiki.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime. Pages this skill writes (if you choose to file an answer) follow the structure in `wiki-schema.md` - both files need to be present.

1. **Identify scope**: Determine your filesystem scope root - the top-level directory your filesystem tool can access.

2. **Scope check - MANDATORY STOP**: If scope is bare drive root (`C:\`, `D:\`, `/`), OS root, or user home (`C:\Users\X`, `/home/X`, `/Users/X`) → **stop immediately. Do not search. Do not attempt to locate wiki-config.md.** Go directly to step 6.

3. **Scan `<available_skills>` for `wiki-config`.** Note whether it's available - this shapes the recommendations below. The bundled `references/setup-help.md` is also available; read it if the user needs orientation or if you get stuck.

4. **Locate and read `wiki-config.md`**: Search recursively (first-match, max 5 levels). If found, read it (`blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`). If not found, skip to step 6.

5. **Locate and read `wiki-schema.md` - mandatory check**: In the same directory as `wiki-config.md`, verify `wiki-schema.md` exists and parses as YAML. **Do not proceed to the Workflow below until you have a definite verdict** (present / missing / malformed). Then:

   - **Present and parses cleanly** → read the schema (`mandatory_fields`, `conditional_fields`, `enums`) and proceed to the Workflow.
   
   - **Missing** → STOP. Do not proceed. Do not deploy. Response depends on whether `wiki-config` is in `<available_skills>` (from step 3):
     
     - **wiki-config available:** Output exactly this pattern - *"Your wiki is missing `wiki-schema.md`. Run `/wiki-config` to deploy it and complete setup. I'll wait for that to be done before proceeding."* End of turn. Do not offer bundled deployment; do not present alternatives. wiki-config's guided flow is the correct path when wiki-config is available.
     
     - **wiki-config not available:** Offer bundled fallback - *"Your wiki is missing `wiki-schema.md` and the wiki-config skill is not installed. I can deploy a default from my bundled reference, but I'd recommend installing wiki-config for the guided setup. Deploy bundled default?"* Wait for explicit confirmation. On OK, deploy from `references/wiki-schema.md`.
   
   - **Malformed** → STOP. Same structure:
     
     - **wiki-config available:** *"Your `wiki-schema.md` is malformed. Run `/wiki-config` - it has a guided repair flow that preserves any customizations you've made. I'll wait."* End of turn. Do not attempt repair or bundled overwrite.
     
     - **wiki-config not available:** Point to `references/setup-help.md` for manual repair guidance. If the user explicitly instructs a reset (not as an automatic fallback), warn that it overwrites any customizations, then deploy the default on explicit OK.

   The same two-branch structure applies if `wiki-config.md` itself was found malformed in step 4: wiki-config available → stop and recommend `/wiki-config`, end of turn; unavailable → guided manual repair via setup-help.md.

6. **Config not found at all**: Ask the user for their wiki root path, search there (bounded, max 5 levels). If still nothing, they don't have a wiki yet - follow the "Missing" branch above (recommend `/wiki-config`; offer bundled deployment if unavailable).

---

## Capability Requirements

This skill requires **filesystem read access**. Write access is only needed if the user chooses to file the answer as a new wiki page.

If running on a surface without filesystem tools (web, mobile), inform the user that this skill cannot access the local knowledge base. Offer to answer from general model knowledge instead, with the caveat that it will not draw on wiki-specific context.

---

## Workflow

Config Discovery has already loaded `wiki-config.md` and `wiki-schema.md` into context. Do not re-read them; proceed from here assuming both are available.

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


