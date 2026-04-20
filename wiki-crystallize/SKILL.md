---
name: wiki-crystallize
description: Distil a long chat thread, accumulated research session, or working document into a structured wiki page capturing the current state of knowledge on the topic. Run when you say /wiki-crystallize, when a chat is getting heavy, or before closing a long-running thread. The chat is the scaffolding; the wiki page is the artefact. Requires filesystem read/write access.
---

<!-- version: 3.0 -->

# Wiki Crystallize

Distils ephemeral conversation into durable, structured wiki knowledge.

---

## Config

Wiki root is the directory containing `wiki-config.md`. Skills derive it at runtime.

**Step 1 - Identify filesystem scope.** Determine the root directory the filesystem tool has access to (for filesystem MCP: `allowedDirectories`; for Claude Code: session CWD; for other surfaces: the equivalent).

**Step 2 - Check scope sensibility.** If the scope root is a bare drive root (`C:\`, `D:\`, `/`), an OS root, or a user home (`C:\Users\X`, `/home/X`, `/Users/X`), do NOT search. Go to Step 4.

**Step 3 - Bounded search.** If scope is sensible, search recursively for `wiki-config.md`. First-match early termination, max 5 directory levels deep. If found: the directory containing it is the wiki root. Read `blacklist`, `index_excludes`, `ingested_folder`, `ingested_subdirs`, `log_format`. Proceed with the skill's operation.

**Step 4 - Ask once.** If not found, or scope was insensible, ask the user:

*"I couldn't find `wiki-config.md` in my accessible scope. Where is your wiki root - the folder containing your notes and `wiki-config.md`? It must be inside my filesystem scope."*

If the user provides a specific path: look for `wiki-config.md` at that path or within it (bounded search, max 5 levels deep). If found, use the containing directory as wiki root, read config, and proceed. If not found at the provided path, go to Step 5.

If the user is unclear, unsure, or declines to answer: go to Step 5.

**Step 5 - Unable to proceed.** The skill cannot meaningfully operate without a configured wiki. State the situation clearly, recommend the proper setup path, and point at the bundled references:

*"I can't operate without a configured wiki. The recommended setup path is the `/wiki-config` skill - install it (if not already available) and run it for interactive setup. If you'd rather set up manually, I have two bundled references in this skill: `references/setup-help.md` (manual setup walkthrough) and `references/wiki-config-template.md` (the default config content). I can read those and walk you through it if you want."*

After this recommendation, let the user drive. If they want to proceed with wiki-config, confirm and stop (they invoke it themselves). If they want manual setup, read the references and assist them - including writing files on their explicit direction. Do not automatically create the config or scaffolding without the user's clear instruction.


---

## Capability Requirements

This skill requires **filesystem read and write access**. If unavailable on the current surface (web, mobile), offer to produce the wiki page as an artifact that the user can save manually.

---

## Purpose

Crystallize compresses the durable signal from a chat (decisions, findings, patterns, open questions) into a structured wiki page that orients any future session faster than re-reading the thread.

Use it liberally - any time something meaningful has happened. It is not reserved for heavy or exhausted threads. It is the session management mechanism: the wiki is the memory that persists across chat cycles; chat history is scaffolding. Crystallize before retiring a chat, not as a cleanup step afterward.

**Keep:** Decisions made, patterns established, lessons learned, open questions, current understanding, key findings.
**Discard:** Exploratory back-and-forth, dead ends, process chat, superseded drafts.

| Level | When to use | Closing posture |
|---|---|---|
| **Single session** | End of a working session; context still valuable | Pause, not close |
| **Topic thread** | Thread getting heavy across sessions | Recommend fresh start |
| **Whole chat** | Thread exhausted or explicitly archived | Strong close |

---

## Input

Provide one of:
1. A summary of the thread (your own words)
2. A pasted transcript or export of the conversation
3. A description of what was accomplished and what was decided

If none are provided, ask the user to describe what this crystallization should capture before proceeding.

---

## Workflow

### Step 1 - Read wiki-config.md and Overview.md

Read configuration (blacklist). Then read `Overview.md` to understand the current state of knowledge; crystallize adds to the wiki, does not duplicate it.

### Step 2 - Analyse the input

Identify durable knowledge (decisions, findings, patterns, open questions, current best understanding) and discard conversational scaffolding (exploratory back-and-forth, superseded drafts, process steps, corrections already incorporated).

### Step 3 - Find an existing home for this knowledge

Before scanning candidates, check whether the topic belongs to an existing domain. If a Domain Home exists for that domain, it is the presumptive target - scan to confirm fit, not to find alternatives. Prefer updating a hub over creating a new file.

Read `index.md` and scan for:
- A Domain Home covering this ground (check first)
- A page whose title matches the topic
- A "Current State" or summary page for this area

**Strong match found:** update it. Proceed to Step 4a.

**Multiple candidates or unclear:** ask the user: *"I found these pages that might be the right home: [list]. Update one, or create new?"* Wait for their answer.

**No existing page fits:** confirm with the user before creating, unless the content is clearly self-contained.

**Key rule:** When in doubt, update a hub rather than create. New pages fragment the graph; rich hubs compound it.

### Step 4a - Update an existing page

Read the full current content of the target page. Integrate the crystallized knowledge:
- Add new decisions, findings, or open questions to the appropriate sections
- Update any sections that are now out of date
- Do not duplicate content already present; synthesise and merge
- Update the frontmatter version, date, and changes fields

### Step 4b - Write a new page

```markdown
---
title: Topic - Current State
version: 1.0
date: YYYY-MM-DD
changes: Crystallized from [source chat / session description]
---

# Topic - Current State

## What Was Established
[2-4 sentences: the core finding or current understanding]

## Key Decisions
[Decisions made, with brief rationale]

## Current Understanding
[The substantive content, structured as appropriate: prose, tables, lists]

## Open Questions
[What remains unresolved. Be specific.]

## Related Pages
[[Related Page 1]], [[Related Page 2]]
```

Choose the most specific placement using the folder structure from `index.md`. Do not place in blacklisted paths, raw/, archive/, or Projects/. Confirm with the user if the location is not obvious.

### Step 5 - Update index.md (new pages only)

If a new page was created in Step 4b, add an entry in the correct section of `index.md`. If an existing page was updated in Step 4a, no index change is needed.

### Step 6 - Update Overview.md if warranted

Update the relevant domain section in Overview.md only if the crystallization represents a significant shift: a major decision reached, a research phase completed, or a new domain established. Do not update for incremental additions.

### Step 7 - Append to log.md

```
## [YYYY-MM-DD] crystallize | <Topic or Chat Name>
Updated [[Existing Page]] with new findings. [or: Created [[New Page]].]
[Brief note on what was captured. Updated Overview.md: yes/no.]
```

### Step 8 - Choose a closing posture

Read two signals to decide which closing to deliver:

**Signal 1: stated intent.** Did the user invoke crystallize with explicit whole-chat language ("before I close this", "wrapping up this thread", "archiving")? If yes, deliver a strong close.

**Signal 2: thread weight.** Check log.md for prior crystallize entries on this same topic. Multiple prior entries mean the thread has been accumulating across sessions and a fresh start is warranted; deliver a strong close. A first crystallize on a topic, or a session that was focused and light, calls for a session wrap.

Note: context window fullness cannot be precisely measured from within the skill. This is a qualitative judgment based on observable signals.

**Session wrap** - thread still has value, established context worth keeping:
> "Captured. [[Topic]] reflects today's decisions and findings. Continue this chat or start a new one; both work. If you continue, the context here still has value; if you start fresh, read [[Topic]] first."

**Strong close** - thread is heavy or explicitly being archived:
> "This chat can now be closed. [[Topic]] captures everything worth keeping. Start your next session by reading that page; it will orient a fresh context faster than carrying this thread forward."

---

## Key Rules

1. **Distil, do not transcribe:** the wiki page should be significantly shorter and more structured than the source
2. **Keep the durable, discard the scaffolding:** be ruthless about what compounds value
3. **Frontmatter on every page:** title, version, date, changes: "Crystallized from [source]"
4. **Never write to blacklisted paths**
5. **Update Overview.md sparingly:** only for genuinely significant knowledge shifts
6. **Wikilinks make it searchable:** always add links to related pages

---

## What this skill does not do

This skill does wiki work: ingesting, synthesising, organising, and querying `.md` pages to compound knowledge over time. It does not modify tool or plugin settings, shell out to manipulate application state, or replicate behaviours that belong to whatever app the user reads their notes in. If a request cannot be satisfied by reading and writing `.md` files inside the wiki root, decline and explain why.


