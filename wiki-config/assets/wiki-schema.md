---
# Wiki Page Schema v1.0
# Read by all wiki skills on boot to know what fields pages use.
# Managed by wiki-config - run /wiki-config to view, edit, reset, or repair.

schema_version: "1.0"

# Mandatory fields - every wiki page must have these
mandatory_fields:
  title: string
  version: string
  date: date          # YYYY-MM-DD
  changes: string     # quoted, <80 chars, no paths
  page_type: enum     # see enums below

# Conditional fields - written when context applies
conditional_fields:
  updated: date                    # any skill touch, YYYY-MM-DD
  status: enum                     # default: active
  description: string              # ~200 chars, quoted
  crystallize_count: integer       # wiki-crystallize only, starts at 1
  source: list                     # always a list, even for single origin
  reliability: enum                # only meaningful when source present

# Valid enum values
enums:
  page_type:
    - knowledge        # synthesised knowledge page
    - reference        # factual reference material
    - survey           # overview/survey of a topic area
    - research-note    # in-progress investigation
    - domain-home      # domain anchor page
    - overview         # multi-topic overview
    - home             # top-level hub
    - log              # append-only operational log
    - index            # catalogue/index page
    - config           # configuration document
  status:
    - active           # current, canonical
    - stub             # placeholder, needs expansion
    - artefact         # kept as historical record
    - archived         # superseded but retained
    - snapshot         # point-in-time capture
  reliability:
    - high             # primary source or well-established
    - medium           # secondary source or partial confidence
    - low              # speculative, hearsay, or unverified
---

# Wiki Schema

Skills read this file on boot alongside `wiki-config.md`. It defines the frontmatter structure all wiki pages follow.

## How skills use this

When writing any page, skills consult this schema to know:
- Which fields are mandatory
- Which conditional fields apply given context
- Which enum values are valid for constrained fields

Skills that write `page_type` choose the appropriate enum value based on what the page is for, and ask the user when unclear.

## Editing this file

Run `/wiki-config` and choose "View/edit schema". The interactive flow warns before changes that can cause erratic skill behaviour.

**Generally safe:** Adding new conditional fields, extending enum value lists with new categories that fit your workflow.

**Risky:** Removing mandatory fields, renaming fields, removing enum values that existing pages use, changing field types.

The bundled default (what `/wiki-config` restores) is the minimum coherent set across all six wiki skills.

## Field semantics

This file defines **structure**. For **meaning** - when and why each field is written, what counts as a valid value, how skills interpret each one - see `LLM Wiki Skills - Conventions.md` in your vault (or the conventions reference bundled with each skill).

## Recovery

If this file is missing, malformed, or unexpectedly modified:

```
/wiki-config
```

Wiki-config bundles the default schema and can restore or repair it. The operational skills (wiki-ingest, wiki-lint, wiki-integrate, wiki-crystallize, wiki-query) refuse to proceed without a valid schema and will redirect you here.
