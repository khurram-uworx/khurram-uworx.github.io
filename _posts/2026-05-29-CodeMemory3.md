---
title: "SQL, Extensibility, and the Democratization of Infrastructure"
date: 2026-05-29
series:
  - "Memori"
  - "LLMs as Equializers"
tags:
  - MCP
  - Semantic Search
  - Code Intelligence
  - SQL
  - Context Engineering
comments: true
---

We started with a graph database. It made sense on paper — codebases are graphs of symbols and dependencies, so query them through a graph traversal API. We built a custom query layer over the dependency graph: `GetDownstreamDependents(symbol, depth)`, `FindCycles(component)`, `TracePath(source, target)`. Clean abstractions. Clear semantics.

LLMs could not use them.

Every query required understanding the graph schema — what nodes exist, what edges connect them, how traversal direction maps to dependency direction. We wrote documentation, examples, prompt templates. The agents still struggled. They hallucinated node names, guessed edge types, and wrote traversal patterns that hit the wrong direction. A tool that required a human to explain "here is how you query this" before every session was not working.

We replaced the entire query layer with SQL. `SELECT * FROM RelationshipRecord WHERE SourceSymbolId = 'x'`. No custom DSL, no traversal semantics to learn, no schema to internalize. SQL is the one query language every developer knows and every LLM was trained on. If you can describe what you want in English, an LLM can translate it to SQL — and get it right on the first attempt.

The graph database is gone. The SQL engine is the most-used tool in the system. That pivot taught us something about how to design infrastructure for the age of coding agents.

## Why SQL is the universal query DSL for agents

The insight is not that SQL is a good language — it is that SQL is a *known* language. The distribution of training data for frontier LLMs includes millions of SQL examples: every Stack Overflow post, every GitHub repository, every tutorial, every production schema migration. An LLM does not need to learn SQL. It already knows it.

When an agent calls CodeMemory's `sql_query` tool, it can write:

```sql
SELECT Name, Kind FROM SymbolRecord WHERE FilePath LIKE '%auth%' LIMIT 10
```

```sql
SELECT FilePath, COUNT(*) AS cnt FROM SymbolRecord GROUP BY FilePath HAVING cnt > 10 ORDER BY cnt DESC
```

```sql
SELECT s.Name, COUNT(*) AS dependents
FROM SymbolRecord s
JOIN RelationshipRecord r ON s.Id = r.TargetSymbolId
WHERE r.RelationshipType = 'References'
GROUP BY s.Name
ORDER BY dependents DESC
LIMIT 5
```

```sql
-- Vector search: find chunks semantically related to query text
SELECT FilePath, Content, __score
FROM ChunkRecord
WHERE Content LIKE '%auth%'
ORDER BY Similarity DESC
LIMIT 10
```

Every one of these is valid SQL. Every one was written by an LLM, not a human. The agent formulates the query based on the table schema it discovered via `tools/list`, composes the SQL, and executes it against the in-memory store — all without a human in the loop.

**The graph database failed because it added cognitive overhead to the agent's decision loop.** The agent had to think about *how to query* instead of *what to query*. SQL removes that overhead. The agent focuses on the question, and SQL translates.

## How the SQL engine works

CodeMemory's SQL query engine lives in `CodeMemory.Mcp.SqlQuery` and runs against the same `InMemoryVectorStore` that stores symbols, chunks, and relationships. The pipeline:

```
SQL text
    ↓
SqlParserCS → AST (abstract syntax tree)
    ↓
Query type detection:
    ├─ Single-table SELECT
    ├─ Multi-table (JOINs, derived tables)
    ├─ Set operations (UNION, INTERSECT, EXCEPT)
    └─ CTE (Common Table Expression)
    ↓
For single-table:
    ├─ SqlExpressionBuilder.BuildFilter<TRecord>(where)
    │   → System.Linq.Expressions.Expression<Func<TRecord, bool>>
    ├─ collection.GetAsync(filter, top) → IAsyncEnumerable<TRecord>
    └─ Client-side: GROUP BY, aggregates, HAVING, ORDER BY, LIMIT, projection
    ↓
For multi-table:
    ├─ evaluateGroupAsync() per TableWithJoins
    ├─ extractJoinInfo → (JoinType, ON condition)
    ├─ cross-join groups → mergeWithJoinType
    └─ Client-side: same post-processing
    ↓
For vector search (ORDER BY Similarity DESC):
    ├─ Extract text from Content LIKE '%pattern%'
    ├─ NgramEmbeddingGenerator → embedding vector
    ├─ collection.SearchAsync(vector, top, options)
    └─ Return rows with __score (0-1)
    ↓
SqlQueryResult(success, rowCount, executionTimeMs, columns, rows)
```

The key abstraction is `SqlExpressionBuilder.BuildFilter` — it takes a SQL `WHERE` clause AST and produces a `System.Linq.Expressions.Expression<Func<TRecord, bool>>`. This is how SQL conditions become LINQ filters over the in-memory vector store:

```csharp
// SQL: WHERE Kind = 'Class' AND FilePath LIKE '%auth%'
// becomes:
record => record.Kind == "Class" && record.FilePath.Contains("auth")
```

The expression tree is compiled and passed to `VectorStoreCollection.GetAsync(filter)`, which applies it as a server-side predicate (or client-side filter for in-memory stores, but the contract is the same).

### Vector search in SQL

The `ORDER BY Similarity DESC` clause is special. When detected, the engine switches from exact filtering to vector search:

1. Extracts the search text from any `WHERE Content LIKE '%pattern%'` clause
2. Embeds it using the registered `IEmbeddingGenerator`
3. Calls `collection.SearchAsync(vector, top, options)` on the vector store
4. Returns results with a synthetic `__score` column (0–1 cosine similarity)

This means agents can combine semantic search with SQL filtering in a single query:

```sql
SELECT FilePath, Content, LineStart, LineEnd, __score
FROM ChunkRecord
WHERE Content LIKE '%payment%' AND Language = 'CSharp'
ORDER BY Similarity DESC
LIMIT 5
```

Returns chunks related to "payment" language, filtered to C# only, ranked by semantic similarity. One query, two retrieval paradigms.

### The "this was insanity months ago" story

A SQL parser and execution engine — even a `SELECT`-only one — is a compiler engineering project. You need a tokenizer, an AST parser, a type system, join planning, aggregate evaluation, and projection logic. The `SqlQueryService` in CodeMemory is 2,200 lines. It handles CTEs, derived tables, set operations, GROUP BY with HAVING, multi-table JOINs (INNER, LEFT, RIGHT, FULL, CROSS), vector search, and client-side post-processing.

We did not hire a database engineer. We did not have a SQL expert on the team. We had an LLM that was trained on SQLite's source code, PostgreSQL's documentation, and every SQL tutorial ever written. We described what we wanted — a `SELECT`-only SQL engine over `IVectorStore` collections — and iterated on implementation with the LLM guiding the parsing mechanics, edge cases, and optimization.

The graph database took weeks and still failed. The SQL engine took a weekend to prototype and has been in production ever since. The difference was not team skill. It was the match between the query language and the LLM's training distribution.

## Extensibility as first principle

The SQL engine is one example of a broader design philosophy. Every major capability in CodeMemory is replaceable through a standard .NET interface:

### Storage providers

The `IStorageService` interface abstracts the storage layer. The MCP (STDIO) host uses `InMemoryVectorStore` — all data in process, ephemeral, zero infrastructure. The ASP.NET host uses `HybridStorageService` — symbols and relationships in EF Core relational tables, chunks in a `VectorStore` provider.

```json
"Storage": {
    "Provider": "sqlite"   // or "inmemory", "pgvector", "sqlserver"
}
```

| Provider | Vector Store | Relational Store | Best for |
|---|---|---|---|
| `inmemory` | `InMemoryVectorStore` | None | CI, agent sessions, ephemeral |
| `sqlite` | `SqliteVectorStore` | EF Core SQLite | Development, single-server |
| `pgvector` | `PostgresVectorStore` | EF Core Npgsql | Production, multi-service |
| `sqlserver` | `SqlServerVectorStore` | EF Core SQL Server | Enterprise, existing infra |

Switching providers changes the connection string. Application code does not change — the same tools, the same queries, the same semantics. The SQL engine on the MCP path uses `SqlParserCS` → LINQ over `InMemoryVectorStore`. The ASP.NET path uses `AspNetSqlQueryTool` forwarding to EF Core for relational queries on symbols and relationships. Both expose the same `sql_query` MCP tool with the same SQL syntax.

### Embedding backends

The `IEmbeddingGenerator<string, Embedding<float>>` interface (from `Microsoft.Extensions.AI.Abstractions`) accepts four providers:

```json
"Embedding": {
    "Provider": "ngram"   // or "onnx", "ollama"
}
```

- **n-gram** (default, offline, deterministic) — character n-gram hashing with random projection. No model download, no API keys, no network calls. Consistent across processes and sessions.
- **ONNX** — BERT-based embedding via ONNX Runtime (bge-micro-v2). Requires model download but runs fully local.
- **Ollama** — delegates to a running Ollama server. Useful when you already have one.

The n-gram default is worth calling out specifically. It uses Memori's `NgramEmbeddingGenerator` — a deterministic character n-gram hash into 1536 dimensions, L2-normalized with `TensorPrimitives.Norm`. It is not *semantic* in the ML sense — synonyms do not match, short queries return weak results — but it is *consistent*. The same codebase produces the same embeddings on every run, on every machine, without any external dependency.

The Memori series covered the math and implementation in the Storage, Recall, and the Middleware Pipeline post, links below. The relevant point for CodeMemory is that this was ML-adjacent infrastructure that would have required a research paper deep-dive and a Python prototyping loop a few years ago. Today, an LLM explains the hashing strategy, implements the projection matrix, and verifies the output distribution — and you do it in C# with `System.Numerics.Tensors`.

### Multi-language parsing

`ILanguageParser` routes to Roslyn (C# — the actual compiler) or Tree-sitter (everything else). Adding a new language means adding a Tree-sitter grammar reference and a `LanguageDetector` mapping entry. The `RoslynSymbolExtractor` and `TreeSitterSymbolExtractor` share the same `ISymbolExtractor` interface. The `IndexingEngine` does not care which parser produced the AST.

## Two deployment models, one codebase

The same codebase ships as a STDIO MCP server (for laptop) and an ASP.NET Streamable HTTP server (for enterprise). No separate code paths, no feature divergence.

**STDIO MCP** — A single process. `InMemoryVectorStore`. N-gram embeddings. Zero external dependencies. Data is ephemeral — restart the process, re-index the repo. This is the `npx @uworx/code-memory` experience.

**ASP.NET / Streamable HTTP** — Persistent storage (SQLite, pgvector, SQL Server). Multi-repo — each repo gets its own MCP endpoint at `/api/mcp/{repoName}`. Embeddings can be ONNX or Ollama. Repository Dashboard at the root URL for managing repos, viewing components, and monitoring metrics. Scheduled re-indexing via cron. Docker Compose with PostgreSQL, Prometheus, Grafana, and Aspire Dashboard.

The per-repo storage isolation is worth calling out. Each repo gets its own `IStorageService` instance, registered at runtime through a thread-safe `IServiceRegistry` — not through keyed DI or named singletons, but through a `ConcurrentDictionary` that maps repo names to storage instances. The `StorageServiceRouter` (a singleton `IStorageService`) delegates to the correct instance based on the current repo context, which flows through `AsyncLocal` from the MCP request handler.

```
Request → MCP routing (/api/mcp/{repoName})
    → RepoContextAccessor sets AsyncLocal<string>
    → StorageServiceRouter.ReadAsync()
        → delegates to storageRegistry.Get(repoName)
    → IStorageService methods execute against repo-specific store
```

No middleware hacks. No per-repo DI containers. Just `AsyncLocal` and a dictionary.

## LLMs as equalizers

This series started with a claim — that LLMs are the great equalizer, that concepts once gated by runtime tribe or academic specialization are now accessible through conversation. Three posts in, the evidence is in the code:

**Parsing.** Roslyn and Tree-sitter are open-source libraries with good documentation, but writing a production-quality extractor over their ASTs used to require compiler expertise. Today, an LLM explains the Roslyn syntax tree API, generates a Tree-sitter query to find method declarations, and helps debug extraction edge cases. The `RoslynSymbolExtractor` and `TreeSitterSymbolExtractor` were built this way — not by compiler engineers, but by engineers who asked LLMs the right questions about the APIs.

**Embeddings.** Character n-gram hashing with random projection is a textbook information retrieval technique. It is also the kind of thing you would previously implement by reading academic papers, prototyping in Python, verifying against a test corpus, and then porting to C#. Today, an LLM describes the hashing strategy, implements the projection, and verifies the distribution — in the language of your choice. The `NgramEmbeddingGenerator` is 150 lines. It would have taken a week of research a few years ago. Today, it takes an afternoon.

**SQL engines.** A `SELECT`-only SQL parser with CTEs, JOINs, set operations, aggregates, and vector search is a compiler engineering project. The `SqlQueryService` is 2,200 lines — small enough for a single developer to understand, large enough that writing it from scratch without guidance would be daunting. But LLMs are fluent in SQL grammar. They have seen every SQL parser ever written. They generate correct AST traversal code on the first or second attempt. The engine was not written by a database specialist. It was written by describing the desired behavior and iterating with an LLM on the output.

**None of this means LLMs write the entire system.** They do not. What they do is flatten the learning curve. They turn "I need to understand this academic paper / compiler API / database internals" into "I need to describe what I want and correct the output." The expertise is still required — you need to know what correct looks like, how to verify the output, and when the LLM is hallucinating. But the *implementation cost* of infrastructure that previously required specialized knowledge drops dramatically.

## A useful rule of thumb

If your coding agent struggles to query your infrastructure, the problem is probably not the agent. It is the query interface. LLMs are fluent in SQL, JSON, and natural language. They are not fluent in your custom graph traversal DSL, your proprietary query language, or your internal schema conventions. The closer your query surface is to what the model was trained on, the fewer errors you will see.

## What comes next

CodeMemory is 0.x, and this is not a pause. The core abstractions are stable — the indexing pipeline, the storage contract, the SQL engine, the MCP tool surface, the two deployment models — but several threads are actively being pulled:

- Deeper integration between CodeMemory's dependency graph and Memori's augmentation pipeline — extracting semantic facts about components, not just symbols
- A specialized `CodeAugmentationClient` tuned for code/git analysis rather than conversational context
- Hotspot-aware incremental indexing — prioritize the files that change most, re-index the rest lazily
- Export and compliance APIs: a user's complete memory footprint, portable and auditable

Each of these deserves its own treatment. When the next one ships, there will be a post about it.

The codebase is open ([MIT](https://github.com/khurram-uworx/CodeMemory), .NET 10), the issues are public, and the architecture is documented. Star the repo, open an issue, or pick up a good first issue — memory is better together.

## Important Links

- CodeMemory repo: [https://github.com/khurram-uworx/CodeMemory](https://github.com/khurram-uworx/CodeMemory)
- npm package: `npx -y @uworx/code-memory`
- Architecture and design notes: [https://github.com/khurram-uworx/CodeMemory/blob/main/ARCHITECTURE.md](https://github.com/khurram-uworx/CodeMemory/blob/main/ARCHITECTURE.md)


{% if page.series %}
{% for s in page.series %}
{% assign series_posts = site.posts | where_exp: "post", "post.series contains s" | sort: 'date' %}
<div class="series-nav">
<h4>More from {{ s }}</h4>
<ul>
{% for post in series_posts %}
<li>
{% if post.url == page.url %}
<strong>{{ post.title }} (Current)</strong>
{% else %}
<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
{% endif %}
</li>
{% endfor %}
</ul>
</div>
{% endfor %}
{% endif %}
