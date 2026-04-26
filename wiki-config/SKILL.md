---
name: wiki-config
description: Set up, validate, and reconfigure the llm-wiki skill suite. Use when the user says /wiki-config, asks to "set up my wiki", "configure wiki", "initialize wiki config", mentions "page structure problems", "header errors", "schema issues", "missing templates", "page templates", or when any wiki skill reports a missing or invalid config, schema, or templates folder. Owns the interactive configuration flow, schema management, and template management for the wiki system.
---

<!-- version: 1.7 -->

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

The bundled `references/setup-help.md` is available throughout this session. Read it if you need more orientation context, if a user asks detailed questions about the wiki system, or if you get stuck.

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

→ **Config not found** - "You don't have a wiki yet. I'll walk you through setup." → Step 2 (Init)

→ **Config found** - Read it, then run a full environment check in the same directory. Check each of the following:

1. `wiki-schema.md` - present and parses cleanly / missing / malformed
2. `templates/` folder - present / missing
3. Template file completeness - which of the 12 expected files are present (`knowledge.md`, `reference.md`, `survey.md`, `domain-home.md`, `overview.md`, `log.md`, `index.md`, `longform.md`, `profile.md`, `established-patterns.md`, `note.md`, `config.md`)
4. `templates_folder:` field in `wiki-config.md` - present / absent

Present a compact, friendly summary - one line per component. For example:

> **Your wiki environment**
>
> - wiki-config.md: found
> - wiki-schema.md: found
> - templates/: present (9 of 12 files)
> - templates_folder field: absent from config
>
> Two items need attention. I can repair both in one step, or you can choose individually.

**If everything is healthy:** offer the main menu directly.

**If any items are missing or broken:** show the summary above, then offer:
- "Repair all" - deploys all missing items in one pass; one confirmation before starting
- Individual choice - user picks which items to address

**Repair actions per component:**
- `wiki-schema.md` missing → deploy from `assets/wiki-schema.md` (non-destructive, no confirmation needed)
- `wiki-schema.md` malformed → warn, offer reset to bundled default; confirm before overwriting
- `templates/` folder missing → introduce templates with the blockquoted message below, then offer to create folder and deploy all 12 files + README from `assets/templates/`
- Individual template files missing → deploy only the missing files from `assets/templates/`; never overwrite files already present
- `templates_folder:` absent from config → write `templates_folder: templates/` to `wiki-config.md`; only do this when `templates/` folder is confirmed present

When `templates/` is missing or incomplete and the user hasn't encountered this before, present:

> **Page Templates**
>
> Templates give every wiki page a consistent, structured starting point. When a skill creates a new page, it reads the matching template from your `templates/` folder and uses it as a scaffold - substituting placeholders like `{{TITLE}}` and `{{DATE}}` with real values.
>
> Your templates are yours to edit. Changes take effect immediately for any new pages; existing pages are never modified. If a template file is missing, the skill falls back to a minimal hardcoded structure.
>
> The default set covers all 12 page types: knowledge, reference, survey, domain-home, overview, log, index, longform, profile, established-patterns, note, and config.

After presenting, ask: "Deploy the default template set to `<wiki_root>/templates/`?"

After any repair: show the updated status and offer the main menu.

**Main menu (all components healthy):**

Offer:
- Validate config (Step 3)
- Edit config settings (Step 4)
- Manage schema (Step 5)
- Manage templates (Step 6)

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
2. Write `templates_folder: templates/` into `<wiki_root>/wiki-config.md` if not already present in the template
3. Copy `assets/wiki-schema.md` to `<wiki_root>/wiki-schema.md`
4. If `<wiki_root>/index.md` missing, create it with header:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Maintained by wiki-ingest and wiki-integrate.*
   ```
5. If `<wiki_root>/log.md` missing, create it with header:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
6. Create `<wiki_root>/raw/` (the ingest queue)
7. Create `<wiki_root>/ingested/` and each subdir in `ingested_subdirs`, plus `<wiki_root>/ingested/assets/` (always)
8. Create `<wiki_root>/templates/` and copy all 12 template files from `assets/templates/` into it
9. Copy `assets/templates/README.md` to `<wiki_root>/templates/README.md`
10. Append an init entry to log.md

Report to the user: list of what was created, explicitly naming `wiki-config.md`, `wiki-schema.md`, and the templates folder (N template files). Suggest next steps (drop files in raw/, run /wiki-ingest).

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

### 6 - Template Management

Template management lets the user view, repair, reset, or inspect the page templates in `<wiki_root>/templates/`.

Present the orientation on first entry to this flow:

> **Page Templates**
>
> Templates give every wiki page a consistent, structured starting point. When a skill creates a new page, it reads the matching template from your `templates/` folder and uses it as a scaffold - substituting placeholders like `{{TITLE}}` and `{{DATE}}` with real values.
>
> Your templates are yours to edit. Changes take effect immediately for any new pages; existing pages are never modified. If a template file is missing, the skill falls back to a minimal hardcoded structure.
>
> The default set covers all 12 page types: knowledge, reference, survey, domain-home, overview, log, index, longform, profile, established-patterns, note, and config.

**Assess current state:**

List files in `<wiki_root>/templates/`. Identify which of the 12 expected files are present and which are missing.

Present the status:

> **Templates folder: [X of 12 default templates present]**
>
> Present: knowledge.md, survey.md, ... [list]
> Missing: reference.md, ... [list, if any]

**Offer actions:**

- **View a template** - read and display the content of any named template file; note that the user can edit it directly in their notes app
- **Deploy missing defaults** - copy any missing files from `assets/templates/` to `<wiki_root>/templates/`; never overwrite files already present
- **Reset a specific template** - overwrite a named template with the bundled default; warn before proceeding
- **Reset all templates** - overwrite all 12 with bundled defaults; warn before proceeding
- **View README** - display `<wiki_root>/templates/README.md`

**Reset safeguard** - show before any reset:

> *"Resetting a template replaces your edits with the bundled default. Your existing wiki pages are not affected - only future pages will use the new structure. This cannot be undone from within this skill."*

Confirm per file on individual reset; confirm once (listing all 12 files) before a full reset.

After any action, offer to continue with another template action or return to the main menu.

---

## Key Rules

1. **This skill owns the setup UX entirely.** The five operational skills all refuse on config-miss and point the user here (or at bundled `references/setup-help.md` for manual setup when this skill is not installed). Only this skill creates scaffolding, validates config, and handles reconfiguration. Only this skill advertises "set up wiki config" triggers.
2. **Never overwrite a live config without explicit confirmation.** Reconfiguration is a per-field dialogue.
3. **Wiki root is the directory containing wiki-config.md.** No wiki_root field in the config. Moving the wiki means moving this file.
4. **Never deploy to a bare drive root, OS root, or user home.** These are almost always user error.
5. **The bundled assets are the single source of truth.** This skill bundles `assets/wiki-config-template.md`, `assets/wiki-schema.md`, and `assets/templates/` (12 template files + README). All are deployed during init. The five operational skills bundle identical read-only copies of `wiki-config-template.md` and `wiki-schema.md` in their `references/` folders. When any shared file changes here, operational copies must update in lockstep. `references/setup-help.md` should also be kept in sync across the operational skills.
6. **wiki-config is the recommended setup, edit, and repair path.** Only wiki-config has the guided schema editor and repair flow. Operational skills can deploy bundled defaults from their `references/` folder as a fallback when wiki-config isn't available or the user prefers to press on, but must recommend `/wiki-config` first when it's available. Never silently overwrite a malformed schema - that case always needs an explicit user decision with an overwrite warning.

---

## What this skill does not do

- Does not perform any wiki operation (ingest, lint, integrate, crystallize, query). Those are the five operational skills.
- Does not manage TaskNotes config, plugin settings, or any non-wiki configuration.
- Does not reach outside the wiki root or modify the filesystem tool's scope.
- Does not edit template content directly - it deploys, repairs, and resets templates; the user edits them in their notes app.
