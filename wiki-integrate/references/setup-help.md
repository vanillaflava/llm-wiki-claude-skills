# Wiki Setup - Mini Guide

This reference helps when `wiki-config.md` cannot be found by any of the llm-wiki skills. The recommended path is to install and run the `/wiki-config` skill - it handles everything below interactively, with validation. Alternatively, the steps here are enough to configure the wiki by hand. For the full context (ecosystem background, privacy model, rationale) see the [llm-wiki-claude-skills README](https://github.com/vanillaflava/llm-wiki-claude-skills/blob/master/README.md).

## Path style - Windows vs Mac/Linux

Examples use Windows backslash (`\`) convention. On Mac or Linux, use forward slash (`/`) instead. YAML itself is agnostic; use your OS's native separator throughout.

## Two scopes to understand

**Filesystem access scope.** The directory your filesystem tool (filesystem MCP, Claude Code, or equivalent) can reach. This is the outer boundary - nothing outside it is visible to any wiki skill. Scope your tool tightly; broad access is a privacy risk (see the repo README's Privacy section for the reasoning).

**Wiki root.** The folder inside the filesystem scope that contains your notes and the wiki system files (`wiki-config.md`, `index.md`, `log.md`, `raw\`, `ingested\`, `templates\`). The skills derive wiki root from the location of `wiki-config.md` - there is no `wiki_root` field in the config.

Wiki root is NOT your machine root (`C:\`, `/`), NOT your user home (`C:\Users\you\`, `~/`), and not necessarily the top of your knowledge base. It is a specific folder you designate for the wiki to live in, inside your filesystem scope.

## Create wiki-config.md

At `<your-wiki-root>/wiki-config.md`, create the file with this content (or copy the `wiki-config-template.md` bundled alongside this help file and rename it):

```yaml
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
```

Edit `blacklist` before using the wiki - the placeholders above are obviously not real folder names; replace with actual paths you want protected (e.g. `Repositories\`, private folders, source trees).

## Directory scaffolding

Create these alongside `wiki-config.md`:

- `index.md` with header `# Wiki Index`
- `log.md` with header `# Wiki Operation Log`
- `raw\` (the ingest queue - drop source files here)
- `ingested\` and each subdir listed in `ingested_subdirs` above
- `ingested\assets\` (always; holds files that couldn't be read)
- `templates\` (reserved for page templates - empty for now)

## Field summary

- `blacklist` - folders that must never receive wiki page writes. File moves to `ingested\` are always permitted. Placeholder values ship; edit before use.
- `index_excludes` - folders that don't appear in `index.md`. Always include `raw\` and `ingested\`.
- `ingested_folder` + `ingested_subdirs` - where processed sources get archived and how they're classified.
- `templates_folder` - reserved for the page-templates system (not yet active). Folder exists empty for now.
- `log_format` - entry format for `log.md`. Don't change unless you know what you're doing.

## Repo

https://github.com/vanillaflava/llm-wiki-claude-skills

## For interactive setup

Install the `wiki-config` skill from the repo for a walk-through that handles all of this, including validation and reconfiguration. The manual path above is fully supported and requires no skill install.

## Retry your command

Once `wiki-config.md` exists at your wiki root and the directory scaffolding is in place, retry whatever command brought you here.
