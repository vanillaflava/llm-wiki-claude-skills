---
title: Wiki Help
version: 1.7
date: 2026-04-27
updated: 2026-04-27
status: active
description: "User-facing help for the llm-wiki skill suite. Fields, write discipline, naming, page types, templates, and tips."
changes: "v1.7 - Frontmatter fields converted to tables"
page_type: reference
---

# Wiki Help

*You installed the wiki skills and ran `/wiki-config`. This file is what the system deployed to your wiki root. It covers how the system actually works, what the skills write and what they leave alone, and a few things that will save you time.*

---

## How do I get started?

You have a wiki scaffold and empty folders. Here is a practical path from zero to a working knowledge base.

### Step 1 - Give your wiki its initial shape with domain homes

Before ingesting anything, decide what domains your wiki will cover. A domain is any area of knowledge or work you want to track - a research project, a job function, a hobby, a health situation. You do not need to plan it all upfront; two or three domains is enough to start.

For each domain, create a Domain Home page. The easiest way: start a chat and describe the domain. Tell the agent what you are working on, what you already know, what questions you are trying to answer. Then run `/wiki-crystallize` and target a new domain home. The agent will create `[Domain] - Home.md` from the domain-home template, and you will have an anchor page for that area of knowledge.

Domain homes are the structural backbone of the wiki. Agents assigned to a domain read the home page first and immediately know the current state, the open questions, and where the work left off. Without them, every chat starts from scratch.

### Step 2 - Start bringing in sources

Once you have at least one domain home, start feeding the wiki. A few paths:

**Sources you already have.** If you have existing notes, PDFs, articles, or documents on a topic, drop them into `raw/` and run `/wiki-ingest`. The skill will read them, synthesise wiki pages, and file the originals in `ingested/`. This is the fastest way to turn an existing collection into structured, interlinked knowledge.

**Web clippings.** If you read things online that you want to remember, a browser-based web clipper is the most frictionless capture tool. Clip an article, it lands in a folder, you drop it into `raw/`, done. Popular free options:

- **Obsidian Web Clipper** (official Obsidian extension) - clips directly into your notes in Obsidian-native Markdown with good metadata. Obsidian-specific.
- **MarkDownload** (browser extension) - saves any web page as a clean Markdown file. Works with any Markdown-based workflow.
- **Readwise Reader** - read-it-later app with highlight export. Outputs structured Markdown with your annotations.
- **Raindrop.io** - bookmarking with full-page saves and tag organisation.

All of these produce files you can drop directly into `raw/`. The wiki does not care where the file came from; `/wiki-ingest` classifies and routes it.

**Starting from conversation.** You do not need source files to get value from the wiki. Open a chat about a domain you care about - ask questions, think through a problem, explore a topic. When the conversation has produced something worth keeping, run `/wiki-crystallize`. The session becomes a knowledge page. This is how the wiki grows from thinking, not just reading.

### Step 3 - Build the habit

The wiki compounds when you use it regularly. The core loop is simple:

1. Drop sources into `raw/`, run `/wiki-ingest` when there are enough to process
2. Run `/wiki-crystallize` before closing any chat where something meaningful happened
3. Run `/wiki-lint` occasionally to keep the graph healthy
4. When a domain home starts to feel stale, update it - either manually or via `/wiki-crystallize`

The wiki does not need to be comprehensive to be useful. Even a small, well-maintained knowledge base with a few active domain homes beats a large, stale one.

---

## What your wiki looks like after first run

When you run `/wiki-config` for the first time, it creates the following in your wiki root. These files are explained in the sections below, but this is the complete picture - nothing is hidden or deferred.

**Config files** - read by skills on every session:

| File | What it is |
|---|---|
| `wiki-config.md` | Your vault's settings - blacklist, ingested subdirs, templates folder path. Edit with `/wiki-config`. |
| `wiki-schema.md` | Field and page type definitions. All skills follow this. Edit with `/wiki-config` > Manage schema. |
| `wiki-help.md` | This file. Human-readable guide. Skills do not read it. |

**Navigation files** - your starting points:

| File | What it is |
|---|---|
| `Home.md` | Navigation hub. Explains the skill workflow and links to major sections. Good first stop for any new agent or new user. |
| `Overview.md` | Living synthesis of what you know across domains. Starts empty - grows via `/wiki-crystallize` as knowledge accumulates. |
| `index.md` | Catalogue of all wiki pages. Maintained automatically by wiki-ingest and wiki-integrate. Use for lookup, not orientation. |
| `log.md` | Prepend-only audit trail of all skill operations. New entries at the top. Skills write here; you read it. |

**Folders:**

| Folder | What it is |
|---|---|
| `raw/` | The ingest queue. Drop source files here, then run `/wiki-ingest`. |
| `ingested/` | Where source files go after processing. Organised into subdirs by content type. |
| `templates/` | Page templates - one per page type. Yours to edit. |

After first run, `Home.md` and `Overview.md` exist but are mostly empty scaffolding. That is correct. The wiki grows from here.

---

## The ingest pipeline - raw/ and ingested/

`raw/` is a flat queue. You drop files in, run `/wiki-ingest`, and the skill reads them, synthesises wiki pages, then moves each file into `ingested/` as an atomic commit. The move is the record - a file still in `raw/` has not been processed yet; a file in `ingested/` has.

`ingested/` is organised into subdirs by content type. Wiki-ingest classifies each file and routes it automatically:

| Subdir | What goes here |
|---|---|
| `clippings/` | Saved web pages, browser clips, web clipper exports |
| `papers/` | Academic papers, PDFs, research material |
| `documentation/` | Product docs, API references, technical specs |
| `articles/` | Blog posts, essays, long-form reading |
| `data/` | CSV, JSON, structured datasets |
| `notes/` | Freeform drafts, quick captures, anything else |
| `assets/` | Files that could not be read or extracted - kept for reference |

You can change the subdir names in `wiki-config.md` to match your workflow. The `assets/` subdir is always created and always used for unprocessable files.

**You do not manage ingested/ directly.** Wiki-ingest puts files there; wiki-lint checks that every ingested file is referenced by a wiki page. If you delete a file from ingested/, wiki-lint will flag its wiki page as having an orphaned source.

---

## How the wiki grows over time

The files created on init are scaffolding. The wiki builds out as you use the skills:

- `/wiki-ingest` creates knowledge pages from your sources and updates `index.md`
- `/wiki-crystallize` distils sessions into domain homes and eventually `Overview.md`
- `/wiki-integrate` adds backlinks when you create pages directly
- Domain homes appear when you first crystallize into a domain - run `/wiki-crystallize` and target a new domain home

`Overview.md` starts as a template stub. The first time you run `/wiki-crystallize` targeting it, it becomes a living synthesis of what the wiki knows. After that, keep it updated whenever knowledge shifts significantly across domains.

---

## The three config files

`wiki-config.md`, `wiki-schema.md`, and `wiki-help.md` are the files skills read on every session (skills don't read wiki-help.md, but it lives alongside them). If any go missing or get corrupted, run `/wiki-config` - it will offer to repair or redeploy from bundled defaults.

---

## Frontmatter fields

Skills write frontmatter. You write the body. That division is mostly clean, but it helps to know what each field is for.

### The five mandatory fields

Every page a skill creates gets these:

| Field | What it holds | Key rules |
|---|---|---|
| `title:` | The page title | Must match the H1 heading exactly. Quote if the value contains a colon. |
| `version:` | Page version number | Starts at 1.0. Minor bump (1.1, 1.2) for additions; major bump (2.0) for structural rewrites. You can bump it on significant manual edits too. |
| `date:` | Creation date | Set once, never changed again - even on full rewrites. `updated:` is what tracks subsequent touches. |
| `changes:` | Description of the most recent change | Always quoted. Under 80 characters. Not a file path. Think of it as a commit message for the page. |
| `page_type:` | What kind of page this is | Controls how skills handle the page. See Page types below for the full list. |

### Conditional fields

Skills add these when they apply. You will see them on most pages after the system has been running for a while.

| Field | Written by | When | Notes |
|---|---|---|---|
| `updated:` | Any skill on touch | Every skill write | YYYY-MM-DD. Tracks last skill contact. |
| `status:` | wiki-ingest on create; any skill on state change | Every page | Default `active`. Values: `stub` (needs work), `artefact` (static deliverable), `archived` (deprecated but kept), `snapshot` (frozen copy). Skills never downgrade. |
| `description:` | wiki-ingest, wiki-crystallize | Every page | ~200 characters. Useful for search and agent orientation. You can edit it; the skill rewrites on a full crystallize pass. |
| `crystallize_count:` | wiki-crystallize only | Every crystallize write | Integer starting at 1. Counts deliberate crystallize events only - not routine edits. |
| `source:` | wiki-ingest on create | Pages with an ingested origin | A list pointing to the file's path in `ingested/`. Omitted on hand-authored pages. |
| `reliability:` | wiki-ingest on create | Only when `source:` is present | Skill's assessment of the originating source. Values: `high` (primary/authoritative), `medium` (secondary, reasonable confidence), `low` (speculative or unverified). |

### The Pending Review section

If wiki-ingest rates a page as medium or low reliability, it adds a `## Pending Review` section to the body. This is not an error - it is a quality signal. The section flags specific claims that would benefit from a stronger source.

When you have corroborated a claim - found a primary source, confirmed it from experience, or decided you are comfortable with the confidence level - remove the relevant lines from Pending Review. Run `/wiki-ingest` again with a better source if you have one; the skill will update `reliability:` and clear Pending Review if the new source resolves the gap.

---

## Write discipline

**Frontmatter is skill territory.** Skills overwrite known frontmatter fields without asking. If you edit `title:` or `status:` manually, the next skill write may overwrite your change. Use `/wiki-config` to change schema-level settings; edit the body for everything else.

**One exception:** unrecognised frontmatter fields are preserved. If you add a custom field the schema does not define, skills leave it alone.

**Body is yours.** Skills add material, suggest restructuring, and flag contradictions - but they do not silently delete body content. If a skill removes something, it will say so. If you disagree, push back.

**`date:` is immutable.** Set on creation, never changed again. `updated:` is the field that tracks ongoing changes.

---

## Naming and linking

**File names:** lowercase, hyphens between words, no spaces. `my-topic.md` not `My Topic.md`. Page title (in frontmatter and H1) can use normal capitalisation; the filename is just for the filesystem.

**Wikilinks:** Use `[[Page Title]]` to link between pages. Most Markdown note-taking apps with wikilink support (Obsidian, Logseq, and others) resolve links by filename rather than path, so you do not need to include the folder. If two files share a name, qualify with the path: `[[Folder/Page Title]]`.

**Do not use em-dashes** in file names, page titles, or wikilink text. Use ` - ` (space-hyphen-space) instead. This matters for wiki-lint, which flags em-dashes in titles and filenames as a hard error.

**Index:** `index.md` in your wiki root is a lookup catalogue, not an orientation page. It lists pages and their one-line descriptions. Skills update it automatically. You do not need to maintain it manually.

---

## Page types

The `page_type:` field controls how skills treat a page. The full enum is in `wiki-schema.md`; here is the plain-language version of the types you will use most:

| Type | When to use it |
|---|---|
| `knowledge` | A synthesised page about a topic - what you know about X. The default for most pages wiki-ingest creates. |
| `reference` | Factual reference material you will look things up in. Conventions guides, glossaries, schema docs. Treated as stable - wiki-lint's staleness check exempts it. |
| `survey` | An overview of a topic area rather than a deep synthesis of it. "What exists in this space" rather than "what do I know about this". |
| `research-note` | Active investigation in progress. Less polished than knowledge. |
| `domain-home` | The anchor page for a knowledge domain. Bootstrap target for agents assigned to that domain. |
| `log` | Prepend-only operational record. Skills write new entries at the top, below the header. |
| `index` | Catalogue or index page. `index.md` uses this type. |
| `config` | Configuration document. `wiki-config.md` and `wiki-schema.md` use this type. Skills treat config pages as sensitive and do not modify them during normal operations. |

If wiki-integrate or wiki-crystallize touches a page and the `page_type:` field is missing, they will ask you what type it should be before proceeding.

---

## Customising your wiki

The wiki is designed to be shaped around how you actually think and work. Three layers give you increasing levels of control.

### Config - what the wiki does

`wiki-config.md` controls the operational behaviour of the skills: which folders they will never write to (the blacklist), how ingested sources get classified and filed, where your templates live. Edit it with `/wiki-config` or directly in any text editor.

The most useful things to customise here:

- **Blacklist** - any folder that should never receive a skill-written wiki page. Your `Repositories/` folder, private areas, plugin data directories. Add these before your first ingest.
- **Ingested subdirs** - the taxonomy inside `ingested/`. The defaults (`clippings`, `papers`, `documentation`, `articles`, `data`, `notes`) cover most workflows but you can rename or add subdirs to match how you think about your sources. wiki-ingest classifies files into whichever subdirs exist.

### Schema - what fields pages carry

`wiki-schema.md` defines the frontmatter structure every skill follows: which fields are mandatory, which are conditional, and which enum values are valid. Edit it with `/wiki-config` > Manage schema.

**Safe to extend:**
- Add new conditional fields - a `project:` field, a `context:` tag, a `reviewed:` date. Skills will preserve any field they do not recognise, so your custom fields survive every skill write.
- Extend enum lists - add new `page_type` values, new `status` values, new `reliability` tiers. Skills validate against whatever is in your schema, so new values become valid immediately.

**Approach with care:**
- Renaming or removing fields that existing pages already use will cause wiki-lint to report schema violations across your whole wiki until you migrate the old values. Do it with `/wiki-config`'s guided flow, which warns you before applying changes.

A well-extended schema is how the wiki learns your vocabulary. A `project:` field means an agent can filter pages by project. A `context:` field means pages can carry the situational tags that matter in your workflow.

### Templates - how new pages are structured

`templates/` holds one Markdown file per page type. When wiki-ingest or wiki-crystallize creates a new page, it reads the matching template and uses it as the scaffold - filling in placeholders like `{{TITLE}}` and `{{DATE}}` and then populating each section with synthesised content.

**Templates are entirely yours to edit.** Open any file in `templates/` in your notes app or any plain text editor and change whatever you like:

- **Sections and headings** - add, remove, or rename H2/H3 sections. If you always want a `## Next Steps` section on knowledge pages, add it. The agent will populate it.
- **Markdown elements** - callout blocks, tables, code fences, checklists, Mermaid diagrams. If you want new survey pages to include a comparison table by default, put one in the template. The agent will fill it in.
- **Placeholder text** - instructional comments inside section bodies are read by the agent as guidance. `<!-- Summarise the key finding in 2-3 sentences -->` will be followed.

Changes take effect immediately for new pages. Existing pages are never touched - templates only shape what gets created, not what already exists.

If you want to reset a template to the default, run `/wiki-config` > Template Management > Reset. If a template file goes missing entirely, the skill falls back to a minimal hardcoded stub and notes it in the session summary.

**The practical upshot:** the schema defines the frontmatter fields agents write; the templates define the body structure agents follow. Together they mean the agent is not guessing how to structure a new page - it is reading your specification.

---

## Tips

**Crystallize before closing a heavy chat.** When a thread has gone long and you are about to close it, run `/wiki-crystallize`. It distils what was learned into the relevant Domain Home or knowledge page. The chat is scaffolding; the wiki page is what persists. If you skip this, the knowledge evaporates with the context window.

**Run `/wiki-lint` occasionally.** It surfaces broken links, orphaned pages, pages that have not been touched in 90 days, and schema compliance issues. It never auto-fixes anything - it just reports. File the things worth fixing; ignore the rest.

**Re-ingest to upgrade a page.** If you find a better source on a topic where you already have a page, drop it into `raw/` and run `/wiki-ingest`. The skill will offer to enrich the existing page and update `source:` and `reliability:` if the new source is clearly stronger.

**Domain Homes are the thing.** Each knowledge domain has a `Domain - Home.md`. This is the page a fresh agent reads to understand the current state of that domain without a briefing. Keep it current. `/wiki-crystallize` targets the Domain Home by default.

**The wiki solves the growing context problem.** Chats with agents accumulate context - every message, every tool call, every back-and-forth adds to the load. At some point the context window fills, responses get slower and less reliable, and you need to start a fresh chat. Without the wiki, you spend the first ten minutes of every new chat re-briefing the agent on where you left off. With the wiki, you do not.

The pattern: when a chat is getting heavy, run `/wiki-crystallize` to distil it into the domain home. Close the chat. Open a new one. The agent reads the domain home and picks up almost exactly where the previous one left off - current state, open questions, what was decided, what is next. The knowledge transfers; only the conversation history does not.

This works across three layers:

- **General instructions** (Personal Preferences in Claude, System Prompt elsewhere) carry your permanent preferences and working style into every chat automatically.
- **Project instructions** prime a chat with domain context before it even starts. In Claude and most other providers, you can attach instructions to a project or workspace that load automatically. Point them at your domain home, or paste a summary.
- **Domain Home as first read** is the final layer - the agent reads the current state of the domain and knows what is in progress, what is open, and what was decided last session. A fresh agent with a fresh context window, equipped as if they had been there all along.

The wiki is the memory that survives session cycling. Crystallize before you close.

**Pair the wiki with task management.** Knowledge and tasks work better together than either does alone. The wiki holds what you know; tasks hold what you intend to do about it. When agents can both read domain knowledge and create, update, and close tasks, they shift from passive responders to active participants - they can track their own work, surface blockers, and hand off cleanly across sessions.

If you are using Obsidian, a companion skill is available at https://github.com/vanillaflava/tasknotes-claude-skill that integrates directly with the wiki skills. It requires the **TaskNotes Community Plugin** enabled in Obsidian. With both installed, agents can run their own Kanban boards anchored to domain homes, and domain homes gain a live task view automatically.

**You do not have to use all the fields.** The schema defines what skills write; it does not require you to manually fill everything. If you create a page by hand and skip `reliability:`, wiki-lint will flag it softly but nothing breaks. The fields that matter most for day-to-day use are `title:`, `page_type:`, and `date:`.

---

## Common problems

| Problem | Likely cause | Fix |
|---|---|---|
| Skill refuses to proceed, says schema is missing | `wiki-schema.md` not found or malformed | Run `/wiki-config` and choose Repair |
| New pages have the wrong structure | Template for that page_type is missing or edited unexpectedly | `/wiki-config` > Template Management > Redeploy |
| `[[Wiki Help]]` link is broken in wiki-schema.md | This file was not deployed | Run `/wiki-config` - it will deploy missing assets |
| A page has orange Properties in Obsidian | Frontmatter contains a nested object or map | Open Source mode; the field is valid YAML and agents read it correctly - only the Properties UI is affected |
| Wiki-lint reports em-dash in filename | A file was named with `—` instead of ` - ` | Rename the file; in Obsidian wikilinks update automatically - in other editors update any links manually |
| `changes:` field causes a YAML parse error | Unquoted value contains a colon-space | Wrap the value in double quotes |

### Broken frontmatter

Frontmatter is the YAML block between the `---` delimiters at the top of a file. When it is valid, most Markdown note-taking apps process it silently - rendering it as editable properties, a collapsible metadata header, or simply hiding it from reading view. When the YAML is broken, that processing fails.

**What you will typically see:** the `---` block appears as plain text at the top of the page in preview or reading mode, instead of being parsed. This means the YAML failed entirely - usually an unquoted colon-space, a stray tab character, or a missing or duplicated `---` delimiter.

How this looks varies by editor:

- **Obsidian** - The Properties panel disappears or shows fields as raw text in Reading mode. A specific sub-case: a nested object or map (rather than a scalar or list) shows as an orange field in Properties rather than a full parse failure - this is actually valid YAML and agents read it correctly, it only affects the Properties panel UI. A field missing from Properties entirely usually means a reserved property name conflict.
- **Logseq** - YAML frontmatter is supported alongside Logseq's native `key::` property format. Invalid YAML in the frontmatter block typically renders as raw text at the top of the page in document view.
- **VS Code and plain text editors** - Frontmatter is always visible as raw text. There is no processed view to break - you are already looking at the raw YAML. Some extensions (Foam, Dendron) add frontmatter processing; broken YAML may show as a warning in those contexts.
- **Typora, iA Writer, and similar visual editors** - The frontmatter header may fail to collapse or render, remaining visible as a text block above the document body.

In all cases the underlying file content is intact. Nothing is lost.

**To repair:**

1. **Open the raw YAML.** In Obsidian switch to Source mode (top-right toggle, or `Ctrl/Cmd + E`). In Logseq switch to the document's source or code view. In VS Code or any plain text editor you are already there.
2. **Look for the common culprits:** a value containing a colon-space that is not quoted (`changes: key: detail` → wrap the whole value in double quotes), an em-dash in a field value, a missing closing `---` delimiter, or a stray tab where spaces are expected.
3. **Ask the agent to repair it.** Open a chat with filesystem access and say "repair the frontmatter on [page path]". The agent will read the page, compare it against the schema, and rewrite the frontmatter cleanly while preserving the body.
4. **Run `/wiki-lint`.** After any repair, lint confirms the page is schema-compliant and flags anything that still needs attention.

Note for Obsidian users: the orange Properties case (nested objects) is not actually broken - you only need to intervene if you want to edit that field interactively in the Properties panel. Otherwise leave it; the file is fine.

*Managed by wiki-config. To update or repair this file, run `/wiki-config`.*
