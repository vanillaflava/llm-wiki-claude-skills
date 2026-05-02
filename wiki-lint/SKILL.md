---
name: wiki-lint
description: Health-check the wiki. Scans all pages for broken wikilinks, orphaned pages, stale index entries, missing connections between related pages, em-dash violations in page titles and filenames, orphaned binary assets, and orphaned sources in ingested/. Produces a dated lint report in archive/. Use when you say /wiki-lint, the user mentions broken links, dead wikilinks, missing pages, orphaned pages, outdated or stale pages, "what's broken in my wiki", "are my links working", "which pages have no connections", or asks for a wiki health check. Never auto-fixes anything; report only. Requires filesystem read access and write access to archive/.
metadata:
  version: "3.12"
---

# Wiki Lint

Health-checks the wiki and produces a report. Never modifies wiki content.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime. Pages this skill lints are checked against the structure in `wiki-schema.md` - both files need to be present.

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

This skill requires **filesystem read access** (to scan all pages) and **write access to `archive/`** (to file the lint report). If read access is unavailable, this skill cannot proceed.---

## Workflow

Config Discovery has already loaded `wiki-config.md` and `wiki-schema.md` into context. Do not re-read them; proceed from here assuming both are available.

### Step 1 - Build the page inventory

List all `.md` files in the vault recursively. Exclude paths in `blacklist` entirely; do not scan content inside them. Paths in `index_excludes` (`raw\`, `archive\`, `ingested\`) are excluded from being treated as wiki pages, but remain readable for link target validation: if a live wiki page links to a file in `archive/` or `ingested/`, that link must resolve to a real file. Build the **in-scope page set** from files outside both blacklist and index_excludes. Also list all non-markdown files outside blacklisted paths, raw\, archive\, and ingested\ as potential orphaned binary assets.

### Step 2 - Read index.md

Read `index.md`. Parse all wikilink references and file paths. Build a set of pages listed in index.md and their referenced paths.

### Step 3 - Check for broken wikilinks

For each in-scope wiki page, extract all `[[wikilinks]]`. Attempt to resolve each to an actual file. If no matching file is found: flag as a **broken wikilink** with the page where found, the broken link text, and a suggested correction if obvious.

Also flag: any wiki page containing a link that points into `raw/`. Once wiki-ingest moves files to `ingested/`, raw/ source links will break. Flag these as **future breakage warnings**: "[[Page]] links to raw/<filename>: will break after ingestion. Update to ingested/<subdir>/<filename> once ingest is complete." These are not yet broken but are guaranteed to become so.

### Step 4 - Check for orphan pages

For each page in the in-scope set, check:
- Is it listed in `index.md`?
- Is it referenced by any wikilink in any other in-scope page?

If neither: flag as an **orphan page**. New pages may be legitimate orphans; flag anyway, the human decides.

### Step 5 - Check for stale index entries

For each page listed in `index.md`, check whether the referenced file exists. If not: flag as a **stale index entry**.

### Step 6 - Qualitative and structural passes

**Conceptual issues:** Read `Overview.md` and scan for obvious contradictions or staleness; claims that appear to contradict specific wiki pages, or pages that seem significantly out of date. Qualitative pass only, not a full content review.

**Missing connections:** Scan `index.md` descriptions for significant term overlap between page pairs that do not link to each other. Pages that share multiple key concepts in their index descriptions but have no mutual wikilinks are candidates for missing connections. Flag pairs with meaningful overlap; do not flag tenuous keyword coincidences. This is a lightweight heuristic pass; wiki-integrate handles the actual linking if the user agrees a connection is genuine.

### Step 6b - Check for em-dash in page titles and filenames

Em-dashes (`—`) in page filenames and `title:` frontmatter fields are a persistent LLM output artifact. They break wikilinks (a link to `[[Topic - Subtopic]]` will not resolve a file named `Topic — Subtopic.md`), make pages unsearchable by their intended name, and violate the vault's em-dash convention.

For each in-scope page:
1. Check the filename for any `—` character
2. Read the `title:` frontmatter field and check for any `—` character

Flag each occurrence as an **em-dash violation** with the file path, where it was found (filename / title field), the offending string, and the suggested fix: replace `—` with ` - ` (space-hyphen-space).

### Step 6c - Check for stale pages

For each in-scope page, check for the following conditions:

**Missing date fields (error):** If a page has neither `date:` nor `updated:` in its frontmatter, flag as a **missing date fields error**. Both fields absent suggests something went wrong at creation - the page was not written by a skill or the frontmatter is malformed.

**Stale (soft warning):** If a page has an `updated:` field and it is more than 90 days before today, flag as a **stale page**. Exempt from this check:
- `status: artefact`, `status: snapshot`, `status: archived` - frozen by definition
- `page_type: reference` - reference pages have mostly static content and infrequent updates are expected

Pages without an `updated:` field (but with `date:`) are silently skipped - absence of `updated:` is not an error.

Flag each stale page with: path, `updated:` date, days since last touch.
Flag each missing-date-fields error with: path, which fields are absent.

### Step 6d - Schema compliance check (provenance fields)

For each in-scope page, validate the four provenance fields (`status:`, `description:`, `source:`, `reliability:`). Two classes of finding:

**Errors (hard flag):**

- **`source:` without `reliability:`** - `source:` is present but `reliability:` is absent. These fields are coupled; one without the other is an internal inconsistency. Flag as a **schema error** with: path, field pair affected.
- **Invalid `status:` value** - `status:` is present but its value is not one of the valid enum values (`active`, `stub`, `artefact`, `archived`, `snapshot`). Flag as a **schema error** with: path, field, invalid value found, valid values.
- **Invalid `reliability:` value** - `reliability:` is present but its value is not one of (`high`, `medium`, `low`). Flag as a **schema error** with: path, field, invalid value found, valid values.
- **Invalid `page_type:` value** - `page_type:` is present but its value is not in the valid enum list from wiki-schema.md. Flag as a **schema error** with: path, field, invalid value found, and the valid values from the schema.

**Soft warnings (informational):**

- **Skill-touched page missing `status:` or `description:`** - page has `updated:` (meaning a skill has written to it post-3b) and `page_type:` is `knowledge`, `research-note`, or `survey`, but `status:` or `description:` is absent. Old pages without `updated:` are silently skipped - they predate 3b and will pick up fields on next touch. Flag as a **missing provenance field** with: path, which field(s) absent.
- **`source:` present but no `## Sources` section** - `source:` in frontmatter implies an ingested origin; the body should have a `## Sources` section for enrichment tracking. Absence is not a hard error but is worth flagging. Flag as a **missing Sources section** with: path.
- **Skill-touched page missing `page_type:`** - page has `updated:` (meaning a skill has written to it) but `page_type:` is absent. Pages without `updated:` are silently skipped - they predate the feature and will pick up the field on next touch. Flag as a **missing page_type** with: path.

### Step 7 - Check for orphaned binary assets

For each non-markdown file outside blacklisted paths, raw\, archive\, and ingested\: search all in-scope pages for any reference to this filename. If none found: flag as an **orphaned binary asset**. A file contextually placed alongside related content (e.g. a PDF in a domain subfolder referenced by domain pages) is not an orphan; only flag files with no wiki references anywhere.

### Step 7a - Check for orphaned sources in ingested/

Every file in `ingested/` should have at least one wiki page that references it; via a Sources section in the page body containing the `ingested/` path. A source with no wiki reference has been processed but left no trace in the knowledge graph.

For each file in `ingested/` (all subdirs, including assets/):
1. Search all in-scope wiki pages for the file's relative path (e.g. `ingested/documentation/foo.md`). Check the full page content: `## Sources` section body, `changes:` frontmatter field, or anywhere else a path reference might appear. Treat any match as a valid reference regardless of where it appears.
2. If no match found: flag as an **orphaned source**: "ingested/[subdir]/filename has no wiki page referencing it"

A source in `ingested/assets/` with no reference is expected (it was unreadable at ingest time); flag it at lower severity as a **note** rather than a warning, so the user knows it exists and can re-attempt ingestion if capabilities have improved.

### Step 8 - Write the lint report

Create `[wiki-root]/archive/lint-YYYY-MM-DD.md`:

```markdown
---
title: Lint Report YYYY-MM-DD
date: YYYY-MM-DD
---

# Lint Report - YYYY-MM-DD

## Summary
- Broken wikilinks: N
- Future breakage warnings (raw/ links): N
- Orphan pages: N
- Stale index entries: N
- Missing connections (candidates): N
- Em-dash violations (titles/filenames): N
- Missing date fields (errors): N
- Stale pages (updated: > 90 days): N
- Schema errors (invalid enum values, missing field pairs): N
- Missing provenance fields (skill-touched pages): N
- Missing page_type: (skill-touched pages): N
- Missing Sources sections: N
- Orphaned binary assets: N
- Orphaned sources in ingested/: N (+ N notes in assets/)
- Conceptual flags: N

## Broken Wikilinks
[page where found, broken link, suggested fix]

## Future Breakage Warnings
[page where found, raw/ link, suggested post-ingest destination]

## Orphan Pages
[page path, reason it may be orphaned]

## Stale Index Entries
[index entry, the path that no longer resolves]

## Missing Connections
[page pair, overlapping terms, suggested action: run wiki-integrate]

## Em-dash Violations
[file path, where found (filename / title field), suggested fix: replace — with ' - ']

## Missing Date Fields
[file path, which of date: / updated: are absent; likely indicates malformed frontmatter or non-skill creation]

## Stale Pages
[file path, updated: date, days since last touch; not flagged if status: artefact/snapshot/archived or page_type: reference]

## Schema Errors
[file path, field(s) affected, nature of error (invalid enum value / source: without reliability: / invalid page_type: value)]

## Missing Provenance Fields
[file path, which of status: / description: are absent; page has updated: and is a fact-establishing page_type]

## Missing page_type:
[file path, page has updated: but page_type: is absent; add page_type: using values from wiki-schema.md, or run wiki-integrate to infer and confirm]

## Missing Sources Sections
[file path, has source: in frontmatter but no ## Sources section in body]

## Orphaned Binary Assets
[file path, reason it has no wiki references]

## Orphaned Sources in ingested/
[file path, no wiki page references this source; consider re-ingesting or archiving]

## Notes: ingested/assets/
[file path, was unreadable at ingest time; re-attempt if capabilities have improved]

## Conceptual Flags
[page, nature of potential issue]
```

### Step 9 - Prepend to log.md

Add the new entry at the top of log.md, below the header line, above all existing entries.

```
## [YYYY-MM-DD] lint | Full wiki pass
Summary: N broken links, N orphans, N stale entries, N missing connections, N date-field errors, N stale pages, N schema errors, N missing provenance/page_type fields, N orphaned assets.
Report: archive/lint-YYYY-MM-DD.md
```

### Step 10 - Present findings

Report the summary. Do not offer to auto-fix anything. Suggest follow-up:
- Broken wikilinks -> fix manually or run wiki-integrate on affected pages
- Future breakage warnings -> update raw/ links to ingested/ paths after the next ingest run
- Orphan pages -> run wiki-integrate, or move to archive/ if deprecated
- Stale index entries -> remove from index.md or update the path
- Missing connections -> run wiki-integrate on the flagged page pairs
- Em-dash violations -> rename the file (replace `—` with ` - `) and update its `title:` field; search for any wikilinks pointing to the old name and update them
- Missing date fields -> inspect the page; if skill-written, the frontmatter is malformed and should be repaired manually
- Stale pages -> review and update; or set `status: artefact`, `snapshot`, or `archived` if the page is intentionally frozen
- Schema errors -> repair frontmatter manually: add missing `reliability:` when `source:` is present, correct invalid enum values for `status:`, `reliability:`, or `page_type:`
- Missing provenance fields -> re-run wiki-ingest or wiki-crystallize on the page to pick up `status:` and `description:`; or add manually following the schema
- Missing page_type: -> add `page_type:` to the page frontmatter using values from the enum in wiki-schema.md; or run wiki-integrate which will infer and confirm the type
- Missing Sources sections -> add a `## Sources` section to the page body referencing the path in `source:`
- Orphaned binary assets -> move to an appropriate location or delete if unwanted
- Orphaned sources in ingested/ -> re-ingest the source file (drop back into raw/ and run wiki-ingest), or investigate why no wiki page was created
- Notes in ingested/assets/ -> retry ingestion if new tools or capabilities are available

---

## Key Rules

1. **Never modify wiki pages** - read-only except for writing the lint report to archive/
2. **Never delete files** - flag only; the human decides
3. **Do not flag contextually-placed files** - a file with wiki references in its domain is not an orphan
4. **Blacklisted paths are skipped entirely** - lint does not scan content inside blacklisted directories
5. **index_excludes are not wiki pages but are valid link targets** - archive/ and ingested/ are readable for link resolution even though they are excluded from the page inventory
6. **Report is filed in archive/, not the wiki root**

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
