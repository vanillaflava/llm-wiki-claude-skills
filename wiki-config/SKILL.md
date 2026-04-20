---
name: wiki-config
description: Set up, validate, and reconfigure wiki-config.md for the llm-wiki skill suite. Use when the user says /wiki-config, asks to "set up my wiki", "configure wiki", "initialize wiki config", or when any wiki skill reports a missing or invalid config. Owns the interactive configuration flow for the wiki system.
---

<!-- version: 1.0 -->

# Wiki Config

Interactive setup, validation, and maintenance for `wiki-config.md`. Pairs with the five operational wiki skills (wiki-ingest, wiki-lint, wiki-integrate, wiki-crystallize, wiki-query) which all read the same config.

---

## Capability Requirements

Filesystem read, write, search, and directory creation. If running on a surface without filesystem tools, inform the user and stop.

---

## Workflow

### 1 - Discover

Identify the filesystem scope root (for filesystem MCP: `allowedDirectories`; for Claude Code: session CWD; for other surfaces: the equivalent).

Check scope sensibility. If insensible (bare drive root, OS root, or user home), skip search and go straight to step 2 (Init).

If sensible, search recursively for `wiki-config.md` with first-match early termination, max 5 directory levels deep.

If found: report the location. Offer to validate current values (step 3) or reconfigure specific fields (step 4).

If not found: proceed to step 2 (Init).

### 2 - Init

Ask the user for their wiki root:

*"Let's set up `wiki-config.md`. I need the absolute path to your wiki root - the folder where your notes live.*

*Wiki root is NOT your machine root (`C:\`, `/`) or your user home (`C:\Users\you\`, `~/`). It is the specific folder where your notes and wiki system files live. It must be inside your filesystem tool's scope.*

*Examples: `C:\Users\you\Notes\`, `/Users/you/Documents/Wiki/`, `/home/you/vault/`"*

Sanity-check the returned path:
- Reject bare drive roots (`C:\`, `D:\`, `/`)
- Reject OS roots and user homes (`/home/`, `/Users/`, `C:\Users\`)
- Confirm the path exists and is writable

If the path fails a check, explain which and ask again.

Deploy the scaffolding:
1. Copy `assets/wiki-config-template.md` to `<wiki_root>/wiki-config.md`
2. If `<wiki_root>/index.md` missing, create it with header:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Maintained by wiki-ingest and wiki-integrate.*
   ```
3. If `<wiki_root>/log.md` missing, create it with header:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
4. Create `<wiki_root>/raw/` (the ingest queue)
5. Create `<wiki_root>/ingested/` and each subdir in `ingested_subdirs`, plus `<wiki_root>/ingested/assets/` (always)
6. Create `<wiki_root>/templates/` (empty; reserved for the page-templates system)
7. Append an init entry to log.md

Report to the user: list of what was created, suggest next steps (drop files in raw/, run /wiki-ingest).

### 3 - Validate

Read the config and check:
- YAML parses without error
- All required fields present: `blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`
- Paths in `blacklist` and `index_excludes` are relative strings (no absolute paths, no protocols)
- `ingested_folder` appears in `index_excludes` and NOT in `blacklist`
- Wiki root (directory containing config) passes the sanity check from step 2
- `blacklist` values are real folder names, not template placeholders (flag if placeholders from the bundled template still appear)

Report findings. For each issue, propose a fix and ask before applying.

### 4 - Reconfigure

If the user wants to change values:
- Show current value
- Prompt for new value with context on what the field does
- Confirm before writing
- Write back to `wiki-config.md` preserving comments and structure

Never silently overwrite. Reconfiguration is always an explicit, confirmed operation.

---

## Key Rules

1. **This skill owns the setup UX entirely.** The five operational skills all refuse on config-miss and point the user here (or at bundled `references/setup-help.md` for manual setup when this skill is not installed). Only this skill creates scaffolding, validates config, and handles reconfiguration. Only this skill advertises "set up wiki config" triggers.
2. **Never overwrite a live config without explicit confirmation.** Reconfiguration is a per-field dialogue.
3. **Wiki root is the directory containing wiki-config.md.** No wiki_root field in the config. Moving the wiki means moving this file.
4. **Never deploy to a bare drive root, OS root, or user home.** These are almost always user error.
5. **The bundled template is the single source of truth.** This skill bundles it in `assets/wiki-config-template.md` (deployed during init). The five operational skills bundle an identical read-only copy at `references/wiki-config-template.md` (pulled into context when helping a user with manual setup). When the template content changes here, all five operational copies must update in lockstep. `references/setup-help.md` should also be kept in sync across the operational skills.

---

## What this skill does not do

- Does not perform any wiki operation (ingest, lint, integrate, crystallize, query). Those are the five operational skills.
- Does not manage TaskNotes config, plugin settings, or any non-wiki configuration.
- Does not reach outside the wiki root or modify the filesystem tool's scope.
- Does not handle page templates, tag taxonomy, or conventions (templates are reserved for a future batch).
