---
name: wiki-integrate
description: Weave a newly created or significantly updated wiki page into the knowledge graph. Adds it to index.md if missing, finds related pages by topic overlap, and adds backlinks in both directions. Use when you say /wiki-integrate, when the user says "nothing links to this page", "this page is an orphan", "connect this page to related pages", "add backlinks to this page", "add this page to the index", "weave this page into the wiki", when a new page has just been created directly in a chat, or when a page has been significantly revised and needs connecting. Lightweight; does not rewrite content, only adds links and index entries. Requires filesystem read/write access.
metadata:
  version: "3.11"
---

# Wiki Integrate

Connects a new or updated wiki page into the knowledge graph by adding backlinks and an index entry.

---

## Config Discovery

**Every invocation starts here.** Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime. Pages this skill writes to follow the structure in `wiki-schema.md` - both files need to be present.

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

This skill requires **filesystem read and write access**. If unavailable, inform the user and stop.

---

## When to Use This Skill

Run wiki-integrate when:
- A new wiki page has been created directly in a chat (not via wiki-ingest)
- An existing page has been significantly revised and may have new connection opportunities
- Pages exist in the vault that are orphaned from the knowledge graph

**wiki-ingest** processes raw source material into wiki pages and links them automatically. **wiki-integrate** links pages that already exist as wiki content. Do not use one as a substitute for the other.

---

## Input

Wiki-integrate operates on a **target page**. Provide the path to the newly created or updated page. If no path is provided, ask the user which page to integrate before proceeding.

---

## Workflow

Config Discovery has already loaded `wiki-config.md` and `wiki-schema.md` into context. Do not re-read them; proceed from here assuming both are available.

### Step 1 - Read the target page

Read the full content of the target page. Identify the main topic, key concepts, existing `[[wikilinks]]`, and the frontmatter title.

**Page_type check:** Read the `page_type` field from the page's frontmatter.
- **Missing:** infer from the page's content and structure using the schema enum (already in context). Present the inferred type to the user and confirm before writing: *"This page doesn't have a `page_type`. Based on the content it looks like a `<type>` - does that match, or would you like a different type?"* Record the confirmed value; it will be written in Step 6 alongside `updated:`.
- **Present but appears outgrown:** if the page's current `page_type` no longer matches its actual scope (e.g. a `knowledge` page that has grown into a comparative survey, or a `stub` that is now substantive), offer a promotion: *"This page is typed as `<type>` but reads more like a `<better-type>` now. Want me to update `page_type` while I'm here?"* Never auto-promote; wait for explicit confirmation.

### Step 2 - Check the blacklist

Confirm the target page is not in a blacklisted path. If it is, stop; wiki skills cannot modify blacklisted paths.

### Step 3 - Check and update index.md

Read `index.md`. If the target page is not listed, add an entry in the correct section:
`- [[Page Title]]: One sentence description. \`relative/path/to/page.md\``

If already listed, verify the description is still accurate. Update if significantly stale.

### Step 4 - Find related pages

From the index catalogue, select 3-8 candidate pages with genuine topic overlap. Connections should be meaningful, not just keyword matches.

### Step 5 - Read candidate pages

Read each candidate in full. Confirm a genuine connection exists: shared concepts, one builds on the other, or a reader would benefit from knowing both. Discard candidates where no real connection exists.

### Step 6 - Add backlinks

For each confirmed pair, add links in both directions:
- In the target page: add `[[Candidate Page Title]]` where it fits naturally
- In the candidate page: add `[[Target Page Title]]` where it fits naturally

Write only the new wikilink; do not restructure or rewrite candidate page content. Check each candidate is not blacklisted before writing.

For every page written to (target and each candidate that received a link), also update the `updated:` frontmatter field to today's date. If a `page_type` value was inferred and confirmed in Step 1, write it to the target page's frontmatter now. Leave all other frontmatter fields unchanged.

### Step 7 - Prepend to log.md

Add the new entry at the top of log.md, below the header line, above all existing entries.

```
## [YYYY-MM-DD] integrate | <Target Page Title>
Added to index.md. Linked from: [[Page A]], [[Page B]]. Linked to: [[Page C]].
```

### Step 8 - Summarise

Report: whether the target was added to index.md, which pages now link to it, which pages it now links to, and any connections ruled out with brief reasoning.

---

## Key Rules

1. **Content is unchanged:** only adds wikilinks and index entries; never rewrites or restructures
2. **Never write to blacklisted paths**
3. **Links must be genuine:** only add links where a real connection exists
4. **Minimal footprint in candidate pages:** add only the new wikilink
5. **One target per run**
6. **Do not integrate archive/ or assets/ pages**

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.


