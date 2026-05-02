# llm-wiki-skills

[![Release](https://img.shields.io/github/v/release/vanillaflava/llm-wiki-skills?style=flat-square)](https://github.com/vanillaflava/llm-wiki-skills/releases/latest) [![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](https://github.com/vanillaflava/llm-wiki-skills/blob/master/LICENSE) [![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-brightgreen?style=flat-square)](https://agentskills.io/specification) [![Claude](https://img.shields.io/badge/Claude-D97757?style=flat-square&logo=claude&logoColor=white)](https://claude.ai) [![Markdown](https://img.shields.io/badge/Markdown-000000?style=flat-square&logo=markdown&logoColor=white)](https://commonmark.org) [![Obsidian](https://img.shields.io/badge/Obsidian-%23483699?style=flat-square&logo=obsidian&logoColor=white)](https://obsidian.md) [![MCP](https://img.shields.io/badge/MCP-filesystem-blue?style=flat-square)](https://modelcontextprotocol.io)

Six agent skills for building and maintaining a personal knowledge wiki using Markdown files, wikilinks, and filesystem access. Developed on Claude with GUI install support; works on Claude Code, Gemini CLI, Codex CLI, and GitHub Copilot.

## What this is

A set of agent skills for incrementally building and maintaining a structured, interlinked knowledge base on your local filesystem. You add sources; the agent reads, synthesises, and files them into wiki pages. You ask questions; the agent reads the wiki and answers with citations. The wiki compounds over time - each source and each good question makes it richer.

This is my personal implementation of the LLM wiki pattern (see below). It is not a product. It has no backend, no API, no cloud sync (unless you provide one), and no dependencies beyond your agent and filesystem access. Everything lives in Markdown files on your disk. If you can read `.md` files, you can use this in Obsidian, Logseq, VS Code, iA Writer, or a plain folder. 

I built this on Claude, but the underlying skill instructions follow the [agentskills.io open standard](https://agentskills.io/specification) and work on Gemini CLI, Claude Code, OpenAI Codex CLI, and GitHub Copilot. The `.skill` packaging and Claude Desktop GUI install are Claude-specific; other agents install skills as folders. See [Installation](#installation).

The skills are (hopefully) honest about what they are: structured instructions to a language model, not code. They are good at their job but not infallible. The wiki is only as good as the sources you feed it and the questions you ask. 

## Background

This implementation grows out of my own history of working with Obsidian and learning LLM capabilities (and their restrictions!) over time. I started with local markdown files, downloading them, struggling with diffs, and generally being too lazy to do the bookkeeping needed to keep everything connected, coherent, and above all fresh. MCPs changed that game significantly for me, once I was able to add file system access. Now I could use LLM agents to read, write, and maintain notes directly on disk, without intermediaries. Editing was way faster now, bootstrapping new agents became a lot easier, but the discipline required to not keep drowning in hundreds semi-connected markdown files (plus their legacy versions) was still a drag. 

What changed all this for me, was [Andrej Karpathy's llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), which describes using an LLM to maintain a persistent, compounding wiki rather than doing one-shot RAG retrieval (and I had barely learned what that even was, before I read about the pattern). The core insight, that the wiki is a compiled artefact that gets richer with every source and query, rather than a pile of documents to retrieve from (which I had up to this point) is wholly Karpathy's and is well worth reading in full. And perhaps even just implementing yourself. I had never pushed anything to GitHub before this little project. I learned so much from just trying this out. It's a little shocking to me that a) I could just do this and b) how incredibly well this pattern actually works. 

What I am adding to it is just my own preferences in workflow (while keeping the skills generic): packaging, configuration, a source-tracing convention, and a few architectural choices that emerged from real sustained use.

## What I figured out along the way

| Aspect             | The pattern (intentionally open-ended)            | This implementation                                                                                                                                                                                            |
| ------------------ | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Raw sources        | Immutable permanent store                         | Flat queue; moving to `ingested/` is the commit - filesystem is the state                                                                                                                                      |
| State tracking     | `log.md` records what's processed                 | Presence in `ingested/` is the record; `log.md` is audit-only (no reading of what has already been ingested, re-ingestion accidental or not works well enough)                                                 |
| Query compounding  | "Good answers can be filed back" (noted as a tip) | `/crystallize` as a first-class operation - primary mechanism for accumulating chat knowledge, every long session, every major change, archiving a heavy chat -> it compounds and enriches the next bootstrap. |
| Session bootstrap  | Schema document                                   | The wiki and domain home pages are the bootstrap - the wiki is the persistent context                                                                                                                          |
| Configuration      | Suggestions for structure                         | `wiki-config.md` read by all six skills; configure once                                                                                                                                                        |
| Source attribution | Not specified                                     | `source:` frontmatter field traces each wiki page to its origin in `ingested/`. `reliability:` (high/medium/low, agent-assessed at ingest time) captures source trust. Pages from lower-confidence sources get a `## Pending Review` section flagging specific claims to corroborate. |
| Page structure     | Not specified                                     | 13 vault-side templates (one per page type) deployed by `wiki-config` on init. Writing skills read the matching template when creating new pages. User-editable without touching skill code; changes take effect immediately for new pages. |

The most satisfying thing that emerged from looking at other implementations and attempting to build this myself was `/crystallize`. Karpathy observes that query answers are valuable and shouldn't disappear into chat history. This isn't about a fully automated self-maintaining archival machine either. He stresses that staying involved is important, asking questions and learning from the answers is the whole point. The LLM is the previously absent bookkeeper, the flagger of stale content, conflicting information and orphaned treasures in a sea of markdown. I had already been prompting summaries of heavy chats, and cycling them out for a fresh instance. Now distilling a long working session into a wiki page became the primary way my projects accumulate knowledge from conversations. Domain home pages (think wiki hubs) serve as structured session bootstraps - previously that context lived in manually written summaries. Now the wiki itself is the persistent context across sessions.

One thing worth knowing before you see it for the first time: the `## Pending Review` section that appears on some ingested pages is not an error. It is a quality signal written by the ingest skill when a page is created from a single lower-confidence source - it flags specific claims that would benefit from a stronger source. When you have one, drop it in `raw/` and re-ingest; the skill will update the trust assessment and remove the section if the gap is resolved.

The other thing I worked out, which took longer than it should have, is how to make session cycling actually comfortable. Agents lose their working memory. The context window fills, things slow down, and you start a fresh chat. Without the wiki, you spend the next twenty minutes re-explaining where you left off.

What I found is that there are three layers that work together to make this nearly seamless, and the wiki is only one of them.

The first layer is whatever your platform calls permanent instructions - Personal Preferences in Claude, system prompt elsewhere. These fire on every conversation automatically. They carry who you are: working style, preferences, hard constraints, tool habits. They do not change by domain.

The second layer is per-project or per-domain instructions - Claude calls these Project Instructions, and similar mechanisms exist on other providers. You attach them to a project and they inject automatically into every chat in that project. I use these to tell the agent which domain it's working in and to direct it to read the relevant domain home before doing anything else.

The third layer is the domain home itself. The agent reads it at session start and knows: what is being worked on, what was decided last session, what is open. No briefing required.

Together: the permanent instructions carry who you are. The project instructions say which domain and point to the home. The domain home carries where you left off. A fresh agent with an empty context window is equipped in the first minute.

This is, I think, the most underappreciated part of the pattern as applied to ongoing work rather than a one-shot task. The wiki is the memory that survives session cycling. The instruction layers are what put a fresh agent on the right track before it even reads the wiki. Both matter, and they are designed to work together.

## The six skills

| Skill              | What it does                                                                                                                                                                                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `wiki-config`      | Sets up, validates, and reconfigures your wiki. On first run: deploys `wiki-config.md`, `wiki-schema.md`, `wiki-help.md`, `Home.md`, `Overview.md`, `index.md`, `log.md`, the `raw/` and `ingested/` folder structure, and 13 page templates. Ongoing: guided schema editor, template management (view, repair, reset), and repair flows for anything that goes missing. Start here.                                                                                            |
| `wiki-ingest`      | Processes files from `raw/` into wiki pages; classifies, synthesises, cross-references, and moves each file to `ingested/` as an atomic commit                                                                                                                              |
| `wiki-query`       | Answers questions by reading the wiki index and relevant pages; cites sources; can file valuable answers as new wiki pages                                                                                                                                                  |
| `wiki-lint`        | Health-checks the wiki: broken links, orphaned pages, missing index entries, unreferenced sources in `ingested/`                                                                                                                                                            |
| `wiki-integrate`   | Weaves a new or updated page into the knowledge graph by adding backlinks and index entries                                                                                                                                                                                 |
| `wiki-crystallize` | The session management mechanism. Distils durable signal from a working session - decisions, findings, open questions - into a structured wiki page. Biased toward updating existing domain homes and `Overview.md` rather than creating new pages. Close every heavy chat with this; the next session picks up from the wiki page, not from scrollback. |

Use them individually or together. A minimal setup is `wiki-config` (once) plus `wiki-ingest` and `wiki-query`. Add the others as your wiki grows.

## Requirements

**An agent with filesystem access.** These skills read and write files on your local disk. How you get filesystem access depends on your platform:

- **Claude Desktop** has no native filesystem access. You need a filesystem MCP - [Desktop Commander](https://github.com/wonderwhy-er/DesktopCommanderMCP) or the [official Anthropic filesystem MCP](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem), both installable via **Customize → Connectors**.
- **Claude Code, Gemini CLI, OpenAI Codex CLI, and GitHub Copilot** have filesystem access built in. No MCP required.
- **Claude.ai web and mobile** cannot reach files on your disk. The skills work with severe limitations and are not recommended for primary use.

**A Markdown wiki directory.** Any folder of `.md` files with a consistent wikilink convention works. The skills were developed in Obsidian but the files themselves are plain Markdown - nothing is Obsidian-specific.

## Installation

### Claude Desktop

**Download from the [latest release](https://github.com/vanillaflava/llm-wiki-skills/releases/latest)** and upload each `.skill` file under **Customize → Skills**:

| Skill | Download |
|---|---|
| wiki-config | [wiki-config.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-config.skill) |
| wiki-ingest | [wiki-ingest.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-ingest.skill) |
| wiki-query | [wiki-query.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-query.skill) |
| wiki-lint | [wiki-lint.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-lint.skill) |
| wiki-integrate | [wiki-integrate.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-integrate.skill) |
| wiki-crystallize | [wiki-crystallize.skill](https://github.com/vanillaflava/llm-wiki-skills/releases/latest/download/wiki-crystallize.skill) |

Skills activate automatically when relevant, or via slash commands (`/wiki-config`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`, `/wiki-integrate`, `/wiki-crystallize`). After a few cycles, the LLM learns to suggest them organically.

If you are comfortable with Git, you can also clone the repo and upload the `.skill` files directly from the repo root - they are kept in sync with each release.

### Other agents (Claude Code, Gemini CLI, Codex CLI, GitHub Copilot)

Filesystem access is built in on these platforms - no MCP required. Copy the skill folder you want into your agent's skills directory:

```
Claude Code:     ~/.claude/skills/<skill-name>/
Codex CLI:       ~/.codex/skills/<skill-name>/
Gemini CLI:      ~/.gemini/skills/<skill-name>/
GitHub Copilot:  configure via chat.agentSkillsLocations in VS Code
```

Clone the repo and copy:

```bash
git clone https://github.com/vanillaflava/llm-wiki-skills.git
cp -r llm-wiki-skills/wiki-query ~/.gemini/skills/wiki-query
```

Copy the **entire skill folder**, not just `SKILL.md` - skills bundle reference files in `references/` and templates in `assets/` that they read at runtime.

Activate with `/wiki-config` or your agent's equivalent slash command. Conversational triggering varies by agent; slash commands are reliable across all tested platforms.

There is no auto-update mechanism. Watch [GitHub Releases](https://github.com/vanillaflava/llm-wiki-skills/releases) and re-copy the skill folder when new versions ship.

## Getting started

**1. Set up your wiki**

Easiest way: run `/wiki-config`. It asks where your wiki should live, then creates the full scaffold in one guided step - config, schema, a user guide (`wiki-help.md`), starter navigation pages (`Home.md`, `Overview.md`), an index, an operation log, the `raw/` and `ingested/` folder structure, and 13 page templates (one per page type). Takes about a minute. You'll then want to replace the placeholder values in `wiki-config.md`'s `blacklist` field with the actual folders you want to keep off-limits to wiki writes.

`wiki-help.md` lands in your wiki root alongside the config files. It covers fields, page types, write discipline, naming conventions, customisation, and tips - the reference you reach for when something is unexpected. Open it in your notes app; it is a plain Markdown file.

**The directory containing `wiki-config.md` is your wiki root.** Skills derive it at runtime from the file's location; you never write a path anywhere. If you relocate the wiki, move `wiki-config.md` with it and everything still works.

If `/wiki-config` is not installed, the other skills can help - each bundles the same templates as read-only references and will offer to get you set up when you run them. Manual setup works too if you prefer to work with files directly; the config and schema YAML are below, along with field explanations.

If you prefer to set things up by hand, create `wiki-config.md` at your intended wiki root with this content:

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

Path style note: examples use Windows backslash (`\`). On Mac or Linux, use forward slash (`/`) instead; YAML itself is agnostic.

Then add this body below the closing `---` (the skill generates this automatically, but it's useful to have when editing manually):

**Wiki root:** The directory containing `wiki-config.md` is the wiki root. The skills derive it from the config file's location; you do not write the path anywhere. If you move the wiki, move this file with it and nothing else changes. Your wiki root must be inside your filesystem tool's allowed scope or the skills cannot reach your notes.

**blacklist:** Paths where wiki page creation is forbidden, relative to wiki root. The placeholder values above MUST be replaced with real folder names before the wiki is useful. Add Git repos, private folders, source trees, or any area that should never receive wiki writes.

**index_excludes:** Paths excluded from `index.md` tracking. `raw\` and `ingested\` are always excluded; source files are not wiki pages.

**ingested_folder:** Folder where processed source files are archived, relative to wiki root. Must be in `index_excludes`; must not be in `blacklist`.

**ingested_subdirs:** Archival taxonomy within `ingested_folder`. The skill classifies each source and routes it to the appropriate subfolder. Adapt freely; these are just suggestions. `ingested/assets/` is always created for files that couldn't be read or extracted.

**templates_folder:** The folder where page templates live. Deployed and managed by `wiki-config` - one template file per page type. Edit templates directly in your notes app; changes take effect immediately for new pages.

**log_format:** Do not change without updating all wiki skills.

The `reliability:` field you will see in frontmatter is written by `wiki-ingest` at ingest time. Values are `high` (primary or authoritative source), `medium` (secondary source, reasonable confidence), or `low` (speculative or unverified). It is only written when `source:` is also present.

Then create `wiki-schema.md` alongside it. This file defines the frontmatter fields every wiki page uses and the valid values for enumerated fields. Skills read it on boot alongside `wiki-config.md` and consult it whenever they write a page.

```yaml
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
    - knowledge
    - reference
    - survey
    - domain-home
    - overview
    - log
    - index
    - config
    - longform
    - profile
    - established-patterns
    - note
  status:
    - active
    - stub
    - artefact
    - archived
    - snapshot
  reliability:
    - high
    - medium
    - low
---
```

The easiest way to get a correct `wiki-schema.md` is to copy it from any skill's `references/wiki-schema.md` - they all bundle an identical read-only reference copy. Running `/wiki-config` does this for you in one step, and also includes a guided schema editor if you later want to extend it.

The bundled defaults are the minimum coherent set. You can safely extend the schema - add conditional fields, extend enum value lists to fit your own categories. Removing mandatory fields or renaming existing ones isn't recommended; skills depend on those being where they expect.

The resulting directory structure looks like this:

```
your-wiki/
├── wiki-config.md     ← your settings
├── wiki-schema.md     ← frontmatter structure for all wiki pages
├── wiki-help.md       ← user guide (fields, types, tips) - open in your notes app
├── Home.md            ← navigation hub (skill workflow, links to major sections)
├── Overview.md        ← living synthesis across all domains (grows via /wiki-crystallize)
├── CLAUDE.md          ← tells the agent how your wiki works (see below)
├── index.md           ← page catalog (agent maintains this)
├── log.md             ← operation log, prepend-only (agent maintains this)
├── raw/               ← drop source files here
├── ingested/          ← agent archives processed files here
│   ├── clippings/
│   ├── documentation/
│   ├── articles/
│   └── notes/
└── templates/         ← 13 page templates, one per page type - edit freely
```

**2. Write a CLAUDE.md**

A short document that tells the agent: where things live, what conventions to follow, and which domain home pages to read for context. Start simple - one paragraph is enough. The wiki improves this document over time.

**3. Start putting it to work**

If you're starting fresh, drop a clipping, a research paper, or a document you've been meaning to read into `raw/`, say `/wiki-ingest`, and watch it become a wiki page. If you already have a notes collection or vault, point the agent at a domain you work in and let it help you set up a Domain Home - a hub page it can use to orient future sessions.

Some good early moves:
- **Ingest a small batch of sources** you've been meaning to read. The first time you query across them, the pattern starts to click.
- **Discuss your existing Domain Homes**, or set up a new one for a topic you care about. These become the bootstrap context for future sessions.
- **End a rich working session with `/wiki-crystallize`.** It distils the durable signal from a conversation into wiki pages, so the next session starts from something structured rather than from chat scrollback.

From here, co-work with your LLM: add sources, ask questions, refine pages. Your LLM will adapt to your personal pattern; use `CLAUDE.md` to nudge its behaviour to your preferences.

## Privacy

These skills are plain instructional language - no code, no scripts, no remote connections of their own. Other skills in the ecosystem may behave differently; always check what you install. But "no connections from the skill itself" does not mean private: **your wiki is not private from your LLM provider.** Every file the agent reads - source material you drop in `raw/`, wiki pages it synthesises, your `CLAUDE.md`, your index - is sent to Anthropic (or whichever provider you use) as part of the conversation. This is true even if everything lives on local disk. The only data boundary that matters is what the agent is allowed to access.

Before you start, it helps to think in two scopes:

```
Your machine
├── Everything else on your filesystem
└── Your knowledge space (Obsidian vault, markdown reader, notes folder, etc.)
    ├── Private/          ← medical, financial, personal - never expose
    ├── Sensitive/        ← work confidential, legal, credentials
    ├── Archive/          ← legacy notes you don't want an LLM near
    └── Agent Access/     ← your wiki root - the ONLY folder the agent needs
        ├── wiki-config.md
        ├── CLAUDE.md
        ├── raw/
        ├── ingested/
        └── ... wiki pages
```

Give the agent-accessible folder an obvious name - something like `Agent Access` makes the boundary legible to you when you're setting things up and when you revisit it later. Everything outside it should be unreachable by the agent. Everything inside it should be something you are comfortable sending to your LLM provider.

Scope your filesystem MCP to that folder and nothing wider. If you use Desktop Commander, its `allowedDirectories` setting is the right lever. Anthropic's own filesystem MCP scopes to whatever directories you configure at setup. Whatever MCP you choose, the scope should be the wiki root - not your knowledge space root, not your documents folder, not your home directory. Agents with native filesystem access, such as Claude Code, are not constrained by MCP-level controls and must be scoped separately through their own configuration.

The `blacklist` field in `wiki-config.md` is a separate and narrower control: it prevents the skills from _writing_ wiki pages to specific folders, but it does not prevent the agent from reading them. Do not rely on the blacklist as a privacy boundary - it is not one. If there are folders in your wiki root you do not want an LLM to read, move them outside the MCP's allowed scope entirely.

The filesystem MCP you use may also send data of its own. Desktop Commander, as an example, sends anonymised telemetry by default unless you disable it in its config. Inspect and understand the privacy behaviour of any MCP tool before connecting it to material you consider sensitive. A useful frame: if you would not be comfortable seeing certain files in your provider's next training run, make sure they are simply not accessible to the agent. Use a structure where the boundary is obvious and easy to remember.

What happens to your data once it reaches your provider depends on their privacy policy and your account settings. Claude's data handling is described at [anthropic.com/privacy](https://www.anthropic.com/privacy). If you use a different provider, check their policy before proceeding.

## TaskNotes (optional)

[tasknotes-skill](https://github.com/vanillaflava/tasknotes-skill) is a complementary skill for basic task management against a local Obsidian vault with the [TaskNotes ](https://tasknotes.dev/) plugin installed. I am not affiliated with that project, but their 'one note per task' principle works extremely well with the llm-wiki pattern and these skills. It was my first 'learning how to build agent skills project' (my complementary Claude skill is basic, the official plugin is anything but). Have a look if you are using Obsidian, and/or have no task management skill or MCP wired up. 

It is not required - any task management approach you prefer has the same effect, and there are many out there. The wiki skills have no dependency on it. The TaskNotes skill is noted here because it was designed to work alongside this system, and adds turning knowledge into action, planning and step-by-step processes. The wiki pattern works equally well with a folder of task files, a paper notebook, or any tool of your choice.

---

## License

MIT
