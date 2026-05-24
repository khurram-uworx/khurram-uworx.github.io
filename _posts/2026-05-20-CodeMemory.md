---
title: "Transform Repositories into Queryable Intelligence"
date: 2026-05-20
series:
  - "Context Engineering"
  - "LLMs as Equializers"
  - "Memori"
  - "Model Context Protocol"
  - "Vectors and Tensors"
tags:
  - MCP
  - Semantic Search
  - rag
  - Code Intelligence
  - Context Engineering
comments: true
---

*Why the next frontier in AI-assisted engineering isn't better code generation — it's repository cognition.*

AI coding agents have crossed a threshold. From Copilot to Claude Code to Cursor, the ecosystem now ships code *fast* — complete functions, test suites, refactors, even entire modules. The industry has spent two years optimizing for *generation speed and quality*.

But there's a gap. Agents today are **smart but amnesiac**. They see files, not architecture. Every conversation starts from zero. Deep understanding — dependency chains, impact boundaries, architectural structure, historical evolution, semantic intent — is scattered across LSP servers, git logs, IDE indexes, and developer intuition. No single system connects these layers into something an agent can query deterministically.

This isn't a problem of generation. It's a problem of *memory*.

## The Missing Layer

What if a codebase had a persistent, queryable "brain" — not a search index, but a structured intelligence layer that captures:

- Every symbol, its kind, location, and relationships
- Every dependency: who calls what, who inherits from whom
- Architecture clusters: which components form logical groups
- Semantic embeddings: what code *means*, not just what it says
- Git evolution: how things changed and how often

And what if this brain was available to any agent, in any tool, through a standard protocol — without API keys, without model downloads, without spinning up a vector database?

That's what CodeMemory is.

## We're Not the First

This idea isn't new. The MCP ecosystem already has strong contenders:

- **Serena** — 24k+ stars, LSP-powered symbol retrieval, JetBrains plugin, agentic refactoring
- **Codebase Memory MCP (cmm)** — 2.5k+ stars, C-based knowledge graph, 155 languages, millisecond indexing
- **ChunkHound** — 1.3k+ stars, Python, tree-sitter chunking with a research-backed "cAST" algorithm
- **jCodeMunch** — 1.8k+ stars, token-efficient AST exploration with PageRank centrality
- **Code Index MCP** — 900+ stars, tree-sitter for 10 languages
- **and more:** Code Pathfinder, GitNexus, KiroGraph, Nella, Graphify

Each takes a different approach. The space is rapidly maturing, which is a good sign — it means the problem is real.

**So why build another one?** Two reasons.

### Reason 1: .NET is an Interesting Runtime for AI/ML Workloads

Most code intelligence tools are written in Python, TypeScript, or C. .NET is rarely the first choice — but this ecosystem has quietly assembled one of the most coherent stacks for this kind of work:

- **Roslyn** — Microsoft's open-source C# compiler platform. Not a third-party grammar binding, but the *actual compiler* as a library. Full semantic model, type resolution, syntax trees with perfect fidelity. If you're indexing C# code, writing a parser that competes with Roslyn is simply not rational.
- **`System.Numerics.Tensors`** — SIMD-accelerated tensor operations baked into the BCL. `TensorPrimitives.Norm()` for L2 normalization, `TensorPrimitives.Dot()` for cosine similarity — both JIT to AVX-512 on capable hardware. No Python-to-native FFI overhead, no numpy dependency.
- **`Microsoft.Extensions.AI.Abstractions`** — Standardized interfaces for `IEmbeddingGenerator<TInput, TEmbedding>` and `IChatClient`. OpenAI, Ollama, Azure, local models — swap providers by changing DI registration. Includes a delegating pipeline pattern (caching, telemetry, rate-limiting middleware) that mirrors .NET's HTTP message handler design. Think of it as `Microsoft.Extensions.Logging` for AI — the same abstraction approach that made structured logging ubiquitous in .NET.
- **`Microsoft.Extensions.VectorData.Abstractions`** — Abstract `VectorStore` base class with out-of-the-box connectors for Azure AI Search, Qdrant, Redis, SQLite, PostgreSQL (pgvector), SQL Server, and InMemory. You prototype with in-memory, swap to production with a connection string change.
- **TreeSitter.DotNet** — For non-C# languages, .NET bindings for tree-sitter provide the same AST quality without leaving the platform.

These aren't bolt-on wrappers. They ship as first-class NuGet packages for .NET 10's `IHostApplicationBuilder` and `Microsoft.Extensions.DependencyInjection` patterns. Building in .NET meant we focused on the *intelligence* layer rather than assembling fragmented language bindings and FFI wrappers.

### Reason 2: Two Deployment Models, Same Codebase

Most MCP servers pick one deployment model — either laptop-only or server-only. We wanted both without maintaining two codebases.

**STDIO MCP (default)** — Zero-infrastructure, ephemeral, local-first. A single process starts indexing non-blocking (MCP server live from second zero), uses an in-memory vector store, offline n-gram embeddings, and zero external dependencies. Data is ephemeral — restart the process, re-index. Perfect for CI/CD pipelines, agent sessions, and onboarding.

**ASP.NET / Streamable HTTP (enterprise)** — When the same codebase needs to serve multiple teams and persist across restarts: hybrid storage (symbols/relationships in EF Core relational tables, chunks in a `Microsoft.Extensions.VectorData` vector store), pluggable backends (SQLite, PostgreSQL with pgvector, SQL Server), and multi-repo out of the box — each repo gets its own MCP endpoint (`/api/mcp/{repoName}`) without middleware hacks or keyed DI.

One codebase. Two transports. Your laptop to production without rewriting anything.

## What CodeMemory Does

CodeMemory exposes repository intelligence through 11 MCP tools. Each returns structured JSON — deterministic, composable, not freeform chat:

| Tool | What it gives you |
|---|---|
| `semantic_search` | Natural language code search (no grep needed) |
| `trace_dependency` | Follow dependency chains: upstream (what this symbol calls) or downstream (what calls this symbol) |
| `impact_analysis` | What breaks if you change this symbol — a focused downstream traversal with depth limits |
| `get_architecture_overview` | Components, languages, file/symbol counts |
| `get_edit_context` | Source + deps + tests — everything an agent needs to edit |
| `get_component_clusters` | Logical groupings by inter-component coupling |
| `get_symbol_history` | Git history for a specific symbol |
| `get_hotspots` | Most frequently changed files |
| `sql_query` | Full SQL over your indexed repo data |
| `find_related_code` | Related symbols by relationship type |
| `ping` | Health check + indexing status |

An agent can call `impact_analysis`, feed the results into `get_edit_context`, and make changes with architectural awareness — all without having read a single file.

CodeMemory is MCP-native by design. No chat interface, no IDE plugin, no "AI for developers" dashboard. It's **infrastructure** — a cognition provider that any MCP-compatible agent (Claude Code, GitHub Copilot, Cursor, Codex CLI) can plug into. MCP is rapidly becoming the universal protocol for agent-tool communication, and CodeMemory avoids competing on UI entirely.

## Try It Now — One Command, Any Machine

- We ship native binaries for Windows x64 and Linux x64 (both under 50MB compressed). Mac support is straightforward — the only native dependency is TreeSitter, which targets all three platforms. It's a CI/CD matrix expansion.

CodeMemory ships as a native binary via npm. No .NET SDK, no build step, no source clone. The binary is a self-contained .NET single-file publish — the npm package is just a thin wrapper that downloads the right binary for your platform on install.

*(Why npm? Practically every coding agent runs on Node.js — Codex CLI, Cursor, Copilot CLI, VS Code, Claude Desktop. Publishing there means zero friction for the tools your teams already use.)*

```bash
# Warm the cache (first run downloads the binary — a few seconds)
npx -y @uworx/code-memory --version

# Point it at any repo
npx -y @uworx/code-memory --repo /path/to/your/project
```

Configure it in your agent:

**Copilot CLI (GitHub)** — `~/.mcp.json`:
```json
{
  "mcpServers": {
    "code-memory": { "command": "npx", "args": ["-y", "@uworx/code-memory"] }
  }
}
```

**Cursor** — `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "code-memory": { "command": "npx", "args": ["-y", "@uworx/code-memory"] }
  }
}
```

**Codex CLI** — `~/.codex/config.json`:
```json
{
  "mcpServers": {
    "code-memory": { "command": "npx", "args": ["-y", "@uworx/code-memory"] }
  }
}
```

Start with `ping`. Wait for `indexingCompleted: true`. Then every other tool is ready.

## Architecture in 30 Seconds

A few design choices worth calling out that didn't fit above:

- **SQL query engine** — `SqlParserCS` → LINQ expression trees over the vector store. Supports `SELECT`/`WHERE`/`ORDER BY`/`GROUP BY`/`HAVING`, CTEs, derived tables, aggregates, and vector search via `ORDER BY Similarity DESC`. On the ASP.NET path, queries route to EF Core for symbols/relationships and the vector store for chunks.
- **Multi-language parsing** — Roslyn for C# (the actual compiler), Tree-sitter for TypeScript, JavaScript, and Java
- **Plug-in embeddings** — `IEmbeddingGenerator<string, Embedding<float>>` from `Microsoft.Extensions.AI.Abstractions`. Default is [Memori](https://github.com/khurram-uworx/Memori)'s n-gram hashing (offline, deterministic, no API keys, sub-millisecond per embedding). N-grams are surprisingly effective for code: variable names, function identifiers, and API surface tokens carry dense semantic signal that n-gram fingerprints capture well, often rivaling lightweight transformer embeddings for retrieval tasks. And the implementation is trivial — hash collisions don't break retrieval, they just group synonyms, which is exactly what you want in code search. Swap to OpenAI, Ollama, or any provider via DI registration. (Memori is our own NuGet — the dependency exists only because this logic shouldn't need reinventing.)
- **Non-blocking by contract** — Indexing runs in the background. Tools return empty/partial results until `ping` reports `indexingCompleted: true`. This is the contract every agent must follow, and it's consistent across both STDIO and ASP.NET deployments.

## What It Enables

Once a codebase has a persistent brain, the ceiling on agent-assisted development changes:

- An agent refactoring `PaymentGateway` calls `impact_analysis` first — sees every downstream dependent before touching a line
- An agent debugging a build failure calls `trace_dependency` — traces the broken import chain in milliseconds
- An agent onboarding to a new repo calls `get_architecture_overview` and `get_component_clusters` — understands structure in seconds
- An agent writing a cross-cutting change calls `get_edit_context` — gathers every file, dependency, and test it needs
- An agent runs `sql_query` — "show me all public methods in the auth module sorted by line count"

None of this requires the agent to have *seen* these files before. The memory is persistent.

---

*CodeMemory is a local-first repository intelligence engine. It is not an IDE, a chat assistant, or a code generator. It is a persistent, queryable semantic memory layer for your coding agents.*

*[GitHub](https://github.com/khurram-uworx/CodeMemory) — MIT — .NET 10*

*We're actively building. Star the repo, open an issue, or pick up a "good first issue" — memory is better together.*

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
