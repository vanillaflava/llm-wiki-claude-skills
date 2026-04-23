---
name: wiki-config
description: Set up, validate, and reconfigure wiki-config.md for the llm-wiki skill suite. Use when the user says /wiki-config, asks to "set up my wiki", "configure wiki", "initialize wiki config", mentions "page structure problems", "header errors", "schema issues", or when any wiki skill reports a missing or invalid config or schema file. Owns the interactive configuration flow and schema management for the wiki system.
---

<!-- version: 1.6 -->

# Wiki Config

Interactive setup, validation, and maintenance for `wiki-config.md`. Pairs with the five operational wiki skills (wiki-ingest, wiki-lint, wiki-integrate, wiki-crystallize, wiki-query) which all read the same config.

---

## Capability Requirements

Filesystem read, write, search, and directory creation. If running on a surface without filesystem tools, inform the user and stop.

---

## Workflow

### Step 0 - Welcome & Initialization

**Every time wiki-config is invoked, start here.**

#### Show the wiki skill ecosystem

Check which wiki skills are available in this session. Scan the `<available_skills>` section in your context for skills matching the `wiki-*` pattern (wiki-config, wiki-ingest, wiki-query, wiki-lint, wiki-integrate, wiki-crystallize). Skills found in `<available_skills>` are enabled and usable - mark as ✓. Skills not found are either not installed or toggled off in the user's settings - mark as ✗.

Present this table with actual status for each skill:

```
## Wiki Skill Ecosystem

| Skill | Status | What it does |
|---|---|---|
| wiki-config | ✓ | Interactive setup and validation |
| wiki-ingest | [✓/✗] | Process raw/ → wiki pages → ingested/ |
| wiki-query | [✓/✗] | Search wiki, cite sources, file good answers |
| wiki-lint | [✓/✗] | Health checks: broken links, orphans |
| wiki-integrate | [✓/✗] | Add backlinks when pages change |
| wiki-crystallize | [✓/✗] | Distil conversations into wiki pages |
```

If any are missing: "Install from https://github.com/vanillaflava/llm-wiki-claude-skills for the full ecosystem."

#### Present the welcome message

"Let me read the setup guide to orient us both..."

Then present:

---

> **Welcome to your LLM-Wiki**
> 
> Before we start, a quick orientation:

> **Filesystem access scope** - The directory your filesystem tool can reach. This is the outer boundary. Scope your tool tightly; broad access is a privacy risk.
>
> **Wiki root** - The folder inside that scope containing your Markdown notes and wiki system files (`wiki-config.md`, `index.md`, `log.md`, `raw\`, `ingested\`, `templates\`). This is where you keep your `.md` files - a subset of your Obsidian vault, your Logseq graph, or wherever you maintain notes. Skills find wiki root by looking for `wiki-config.md`.

> **Important:** Wiki root should not be your machine root (`C:\`, `/`) or user home - those are privacy risks. It also doesn't need to be your entire vault or graph. 
>
> We recommend setting wiki root as a subdirectory of your knowledge base - only the folders you're comfortable sharing with an agent. 
>
> The wiki works with your existing notes and helps you synthesize sources (PDFs, articles, documents) into interlinked wiki pages. Each source you add and each question you ask makes the knowledge base richer - it compounds over time.

---

If the user seems uncertain or asks for more context, offer: "Want me to show you the full setup guide?" Then read and present `references/setup-help.md`.

Otherwise proceed to assess state.

#### Assess state and branch

**Check filesystem scope:**

Identify scope root (allowedDirectories for MCP, CWD for Code, equivalent for other surfaces).

**If scope is insensible** (bare drive root, OS root, or user home):

"Your filesystem access is set to `[scope]` - this is very broad and a privacy risk. I recommend scoping to just your wiki folder after setup.

I need an absolute path to your wiki root. Examples:
- Windows: `C:\Users\YourName\Documents\wiki`
- Mac: `/Users/YourName/Documents/wiki`

What's the path?"

**Privacy discipline:** Never list users or enumerate system folders. Ask for path directly.

**If scope is sensible:** Search for `wiki-config.md` recursively (max 5 levels deep).

**Branch based on findings:**

→ **Config found** - Check for `wiki-schema.md` alongside it in the same directory:
  - **Both present**: Read and show the config. "You're already set up. Want to validate the config, view/edit the schema, or change something?" → Step 3 (Validate), Step 4 (Reconfigure), or Step 5 (Schema Management)
  - **Schema missing or malformed**: "Your `wiki-config.md` exists but `wiki-schema.md` is missing or unreadable. I can deploy the default schema now." → On user confirmation, copy `assets/wiki-schema.md` to the wiki root, then offer the main menu

→ **Config not found** - "You don't have a wiki yet. I'll walk you through setup." → Step 2 (Init)

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
2. Copy `assets/wiki-schema.md` to `<wiki_root>/wiki-schema.md`
3. If `<wiki_root>/index.md` missing, create it with header:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Maintained by wiki-ingest and wiki-integrate.*
   ```
4. If `<wiki_root>/log.md` missing, create it with header:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
5. Create `<wiki_root>/raw/` (the ingest queue)
6. Create `<wiki_root>/ingested/` and each subdir in `ingested_subdirs`, plus `<wiki_root>/ingested/assets/` (always)
7. Create `<wiki_root>/templates/` (empty; reserved for the page-templates system)
8. Append an init entry to log.md

Report to the user: list of what was created (explicitly mentioning both `wiki-config.md` and `wiki-schema.md`), suggest next steps (drop files in raw/, run /wiki-ingest).

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

### 5 - Schema Management

Schema management lets the user view, edit, reset, or repair `wiki-schema.md`.

**Before anything else, read `assets/wiki-schema.md` from this skill's own assets folder.** That bundled file is the default schema - the minimum coherent set of fields and enums across all six wiki skills. You need it in context to know what "default" means for reset/repair operations, and to describe safe vs risky edits accurately.

Then read the deployed `<wiki_root>/wiki-schema.md`.

**Compare and present:**

If deployed schema parses cleanly → "Here is your current schema:" show the frontmatter YAML. Offer:
- **Keep as-is** → return to main menu
- **Reset to bundled default** → overwrite with bundled template after explicit confirmation
- **Edit a field or enum** → interactive modification (see below)

If deployed schema is malformed (YAML parse error, missing critical sections) → "Your `wiki-schema.md` is malformed: [specific error]. I can reset it to the bundled default or you can edit manually. What would you prefer?"

If deployed schema is missing → already handled at Step 0 branch, shouldn't reach here, but if so: deploy from bundled template after confirmation.

**Editing a schema field:**

When the user wants to modify the schema, show this warning before accepting changes:

> *"The bundled schema defines the minimum coherent set. Removing mandatory fields, renaming fields, removing enum values that existing pages use, or changing field types can cause erratic skill behaviour (failed writes, invalid pages, lint false-positives). Adding new conditional fields or extending enum value lists is generally safe."*

Then ask what they want to change: a mandatory field, a conditional field, or an enum list.

For each change:
- Show the current value
- Accept the new value
- Validate: mandatory_fields must still include title, version, date, changes, page_type
- Validate: all three enums must still have at least their default values present unless user explicitly confirms removal
- Confirm before writing
- Write back to `wiki-schema.md` preserving comments and structure

**Reset / repair:**

On user request, copy `assets/wiki-schema.md` over `<wiki_root>/wiki-schema.md`. Log the operation. Confirm: "Schema reset to bundled default. Skills will use this on next invocation."

Never silently overwrite a custom schema. Reset is always an explicit, confirmed operation.

---

## Key Rules

1. **This skill owns the setup UX entirely.** The five operational skills all refuse on config-miss and point the user here (or at bundled `references/setup-help.md` for manual setup when this skill is not installed). Only this skill creates scaffolding, validates config, and handles reconfiguration. Only this skill advertises "set up wiki config" triggers.
2. **Never overwrite a live config without explicit confirmation.** Reconfiguration is a per-field dialogue.
3. **Wiki root is the directory containing wiki-config.md.** No wiki_root field in the config. Moving the wiki means moving this file.
4. **Never deploy to a bare drive root, OS root, or user home.** These are almost always user error.
5. **The bundled template and schema are the single source of truth.** This skill bundles `assets/wiki-config-template.md` and `assets/wiki-schema.md` (both deployed during init). The five operational skills bundle identical read-only copies at `references/wiki-config-template.md` and `references/wiki-schema.md` (pulled into context on every invocation - schema for page writes, config for manual setup assistance). When either file changes here, all five operational copies must update in lockstep. `references/setup-help.md` should also be kept in sync across the operational skills.
6. **wiki-config is the recommended setup, edit, and repair path.** Only wiki-config has the guided schema editor and repair flow. Operational skills can deploy bundled defaults from their `references/` folder as a fallback when wiki-config isn't available or the user prefers to press on, but must recommend `/wiki-config` first when it's available. Never silently overwrite a malformed schema - that case always needs an explicit user decision with an overwrite warning.

---

## What this skill does not do

- Does not perform any wiki operation (ingest, lint, integrate, crystallize, query). Those are the five operational skills.
- Does not manage TaskNotes config, plugin settings, or any non-wiki configuration.
- Does not reach outside the wiki root or modify the filesystem tool's scope.
- Does not handle page templates, tag taxonomy, or conventions (templates are reserved for a future batch).
