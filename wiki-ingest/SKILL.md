---
name: wiki-ingest
description: Process source files from the raw/ folder into synthesised wiki pages. Run after dropping articles, PDFs, notes, or data files into raw/, or when you say /wiki-ingest. Treats raw/ as a flat queue; scans all files, extracts and synthesises knowledge, creates or updates wiki pages with wikilink backlinks, then moves each source file to the appropriate ingested/ subfolder as an atomic commit; the move is the record of completion. Unreadable files move to ingested/assets/. Updates index.md and logs the operation. Blacklist applies to wiki page creation only, not file moves. Requires filesystem read/write/move access.
---

<!-- version: 4.2 -->

# Wiki Ingest

Processes source files from the `raw/` queue into synthesised, interlinked wiki pages. The move of each file into `ingested/` is the atomic commit; the filesystem is the truth, not the log.

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

This skill requires **filesystem read, write, move, and search access**. If running on a surface without filesystem tools (web, mobile), inform the user and stop.

- **Filesystem search:** Required to locate `wiki-config.md`
- **Filesystem read/write/move:** Required for reading raw/ files, writing wiki pages, moving files to ingested/, and updating index.md and log.md
- **PDF extraction:** Optional. Used for .pdf files. If unavailable, PDFs move to ingested/assets/ with reason logged
- **Vision (image reading):** Optional. Used for image files. If unavailable, images move to ingested/assets/ with reason logged

---

## Workflow

### Step 1 - Scan raw/ for all files

List all files directly in `<wiki_root>/raw/` (flat; raw/ has no subdirs in this architecture). If raw/ is empty, report "No files in raw/ to process." and stop; do not append to log.md for a no-op run.

Every file found will be processed. There is no pre-filter based on log history; the filesystem is the truth. Files already ingested in a previous run will simply be re-ingested; the synthesis step will surface whether the content is unchanged, updated, or contradictory.

### Step 2 - Process each file

For each file in raw/, in order:

#### 2a. Read/extract content

**Always attempt to read the file first;** do not pre-filter by extension.

**Natively readable without tools** (read directly):
- Plain text: `.md`, `.txt`, `.csv`, `.tsv`, `.json`, `.yaml`, `.yml`, `.html`, `.xml`
- Code files: `.py`, `.js`, `.ts`, `.r`, and most other text-based source files

**Requires tools** (attempt with available tool):
- `.pdf`: use PDF extraction tool if available
- Images (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.heic`): use vision if available
- `.docx`, `.xlsx`, `.pptx`, `.epub`: use document extraction tool if available

If reading fails or the file type is unsupported: note the reason, then proceed to step 2f and move the file to `ingested/assets/`. Do not skip it; unreadable files are acknowledged, not ignored.

#### 2b. Understand the content and classify for archival

Read the extracted content carefully. Identify the main topic and domain, key concepts and findings, and how this connects to what is already in the wiki.

**Subject over document type:** An article titled "Building a Wiki System" belongs in AI Learning if that is what a reader gains from it, not in wiki infrastructure merely because the title says "wiki." Always ask: *what does a reader learn from this?* That drives placement.

**Classify the source:** determine which `ingested_subdir` this file belongs in. Use the content and any available metadata (frontmatter `source:`, `tags:`, file extension, structure) to decide:
- `clippings`: has a source URL, web clipper metadata, or is clearly a saved web page
- `papers`: academic structure (abstract, methodology, references), DOI/arxiv, PDF from a journal
- `documentation`: product docs, API reference, technical spec from an official source
- `articles`: blog post, news article, opinion piece, long-form essay
- `data`: content is primarily structured data (CSV, JSON) not prose
- `notes`: freeform, quick capture, or nothing else fits

**Content type vs user-applied tags.** If a source has `tags: clippings` from a web clipper but the content appears to be something other than a generic web clip (for example, clearly official product documentation or an academic paper), do not silently override the tag. Instead, flag the apparent mismatch and ask the user: *"This file is tagged 'clippings' but reads like [documentation/a paper/etc.]. I'd suggest routing it to `documentation/`; does that match your intent, or should I use `clippings/`?"* The clipping tool may have been configured deliberately, or the user may have applied the tag for their own reasons. When in doubt, ask with a suggestion rather than deciding unilaterally.

Record the chosen subdir; it is used in steps 2f and 2d.

#### 2c. Consult index.md and filesystem

Read `index.md`. Do two things:
1. **Find integration points:** identify existing wiki pages related to the source; candidates for backlinks or updates. If this source appears to have been ingested before, note the existing page; you may be updating it rather than creating a new one.
2. **Map subfolder structure:** scan the actual filesystem (via list_directory) for existing subfolders within the relevant wiki sections. Use this as the primary placement guide. Use index.md headings only to determine where to file the index entry, not to infer what subfolders exist on disk.

**Duplicate page check:** Before creating a new page in step 2d, scan index.md descriptions for significant topic overlap with the incoming source. If a strong match exists (another page that already covers this subject), prefer updating that page rather than creating a new one. Flag the decision in the session summary: "updated [[Existing Page]] rather than creating a new page."

#### 2d. Determine output: create or update wiki pages

Decide whether to create new wiki page(s), update an existing page, or both.

When creating a new page:
- Choose the most specific placement using the filesystem subfolder map from step 2c
- Name clearly: `Topic - Aspect.md` or `Domain - Subtopic.md`
- Include YAML frontmatter:
  ```yaml
  ---
  title: Note Title
  version: 1.0
  date: YYYY-MM-DD
  changes: Created by wiki-ingest from [source-filename]
  ---
  ```
  `changes:` must be a brief description only; never a file path or URL. The ingested/ path lives in the body Sources section (see below).
- Add a Sources section at the end of the page body:
  ```markdown
  ## Sources
  [source-filename](https://the-source-url) · `ingested/[subdir]/source-filename.md`
  ```
  If the source has a `source:` field in its frontmatter (web-clipped or known URL), both the URL and the local path are required. If no URL is available, the local path alone is sufficient. This section is how wiki-lint confirms the source is not orphaned; do not omit it.
- Write synthesised markdown, not a raw copy of the source
- Add `[[wikilinks]]` to related pages identified in step 2c

When updating an existing page (including re-ingestions):
- Read the full current page first
- **Treat an update with the same synthesis ambition as a new page.** Do not anchor on what is already there. Ask: *what does a reader of this page not yet know that this source would teach them?* That gap is the synthesis target. If the source is large (>100 lines) and the page has a clear existing structure, check every major section of the source against the current page; missing coverage of a major section is a synthesis gap, not a design decision.
- Integrate new knowledge naturally; flag contradictions or significant updates in the session summary
- If the page does not yet have a Sources section, add one using the same format above. If a Sources section already exists, check whether it references a `raw/` path for this source; if so, update it to the correct `ingested/[subdir]/[filename]` path. The Sources section should always reflect the post-move destination, never the raw/ queue.
- If new content from this source would substantially change the scope or length of the page (rough signal: new content exceeds current content), consider whether the page should be split or a companion page created; offer this to the user as a suggestion, do not act without consent
- Update the frontmatter version and changes fields

**Large or dense reference documents - third path.** Some files are readable but are too large or specialised to synthesise meaningfully into wiki pages. A 400-page medical advisory, a full technical standard, or a comprehensive legal document may be worth keeping on hand for direct consultation without being ingested. When a source is readable but synthesis would produce noise rather than signal, surface the judgment to the user before proceeding:

*"[filename] is [N pages / very large]. I can: (a) create a wiki page synthesising the key findings, (b) enrich an existing related page with what's relevant, or (c) acknowledge it exists with a stub entry and a pointer - useful if you want it indexed but don't want to synthesise it now. Which would you prefer?"*

If the user chooses (c): create a minimal stub page with a `## Sources` section pointing to the file's location, and move the file to the relevant domain `Assets/` folder (not `ingested/` - this file was never fed to the ingest queue by design). Note that `Knowledge/[Domain]/Assets/` is distinct from `ingested/assets/`: the former holds files deliberately kept as reference material; the latter holds files that failed ingest and may be retried later.

#### 2e. Add reciprocal backlinks

For each existing wiki page that should reference the new content, add a `[[New Page Title]]` link using this test: would a reader of that page benefit from knowing about this content in a normal reading context, not just because they share a keyword, but because one genuinely informs the other? If yes, add the link. If the connection only exists at the surface level, omit it. Graph density is not the goal.

#### 2f. Move file to ingested/ - the atomic commit

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
`- [[Note Title]]: One sentence description. \`relative/path/to/page.md\``

Place under the correct subfolder heading, consistent with step 2d placement. Do not add entries for blacklisted paths, raw/ sources, ingested/ source files, or binary files.

If a page was updated rather than created, assess whether its current index.md description still accurately reflects the page's scope. If the update added a major new section or substantially changed coverage, update the description. A stale index entry misleads future searches and agent orientation.

#### 2h. Append to log.md - audit trail

Add a new entry at the top (below the header, above existing entries):
```
## [YYYY-MM-DD] ingest | <source filename>
Brief summary: pages created or updated, destination in ingested/, any files flagged.
```

Never edit existing log entries. The log is an audit trail only; it is not used to determine what has been processed (the filesystem handles that).

### Step 3 - Summarise

Report: files processed with outcomes and destinations, files moved to assets/ with reasons, new wiki pages created, existing pages updated, any contradictions or significant re-ingestion findings, suggestions for follow-up (e.g. run wiki-lint to validate links to ingested/).

---

## Key Rules

1. **Blacklist governs wiki page creation only:** never create a wiki page in a blacklisted path; file moves to ingested/ are permitted regardless of blacklist
2. **raw/ is a queue:** wiki-ingest is the only agent that moves files out of raw/, and only ever to ingested/; never delete files from raw/
3. **Every file exits the queue:** processable files go to ingested/[subdir]/; unprocessable files go to ingested/assets/; no file is left behind
4. **Source links point to ingested/, never to raw/:** wiki page frontmatter and body links reference the post-move destination
5. **The body Sources section is the trace:** every wiki page created from a source must include a Sources section with the `ingested/[subdir]/[filename]` path; this is how wiki-lint confirms a source is not orphaned. The `changes:` frontmatter field contains only a brief human-readable description, never a file path or URL
6. **log.md is append-only and an audit trail only:** never use it to determine processing state; never edit existing entries
7. **Synthesise, do not copy:** wiki pages contain synthesised knowledge, not verbatim transcripts
8. **Frontmatter on every new page:** always include title, version, date, changes
9. **One source can produce multiple pages:** a rich PDF may warrant several wiki pages; each page's `changes:` field should reference the source
10. **If unsure whether a path is blacklisted, stop and ask:** never guess

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
