# Wiki Setup - Quick Start Guide

**What this is:** A personal knowledge wiki that compounds over time. You add sources; your agent reads, synthesises, and files them into interlinked wiki pages. You ask questions; your agent reads the wiki and answers with citations. Each source and each good question makes it richer.

This guide helps when `wiki-config.md` or `wiki-schema.md` cannot be found. The recommended path is `/wiki-config` - it handles everything below interactively with validation, and it is the only skill that can deploy `wiki-schema.md`. For full context (ecosystem background, privacy model, Karpathy's pattern), see the [llm-wiki-skills README](https://github.com/vanillaflava/llm-wiki-skills).

---

## The six skills

| Skill | What it does |
|---|---|
| **wiki-config** | Sets up, validates, and reconfigures your wiki. Run once at cold start; anytime you want to inspect or change settings |
| **wiki-ingest** | Processes files from `raw/` into wiki pages; synthesises, cross-references, archives to `ingested/` |
| **wiki-query** | Answers questions by reading the wiki; cites sources; can file valuable answers as new pages |
| **wiki-lint** | Health checks: broken links, orphaned pages, unreferenced sources |
| **wiki-integrate** | Weaves new/updated pages into the knowledge graph with backlinks |
| **wiki-crystallize** | Distils working sessions into structured wiki pages; updates hubs and overview |

**First-time setup - mandatory:** Run `/wiki-config` at least once before any other wiki skill. It deploys both `wiki-config.md` (settings) and `wiki-schema.md` (the frontmatter structure for all wiki pages). The operational skills refuse to proceed if `wiki-schema.md` is missing or malformed, and only `/wiki-config` can deploy or repair it.

**Minimal setup:** wiki-config (once) + wiki-ingest + wiki-query. Add the others as your wiki grows.

---

## Two scopes to understand

**Filesystem access scope.** The outer boundary - the directory your filesystem tool can reach. Nothing outside it is visible to any wiki skill. **Scope tightly; broad access is a privacy risk.**

Example scope hierarchy:
```
Your machine
├── Everything else
└── Your knowledge space (vault/notes)
    ├── Private/       ← never expose to agent
    ├── Sensitive/     ← work confidential
    ├── Archive/       ← legacy notes
    └── Agent Access/  ← wiki root (ONLY folder agent needs)
        ├── wiki-config.md
        ├── wiki-schema.md
        ├── index.md, log.md
        ├── raw/, ingested/, templates/
        └── ... wiki pages
```

**Wiki root.** The specific folder inside that scope containing your notes and wiki system files. Skills find it by looking for `wiki-config.md` - there's no separate path field. If you move the wiki, move this file with it.

Wiki root is **NOT** your machine root (`C:\`, `/`), **NOT** your user home (`~/`, `C:\Users\you`), and not necessarily your vault root. It's the folder you designate for the wiki.

---

## Quick setup (manual path)

### 1. Create wiki-config.md and wiki-schema.md

**Recommended:** Run `/wiki-config` - it deploys both files with correct defaults and is the only reliable way to get `wiki-schema.md` right.

**Manual path:** at `<your-wiki-root>/`, create both files.

First, `wiki-config.md`:

```yaml
---
blacklist:
  - Repositories\
  - Private\
  - (add folders where wiki should never write)

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

Then `wiki-schema.md`: copy from the skill's `references/wiki-schema.md` - every operational skill bundles an identical read-only reference copy. Do not hand-write the schema; the bundled file is the authoritative default. The operational skills will refuse to proceed if the schema is missing or malformed.

**Path style:** Examples use Windows backslash (`\`). On Mac/Linux, use forward slash (`/`).

### 2. Create directory scaffolding

Alongside `wiki-config.md` and `wiki-schema.md`:
- `index.md` with header `# Wiki Index`
- `log.md` with header `# Wiki Operation Log`  
- `raw/` (drop source files here)
- `ingested/` with subdirs: `clippings/`, `documentation/`, `papers/`, `articles/`, `data/`, `notes/`, `assets/`
- `templates/` (reserved for future; empty for now)

### 3. Write CLAUDE.md (optional but recommended)

A short doc telling your agent where things live and what conventions to follow. One paragraph is enough to start. The wiki improves this over time.

### 4. Drop a file in raw/ and run `/wiki-ingest`

Your agent reads the source, synthesises a wiki page, updates the index, and archives the file to `ingested/`. That's the basic loop.

---

## Field reference

**blacklist** - Folders where wiki page creation is forbidden (relative to wiki root). File moves to `ingested/` are always permitted. **Edit the placeholder values before first use** - they're obviously not real folder names.

**index_excludes** - Folders excluded from `index.md`. Always include `raw/` and `ingested/` (source files aren't wiki pages).

**ingested_folder** - Where processed sources get archived. Must be in `index_excludes`; must NOT be in `blacklist`.

**ingested_subdirs** - Archival taxonomy. wiki-ingest classifies each source and routes to the matching subfolder. Adapt these to your workflow. `ingested/assets/` is always created for unreadable files.

**templates_folder** - Reserved for page-templates system (future release). Nothing reads this yet; safe to ignore.

**log_format** - Entry format for `log.md`. Don't change unless you know why.

---

## For interactive setup

Install `/wiki-config` for guided setup with validation and reconfiguration. It is the recommended path because it is the only skill that can deploy and repair `wiki-schema.md` reliably. Manual setup works if wiki-config is not installed, but you must copy the bundled `wiki-schema.md` from any operational skill's `references/` folder; do not hand-write it.

## Retry your command

Once `wiki-config.md`, `wiki-schema.md`, and directory scaffolding exist at your wiki root, retry whatever command brought you here.
