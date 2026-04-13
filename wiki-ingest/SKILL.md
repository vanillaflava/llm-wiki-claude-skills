---
name: wiki-ingest
description: Process source files from the raw/ folder into synthesised wiki pages. Run after dropping articles, PDFs, notes, or data files into raw/, or when you say /wiki-ingest. Treats raw/ as a flat queue; scans all files, extracts and synthesises knowledge, creates or updates wiki pages with wikilink backlinks, then moves each source file to the appropriate ingested/ subfolder as an atomic commit — the move is the record of completion. Unreadable files move to ingested/assets/. Updates index.md and logs the operation. Blacklist applies to wiki page creation only, not file moves. Requires filesystem read/write/move access.
compatibility: Works with any markdown knowledge base using the wikilink convention — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "3.4"
---

# Wiki Ingest

Processes source files from the `raw/` queue into synthesised, interlinked wiki pages. The move of each file into `ingested/` is the atomic commit — the filesystem is the truth, not the log.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `wiki_root` — absolute path to the wiki root (may be a subfolder of a larger system)
- `blacklist` — paths where wiki page creation is forbidden (relative to vault_root)
- `index_excludes` — paths excluded from index.md
- `ingested_folder` — path to the archival folder (relative to vault_root)
- `ingested_subdirs` — archival taxonomy within ingested_folder
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
5. Create `ingested/` and each subdir listed in `ingested_subdirs`, plus `ingested/assets/` (always)
6. Confirm what was created and proceed with the ingest

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
This applies to wiki page creation only — not to file moves between raw/ and ingested/.
Add Git repos, source code folders, or any area that should never receive wiki writes.

**index_excludes** — Paths excluded from index.md tracking.
`raw\` is always excluded — source material is indexed only after ingestion.
`ingested\` must also be excluded — source files are not wiki pages.
`archive\` keeps deprecated pages out of the live catalogue.

**ingested_folder** — Path to the archival folder (relative to vault_root). Wiki-ingest moves
every processed file here as an atomic commit. Must be in index_excludes; must NOT be in
blacklist (ingest needs write access to move files here).

**ingested_subdirs** — Archival taxonomy within ingested_folder. Wiki-ingest classifies each
source and routes it to the appropriate subdir. Adapt freely — these are suggestions:
- `clippings` — web saves, browser clips, web clipper output
- `documentation` — product docs, API references, technical specifications
- `papers` — academic papers, PDFs, research material
- `articles` — blog posts, news, long-form reading
- `data` — CSV, JSON, structured datasets
- `notes` — freeform drafts, quick captures, anything else
`ingested/assets/` is always created — it holds files that could not be read or extracted.

**Property type conflict warning:** Some note-taking apps infer a type for each frontmatter
property (text, list, date, etc.). If any property name in this config conflicts with a property
of a different type elsewhere in your vault or used by a plugin, rename the field here and in
your live wiki-config.md. Inform your agent of any change so it reads the correct field names.

**log_format** — Do not change without updating all wiki skills.
```

---

## Capability Requirements

This skill requires **filesystem read, write, move, and search access**. If running on a surface without filesystem tools (web, mobile), inform the user and stop.

- **Filesystem search:** Required to locate `wiki-config.md`
- **Filesystem read/write/move:** Required for reading raw/ files, writing wiki pages, moving files to ingested/, and updating index.md and log.md
- **PDF extraction:** Optional. Used for .pdf files. If unavailable, PDFs move to ingested/assets/ with reason logged
- **Vision (image reading):** Optional. Used for image files. If unavailable, images move to ingested/assets/ with reason logged

---

## Workflow

### Step 1 — Scan raw/ for all files

List all files directly in `<wiki_root>/raw/` (flat — no subdirectory traversal needed; raw/ has no subdirs in this architecture). If raw/ is empty, report "No files in raw/ to process." and stop — do not append to log.md for a no-op run.

Every file found will be processed. There is no pre-filter based on log history — the filesystem is the truth. Files already ingested in a previous run will simply be re-ingested; the synthesis step will surface whether the content is unchanged, updated, or contradictory.

### Step 2 — Process each file

For each file in raw/, in order:

#### 2a. Read/extract content

**Always attempt to read the file first** — do not pre-filter by extension.

**Natively readable without tools** (read directly):
- Plain text: `.md`, `.txt`, `.csv`, `.tsv`, `.json`, `.yaml`, `.yml`, `.html`, `.xml`
- Code files: `.py`, `.js`, `.ts`, `.r`, and most other text-based source files

**Requires tools** (attempt with available tool):
- `.pdf` — use PDF extraction tool if available
- Images (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.heic`) — use vision if available
- `.docx`, `.xlsx`, `.pptx`, `.epub` — use document extraction tool if available

If reading fails or the file type is unsupported: note the reason, then proceed to step 2f — move the file to `ingested/assets/`. Do not skip it; unreadable files are acknowledged, not ignored.

#### 2b. Understand the content and classify for archival

Read the extracted content carefully. Identify the main topic and domain, key concepts and findings, and how this connects to what is already in the wiki.

**Subject over document type:** An article titled "Building a Wiki System" belongs in AI Learning if that is what a reader gains from it — not in wiki infrastructure merely because the title says "wiki." Always ask: *what does a reader learn from this?* That drives placement.

**Classify the source** — determine which `ingested_subdir` this file belongs in. Use the content and any available metadata (frontmatter `source:`, `tags:`, file extension, structure) to decide:
- `clippings` — has a source URL, web clipper metadata, or is clearly a saved web page
- `papers` — academic structure (abstract, methodology, references), DOI/arxiv, PDF from a journal
- `documentation` — product docs, API reference, technical spec from an official source
- `articles` — blog post, news article, opinion piece, long-form essay
- `data` — content is primarily structured data (CSV, JSON) not prose
- `notes` — freeform, quick capture, or nothing else fits

Record the chosen subdir — it is used in steps 2f and 2d.

#### 2c. Consult index.md and filesystem

Read `index.md`. Do two things:
1. **Find integration points:** identify existing wiki pages related to the source — candidates for backlinks or updates. If this source appears to have been ingested before, note the existing page — you may be updating it rather than creating a new one.
2. **Map subfolder structure:** scan the actual filesystem (via list_directory) for existing subfolders within the relevant wiki sections. Use this as the primary placement guide. Use index.md headings only to determine where to file the index entry, not to infer what subfolders exist on disk.

**Duplicate page check:** Before creating a new page in step 2d, scan index.md descriptions for significant topic overlap with the incoming source. If a strong match exists — another page that already covers this subject — prefer updating that page rather than creating a new one. Flag the decision in the session summary: "updated [[Existing Page]] rather than creating a new page."

#### 2d. Determine output: create or update wiki pages

Decide whether to create new wiki page(s), update an existing page, or both.

When creating a new page:
- Choose the most specific placement using the filesystem subfolder map from step 2c
- Name clearly: `Topic — Aspect.md` or `Domain — Subtopic.md`
- Include YAML frontmatter:
  ```yaml
  ---
  title: Note Title
  version: 1.0
  date: YYYY-MM-DD
  changes: Created by wiki-ingest from ingested/[subdir]/[source-filename]
  ---
  ```
- Write synthesised markdown — not a raw copy of the source
- Add `[[wikilinks]]` to related pages identified in step 2c

When updating an existing page (including re-ingestions):
- Read the full current page first
- Integrate new knowledge naturally; flag contradictions or significant updates in the session summary
- Update the frontmatter version and changes fields

#### 2e. Add reciprocal backlinks

For each existing wiki page that should reference the new content, add a `[[New Page Title]]` link using this test: would a reader of that page benefit from knowing about this content in a normal reading context — not just because they share a keyword, but because one genuinely informs the other? If yes, add the link. If the connection only exists at the surface level, omit it. Graph density is not the goal.

#### 2f. Move file to ingested/ — the atomic commit

This step is the record of completion. Execute it after wiki page creation/update is done.

**For processable files:**
Move `raw/[filename]` to `ingested/[classified-subdir]/[filename]`.
- If the destination subdir does not exist, create it first
- If a file with the same name already exists in the destination, overwrite it (re-ingestion)
- Move success = ingestion complete for this file

**For unprocessable files** (reading failed, unsupported format, tool unavailable):
Move `raw/[filename]` to `ingested/assets/[filename]` instead.
- Log the reason the file could not be processed
- The file is acknowledged and out of the queue; a future capability improvement may enable ingestion

**If the move fails for any reason:** do not delete the file from raw/. Leave it in place, report the failure, and continue with the next file. A file that stays in raw/ will be retried on the next run.

#### 2g. Update index.md

For each new wiki page created, add an entry in the correct section:
`- [[Note Title]] — One sentence description. \`relative/path/to/page.md\``

Place under the correct subfolder heading, consistent with step 2d placement. Do not add entries for blacklisted paths, raw/ sources, ingested/ source files, or binary files.

If a page was updated rather than created, no index entry change is needed.

#### 2h. Append to log.md — audit trail

Add a new entry at the top (below the header, above existing entries):
```
## [YYYY-MM-DD] ingest | <source filename>
Brief summary: pages created or updated, destination in ingested/, any files flagged.
```

Never edit existing log entries. The log is an audit trail only — it is not used to determine what has been processed (the filesystem handles that).

### Step 3 — Summarise

Report: files processed with outcomes and destinations, files moved to assets/ with reasons, new wiki pages created, existing pages updated, any contradictions or significant re-ingestion findings, suggestions for follow-up (e.g. run wiki-lint to validate links to ingested/).

---

## Key Rules

1. **Blacklist governs wiki page creation only** — never create a wiki page in a blacklisted path; file moves to ingested/ are permitted regardless of blacklist
2. **raw/ is a queue** — wiki-ingest is the only agent that moves files out of raw/, and only ever to ingested/; never delete files from raw/
3. **Every file exits the queue** — processable files → ingested/[subdir]/; unprocessable files → ingested/assets/; no file is left behind
4. **Source links point to ingested/, never to raw/** — wiki page frontmatter and body links reference the post-move destination
5. **The `changes:` frontmatter field is the trace** — every wiki page created from a source must include `changes: Created by wiki-ingest from ingested/[subdir]/[filename]`; this is how wiki-lint confirms a source is not orphaned
6. **log.md is append-only and an audit trail only** — never use it to determine processing state; never edit existing entries
7. **Synthesise, do not copy** — wiki pages contain synthesised knowledge, not verbatim transcripts
8. **Frontmatter on every new page** — always include title, version, date, changes
9. **One source can produce multiple pages** — a rich PDF may warrant several wiki pages; each page's `changes:` field should reference the source
10. **If unsure whether a path is blacklisted, stop and ask** — never guess

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
