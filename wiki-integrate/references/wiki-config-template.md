---
blacklist:
  - Folder(s) where the wiki should not write
  - You can edit these

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

templates_folder: templates\

log_format: "## [YYYY-MM-DD] {type} | {subject}"
---

## Configuration Guide

**Path style - Windows vs Mac/Linux.** Examples below use Windows backslash (`\`) directory convention. On Mac or Linux, use forward slash (`/`) instead. YAML itself is agnostic; the convention just keeps paths consistent with your OS's native separator.

**Wiki root.** The directory containing `wiki-config.md` IS the wiki root. Skills derive it from this file's location; you do not write the path anywhere. If you move the wiki, move this file with it and nothing else changes. Your wiki root must be inside your filesystem tool's allowed scope or the skills cannot reach your notes.

Wiki root is NOT your machine root (`C:\`, `/`), NOT your user home (`C:\Users\you\`, `~/`), and not necessarily the top of your knowledge base. It is the specific folder where your notes and the wiki system files (`index.md`, `log.md`, `raw\`, `ingested\`, `templates\`) live. See the repo README for scope and privacy guidance.

**blacklist.** Paths where wiki page creation is forbidden, relative to wiki root. The placeholder values above MUST be replaced with real folder names before the wiki is useful. Applies to wiki page creation only; file moves to `ingested\` are always permitted. Typical entries: `Repositories\` (Git repos), private folders, source code trees, or any area that should never receive wiki writes.

**index_excludes.** Paths excluded from `index.md` tracking. `raw\` and `ingested\` should always stay excluded (source material is not wiki content). `archive\` keeps deprecated pages out of the live catalogue. Add other paths you want hidden from the index.

**ingested_folder.** The archival folder where `wiki-ingest` moves every processed source file. Must appear in `index_excludes`. Must NOT appear in `blacklist` (ingest needs write access to move files here).

**ingested_subdirs.** Archival taxonomy inside `ingested_folder`. wiki-ingest classifies each source and routes to the matching subdir. Adapt freely to your workflow:
- `clippings` - web saves, browser clips
- `documentation` - product docs, API references, technical specs
- `papers` - academic papers, PDFs, research material
- `articles` - blog posts, essays, long-form reading
- `data` - CSV, JSON, structured datasets
- `notes` - freeform drafts, quick captures, anything else

`ingested/assets/` is always created automatically; it holds files that could not be read or extracted at ingest time.

**templates_folder.** The folder holding page templates. Reserved for the page-templates system landing in a future release. The folder is created empty during setup. When the templates system ships, user-editable templates placed here will control page structure; no skill in the current release reads this field yet. Safe to leave as-is.

**log_format.** Format string for log entries. Do not change without updating all wiki skills.

**Property type conflict warning:** Some note-taking apps infer a type for each frontmatter property (text, list, date, etc.). If any property name here conflicts with a property of a different type elsewhere in your vault or used by a plugin, rename it here and inform your agent of the change.

**For interactive setup, validation, and reconfiguration:** install and run the `/wiki-config` skill. It walks through each field with explanations, validates the YAML, and handles schema migration across skill versions. Direct edits to this file are equally valid; the wiki-config skill is a convenience, not a requirement.
