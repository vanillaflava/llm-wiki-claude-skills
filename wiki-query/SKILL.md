---
name: wiki-query
description: Answer a question using the compiled wiki knowledge base. Reads index.md to identify relevant pages, synthesises an answer with [[wikilink]] citations, and optionally files valuable answers as new wiki pages. Use when you say /wiki-query or when the user wants to draw on accumulated wiki knowledge rather than general model knowledge. Requires filesystem read access.
compatibility: Works with any markdown knowledge base supporting [[wikilinks]] — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "2.1"
---

# Wiki Query

Answers questions by reading and synthesising knowledge from the compiled wiki.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `vault_root` — absolute path to the wiki root
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

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
