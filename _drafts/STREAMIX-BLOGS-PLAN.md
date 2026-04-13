# Streamix Blog Series Plan

## Goal

Create a short, credible blog series that explains why Streamix exists, how to think about it, and where it fits in real .NET systems.

The series should help experienced engineers answer three questions:

- why would I use this instead of raw `Task`, `IAsyncEnumerable<T>`, and channels?
- what are the actual semantics around execution, ordering, cancellation, and backpressure?
- where does Streamix fit into production .NET code without requiring a full platform rewrite?

## Intended Audience

- software engineers with roughly 5 years of experience
- comfortable with C#, async/await, `IAsyncEnumerable<T>`, and service/backend code
- looking for sharper abstractions and clearer semantics, not beginner async tutorials

## Recommended Series Size

Recommended total: 4 posts

Why 4:

- enough space to cover motivation, mental model, semantics, and applied usage
- short enough to stay focused while the project is still early-stage
- avoids fragmenting the core story across too many narrowly scoped posts

## Series Arc

1. Why Streamix Exists in Modern .NET
2. The Core Mental Model: `Stream<T>`, `Single<T>`, and `IAsyncEnumerable<T>`
3. Hot vs Cold Streams, Ordering, and Async Composition
4. Backpressure, Interop, and Streaming ASP.NET Core Responses

## Post Plan

### 1. Why Streamix Exists in Modern .NET

Purpose:

- define the gap between low-level async primitives and a fluent stream abstraction
- position Streamix as .NET-native rather than as a direct clone of Rx or Reactor

Key points:

- `Task`, `IAsyncEnumerable<T>`, and channels are strong primitives but awkward to compose at scale
- larger pipelines need explicit semantics for concurrency, ordering, cancellation, and errors
- Streamix provides a small, fluent abstraction over those concerns

Examples:

- async enrichment pipeline
- paginated API aggregation
- event ingestion pipeline

### 2. The Core Mental Model: `Stream<T>`, `Single<T>`, and `IAsyncEnumerable<T>`

Purpose:

- explain the shape of the API and how it maps to existing .NET concepts

Key points:

- `Stream<T>` means 0..N values
- `Single<T>` means 0..1 values
- streams are cold and pull-based by default
- `IAsyncEnumerable<T>` remains the default mental model
- terminal execution is explicit

Examples:

- converting `await foreach` code into fluent composition
- combining `Single<T>` and `Stream<T>` in service code

### 3. Hot vs Cold Streams, Ordering, and Async Composition

Purpose:

- explain the behavioral semantics most likely to affect correctness and throughput

Key points:

- when a stream is re-executed versus shared
- when ordering is preserved and when it is intentionally relaxed
- why concurrent operators need explicit semantics
- how to choose between sequential, ordered concurrent, and unordered concurrent composition

Examples:

- expensive upstream source shared through `Publish()` / `RefCount()`
- ordered versus unordered enrichment calls
- 1:N expansion with `FlatMap`, `FlatMapOrdered`, and `ConcatMap`

### 4. Backpressure, Interop, and Streaming ASP.NET Core Responses

Purpose:

- finish with practical integration guidance

Key points:

- pull-based backpressure and explicit channel boundaries
- interop with `IAsyncEnumerable<T>`, channels, and AsyncRx.NET
- practical server-side streaming in ASP.NET Core
- how to adopt Streamix incrementally inside an existing system

Examples:

- bounded pipeline with channel-backed execution
- adapting existing async enumerable or channel producers
- SSE or JSON response streaming endpoint

## Editorial Guidelines

- Keep each post anchored to `README.md`, `GETTING-STARTED.md`, and `ARCHITECTURE.md`.
- Do not present roadmap items as fully implemented.
- Prefer concrete semantics over abstract "reactive programming" language.
- Use production-shaped examples rather than toy-only examples.
- Compare against raw .NET primitives more often than against Java ecosystem vocabulary.

## Risks to Avoid

- writing generic reactive-programming content instead of Streamix-specific guidance
- overselling early-stage capabilities
- explaining basic async concepts this audience already knows
- separating closely related semantics into too many disconnected posts
