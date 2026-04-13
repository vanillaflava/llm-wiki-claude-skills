---
name: wiki-crystallize
description: Distil a long chat thread, accumulated research session, or working document into a structured wiki page capturing the current state of knowledge on the topic. Run when you say /wiki-crystallize, when a chat is getting heavy, or before closing a long-running thread. The chat is the scaffolding; the wiki page is the artefact. Requires filesystem read/write access.
compatibility: Works with any markdown knowledge base supporting [[wikilinks]] — Obsidian, Logseq, Foam, Dendron, or a plain folder of .md files.
metadata:
  version: "2.3"
---

# Wiki Crystallize

Distils ephemeral conversation into durable, structured wiki knowledge.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` by filename across accessible directories. Do not assume a path. If found, read it and extract:
- `vault_root` — absolute path to the wiki root
- `blacklist` — paths never to write to
- `index_excludes` — paths excluded from index.md
- `log_format` — format string for log entries

**If `wiki-config.md` is not found — run init:**
1. Ask the user: *"I couldn't find wiki-config.md. Please provide the path to your knowledge base root — the top-level folder containing your notes."*
2. Write `wiki-config.md` at that path using the template below
3. If `index.md` does not exist, create it:
   ```
   # Wiki Index
   *Catalogue of all wiki pages. Updated by wiki-ingest and wiki-integrate.*
   ```
4. If `log.md` does not exist, create it:
   ```
   # Wiki Operation Log
   *Append-only. New entries go at the top.*
   ```
5. Confirm what was created and proceed

**Config template — write frontmatter block first, then body:**

```
---
vault_root: /absolute/path/to/your/knowledge-base

blacklist:
  - Repositories\

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

log_format: "## [YYYY-MM-DD] {type} | {subject}"
---
```

Then write this body after the closing `---`:

```markdown
## Configuration Guide

**vault_root** — Absolute path to your knowledge base root.
Windows example: `C:\Users\yourname\Documents\MyWiki`
macOS example: `/Users/yourname/Documents/MyWiki`

**blacklist** — Paths where wiki page creation is forbidden (relative to vault_root).
Add Git repos, source code folders, or any area that should never receive wiki writes.

**index_excludes** — Paths excluded from index.md tracking.
`raw\` and `ingested\` are always excluded — source files are not wiki pages.
`archive\` keeps deprecated pages out of the live catalogue.

**ingested_folder / ingested_subdirs** — Archive configuration for wiki-ingest.
Carried here so the config is valid regardless of which skill runs first and triggers init.

**log_format** — Do not change without updating all wiki skills.
```


---

## Capability Requirements

This skill requires **filesystem read and write access**. If unavailable on the current surface (web, mobile), offer to produce the wiki page as an artifact that the user can save manually.

---

## Purpose

Chats accumulate context that is costly to carry, hit-and-miss to search, and increasingly dominated by superseded content. Crystallize compresses the durable signal — decisions, findings, patterns, open questions — into a structured wiki page that orients any future session faster and more reliably than re-reading the thread.

The name is precise: raw carbon accumulates; crystallize turns it into something dense, structured, and permanent. The chat is the scaffolding. The wiki page is the artefact.

**Keep:** Decisions made, patterns established, lessons learned, open questions, current understanding, key findings, agreed frameworks.
**Discard:** Exploratory back-and-forth, dead ends, process chat, superseded drafts, corrections already incorporated.

Crystallize serves at multiple stages — it is not exclusively a thread-ending operation:

| Level | When to use | Closing posture |
|---|---|---|
| **Single session** | End of a working session; established context still valuable | Pause, not close — continue or return |
| **Topic thread** | After several sessions; thread getting heavy | Recommend fresh start for next session |
| **Whole chat** | Thread is exhausted or explicitly being archived | Strong close — wiki page is the successor |

---

## Input

Provide one of:
1. A summary of the thread (your own words)
2. A pasted transcript or export of the conversation
3. A description of what was accomplished and what was decided

If none are provided, ask the user to describe what this crystallization should capture before proceeding.

---

## Workflow

### Step 1 — Read wiki-config.md and Overview.md

Read configuration (vault root, blacklist). Then read `Overview.md` to understand the current state of knowledge — crystallize adds to the wiki, not duplicates it.

### Step 2 — Analyse the input

Identify durable knowledge (decisions, findings, patterns, open questions, current best understanding) and discard conversational scaffolding (exploratory back-and-forth, superseded drafts, process steps, corrections already incorporated).

### Step 3 — Find an existing home for this knowledge

Before creating anything new, check whether the knowledge already has a home in the wiki.

Read `index.md` and scan for pages that could serve as the hub for this topic:
- A page whose title matches the topic of the crystallization
- A domain home page that covers this ground
- A "Current State" or summary page for this area

For each candidate, read the page briefly to assess fit.

**If a strong match is found:** Prefer enriching and updating it. Proceed to Step 4a.

**If multiple candidates exist, or it's unclear:** Ask the user: *"I found these existing pages that might be the right home for this: [list]. Should I update one of these, or create a new page?"* Wait for their answer.

**If no existing page fits:** Confirm with the user before creating a new page, unless the session was lightweight and the content is clearly self-contained.

### Step 4a — Update an existing page

Read the full current content of the target page. Integrate the crystallized knowledge:
- Add new decisions, findings, or open questions to the appropriate sections
- Update any sections that are now out of date
- Do not duplicate content already present — synthesise and merge
- Update the frontmatter version, date, and changes fields

### Step 4b — Write a new page

```markdown
---
title: Topic — Current State
version: 1.0
date: YYYY-MM-DD
changes: Crystallized from [source chat / session description]
---

# Topic — Current State

## What Was Established
[2-4 sentences: the core finding or current understanding]

## Key Decisions
[Decisions made, with brief rationale]

## Current Understanding
[The substantive content — structured as appropriate: prose, tables, lists]

## Open Questions
[What remains unresolved. Be specific.]

## Related Pages
[[Related Page 1]], [[Related Page 2]]
```

Choose the most specific placement using the folder structure from `index.md`. Do not place in blacklisted paths, raw/, archive/, or Projects/. Confirm with the user if the location is not obvious.

### Step 5 — Update index.md (new pages only)

If a new page was created in Step 4b, add an entry in the correct section of `index.md`. If an existing page was updated in Step 4a, no index change is needed.

### Step 6 — Update Overview.md if warranted

Update the relevant domain section in Overview.md only if the crystallization represents a significant shift: a major decision reached, a research phase completed, or a new domain established. Do not update for incremental additions.

### Step 7 — Append to log.md

```
## [YYYY-MM-DD] crystallize | <Topic or Chat Name>
Updated [[Existing Page]] with new findings. [or: Created [[New Page]].]
[Brief note on what was captured. Updated Overview.md: yes/no.]
```

### Step 8 — Choose a closing posture

Read two signals to decide which closing to deliver:

**Signal 1 — stated intent.** Did the user invoke crystallize with explicit whole-chat language ("before I close this", "wrapping up this thread", "archiving")? If yes → strong close.

**Signal 2 — thread weight.** Check log.md for prior crystallize entries on this same topic. Multiple prior entries = the thread has been accumulating across sessions and a fresh start is warranted → strong close. A first crystallize on a topic, or a session that was focused and light → session wrap.

Note: context window fullness cannot be precisely measured from within the skill. This is a qualitative judgment based on observable signals.

**Session wrap** — thread still has value, established context worth keeping:
> "Captured. [[Topic]] reflects today's decisions and findings. Continue this chat or start a new one — both work. If you continue, the context here still has value; if you start fresh, read [[Topic]] first."

**Strong close** — thread is heavy or explicitly being archived:
> "This chat can now be closed. [[Topic]] captures everything worth keeping. Start your next session by reading that page — it will orient a fresh context faster than carrying this thread forward."

---

## Key Rules

1. **Distil, do not transcribe** — the wiki page should be significantly shorter and more structured than the source
2. **Keep the durable, discard the scaffolding** — be ruthless about what compounds value
3. **Frontmatter on every page** — title, version, date, changes: "Crystallized from [source]"
4. **Never write to blacklisted paths**
5. **Update Overview.md sparingly** — only for genuinely significant knowledge shifts
6. **Wikilinks make it searchable** — always add links to related pages

---

## Cloud-Synced Vaults

Vaults stored in cloud sync services may have files not locally downloaded, appearing as zero-byte placeholders. If a file read returns empty unexpectedly, flag it as a possible sync issue and ask the user to confirm before retrying. Do not treat a zero-byte file as a successfully processed empty file.
