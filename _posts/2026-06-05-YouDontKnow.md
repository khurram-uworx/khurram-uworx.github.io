---
title: "The C# You Don't Know"
date: 2026-06-12
tags:
  - C#
comments: true
---
It started with a single keyword.

I was reading some code when I came across this:

```csharp
static (a, b, c) => a + b + c
```

I stopped.

Not because the code was complicated.

Because I had no idea that was legal C#.

Static lambdas?

Since when?

I've been writing C# for years. I've written services, APIs, libraries, desktop applications, cloud workloads. I've spent enough time in the language that I rarely encounter syntax I don't recognize.

Yet there it was.

A feature I didn't know existed.

The answer, as it turns out, is that static lambdas arrived in C# 9.

C# 9.

Not last month.

Not last year.

Years ago.

That's when an uncomfortable thought crossed my mind.

If this happened to me, it's probably happening to a lot of other developers too.

Because most of us don't actually know C#.

We know *a version* of C#.

Usually the version we spent the most time with.

And that's an important distinction.

---

## The Snapshot Problem

Every developer carries around a mental snapshot of their language.

For some people it's C# 5.

For others it's C# 7.

Maybe it's whatever version was current when they built the project that defined a chunk of their career.

The snapshot becomes the language.

Everything else becomes "new features."

Every developer carries a snapshot of their language too.

The problem is that C# never stopped evolving.

Not dramatically.

Not loudly.

Quietly.

Relentlessly.

While many languages reinvent themselves through revolutions, C# evolves through accumulation.

Features arrive.

They fit naturally into the language.

They don't force rewrites.

They don't invalidate old code.

They simply become available.

Which means it's entirely possible to write C# every day and still miss years of language evolution.

That's exactly what happened to me.

And maybe to you.

---

## The Features You Might Have Missed

Once I noticed that gap in my knowledge, I started noticing others.

Features I'd seen before but never explored.

Features I'd heard about and mentally filed under "I'll look at that later."

Features that had quietly become part of modern C#.

---

### Target-typed new

For example, object creation used to look like this:

```csharp
Dictionary<string, int> counts =
    new Dictionary<string, int>();
```

Today:

```csharp
Dictionary<string, int> counts = new();
```

The compiler already knows the type.

Why repeat yourself?

Simple.

Obvious.

Easy to take for granted.

---

### Collection expressions

Even collections became cleaner.

Instead of:

```csharp
var values = new[] { 1, 2, 3, 4 };
```

Modern C# allows:

```csharp
var values = [1, 2, 3, 4];
```

Or:

```csharp
var combined =
[
    ..defaults,
    42,
    ..overrides
];
```

The syntax feels natural.

As if it should have always existed.

---

### Pattern matching

This one stopped me the most.

What once looked procedural:

```csharp
if (user != null &&
    user.IsActive &&
    user.Role == "Admin")
{
    ...
}
```

Can now express shape directly:

```csharp
if (user is
{
    IsActive: true,
    Role: "Admin"
})
{
    ...
}
```

The code reads closer to the idea it's expressing.

Not "check these three conditions."

Not "make sure this isn't null."

But "is this thing an admin?"

That shift — from telling the computer *how* to check to describing *what* you're looking for — is the quiet revolution of modern C#.

---

## One Function, One Career

The breadth is easy to spot once you start looking.

But there's another kind of "Wait" I've been thinking about.

It's the one that doesn't happen across the language.

It happens across your career.

---

### The junior version

I can almost see him.

Someone just starting out. Green. Eager. Writing his first real function.

```csharp
static bool ShapeEquals(int[] left, int[] right)
{
    if (left.Length != right.Length)
        return false;

    for (int i = 0; i < left.Length; i++)
    {
        if (left[i] != right[i])
            return false;
    }

    return true;
}
```

He was proud of this.

It worked. It was correct. It was his.

And that's the thing. He didn't know it yet, but this tiny function was about to teach him the entire language.

---

### The LINQ awakening

Then he discovered LINQ.

```csharp
static bool ShapeEquals(int[] left, int[] right)
    => left.SequenceEqual(right);
```

He stared at the screen.

One line?

Wait. C# can do that?

He replaced everything he could find with LINQ. Every loop became a query. Everything looked like a nail.

I remember doing the same thing.

---

### The abstraction question

But LINQ wasn't the end for him either. It was the beginning of a harder question.

What does this function *actually* need?

```csharp
static bool ShapeEquals(IEnumerable<int> left, IEnumerable<int> right)
    => left.SequenceEqual(right);
```

`IEnumerable<int>` says: I only need to enumerate.

But his original algorithm used `Length` and indexers. Those are random-access operations. Switching to `IEnumerable` without thinking means either losing efficiency or rewriting everything.

So what does this function actually need?

Count and indexing.

```csharp
static bool ShapeEquals(IReadOnlyList<int> left, IReadOnlyList<int> right)
    => left.SequenceEqual(right);
```

He started reading every type as a promise:

| Type | What it promises |
|---|---|
| `IEnumerable<T>` | I can enumerate one-by-one |
| `IReadOnlyCollection<T>` | I can enumerate and know the count |
| `IReadOnlyList<T>` | I can enumerate, count, and index |
| `T[]` | I specifically require an array |

Most developers jump from `T[]` straight to `IEnumerable<T>`. But the forgotten middle — `IReadOnlyCollection<T>`, `IReadOnlyList<T>` — often describes the requirements far more accurately.

He was learning that. Slowly. One function at a time.

I was learning it too.

---

### The memory layer

Then he found Span.

```csharp
static bool ShapeEquals(ReadOnlySpan<int> left, ReadOnlySpan<int> right)
    => left.SequenceEqual(right);
```

He stopped.

`ReadOnlySpan<int>` isn't a collection. It's not a sequence.

It's *memory*.

Contiguous memory. Read-only view. No allocation. Works with arrays, slices, stackalloc buffers.

The JIT sees `left[i]` as direct memory access. Zero allocations. Bounds-check elimination. Vectorization.

He felt like he was seeing C# for the first time.

There's a whole layer of the language he had never touched. A layer designed for parsers, serializers, tensor operations, image processing, numerical computing. Code that cares about *where* data lives, not just *what* data contains.

Span is stack-only. It can't be stored in fields. It can't cross async boundaries. It can't be captured by lambdas.

And that's the point.

It's not for everyday business logic.

It's for when performance matters and you need to say: *this is a view into memory, nothing more.*

---

### The bottom that isn't there

But even Span wasn't the end.

He kept digging.

```csharp
using System.Runtime.CompilerServices;

[MethodImpl(MethodImplOptions.AggressiveInlining)]
static bool ShapeEquals(ReadOnlySpan<int> left, ReadOnlySpan<int> right)
    => left.SequenceEqual(right);
```

That `SequenceEqual` — for spans — is not the LINQ `SequenceEqual` he knew. It's a highly optimized memory comparison. The JIT can vectorize it. SIMD instructions. The framework already solved this, with the kind of performance he didn't even know to ask for.

Beyond that? Tensor primitives. `System.Numerics`. `System.Buffers`. Math-optimized routines baked into the runtime itself.

```csharp
int[] a = [1, 3, 224, 224];
int[] b = [1, 3, 224, 224];

bool same = ShapeEquals(a, b);
```

Same function call. Same arrays.

Completely different world under the hood.

---

He thought he had found the bottom.

There was no bottom.

That single function — `ShapeEquals` — took him from junior to senior to platform engineer to performance specialist. Not because he got smarter. Because the language kept revealing new layers every time he was ready to see them.

I know. I was there.

---

## The Clever Trick C# Pulled Off

Here's what I find most impressive.

Most languages face a difficult choice.

Move forward aggressively.

Or preserve familiarity.

Innovation often comes at the cost of stability.

C# spent two decades trying to have both.

A codebase from fifteen years ago still looks recognizably like C#.

A codebase written today can look remarkably modern.

Both compile.

Both are valid.

Both belong to the same language.

That's not easy.

Adding features is straightforward.

Adding features while making old code feel respected is much harder.

---

## The C# You Don't Know

The lesson from my static lambda discovery wasn't that I had missed a feature.

It was that I had mistaken familiarity for completeness.

I thought I knew C#.

What I actually knew was the version of C# that had settled into my habits.

The language had continued evolving while I was busy writing software.

And that's perfectly normal.

Most developers aren't actively studying release notes every year.

We're shipping products.

Fixing bugs.

Meeting deadlines.

Building things.

Which means every now and then we encounter a piece of code that reminds us the language has been growing without asking for our attention.

That's a good thing.

It means the language is alive.

And it means there are probably dozens of features waiting just outside the borders of our mental snapshot.

Which is why seasoned C# developers keep having the same experience.

They encounter a piece of code.

Pause.

Lean closer.

And ask the most delightful question a programming language can provoke:

**"Wait. C# can do that?"**
