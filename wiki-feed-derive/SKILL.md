---
name: wiki-feed-derive
description: Derive machine-readable concentrated notes from pages/ into lightrag-feed/ subdirectories (glossary, hubs, contrasts, crystallized). Run when the user says "feedに書き出して", "用語辞典エントリを作って", "対比ノートをfeedに", "ハブを作って", or /wiki-feed-derive. Reads an existing pages/ entry, applies the appropriate feed template (templates/glossary-entry.md, hub.md, contrast.md, crystallize.md), and writes a dictionary-style definition-explicit version with enumerated related terms. NOT a plain copy — the feed version is rewritten specifically for entity/relation extraction by LightRAG. Preserves source attribution via `source:` frontmatter. Logs to log.md. Requires wiki-config.md and templates/ to exist.
---

<!-- version: 1.0 -->

# Wiki Feed Derive

Derives **machine-readable concentrated notes** from `pages/` into `lightrag-feed/` subdirectories. This is the Vault-side half of the "Derived Bridge" pattern connecting an Obsidian Wiki to a LightRAG instance. LightRAG reads from `lightrag-feed/` via symlink and builds its knowledge graph from these derivatives rather than raw sources. Because derivatives are written in **the user's own vocabulary** (editorial prose concentrated into dictionary form), LightRAG's extracted entities and relations align with how the user actually thinks about their domain.

This skill does NOT copy `pages/` content verbatim. It rewrites into a different format optimized for machine extraction.

---

## Configuration

**Finding your config:** Search for `wiki-config.md` across accessible directories. Extract:

- `wiki_root` — absolute path to the Vault root
- `blacklist` — ignore these paths when reading sources
- `index_excludes` — confirms `lightrag-feed/` is expected to exist

**If `wiki-config.md` is not found:** Stop and ask the user for the Vault path. Do not assume.

**Expected directory structure** (created by prior Vault setup):

```
{wiki_root}/
├── pages/{concepts,entities,summaries,answers}/
├── lightrag-feed/{glossary,hubs,contrasts,crystallized}/
├── lightrag-feed/overview-snapshot.md
├── templates/{glossary-entry,hub,contrast,crystallize}.md
└── log.md
```

If `templates/` or `lightrag-feed/` subdirectories are missing, stop and ask the user to run Vault initialization.

---

## Feed Types

| Type | Source Pages | Output Path | Template |
|---|---|---|---|
| glossary | 1 × pages/concepts/ | lightrag-feed/glossary/{term}.md | templates/glossary-entry.md |
| hub | N × pages/concepts/ | lightrag-feed/hubs/{domain}.md | templates/hub.md |
| contrast | 2 × pages/ | lightrag-feed/contrasts/{A}-vs-{B}.md | templates/contrast.md |
| crystallized | conversation session | lightrag-feed/crystallized/{title}-{date}.md | templates/crystallize.md |
| overview-snapshot | overview.md | lightrag-feed/overview-snapshot.md (overwrite) | self-contained (see below) |

---

## Execution Flow

### 1. Identify the derivation type

Infer from the user's instruction:

- "用語辞典" / "glossary" / "定義明確に" → **glossary**
- "俯瞰" / "hub" / "○○分野の地図" / "全景" → **hub**
- "違い" / "対比" / "A vs B" → **contrast**
- "対話をまとめて" / "セッションを残して" / "crystallize" → **crystallized**
- "overview更新" / "スナップショット" → **overview-snapshot**

If ambiguous, ask the user which type.

### 2. Read the source page(s)

- **glossary**: 1 source page (usually pages/concepts/)
- **contrast**: 2 source pages (A and B) — confirm both with user
- **hub**: multiple source pages related to a domain — ask user which pages or which domain
- **crystallized**: current conversation session context
- **overview-snapshot**: read overview.md directly

Read each source fully. Note frontmatter (created, updated, tags, sources).

### 3. Check if target already exists

```
target_path = {wiki_root}/lightrag-feed/{type}/{filename}.md
if exists(target_path):
    ask user: "既存の feed 頁 ({filename}) があります。上書きしますか？ それとも新規名で作りますか？"
    if overwrite:
        preserve `created` date, set `updated` to today
    if new_name:
        suggest {filename}-v2 or ask for name
```

### 4. Apply the template

Read `templates/{type}.md`. Replace all `{...}` tokens with actual content extracted from source page(s).

**Critical**: Do NOT paste source page prose into the template. Rewrite:

- **Dictionary style**: "{Term}は{X}が{Y}する{Z}のことである。"
- **Explicit definitions**: start with "{Term}とは〜" construction
- **Enumerated relations**: list `[[related-term]]: {relation-kind}` lines where relation-kind is one of: is-a, uses, contrasts-with, part-of, instance-of, precedes, etc.
- **Concise**: aim for 30-60% of the source page's length
- **Preserve key terminology**: use the same terms the user uses in pages/

### 5. Set frontmatter

```yaml
---
type: glossary | hub | contrast | crystallized
created: {today if new, else preserve existing}
updated: {today}
source: {source path(s)}
derived_from: [list of source page paths]
tags: {inherit from source}
---
```

For `overview-snapshot`:

```yaml
---
type: overview-snapshot
created: {preserve if exists, else today}
updated: {today}
source: overview.md
snapshot_of: {wiki_root}
---
```

### 6. Write the file

Write to `target_path`. If overwriting, replace atomically.

### 7. Update log.md

Append to `{wiki_root}/log.md`:

```markdown
## [YYYY-MM-DD] feed-derive | {type}/{filename}
```

### 8. Report to user

Brief summary:

- **File**: created/updated {path}
- **Source**: {source pages listed}
- **Compression**: {source chars} → {feed chars} ({ratio}%)
- **Next suggestions**: 1-3 candidates based on the just-derived content (e.g., "次は {関連概念} の glossary を derive しますか？")

---

## overview-snapshot Special Format

Different from others because the source is overview.md (user-written narrative) and the output overwrites a single fixed file.

Use this format directly (no separate template needed):

```markdown
---
type: overview-snapshot
created: {date}
updated: {today}
source: overview.md
snapshot_of: {wiki_root}
---

# {Vault basename} Overview Snapshot

## Domain scope
{1-paragraph summary of what this Vault covers, derived from overview.md}

## Key concepts (wikilinked)
- [[{concept 1}]]
- [[{concept 2}]]
...

## Key entities
- [[{entity 1}]]
...

## Recent activity summary
{Summarize the last 5-10 entries from log.md tail}

## Structural outline
- pages/concepts/: {count} entries
- pages/entities/: {count} entries
- pages/summaries/: {count} entries
- pages/answers/: {count} entries
- lightrag-feed/glossary/: {count} entries
```

---

## Guardrails

1. **NEVER** copy source prose verbatim. Always rewrite in dictionary style.
2. **NEVER** write to `pages/` from this skill — only to `lightrag-feed/`
3. **NEVER** delete existing feed files — only create or update
4. **PRESERVE** source attribution in frontmatter (`source:` and `derived_from:` fields)
5. **ASK** before overwriting existing feed files
6. **STOP** if `templates/` is missing — do not write without a template
7. **RESPECT** `blacklist` from wiki-config.md
8. **DO NOT** auto-invoke LightRAG re-ingest. After writing, suggest but wait for user confirmation.

---

## Usage Examples

### Example 1: Create glossary entry

```
User: RAGの用語辞典エントリをfeedに作って

Skill flow:
  1. Find wiki-config.md → wiki_root
  2. Read pages/concepts/RAG.md
  3. Read templates/glossary-entry.md
  4. Rewrite into dictionary form:
     "RAG（Retrieval-Augmented Generation）とは、LLMが外部知識を検索して回答に組み込む技術のことである。"
     + enumerate related terms: [[ベクトル検索]]: uses, [[知識グラフ]]: alternative-to, [[LLM]]: enhances
  5. Write lightrag-feed/glossary/RAG.md with frontmatter
  6. Append to log.md
  7. Report: "lightrag-feed/glossary/RAG.md 作成完了（420 chars, 元頁1,280 charsから33%に圧縮）。
              次は対比候補 RAG vs LightRAG を contrast で作りますか？"
```

### Example 2: Create contrast note

```
User: LightRAG と 通常RAG の contrast ノート作って

Skill flow:
  1. Ask: "pages/entities/LightRAG.md と pages/concepts/RAG.md を対比元としてよいですか？"
  2. Read both + templates/contrast.md
  3. Extract dimensions of contrast:
     - データ構造: LightRAG=KG, RAG=ベクトルのみ
     - 検索方式: LightRAG=mix mode, RAG=top-k
     - スケール特性: ...
  4. Write lightrag-feed/contrasts/LightRAG-vs-RAG.md
  5. Log + report
```

### Example 3: Overview snapshot

```
User: overview-snapshot を更新して

Skill flow:
  1. Read overview.md
  2. Read tail(log.md, 20) for recent activity
  3. Count entries in each pages/ subdirectory
  4. Write (overwrite) lightrag-feed/overview-snapshot.md
  5. Log
  6. Report: "overview-snapshot 再生成完了。LightRAG に再 ingest させますか？
              MCPの lightrag-research サーバーの lightrag_insert ツールを呼べます。"
```

### Example 4: Ambiguous request

```
User: これをfeedに

Skill flow:
  1. Ask: "feed化したい内容を確認させてください。以下のどれですか？
     A) 今開いているページをglossaryに
     B) 直近の対話をcrystallizedに
     C) 特定の対比ノートを新規作成
     D) overview-snapshot 再生成"
  2. Proceed based on user's choice
```

---

## Post-conditions and Next Steps

After successful run, propose to the user:

1. **LightRAG へ再 ingest**: "LightRAG にこのfeed を読み込ませますか？ MCPの lightrag-research サーバー経由で lightrag_insert / lightrag_query を呼べます"（ただし自動実行はしない、確認を得る）
2. **関連する次の濃縮候補**: just-derived 内容から派生する 1-3 件の候補を提案
3. **wiki-lint の提案**: feed 乖離チェックが必要そうなら "wiki-lint で整合性チェックしますか？"

**Do not** auto-invoke follow-up actions. Always wait for user confirmation (A = minimum manual operation principle).
