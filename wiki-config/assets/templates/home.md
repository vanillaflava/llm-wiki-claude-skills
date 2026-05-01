---
title: "{{TITLE}}"
version: 1.0
date: "{{DATE}}"
updated: "{{DATE}}"
status: active
description: "{{DESCRIPTION}}"
changes: "v1.0 - Created by wiki-config on init"
page_type: domain-home
---

# {{TITLE}}

*Navigation hub for the wiki. This page orients you and any agent working in this vault: what the skills do, how to use them, and where to find things. Update it as your wiki grows.*

---

## The skill workflow

Six skills cover the full lifecycle. You invoke them by name in any chat with filesystem access.

| Skill | When to use it | What it does |
|---|---|---|
| `/wiki-config` | Setup, repair, or reconfigure | Creates the wiki scaffold, manages schema and templates, repairs missing files |
| `/wiki-ingest` | After dropping files into `raw/` | Reads source files, synthesises wiki pages, moves sources to `ingested/` |
| `/wiki-query` | To retrieve knowledge | Answers questions from the wiki with wikilink citations |
| `/wiki-integrate` | After creating a page directly | Weaves it into the link graph - adds backlinks, updates index |
| `/wiki-crystallize` | Before closing a heavy chat | Distils the session into a wiki page or domain home |
| `/wiki-lint` | Periodic health check | Surfaces broken links, orphaned pages, stale pages, schema issues |

**The basic loop:** drop sources into `raw/` → `/wiki-ingest` → knowledge accumulates → `/wiki-crystallize` to distil sessions → `/wiki-query` to retrieve → `/wiki-lint` occasionally to keep the graph healthy.

---

## Key pages

| Page | What it is |
|---|---|
| [[Overview]] | Living synthesis of what you currently know across all domains. The vault-level crystallize target. Start here for orientation. |
| [[index]] | Catalogue of all wiki pages. Maintained automatically by wiki-ingest and wiki-integrate. Use for lookup. |
| [[log]] | Append-only audit trail of all skill operations. New entries at the top. |
| [[Wiki Help]] | Field conventions, write discipline, naming rules, page types, tips. Read this if you're not sure how something works. |

---

## Your domains

*Add a link and one-line description for each domain home as your wiki grows. Domain homes are the anchor pages for each knowledge area - they are what an agent reads to orient itself before working in a domain.*

| Domain | Home page |
|---|---|
| [Domain name] | [[Domain - Home]] |

---

## How knowledge compounds

1. **Source material** lands in `raw/` - articles, PDFs, notes, clippings
2. **Wiki-ingest** synthesises them into wiki pages and files the originals in `ingested/`
3. **Sessions** with agents produce new understanding - `/wiki-crystallize` at the end distils it
4. **Domain homes** accumulate current state across sessions; a fresh agent reads one and knows where things stand
5. **Overview.md** is the vault-level synthesis - updated by `/wiki-crystallize` when knowledge shifts significantly across domains

The wiki is the memory that persists when chat history does not.

---

## Related

[[Overview]] | [[index]] | [[log]] | [[Wiki Help]]
