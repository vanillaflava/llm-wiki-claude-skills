---
name: wiki-ingest
description: Process new source files from the raw/ folder into synthesised wiki pages. Run after dropping articles, PDFs, notes, or data files into raw/ — or when you say /wiki-ingest. Scans for unprocessed files, extracts knowledge, creates or updates wiki pages with wikilink backlinks, updates index.md, and logs the operation. Never writes to blacklisted paths. Requires filesystem read/write access.
compatibility: Works with any markdown knowledge base supporting [[wikilinks]] — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "2.1"
---

# Wiki Ingest

Processes unprocessed files from `raw/` into synthesised, interlinked wiki pages.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `vault_root` — absolute path to the wiki root
- `blacklist` — paths never to write to (relative to vault_root)
- `index_excludes` — paths excluded from index.md
- `raw_subdirs` — subdirectories inside raw/
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
5. Create any `raw/` subdirectories listed in `raw_subdirs` that do not already exist
6. Confirm what was created and proceed with the ingest

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

This skill requires **filesystem read, write, and search access**. If you are running on a surface without filesystem tools (web, mobile), you cannot proceed — inform the user and stop.

- **Filesystem search:** Required to locate `wiki-config.md`
- **Filesystem read/write:** Required for reading raw/ files and writing wiki pages, index.md, and log.md
- **PDF extraction:** Optional. Used for .pdf files in raw/. If unavailable, PDF files will be flagged and skipped
- **Vision (image reading):** Optional. Used for image files. If unavailable, image files will be flagged and skipped

---

## Workflow

### Step 1 — Read the log to identify already-processed files

Read `log.md` from the vault root. Extract the list of files already ingested (look for `ingest |` entries and filenames mentioned in them). Build a set of already-processed file paths. If the log is empty or new, treat the processed set as empty.

### Step 2 — Scan raw/ for unprocessed files

List all files under `<vault_root>/raw/` recursively. Check each against the already-processed set from Step 1. Collect the list of unprocessed files. If none are found, report "No new files in raw/ to process." and stop — do not append to log.md for a no-op run.

### Step 3 — Process each unprocessed file

For each unprocessed file, in order:

#### 3a. Read/extract content

**Always attempt to read the file first** — do not pre-filter by extension.

**Natively readable without tools** (read directly):
- Plain text: `.md`, `.txt`, `.csv`, `.tsv`, `.json`, `.yaml`, `.yml`, `.html`, `.xml`
- Code files: `.py`, `.js`, `.ts`, `.r`, and most other text-based source files

**Requires tools** (attempt with available tool; skip and flag if unavailable):
- `.pdf` — use PDF extraction tool if available
- Images (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.heic`) — use vision if available
- `.docx`, `.xlsx`, `.pptx`, `.epub` — use document extraction tool if available

If reading fails or returns unreadable content: tell the user which file could not be read and why, note what was attempted, suggest what tool might enable ingestion in a future run. Then continue with remaining files.

#### 3b. Understand the content

Read the extracted content carefully. Identify the main topic and domain, key concepts and findings, and how this connects to what is already in the wiki.

**Important:** Distinguish the *subject matter* of the source from its *document type*. An article titled "Building a Wiki System" is about AI tools and workflows if that is what a reader would learn from it — it belongs in AI Learning, not in wiki infrastructure, even if the word "wiki" features prominently in the title. Always ask: *what does a reader gain from this?* That drives placement.

#### 3c. Consult index.md

Read `index.md`. Do two things:
1. **Find integration points:** identify existing wiki pages related to the source material — candidates for receiving backlinks
2. **Map the subfolder structure:** note existing subfolders within each section — you will need this in Step 3d to place new pages precisely

#### 3d. Determine output: create or update wiki pages

Decide whether to create new wiki page(s), update an existing page, or both.

When creating a new page:
- Choose the most specific placement using the subfolder map from Step 3c — do not default to the section root
- Name clearly: `Topic — Aspect.md` or `Domain — Subtopic.md`
- Include YAML frontmatter:
  ```yaml
  ---
  title: Note Title
  version: 1.0
  date: YYYY-MM-DD
  changes: Created by wiki-ingest from raw/<source-filename>
  ---
  ```
- Write synthesised markdown — not a raw copy of the source
- Add `[[wikilinks]]` to related pages identified in Step 3c

When updating an existing page:
- Read the full current page first
- Integrate new knowledge naturally or as a dated new section
- Update the frontmatter version and changes fields

#### 3e. Add reciprocal backlinks

For each existing wiki page that should reference the new content, add a `[[New Page Title]]` link where the connection is genuinely useful. Do not add backlinks mechanically.

#### 3f. Update index.md

For each new wiki page created, add an entry in the correct section:
`- [[Note Title]] — One sentence description. \`relative/path/to/page.md\``

Place under the correct subfolder heading, consistent with Step 3d placement. Do not add entries for blacklisted paths, raw/ sources, or binary files.

#### 3g. Append to log.md

Add a new entry at the top (below the header, above existing entries):
```
## [YYYY-MM-DD] ingest | <source filename>
Brief summary: pages created or updated, any files flagged.
```

Never edit existing log entries.

### Step 4 — Summarise

Report: files processed with outcomes, files skipped with reasons, new pages created, existing pages updated, suggestions for follow-up (e.g. run wiki-integrate on new pages, run wiki-lint).

---

## Key Rules

1. **Never write to blacklisted paths** — check the blacklist before every write
2. **raw/ is read-only** — never create, modify, or delete anything in raw/
3. **log.md is append-only** — never edit existing entries; add new entries at the top
4. **Synthesise, do not copy** — wiki pages contain synthesised knowledge, not verbatim transcripts
5. **Frontmatter on every new page** — always include title, version, date, changes
6. **One source can produce multiple pages** — a rich PDF may warrant several wiki pages
7. **If unsure whether a path is blacklisted, stop and ask** — never guess

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
