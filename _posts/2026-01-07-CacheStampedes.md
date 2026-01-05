---
title: "The Best Way to Prevent Cache Stampedes"
date: 2026-01-07
tags:
    - dotnet
    - caching
comments: true

---

In **2026 / .NET 10**, the *most elegant* and idiomatic solution to the **cache stampede (dogpile) problem** in C# is:

> **Use built-in “single-flight” caching via `HybridCache.GetOrCreateAsync`**
> (or `IMemoryCache.GetOrCreateAsync` + async locking if HybridCache isn’t available)

This gives you **correctness, simplicity, and async safety** without reinventing concurrency primitives.

---

## 1️⃣ The Modern Best Practice: `HybridCache` (Recommended)

### Why this is the elegant solution

`Microsoft.Extensions.Caching.Hybrid` (introduced in .NET 8 and mature by .NET 10):

* ✅ Built-in **cache stampede protection**
* ✅ Ensures **only one factory runs per key**
* ✅ Async-safe, cancellation-aware
* ✅ Supports **L1 memory + L2 distributed cache**
* ✅ No custom locking code

### Example

```csharp
public async Task<Product> GetProductAsync(
    int id,
    CancellationToken ct)
{
    return await _cache.GetOrCreateAsync(
        $"product:{id}",
        async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            return await _repository.GetProductAsync(id, ct);
        },
        ct);
}
```

### What happens under load?

* 1,000 concurrent requests hit a cold cache
* **Only one request** executes `_repository.GetProductAsync`
* Others **await the same Task**
* No lock contention, no duplicated DB calls

✔ This is *true single-flight caching*

---

## 2️⃣ If You’re Using Only `IMemoryCache`

`IMemoryCache.GetOrCreateAsync` **also prevents stampedes** as long as:

* You **only cache the result**
* You **don’t do external locking**
* Your factory is fully async

### Correct pattern

```csharp
public Task<User> GetUserAsync(Guid id)
{
    return _cache.GetOrCreateAsync(
        $"user:{id}",
        async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(2);
            return await LoadUserAsync(id);
        });
}
```

### Why this works

Internally, `MemoryCache`:

* Locks per key
* Shares the in-flight Task
* Releases waiters once completed

⚠️ **Common mistake**

```csharp
if (!_cache.TryGetValue(key, out var value))
{
    value = await LoadAsync(); // ❌ stampede risk
    _cache.Set(key, value);
}
```

---

## 3️⃣ Advanced / Custom: `AsyncLazy<T>` or `Task<T>` Caching

If you need **full control**, the classic single-flight pattern still works:

```csharp
private readonly ConcurrentDictionary<string, Lazy<Task<T>>> _inflight = new();

public Task<T> GetAsync(string key)
{
    var lazy = _inflight.GetOrAdd(
        key,
        _ => new Lazy<Task<T>>(LoadAsync));

    return lazy.Value.ContinueWith(t =>
    {
        _inflight.TryRemove(key, out _);
        return t.Result;
    });
}
```

### Use this only if:

* You need **non-expiring inflight coordination**
* You’re implementing a **custom cache**
* You need behavior not supported by `HybridCache`

Otherwise, this is **more code than value** in 2025.

---

## 4️⃣ What *Not* to Use Anymore ❌

| Pattern                 | Why                      |
| ----------------------- | ------------------------ |
| Global `SemaphoreSlim`  | Kills throughput         |
| Per-key `SemaphoreSlim` | Hard to manage lifecycle |
| Double-check locking    | Error-prone with async   |
| Polly-only locks        | Not caching-aware        |
| Manual Redis locks      | Overkill for read caches |

---

## ✅ Final Recommendation (2025)

### 🥇 Best Overall

**`HybridCache.GetOrCreateAsync`**

> Clean, safe, scalable, async-first, future-proof

### 🥈 Good Enough

**`IMemoryCache.GetOrCreateAsync`**

### 🥉 Only If Necessary

**Custom single-flight via `Lazy<Task<T>>`**

---

If you **must implement it yourself** in 2025—no `IMemoryCache`, no `HybridCache`, no “fancy” caching libraries—the **most elegant, correct, and idiomatic C# solution** is a **single-flight async cache** using:

> **`ConcurrentDictionary<TKey, Task<TValue>>` (or `Lazy<Task<T>>`)**

This is the simplest design that:

* Prevents cache stampede
* Is fully async-safe
* Requires no locks
* Scales under high concurrency
* Is easy to reason about

---

## ✅ The Canonical DIY Solution (Recommended)

### Core idea

Cache **the in-flight Task**, not the result.

All concurrent callers await the same task.

---

### Minimal, correct implementation

```csharp
public sealed class SingleFlightCache<TKey, TValue>
    where TKey : notnull
{
    private readonly ConcurrentDictionary<TKey, Task<TValue>> _tasks = new();

    public async Task<TValue> GetAsync(
        TKey key,
        Func<CancellationToken, Task<TValue>> factory,
        CancellationToken ct = default)
    {
        var task = _tasks.GetOrAdd(key, _ =>
            CreateTask(key, factory, ct));

        try
        {
            return await task.ConfigureAwait(false);
        }
        catch
        {
            // Important: do not cache failures
            _tasks.TryRemove(key, out _);
            throw;
        }
    }

    private async Task<TValue> CreateTask(
        TKey key,
        Func<CancellationToken, Task<TValue>> factory,
        CancellationToken ct)
    {
        try
        {
            return await factory(ct).ConfigureAwait(false);
        }
        finally
        {
            // Remove only after completion
            _tasks.TryRemove(key, out _);
        }
    }
}
```

---

## 🧠 Why this is elegant

### ✔ No locks

`ConcurrentDictionary` already gives you atomicity.

### ✔ No `SemaphoreSlim`

Avoids contention and deadlocks.

### ✔ Natural async behavior

Callers just `await`.

### ✔ Automatic stampede prevention

First caller executes `factory`, others await.

### ✔ Failure-safe

Exceptions are **not cached**.

---

## ⚠️ Critical Design Details (Most People Miss These)

### 1️⃣ **Do NOT cache failed Tasks**

Otherwise one failure poisons all future callers.

✔ Remove the entry on exception.

---

### 2️⃣ **Do NOT pass per-request CancellationTokens**

If one caller cancels, all callers cancel.

❌ Bad:

```csharp
GetAsync(key, factory, requestCt)
```

✔ Good:

```csharp
GetAsync(key, factory, CancellationToken.None)
```

Or:

```csharp
using var cts = CancellationTokenSource
    .CreateLinkedTokenSource(appShutdownToken);
```

---

### 3️⃣ **Don’t remove before completion**

Removing too early causes double execution.

❌ Wrong:

```csharp
_tasks.TryRemove(key, out _); // before await
```

✔ Correct:

```csharp
finally { _tasks.TryRemove(key, out _); }
```

---

## 🔁 Variant: `Lazy<Task<T>>` (Stronger Guarantees)

If you want **exactly-once execution**, even under extreme races:

```csharp
private readonly ConcurrentDictionary<TKey, Lazy<Task<TValue>>> _cache = new();

public Task<TValue> GetAsync(TKey key, Func<Task<TValue>> factory)
{
    var lazy = _cache.GetOrAdd(
        key,
        _ => new Lazy<Task<TValue>>(
            factory,
            LazyThreadSafetyMode.ExecutionAndPublication));

    return lazy.Value.ContinueWith(t =>
    {
        _cache.TryRemove(key, out _);
        return t.Result;
    });
}
```

### When to use this

* You absolutely must guarantee only one factory invocation
* You accept slightly more complexity

---

## ❌ Patterns to Avoid (Even DIY)

| Pattern               | Problem                          |
| --------------------- | -------------------------------- |
| `lock`                | Blocks threads                   |
| `SemaphoreSlim`       | Easy to leak / forget to release |
| Double-check locking  | Broken with async                |
| Manual spinlocks      | CPU waste                        |
| One semaphore per key | Lifecycle hell                   |

---

## 🧪 Bonus: Add TTL (Still Simple)

```csharp
record CacheEntry(Task<TValue> Task, DateTimeOffset Expiry);

private readonly ConcurrentDictionary<TKey, CacheEntry> _cache = new();

public Task<TValue> GetAsync(TKey key, Func<Task<TValue>> factory)
{
    var now = DateTimeOffset.UtcNow;

    var entry = _cache.AddOrUpdate(
        key,
        _ => new CacheEntry(factory(), now.AddMinutes(5)),
        (_, old) =>
            old.Expiry > now ? old : new CacheEntry(factory(), now.AddMinutes(5)));

    return entry.Task;
}
```

✔ Still stampede-safe
✔ Still async
✔ No locks

---

Great question. This is where the **design philosophies really diverge**, even though they solve the same *cache stampede* problem.

Below is a **clear, practical comparison** of:

* **.NET single-flight caching** (HybridCache / IMemoryCache / Task-based)
* **Java’s Caffeine**
* **Go’s `singleflight`**

with emphasis on **mechanics, guarantees, and ergonomics** rather than marketing.

---

## 1️⃣ Conceptual Model (One-Line Summary)

| Platform                | Mental Model                              |
| ----------------------- | ----------------------------------------- |
| **.NET**                | *Cache the in-flight `Task<T>`*           |
| **Java (Caffeine)**     | *Block competing threads on the same key* |
| **Go (`singleflight`)** | *Deduplicate concurrent function calls*   |

---

## 2️⃣ Core API Comparison

### .NET (HybridCache / IMemoryCache)

```csharp
await cache.GetOrCreateAsync(key, async entry =>
{
    return await LoadAsync();
});
```

### Java (Caffeine)

```java
cache.get(key, k -> load(k));
```

### Go (singleflight)

```go
v, err, _ := group.Do(key, func() (any, error) {
    return load()
})
```

---

## 3️⃣ Concurrency Mechanics (Critical Differences)

### 🟦 .NET — *Task sharing (async-first)*

* First caller starts `Task<T>`
* Other callers **await the same Task**
* No threads blocked
* Naturally async & scalable

```text
Requests ──► same Task<T> ──► await
```

✔ **Best fit for async I/O**

---

### 🟨 Java (Caffeine) — *Thread blocking*

* First thread computes value
* Other threads **block** on the same key
* Uses locks / condition variables

```text
Threads ──► lock ──► wait ──► wake
```

✔ Very efficient for **CPU-bound work**
❌ Less ideal for async/reactive models

---

### 🟥 Go — *Goroutine deduplication*

* Goroutines wait on channels
* No OS thread blocking
* Explicitly **not a cache**

```text
goroutines ──► channel wait ──► resume
```

✔ Extremely lightweight
✔ Explicit failure semantics
❌ No TTL, eviction, or storage

---

## 4️⃣ Failure & Cancellation Semantics

| Aspect            | .NET                          | Caffeine            | Go singleflight |
| ----------------- | ----------------------------- | ------------------- | --------------- |
| Exception cached? | ❌ No (unless you do it wrong) | ❌ No                | ❌ No            |
| Cancellation      | Shared Task → risky           | Thread interruption | Context-based   |
| Retry on failure  | Automatic                     | Automatic           | Manual          |
| Partial success   | Possible                      | No                  | No              |

### Subtle .NET gotcha

Passing a **request-scoped `CancellationToken`** can cancel *everyone*.

✔ Best practice:

```csharp
CancellationToken.None
```

---

## 5️⃣ TTL & Eviction

| Feature             | .NET        | Caffeine        | Go |
| ------------------- | ----------- | --------------- | -- |
| TTL                 | Yes         | Yes             | ❌  |
| Size-based eviction | HybridCache | Yes (W-TinyLFU) | ❌  |
| Background refresh  | Manual      | Built-in        | ❌  |
| Multi-level cache   | Yes         | No              | ❌  |

🏆 **Caffeine wins eviction sophistication**
🏆 **.NET wins multi-layer caching**
🏆 **Go keeps things orthogonal**

---

## 6️⃣ Performance Characteristics

### Latency under stampede

| Platform   | Result                   |
| ---------- | ------------------------ |
| .NET async | Lowest                   |
| Go         | Near-zero                |
| Java       | Higher (blocked threads) |

### Throughput under load

| Platform | Behavior               |
| -------- | ---------------------- |
| .NET     | Scales with async I/O  |
| Java     | Limited by thread pool |
| Go       | Massive concurrency    |

---

## 7️⃣ Elegance vs Explicitness

| Dimension        | .NET           | Java              | Go                 |
| ---------------- | -------------- | ----------------- | ------------------ |
| Boilerplate      | Very low       | Low               | Medium             |
| Explicit control | Medium         | Low               | High               |
| Footguns         | Cancellation   | Thread exhaustion | Forgetting cache   |
| Philosophy       | Runtime-driven | Library-driven    | Minimal primitives |

---

## 8️⃣ Design Philosophy Comparison

### .NET

> *“Async is the primitive.”*

Single-flight emerges naturally from `Task<T>`.

---

### Java

> *“Threads are the primitive.”*

Single-flight implemented via blocking + eviction heuristics.

---

### Go

> *“Coordination, not caching.”*

`singleflight` intentionally avoids policy.

---

## 9️⃣ Who Wins What?

| Use Case                        | Winner      |
| ------------------------------- | ----------- |
| Async microservices             | 🟦 .NET     |
| High-throughput in-memory cache | 🟨 Caffeine |
| Lightweight RPC deduplication   | 🟥 Go       |
| Multi-tier caching              | 🟦 .NET     |
| Explicit control                | 🟥 Go       |

---

## 🏁 Final Verdict

> **The most elegant single-flight implementation depends on the runtime model:**

* **.NET** → *Task-based caching is the cleanest*
* **Java** → *Caffeine is unmatched for eviction*
* **Go** → *`singleflight` is minimal and perfect*

There is **no universal winner**, only alignment with the language’s concurrency philosophy.

If we’re handing out the “Single-Flight Oscar” 🏆 based purely on **elegance, safety, and developer ergonomics in 2025**, the winner is:

### **🥇 .NET (Task-based single-flight caching)**

Here’s why it takes the gold:

---

### 1️⃣ Async-first by design

* No threads blocked
* Works naturally with `async/await`
* Fits modern microservices and I/O-heavy workloads

### 2️⃣ Minimal boilerplate

* `GetOrCreateAsync` + `Task<T>` sharing = **no locks, no semaphores**
* Failure-safe and exception-aware
* Cancellation-aware if done right

### 3️⃣ Multi-tier ready

* Supports L1 memory + L2 distributed cache
* Handles cache eviction gracefully
* Plays well with background refresh / stale-while-revalidate

### 4️⃣ Low footgun risk

* No deadlocks
* No thread pool exhaustion
* Natural single-flight semantics

---

### 🥈 Silver: Go `singleflight`

* Brilliantly minimal and explicit
* Extremely lightweight under high concurrency
* But it’s **not a cache**; TTL, eviction, multi-layer caching all have to be added manually

### 🥉 Bronze: Java Caffeine

* Very fast and sophisticated for CPU-bound caches
* Built-in eviction policies like W-TinyLFU
* But it **blocks threads**, which makes async/reactive systems less elegant

---

💡 **TL;DR**:

> .NET wins the **“most elegant, production-ready single-flight caching” Oscar** in 2025.

It’s async-native, safe, concise, and multi-tier ready—basically what other languages are still hacking around to replicate.

---
