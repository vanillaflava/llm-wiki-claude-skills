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

### Mandatory fields

Every page a skill writes must include all five:

| Field | Rule |
|---|---|
| `title:` | Must match the H1 heading exactly. Quote if the value contains a colon. |
| `version:` | Start at 1.0. Increment minor (1.1) for additions, major (2.0) for structural rewrites. |
| `date:` | Creation date. Set once at page creation, never modified by skills on subsequent writes. YYYY-MM-DD only, no timestamp. |
| `changes:` | One sentence. Never a file path. Never an em-dash. Always quoted. |
| `page_type:` | Controls how skills handle the page. Use values from the `page_type` enum above. |

### Conditional fields

Written by skills when context applies. Not required on every page.

| Field | Written by | When | Key rules |
|---|---|---|---|
| `updated:` | Any wiki skill on touch | Every skill write | YYYY-MM-DD; captures last skill contact date. On page creation, equals `date:` - this is expected and correct. Writing `updated:` alone does not require a version bump. wiki-lint staleness threshold: 90 days; exempt status artefact, snapshot, archived. |
| `status:` | wiki-ingest on create; any skill on state change | Every page | Default `active`; write `stub` if body is minimal at create; skills never downgrade existing value |
| `description:` | wiki-ingest, wiki-crystallize | Every page | ~200 chars, quoted; LLM bookkeeping - not human territory |
| `crystallize_count:` | wiki-crystallize only | Every crystallize write | Integer. Read current value and add 1; if field is absent, write 1. Counts deliberate crystallize events only - not general edits. |
| `source:` | wiki-ingest on create | When page has an ingested origin | Always a list, even for single origin: `["ingested/clippings/foo.md"]` |
| `reliability:` | wiki-ingest on create | Only when `source:` is present | Use minimum value when multiple sources contributed; omit entirely when no source |

### Write discipline

**Frontmatter is skill territory.** Skills overwrite known fields without asking. Humans who edit these values should expect them to be overwritten on the next skill touch.

**Body is human territory.** Skills append and add sections; they never delete human-written prose. Unrecognised frontmatter fields are preserved unchanged.

### Key anti-patterns

| Anti-pattern | Fix |
|---|---|
| `changes: key: value` (colon-space unquoted) | Quote the `changes:` value |
| Em-dash (`—`) in any field value or title | Use ` - ` (space-hyphen-space) |
| File path in `changes:` | Put paths in body `## Sources` section; `changes:` is a short description only |

For full conventions, page type guidance, and tips, see `[[Wiki Help]]` in your vault (deployed by `/wiki-config` on first run).

## Recovery

If this file is missing, malformed, or unexpectedly modified:

```
/wiki-config
```

Wiki-config bundles the default schema and can restore or repair it. The operational skills (wiki-ingest, wiki-lint, wiki-integrate, wiki-crystallize, wiki-query) refuse to proceed without a valid schema and will redirect you here.
