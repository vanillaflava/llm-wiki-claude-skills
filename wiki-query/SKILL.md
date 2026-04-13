---
name: wiki-query
description: Answer a question using the compiled wiki knowledge base. Reads index.md to identify relevant pages, synthesises an answer with [[wikilink]] citations, and optionally files valuable answers as new wiki pages. Use when you say /wiki-query or when the user wants to draw on accumulated wiki knowledge rather than general model knowledge. Requires filesystem read access.
---

<!-- version: 2.6 -->

# Wiki Query

Answers questions by reading and synthesising knowledge from the compiled wiki.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `wiki_root` — absolute path to the wiki root (may be a subfolder of a larger system)
- `blacklist` — paths to exclude from search scope
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

This skill requires **filesystem read access**. Write access is only needed if the user chooses to file the answer as a new wiki page.

If running on a surface without filesystem tools (web, mobile), inform the user that this skill cannot access the local knowledge base. Offer to answer from general model knowledge instead, with the caveat that it will not draw on wiki-specific context.

---

## Workflow

### Step 1 — Read the index for orientation

Read `index.md` from the vault root. Scan the full catalogue to identify pages likely to be relevant to the query. Do not read every page — use the one-line descriptions to target your search.

Also read `Overview.md` if the query is broad or cross-domain — it provides a fast orientation to the current state of knowledge.

### Step 2 — Identify target pages

Select the 2–6 pages most likely to contain information relevant to the query. Consider:
- Direct topic match (page title or description mentions the subject)
- Domain relevance (if the query is about FASD, read FASD pages)
- Cross-domain connections (the query may span multiple sections)

If the query is highly specific, prefer targeted pages. If broad, start with Overview.md and follow backlinks.

### Step 3 — Read target pages

Read each identified page in full. Note: direct answers to the query, related context, wikilinks to other relevant pages, and gaps or open questions flagged in the notes.

### Step 4 — Follow backlinks if needed

If initial pages reference others that seem directly relevant, read those too. Limit follow-up to 2–3 additional pages. Stop when new pages are not adding information relevant to the query.

### Step 5 — Synthesise the answer

Write a clear answer to the query:
- Ground every claim in specific wiki pages using `[[wikilink]]` citations
- Distinguish between: well-evidenced claims, claims requiring nuance, open questions or gaps
- Use the language and framing of the wiki
- Be honest about what the wiki does and does not contain — do not fill gaps with general model knowledge without flagging it

Format: prose for narrative answers; tables or lists for comparative or structured information.

### Step 6 — Offer to file the answer

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

**In scope:** All wiki pages in the vault root except paths in the blacklist.
**Out of scope:** `raw/` source files — the wiki is the synthesised view, not the raw sources.

If a query requires information only available in raw source files, note this and suggest running wiki-ingest first if the source hasn't been processed.

---

## Key Rules

1. **Cite the wiki, not general knowledge** — every factual claim should trace to a specific wiki page
2. **Flag gaps honestly** — if the wiki doesn't contain the answer, say so clearly
3. **Do not modify files unless the user explicitly asks** — reading is always safe; writing requires consent
4. **Do not access blacklisted paths** — even for reading in query context


