---
name: wiki-query-hybrid
description: Answer questions using a Wiki-first, LightRAG-as-backup hybrid retrieval strategy. Run when the user asks for in-depth or cross-cutting information ("深く", "横断的", "詳しく"), or when standard wiki-query produced a shallow answer, or /wiki-query-hybrid. First searches pages/ via wiki-query logic, assesses completeness, then optionally calls the LightRAG MCP tool `lightrag_query` (from server `lightrag-research`) in mix mode to supplement. Merges both answers, cites all sources (Wiki wikilinks + LightRAG feed paths), and offers to crystallize the result into pages/answers/. Requires wiki-config.md. Uses lightrag-research MCP when available, falls back to Wiki-only gracefully when unavailable.
---

<!-- version: 1.0 -->

# Wiki Query Hybrid

Answers questions using a **Wiki-first, LightRAG-as-backup** retrieval strategy. This is the query-side half of the "Derived Bridge" pattern. Wikipedia editing metaphor: the Wiki pages are the primary, editorially-curated source; LightRAG is a specialized index that can surface relations the Wiki editor didn't explicitly encode.

---

## Configuration

**Finding your config:** Search for `wiki-config.md`. Extract `wiki_root` and `blacklist`.

**LightRAG MCP availability:** Check if the MCP server `lightrag-research` is connected and exposes the `lightrag_query` tool. If not available, fall back to Wiki-only mode transparently (do not fail).

**Note on MCP server configuration:**

- Server name: `lightrag-research` (configured in Claude Desktop or `.mcp.json`)
- Tools exposed: `lightrag_query`, `lightrag_insert`, `lightrag_health`
- Retrieval modes for `lightrag_query`: `local`, `global`, `hybrid`, `naive`, `mix`
- Parameters: `query` (required string), `mode` (default `hybrid`, enum of 5 modes)

---

## Execution Flow

### Phase 1: Wiki-side retrieval

1. Read `{wiki_root}/index.md` to see the available pages catalog
2. Identify candidate pages matching the query by:
   - Keyword match in title / tags / content
   - Wikilink graph proximity (pages linked from/to the query topic)
   - Recency (prefer recent pages when tied)
3. Read up to 5 candidate pages fully
4. Draft a preliminary answer from `pages/` content alone
5. **Self-assess completeness** on:
   - **Coverage**: does the answer address all parts of the question? (score 0-100%)
   - **Depth**: is each part supported by at least one Wiki citation?
   - **Confidence**: are there parts where pages/ gave only vague hints?

### Phase 2: Decide whether to consult LightRAG

**Call LightRAG if any of these apply:**

- Self-assessment completeness < 80%
- Question contains markers: "深く" / "横断的" / "詳しく" / "比較" / "関連して"
- User explicitly asked for hybrid / cross-reference search

**Skip LightRAG if:**

- Wiki answer is complete and confident (completeness ≥ 80%)
- LightRAG MCP unavailable (record this fact, proceed Wiki-only with transparency note)
- User indicated Wiki-only preference

### Phase 3: LightRAG consultation (when invoked)

1. Call MCP tool: `lightrag_query` from server `lightrag-research`
   - Parameters:
     - `query`: the original question (possibly reformulated for clarity)
     - `mode`: `mix` (combines KG + vector retrieval — best for cross-cutting questions)
   - Alternative modes to consider:
     - `local`: when the question focuses on a specific entity
     - `global`: when the question is about broad themes
     - `hybrid`: balanced (default)
     - `naive`: pure vector search fallback
2. Parse the response:
   - Extracted entities
   - Highlighted relations
   - Source chunks (paths to lightrag-feed/ files)
3. Cross-check: which extracted entities correspond to `pages/` entries? Which don't?
4. If LightRAG returns few or vague results, try `hybrid` mode as a second attempt

### Phase 4: Answer composition

Structure the response as:

```markdown
## Answer

{Concise direct answer, 2-3 paragraphs}

## Wiki からの理解

{Points from pages/, each with [[wikilink]] citations}

## LightRAG からの補完  <!-- only if LightRAG was consulted and returned useful content -->

{Points that Wiki alone did not give, with feed/ source attribution}

## 矛盾する観点  <!-- only if any contradictions found -->

> [!warning]
> {Where Wiki and LightRAG disagree, state both views without arbitrating}

## Sources

- Wiki: [[page1]], [[page2]], [[page3]]
- LightRAG feed: lightrag-feed/glossary/X.md, lightrag-feed/contrasts/A-vs-B.md
```

### Phase 5: Offer crystallization

At the end of the answer, ask:

> "この回答を `pages/answers/` に保存しますか？"

If yes:

- Invoke wiki-crystallize behavior (or do it inline with templates/answer.md)
- Frontmatter includes `used_lightrag: true` if LightRAG was consulted
- Filename: `{question-kebab-case}-YYYY-MM-DD.md` or user-suggested name
- Update index.md
- Append to log.md

---

## Graceful Degradation

### Case: LightRAG MCP unavailable

```
Detection: lightrag_query tool call fails (ConnectionError, Timeout, or MCP server not responding)
Behavior:
  - Complete Phase 1 + Phase 4 only
  - Skip LightRAG sections in answer
Transparency note (always include):
  "Note: LightRAG (lightrag-research MCP) に到達できないため、Wiki のみで回答しています。
  LightRAGサーバー (port 9622) を起動してから再実行すると、より横断的な回答が得られる可能性があります。"
```

### Case: LightRAG returns empty / low-quality

```
Detection:
  - No entities extracted
  - Response text below 50 characters
  - "no information found" equivalent
Behavior:
  - Include a note in answer: "LightRAG からは有意な追加情報なし"
  - Proceed with Wiki-only answer
  - Suggest: "feed がまだ薄い可能性があります。wiki-feed-derive で関連用語を濃縮してみませんか？"
```

### Case: Wiki has no relevant pages

```
Detection: after reading index.md and searching, 0-1 pages match, none substantive
Behavior:
  - Do NOT fabricate an answer
  - Do NOT rely solely on LightRAG (feed likely empty for this topic)
  - Suggest: "pages/ に関連情報が少ないようです。raw/ に関連ソースを投入してから wiki-ingest すると、
    より実のある回答ができます。"
```

---

## Guardrails

1. **NEVER** fabricate citations. Every `[[wikilink]]` must resolve to an existing file, every `lightrag-feed/` path must exist.
2. **ALWAYS** cite sources when stating facts — no uncredited claims.
3. **NEVER** auto-save the answer without asking. Always confirm with user before writing to `pages/answers/`.
4. **NEVER** trust LightRAG output blindly — cross-reference with Wiki.
5. **PRESERVE** contradictions with `> [!warning]` callout. Do not arbitrate between sources.
6. **STOP** if `wiki-config.md` not found. Ask for Vault path.
7. **FALL BACK** gracefully when LightRAG unavailable — don't fail the whole query.
8. **DO NOT** trigger LightRAG re-ingest from within this skill. Querying only.

---

## Usage Examples

### Example 1: Cross-cutting deep question

```
User: LightRAGと通常RAGの違いを、少量データで弱い理由も含めて深く

Skill flow:
  Phase 1: Read index.md
    → pages/concepts/RAG.md and pages/entities/LightRAG.md exist
    → Both have basic content but sparse on scale issues
  Phase 2: Completeness self-assessment ~50%, "深く" keyword present
    → consult LightRAG
  Phase 3: Call lightrag_query(query="LightRAG vs traditional RAG, scale issues",
                                mode="mix")
    → Returns entities: [RAG, LightRAG, KG density, community summary]
    → Relations: [LightRAG uses KG, KG needs entities, entities need density]
    → Source chunks: lightrag-feed/glossary/RAG.md,
                      lightrag-feed/contrasts/LightRAG-vs-RAG.md
  Phase 4: Compose answer
    - Wiki-derived: basic architectural difference (pages references)
    - LightRAG-surfaced: KG density bottleneck, community summary requirements,
      critical scale threshold
    - Sources: Wiki [[RAG]], [[LightRAG]]
              LightRAG lightrag-feed/glossary/RAG.md + contrasts/LightRAG-vs-RAG.md
  Phase 5: Ask: "この回答を pages/answers/ に保存しますか？"
  User: yes
  Skill: writes pages/answers/LightRAGが少量データで弱い理由-YYYY-MM-DD.md
         updates index.md
         appends to log.md with "query-hybrid" type
```

### Example 2: Wiki-sufficient factual question

```
User: LightRAG のデフォルトストレージは何？

Skill flow:
  Phase 1: Read pages/entities/LightRAG.md
    → directly answers (NetworkX + JsonKV + NanoVectorDB)
  Phase 2: Completeness 95%, Wiki-only sufficient
    → skip LightRAG
  Phase 3: (skipped)
  Phase 4: Short answer with [[LightRAG]] citation
  Phase 5: Ask crystallize — user likely declines for factual one-liner
```

### Example 3: LightRAG unavailable

```
User: 深く調べて: RAGとKnowledge Graphの関係

Skill flow:
  Phase 1: Read pages/ → partial material
  Phase 2: Completeness ~60%, "深く" marker → attempt LightRAG
  Phase 3: lightrag_query call → ConnectionError
    → Fall back to Wiki-only
  Phase 4: Answer with Wiki sources + transparency note:
    "Note: LightRAG (lightrag-research MCP) に到達できないため、Wiki のみで回答。
    `lightrag-server` の起動後に再実行をお勧めします。"
```

### Example 4: Empty Wiki fallback

```
User: 量子コンピューティングについて

Skill flow:
  Phase 1: Read index.md → no relevant pages
  Phase 2: LightRAG feed also unlikely to have content
  Phase 3: Optionally try LightRAG, expect empty response
  Phase 4: Instead of fabricating, respond:
    "pages/ にも lightrag-feed/ にも関連情報がありません。
    raw/papers/ や raw/articles/ に量子コンピューティング関連ソースを投入してから
    wiki-ingest すると、この問いに答えられるようになります。"
```

---

## Post-conditions

After successful run:

- **If answer saved** → suggest: "wiki-integrate でバックリンクを整理しますか？"
- **If LightRAG was consulted** → consider whether feed needs update:
  "今回 LightRAG が浮上させた {X} について、pages/ にはまだ頁がありません。
  ingest 候補として raw/ に関連ソース投入を提案しますか？"
- **If feed gaps detected** → suggest wiki-feed-derive for missing terms

Do not auto-invoke follow-ups. Always ask (A = minimum manual principle).
