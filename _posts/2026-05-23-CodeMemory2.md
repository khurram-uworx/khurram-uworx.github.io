---
title: "From Files to Structured Intelligence"
date: 2026-05-23
series:
  - "Memori"
tags:
  - MCP
  - Semantic Search
  - Code Intelligence
  - Indexing
  - Context Engineering
comments: true
---

A codebase is not a text file collection. It is a graph of symbols, relationships, and semantics — but most tools never extract that structure. They search text. They grep. They match patterns against string content, not against meaning.

CodeMemory's first job is to turn raw files into structured intelligence. That means extracting every symbol, every relationship, and every semantic boundary from a repository and storing it in a form that agents can query deterministically.

The pipeline that does this — crawling, parsing, extracting, chunking, embedding, storing — is the engine room of the library. It is also a case study in how LLMs have changed what a small team can build in a .NET codebase.

## The pipeline

```
File fetch
    ↓
Language detection (extension-based, .gitignore-aware)
    ↓
Language routing:
    .cs       → RoslynCSharpParser (the actual C# compiler)
    .ts/.js   → TreeSitterParser
    .java     → TreeSitterParser
    .py/.go/.rs/.c/.cpp → TreeSitterParser
    .txt/.md  → text reader (no extraction, still chunked)
    ↓
Symbol extraction (ISymbolExtractor — 23 kinds)
    ↓
Relationship extraction (IRelationshipExtractor — static, syntax-only)
    ↓
Semantic chunking (type-level + member-level, SHA256 identity)
    ↓
Embedding generation (IEmbeddingGenerator — n-gram default, pluggable)
    ↓
Storage (IStorageService — three collections)
```

Each stage is an interface. Each stage can be replaced. The defaults work out of the box — but the abstractions are there for teams that outgrow them.

## Language parsing: Roslyn vs Tree-sitter

The first question a code intelligence tool must answer: how do you parse source code?

There are two paths. You can write or bind a parser for each language — hand-crafted grammars, AST visitors, symbol tables. Or you can use a platform that already has a compiler.

For C#, we made the obvious choice. Roslyn is Microsoft's open-source C# compiler platform. It is not a third-party grammar binding. It is the actual compiler exposed as a library. `CSharpSyntaxTree.ParseText` gives you a full syntax tree with perfect fidelity. The symbol extractor walks `ClassDeclarationSyntax`, `MethodDeclarationSyntax`, `PropertyDeclarationSyntax` — the compiler's own types. There is no abstraction layer, no grammar version mismatch, no "this feature isn't parsed yet."

```csharp
var syntaxTree = CSharpSyntaxTree.ParseText(text, path: filePath);
var root = await syntaxTree.GetRootAsync();

foreach (var typeDecl in root.DescendantNodes().OfType<TypeDeclarationSyntax>())
{
    var kind = typeDecl switch
    {
        InterfaceDeclarationSyntax => CodeSymbolKind.Interface,
        StructDeclarationSyntax    => CodeSymbolKind.Struct,
        RecordDeclarationSyntax    => CodeSymbolKind.Record,
        _                          => CodeSymbolKind.Class,
    };
    // ... extract name, fullName, line range, modifiers, documentation
}
```

For everything else — TypeScript, JavaScript, Java, Python, Go, Rust, C, C++ — we use Tree-sitter. Tree-sitter is a parser generator with pre-built grammars for dozens of languages. `TreeSitter.DotNet` provides .NET bindings that give us the same AST quality without leaving the platform.

```csharp
using var parser = new Parser(new TreeSitter.Language("TypeScript"));
var tree = parser.Parse(text);
// Walk tree.RootNode with Tree-sitter queries
```

The Roslyn path is richer: it resolves types, understands generics, reads XML doc comments. The Tree-sitter path is language-agnostic: it does not resolve cross-file references, but it does extract every function declaration, class definition, and variable binding in the file.

**This used to require compiler engineering expertise.** Writing a production-quality parser for a single language is a multi-month project. Supporting ten languages is a team of specialists. But Roslyn and Tree-sitter — combined with LLMs that can explain their APIs, generate Tree-sitter query patterns, and debug parse failures — collapse the cost dramatically. The `RoslynSymbolExtractor` walks the compiler's own syntax tree. The `TreeSitterSymbolExtractor` navigates a standardized AST. Neither required writing a grammar. Both required knowing *what questions to ask* — and LLMs are excellent at answering "how do I find all method declarations in a TypeScript AST with Tree-sitter?"

The `ILanguageParser` interface abstracts the difference. The `IndexingEngine` routes files by extension:

```csharp
parsers = new Dictionary<Language, ILanguageParser>
{
    [Language.CSharp]      = roslynParser,
    [Language.TypeScript]  = tsParser,
    [Language.JavaScript]  = tsParser,
    [Language.Java]        = tsParser,
    [Language.Python]      = tsParser,
    [Language.Go]          = tsParser,
    [Language.Rust]        = tsParser,
    [Language.C]           = tsParser,
    [Language.Cpp]         = tsParser,
    [Language.HTML]        = tsParser,
};
```

Adding a new language means adding a Tree-sitter grammar reference and a `LanguageDetector` mapping entry. The extractors and chunkers do not care which parser produced the AST.

## Symbol extraction

The `ISymbolExtractor` interface is minimal:

```csharp
public interface ISymbolExtractor
{
    IReadOnlyList<Symbol> Extract(ParseResult result, string filePath);
}
```

`Symbol` carries everything an agent needs to navigate the codebase:

```csharp
public sealed record Symbol(
    string Name,
    CodeSymbolKind Kind,        // 23 kinds: Class, Method, Property, Field, ...
    string FilePath,
    LineRange LineRange,        // Start and end lines
    string FullName,            // Namespace.Class.Method
    IReadOnlyList<string> Modifiers,  // public, static, async, ...
    string? Documentation);     // XML doc comment
```

The `RoslynSymbolExtractor` walks the C# syntax tree recursively — types contain members, members may contain nested types. Each symbol gets a `FullName` that is unique within the repo: `CodeMemory.Services.IndexingEngine.RunIndexingAsync`. This is the identifier used for dependency tracking, impact analysis, and cross-file references.

Twenty-three symbol kinds cover the language surface: `Class`, `Interface`, `Struct`, `Enum`, `Record`, `Method`, `Property`, `Field`, `Event`, `Function`, `Variable`, `TypeAlias`, `Module`, `Constructor`, `Annotation`, `Decorator`. Not every language uses all of them. Not every symbol kind is relevant to every query. But the taxonomy is broad enough that an agent asking "what interfaces does this class implement?" or "show me all public methods in the auth module" gets structured results, not grep noise.

## Relationship extraction

Symbols alone are a dictionary. Relationships are the index — the cross-references that turn a flat list of definitions into a dependency graph.

The `IRelationshipExtractor` finds four kinds of relationships by walking the syntax tree:

- **Inherits**: `class PaymentGateway : IPaymentGateway`
- **Implements**: interface member implementations
- **Calls**: method invocations within method bodies
- **References**: type usage in field declarations, property types, parameter types, variable declarations

The extractor is syntax-only. It does not use Roslyn's `SemanticModel` for type resolution. That means it cannot distinguish between `MyClass.Method()` and `SomeOtherClass.Method()` when both have the same name — but it captures every intra-file reference, and cross-file references are resolved by matching full names. For an MVP that covers the common cases — "what calls this method?" and "what depends on this class?" — syntax-only extraction covers roughly 90% of the practical surface. The remaining 10% (overloaded methods, extension methods, dynamic dispatch) is a known limitation documented in the architecture.

Every relationship is stored as a triple with a composite ID:

```
{sourceGuid}->{targetGuid}:{relationshipType}
```

The composite key ensures deduplication at the storage level. Same source, same target, same type — one record, no matter how many times the extractor encounters it.

## Semantic chunking

Parsing and extraction give you symbols and relationships. But agents also need to search code by meaning, not just by name. That is what semantic search provides — and semantic search needs chunks.

`SemanticChunker` divides each file into two kinds of chunks:

- **Type chunks**: a class, interface, struct, enum, or record — including its file-level context (usings, imports, namespace declarations)
- **Member chunks**: individual methods, properties, fields, and events within a type — each with a `// Parent:` header for context

```csharp
var typeChunk = createTypeChunk(typeSymbol, fileLines, fileContext, filePath, language);
var memberChunk = createMemberChunk(memberSymbol, fileLines, filePath, language);
```

Each chunk gets a content-hash ID via SHA256 of `symbolId|content|filePath`. This makes chunk identity deterministic across indexing runs — the same symbol in the same file always produces the same chunk ID. Incremental indexing can skip unchanged chunks without comparing byte streams.

For text files (.md, .txt) that have no symbols, a single file-level chunk is created. This means README files, documentation, and configuration guides are searchable alongside code.

## Embedding: deterministic by default, pluggable by design

CodeMemory uses `IEmbeddingGenerator<string, Embedding<float>>` from `Microsoft.Extensions.AI.Abstractions` — the same abstraction used by Memori, Semantic Kernel, and the broader .NET AI ecosystem. The default implementation is Memori's `NgramEmbeddingGenerator`.

Memori's earlier posts (links below, Storage, Recall, and the Middleware Pipeline one) covered the embedding `DeterministicEmbeddingGenerator` and `NgramEmbeddingGenerator` in detail — hash tokens into a `float[]`, L2-normalize with `TensorPrimitives.Norm`, store in the `ChunkRecord.Embedding` field. The mechanics are the same here, so I will not repeat them.

What matters for CodeMemory is the plugin architecture:

```csharp
// appsettings.json for the ASP.NET host
"Embedding": {
    "Provider": "ngram"   // or "onnx" or "ollama"
}
```

Swap from deterministic n-gram hashing to ONNX-based BERT embeddings (bge-micro-v2) or an Ollama-hosted model by changing a config value. The pipeline does not care. The embedding generator is registered at startup, injected into `IndexingEngine`, and called after chunking produces document blocks.

The L2 normalization is explicit because the `IEmbeddingGenerator` contract does not guarantee normalized vectors. `TensorPrimitives.Norm()` from `System.Numerics.Tensors` handles it with SIMD acceleration — AVX-512 on capable hardware, scalar fallback everywhere else. No Python dependency, no numpy import, no native FFI call. Just `float[]` math in the BCL.

## Storage: three collections, one contract

The `IStorageService` interface abstracts over three data stores:

| Collection | Record Type | Key | Vector? |
|---|---|---|---|
| `symbols` | `SymbolRecord` | GUID | No |
| `chunks` | `ChunkRecord` | SHA256 hash | Yes (float32, cosine) |
| `relationships` | `RelationshipRecord` | `source->target:type` composite | No |

The MCP (STDIO) host uses `InMemoryVectorStore` — all three collections live in the same vector store process. Data is ephemeral. Zero infrastructure. The ASP.NET host uses `HybridStorageService` — symbols and relationships in EF Core relational tables (for efficient SQL queries via `AspNetSqlQueryTool`), chunks in a `VectorStore` provider (for similarity search).

This dual-path storage is the CodeMemory equivalent of Memori's two-storage split — covered in detail in the Storage, Recall, and the Middleware Pipeline post (link below). The principle is the same: relational data (symbols, relationships) and vector data (chunks, embeddings) have different access patterns, different consistency requirements, and different storage characteristics. Mismatching them is how memory layers become slow, expensive, or both.

## Non-blocking by contract

Indexing a repository takes time — seconds for small projects, minutes for large monorepos. Blocking the MCP server until indexing completes would make the first experience feel broken. So indexing runs in the background, and the `ping` tool reports status:

```json
{"status":"ok","indexingCompleted":false,"message":"Indexing in progress. Retry tools in a few seconds."}
```

Once done:

```json
{"status":"ok","indexingCompleted":true,"fileWatcherActive":true,"version":"0.5.5+..."}
```

This contract is consistent across both hosts:
- **STDIO MCP**: `Task.Run` starts indexing before `host.RunAsync()`. MCP server is live from second zero.
- **ASP.NET**: `IndexingHostedService` (BackgroundService) initializes storage upfront, then indexes each repo sequentially. MCP endpoints are live immediately.

The `IndexingState` static class (ConcurrentDictionary-based, process-scoped) tracks completion per repo. Tools check `IndexingState.IsCompleted()` before serving results — empty or partial results if called too early. This is not a failure; it is a design constraint. Agents must poll `ping` until `indexingCompleted` is true.

On the STDIO path, a `FileWatcherService` starts after indexing completes, monitoring the repo via `FileSystemWatcher` with a 1-second debounce. Changed files are re-indexed incrementally — delete old symbols and chunks, parse, extract, store new ones. The file watcher means the index stays fresh during development without a full re-index cycle.

## A useful rule of thumb

Indexing speed matters less than indexing correctness. A fast pipeline that misses symbols is worse than a slow pipeline that captures everything.

If an agent's `impact_analysis` returns zero results because a relationship was missed during indexing, the agent makes changes without understanding consequences. That is worse than waiting three extra seconds for indexing to complete. Optimize for completeness before optimizing for speed — and rely on the file watcher for incremental freshness.

## What comes next

This pipeline transforms a raw repository into structured, queryable intelligence. But the real power is what happens when agents can query that intelligence through a language they already know.

CodeMemory's SQL query engine — the subject of the next post — lets agents write `SELECT Name, Kind FROM SymbolRecord WHERE FilePath LIKE '%auth%' ORDER BY Similarity DESC` and get structured results from the same store. We did not start there. We tried a graph database approach first — querying the dependency graph through a custom query layer. LLMs struggled with it. Every query required learning a custom DSL, understanding the graph schema, and writing traversal patterns that felt unnatural. Then we switched to SQL — and the same agents that failed at graph queries wrote correct SQL on the first try, every time.

That story, and the SQL engine we built because of it, is the subject of the next post.

The same principle applies to the parsing pipeline described here, the embedding strategy, and the storage architecture. Each was built by asking LLMs to help navigate existing platform abstractions — Roslyn, Tree-sitter, IEmbeddingGenerator, VectorStore — rather than writing everything from scratch. The result is a code intelligence layer that would have required a specialized team to build a few years ago, now achievable by a focused effort on top of well-chosen primitives.

## Important Links

CodeMemory repo: [https://github.com/khurram-uworx/CodeMemory](https://github.com/khurram-uworx/CodeMemory)

npm package: `npx -y @uworx/code-memory`

Architecture and design notes: [https://github.com/khurram-uworx/CodeMemory/blob/main/ARCHITECTURE.md](https://github.com/khurram-uworx/CodeMemory/blob/main/ARCHITECTURE.md)

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
