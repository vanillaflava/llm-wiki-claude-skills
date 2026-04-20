---
name: wiki-config
description: Set up, validate, and reconfigure wiki-config.md for the llm-wiki skill suite. Use when the user says /wiki-config, asks to "set up my wiki", "configure wiki", "initialize wiki config", or when any wiki skill reports a missing or invalid config. Owns the interactive configuration flow for the wiki system.
---

<!-- version: 1.1  -->

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

Check which wiki skills are installed. Skills are typically found at `/mnt/skills/user/` or similar paths in the agent's environment. If you're unsure where skills are located on this surface, search for documentation on skill installation before proceeding.

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

> **Welcome to the wiki system**
> 
> Before we start, here's what you need to understand:
> 
> **Filesystem access scope** - The directory your filesystem tool can reach. This is the outer boundary. Scope your tool tightly; broad access is a privacy risk.
> 
> **Wiki root** - The folder inside that scope containing your Markdown notes and wiki system files (`wiki-config.md`, `index.md`, `log.md`, `raw\`, `ingested\`, `templates\`). This should be a place where you normally read your `.md` files - like a subset of your Obsidian vault, your Logseq graph, or simply the folder where you keep your Markdown notes. Skills find wiki root by looking for `wiki-config.md`.
> 
> Wiki root is NOT your machine root (`C:\`, `/`), NOT your user home, and not necessarily your vault root. It's a specific folder you choose for the wiki. **This is Markdown-only** - we're building a wiki of interlinked `.md` files, not Word docs or PDFs (though you can drop those in `raw\` as sources to be synthesised).

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

→ **Config found** - Read and show it. "You're already set up. Want to validate the config or change something?" → Step 3 (Validate) or Step 4 (Reconfigure)

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
