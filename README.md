# llm-wiki-claude-skills

Five Claude agent skills for building and maintaining a personal knowledge wiki using Markdown files, wikilinks, and filesystem access.

## What this is

A set of installable skills for Claude that let an LLM agent incrementally build and maintain a structured, interlinked knowledge base on your local filesystem. You add sources; the agent reads, synthesises, and files them into wiki pages. You ask questions; the agent reads the wiki and answers with citations. The wiki compounds over time - each source and each good question makes it richer.

This is my personal implementation of the LLM wiki pattern (see below). It is not a product. It has no backend, no API, no cloud sync (unless you provide one), and no dependencies beyond Claude and filesystem access. Everything lives in Markdown files on your disk. If you can read `.md` files, you can use this in Obsidian, Logseq, VS Code, iA Writer, or a plain folder. 

I targeted Claude because that's the platform I'm learning most with - the `.skill` packaging is Claude-specific. The underlying instructions are plain markdown though, and the pattern adapts easily to other models; Codex uses `AGENTS.md` for the same purpose, and the instructions would need only minor adaptation.

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
| Configuration      | Suggestions for structure                         | `wiki-config.md` read by all five skills; configure once                                                                                                                                                       |
| Source attribution | Not specified                                     | `changes:` frontmatter field traces each wiki page back to its source in `ingested/` (still a work in process, as many sources accumulate over time)                                                           |

The most satisfying thing that emerged from looking at other implementations and attempting to build this myself was `/crystallize`. Karpathy observes that query answers are valuable and shouldn't disappear into chat history. This isn't about a fully automated self-maintaining archival machine either. He stresses that staying involved is important, asking questions and learning from the answers is the whole point. The LLM is the previously absent bookkeeper, the flagger of stale content, conflicting information and orphaned treasures in a sea of markdown. I had already been prompting summaries of heavy chats, and cycling them out for a fresh instance. Now distilling a long working session into a wiki page became the primary way my projects accumulate knowledge from conversations.  Domain home pages (think -> wiki hubs) serve as structured session bootstraps - previously that context lived in manually written summaries. Now the wiki itself is the persistent context across sessions.

## The five skills

| Skill              | What it does                                                                                                                                                                                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `wiki-ingest`      | Processes files from `raw/` into wiki pages; classifies, synthesises, cross-references, and moves each file to `ingested/` as an atomic commit                                                                                                                              |
| `wiki-query`       | Answers questions by reading the wiki index and relevant pages; cites sources; can file valuable answers as new wiki pages                                                                                                                                                  |
| `wiki-lint`        | Health-checks the wiki: broken links, orphaned pages, missing index entries, unreferenced sources in `ingested/`                                                                                                                                                            |
| `wiki-integrate`   | Weaves a new or updated page into the knowledge graph by adding backlinks and index entries                                                                                                                                                                                 |
| `wiki-crystallize` | Distils a working session or accumulated conversation into a structured wiki page (biased toward updating existing hubs and the overview.md, as the top level summary of everything that is known). You can teach your LLM to adopt a structure that serves your workflow.  |

Use them individually or together. A minimal setup is `wiki-ingest` and `wiki-query`. Add the others as your wiki grows.

## Requirements

**Claude with filesystem access.** These skills read and write files on your local disk. They require an MCP that provides filesystem access - [Desktop Commander](https://github.com/wonderwhy-er/DesktopCommanderMCP) is what this was developed and tested with; the [official Anthropic filesystem MCP](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) works equally well. Claude Code has native access built in. Without filesystem access, the skills are severely limited (to markdown files created by the LLM you need to manually copy to your local wiki.) 

**A Markdown wiki directory.** Any folder of `.md` files with a consistent wikilink convention works. The skills were developed in Obsidian but the files themselves are plain Markdown - nothing is Obsidian-specific.

**Claude Desktop** (recommended) or Claude Code. The skills have been tested on Claude Desktop with Desktop Commander via MCP. Claude Code users will need to adapt the CLAUDE.md bootstrap convention. Mobile and web surfaces work with the limitations mentioned.

## Installation

1. Download the `.skill` files from this repository
2. In Claude Desktop, go to **Customize → Skills**
3. Upload each `.skill` file

That's it. The skills appear in your skill list and activate automatically when relevant via conversation (the header visible in the install GUI hints at what kind of things trigger it), or via slash commands (`/wiki-ingest`, `/wiki-query`, `/wiki-lint`, `/wiki-integrate`, `/wiki-crystallize`). After a few cycles, the LLM learns to suggest them organically. 

## Getting started

**1. (optional) Set up before first use**

The skills will search for `wiki-config.md` anywhere inside the MCP-accessible scope and offer to create one on first use if it's not found. **The directory containing `wiki-config.md` is the wiki root.** If you'd rather set it up yourself first, create a file called `wiki-config.md` at your intended wiki root with this content:

```yaml
---
blacklist:
  - Repositories\
  - PrivateFolder\

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

Then add this body below the closing `---` (the skills generate this automatically, but it's useful to have when editing manually):

**Wiki root:** The directory containing `wiki-config.md` is the wiki root. The skills derive it from the config file's location; you do not write the path anywhere. If you move the wiki, move this file with it and nothing else changes. Your wiki root must be inside your MCP's allowed scope or the skills cannot reach your notes.

**blacklist:** Paths where wiki page creation is forbidden, relative to wiki root. Add Git repos, source code folders, or any area that should never receive wiki writes.

**index_excludes:** Paths excluded from `index.md` tracking. `raw\` and `ingested\` are always excluded; source files are not wiki pages.

**ingested_folder:** Folder where processed source files are archived, relative to wiki root. Must be in `index_excludes`; must not be in `blacklist`.

**ingested_subdirs:** Archival taxonomy within `ingested_folder`. The skill classifies each source and routes it to the appropriate subfolder. Adapt freely; these are just suggestions. `ingested/assets/` is always created for files that couldn't be read or extracted.

**log_format:** Do not change without updating all wiki skills.

The resulting directory structure looks like this:

```
your-wiki/
├── wiki-config.md
├── CLAUDE.md          ← tells the agent how your wiki works (see below)
├── index.md           ← page catalog (agent maintains this)
├── log.md             ← audit trail (agent maintains this)
├── raw/               ← drop source files here
└── ingested/          ← agent archives processed files here
    ├── clippings/
    ├── documentation/
    ├── articles/
    └── notes/
```

**2. Write a CLAUDE.md**

A short document that tells the agent: where things live, what conventions to follow, and which domain home pages to read for context. Start simple - one paragraph is enough. The wiki improves this document over time.

**3. Drop a file in raw/ and say `/wiki-ingest`**

The agent will read the source, synthesise a wiki page, update the index, and move the file to `ingested/`. That's the basic loop. 

From here you can co-work with your LLM on the wiki to add new sources, ask question, and refine them; and when you reach milestones, use /wiki-crystallize to use the LLM's build-up context to synthesise and update all pages this new concept is connected to (and correct stale references, wikilinks etc.). Your LLM will adapt to your personal pattern and you can use CLAUDE.md to nudge its behaviour to your preferences.

## Privacy

These skills are plain instructional language - no code, no scripts, no remote connections of their own. Other skills in the ecosystem may behave differently; always check what you install. But "no connections from the skill itself" does not mean private: **your wiki is not private from your LLM provider.** Every file the agent reads - source material you drop in `raw/`, wiki pages it synthesises, your `CLAUDE.md`, your index - is sent to Anthropic (or whichever provider you use) as part of the conversation. This is true even if everything lives on local disk. The only data boundary that matters is what the agent is allowed to access.

Before you start, it helps to think in two scopes:

```
Your machine
├── Everything else on your filesystem
└── Your knowledge space (Obsidian vault, markdown reader, notes folder, etc.)
    ├── Private/          ← medical, financial, personal — never expose
    ├── Sensitive/        ← work confidential, legal, credentials
    ├── Archive/          ← legacy notes you don't want an LLM near
    └── Agent Access/     ← your wiki root — the ONLY folder the agent needs
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

[tasknotes-claude-skill](https://github.com/vanillaflava/tasknotes-claude-skill) is a complementary skill for basic task management against a local Obsidian vault with the [TaskNotes ](https://tasknotes.dev/) plugin installed. I am not affiliated with that project, but their 'one note per task' principle works extremely well with the llm-wiki pattern and these skills. It was my first 'learning how to build agent skills project' (my complementary Claude skill is basic, the official plugin is anything but). Have a look if you are using Obsidian, and/or have no task management skill or MCP wired up. 

It is not required - any task management approach you prefer has the same effect, and there are many out there. The wiki skills have no dependency on it. The TaskNotes skill is noted here because it was designed to work alongside this system, and adds turning knowledge into action, planning and step-by-step processes. The wiki pattern works equally well with a folder of task files, a paper notebook, or any tool of your choice.

## License

MIT
