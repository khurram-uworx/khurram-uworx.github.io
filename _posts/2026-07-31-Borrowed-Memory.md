---
title: "Borrowed Memory"
date: 2026-07-31
series:
  - "Nivara"
  - "The C# Diaries"
tags:
  - C#
  - System.Buffers
  - Rust
  - Nivara
  - Performance
comments: true
---

The loss curve started at NaN.

Not after a hundred steps. Not after a thousand.

Step one.

I stared at the terminal. Everything downstream of a NaN input is NaN. The entire run was a wash.

This was Nivara — the C# dataframe library I wrote about in [LLMs as Equalizers](https://khurram-uworx.github.io/2026/01/12/LLMs-Equalizers.html). I had just wired up Adam, the optimizer. First training run. First step. Garbage.

I went looking. That's the whole bug.

`ArrayPool.Rent()` does not zero buffers.

The pool handed me an array that belonged to someone else five seconds ago. I treated it like fresh memory. The first step of training read values nobody had written.

I remember the thought that crossed my mind.

And I should be honest about this: I'm not a Rust developer.

I've read about Rust the way you read about a place you admire from afar. Fascinated. Impressed. Still here, still in C#, because the language keeps unwrapping layers below the ones I thought were the bottom — before I ever feel the need to leave.

In Rust, this is a compile error.

Uninitialized memory doesn't exist in safe Rust. Reach for a buffer that isn't yours, and the compiler stops you. Before it ships. Before it poisons a run. At the moment you wrote it.

In C#, it was a silent first step.

And that moment opened a door I had been walking past for years.

---

## Rent

I'd seen `ArrayPool<T>.Shared.Rent(...)` in ASP.NET source code. Kestrel swims in it. I'd read the tutorials, nodded along, closed the tab. I had never needed it.

Then I built a dataframe library.

Dataframe libraries are unglamorous. They don't do one thing fast; they do a thousand things adequately. Sort, filter, join, group, transpose, aggregate. Every operation touches every element.

And every operation, written naively, does this:

```csharp
var scratch = new T[n];
```

The garbage collector was built for exactly this. Short-lived objects. Gen-0. Cheap. Allocation is fast.

But this wasn't one allocation. This was a matmul over a million rows. A sort over a million elements. Every `new T[n]` became a small pause, inside a loop, repeated.

The GC is a feature. It's also a tax. And in the hot paths of a data library, you want to pay that tax zero times.

I'd already walked most of the way down this ladder once. In [The C# You Don't Know](https://khurram-uworx.github.io/2026/06/12/YouDontKnow.html), I followed a single function down through the layers of the language — `int[]`, then `IEnumerable<T>`, then `IReadOnlyList<T>`, then `ReadOnlySpan<T>`. Each layer was a promise about the data. Span was the bottom I'd found: *a view into memory, nothing more.*

There was a layer below even that.

I just hadn't looked behind the view.

So you rent:

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(1024);

try
{
    // use it
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer);
}
```

`Rent` borrows an array from a shared pool. `Return` gives it back. The next caller receives the same memory. Same bytes, reused, forever. No allocation. No collection. No pause.

Microsoft's own documentation describes `Rent` in the words I had already reached for: *this buffer is loaned to the caller.*

And the numbers make the argument concrete. Their example reads a thousand files — `new byte[n]` on every read — and watches the total allocation climb past a megabyte. Rented from a pool, the same loop allocates barely a kilobyte. Same work. One buffer. Recycled a thousand times.

The discipline shows up in the code as a threshold. Nivara rents big, allocates small:

```csharp
var rowMajor = totalLength >= 1024
    ? (pooledRowMajor = ArrayPool<T>.Shared.Rent(totalLength)).AsSpan(0, totalLength)
    : (new T[totalLength]).AsSpan();
```

1024 elements. Below it, the ceremony of renting isn't worth it. Above it, the hot loop can't be allowed to churn the heap.

There's a catch hiding in that signature. `Rent` takes a *minimum* length. The pool keeps arrays in a handful of bucket sizes, and it hands you whichever bucket fits — sometimes a bigger one than you asked for. The array I get back doesn't know how much of it I actually wanted.

That's why every call site does `.AsSpan(0, totalLength)` right away.

A span carries its own length. A rented array doesn't. The size is mine to remember.

I wrote that line. Then I wrote it again, in another file. Then again.

Everywhere a hot path needed scratch memory, the pattern appeared.

---

## The Borrow Checker That Isn't There

Here's what I didn't see at first.

For weeks, I had been using something shaped exactly like Rust's memory model. Not in Rust — in C#. I just didn't recognize it, because it was wearing C#'s clothes.

| C# construct | Rust parallel | What it gives you |
|---|---|---|
| `ArrayPool<T>` | `Vec<T>` reuse / object pools | Reused heap memory, no GC churn |
| `Memory<T>` / `ReadOnlyMemory<T>` | `&[T]` slices | A view into memory, no ownership |
| `Span<T>` / `ReadOnlySpan<T>` | `&[T]` / `&mut [T]` | Stack-only views, lifetime-limited to scope |
| `IMemoryOwner<T>` | RAII wrapper | `Dispose` returns the buffer to the pool |

`ArrayPool<T>` is a `Vec` handed back to the arena instead of dropped.

`Memory<T>` is a slice — a view of a buffer, not the buffer itself.

`Span<T>` is a slice that lives only on the stack, bounded to its scope the way Rust bounds a borrow.

`IMemoryOwner<T>` is RAII — a `Drop` that returns the memory where it found it.

Line by line, the table writes itself.

The ownership rules survive the translation, too. Microsoft's usage guidelines say a memory buffer can have many consumers but *only one owner at a time* — and that a method signature accepting an `IMemoryOwner<T>` means the method is accepting ownership. That isn't a C# idea. That's Rust's ownership model, wearing an interface.

Then the crucial difference.

Rust enforces all of this at compile time. You cannot keep a slice alive past the buffer it points to. You cannot hold two mutable borrows. The compiler proves it, every build, forever.

C# enforces none of it. Nothing stops you.

Keep using a rented array after you've returned it, and the pool may already have handed that memory to someone else. Microsoft's own best-practices guide calls it by the name Rust made famous — a use-after-free. In safe Rust that bug is impossible; the compiler won't let a borrow outlive its owner. In C#, it's a warning in the documentation, and the discipline is yours.

```rust
let mut buffer = vec![0u8; 1024]; // allocated, used, freed — automatically
```

The C# version needed a `finally` and a `Return`. The Rust version needs nothing.

That's not a criticism of either language. It's the difference between a guarantee and an offer.

Rust says: do it this way, and I'll prove it's safe.

C# says: do it this way, and you get the performance — but the discipline is yours.

And the discipline is real. The pool doesn't zero buffers. Someone has to decide who is responsible for the memory before it's yours.

Nivara chose a rule, and the rule is visible in every hot path:

```csharp
var expAvg = ArrayPool<T>.Shared.Rent(size);
expAvg.AsSpan(0, size).Clear();
```

Clear on rent. The previous tenant left their values behind.

And clear on return:

```csharp
ArrayPool<T>.Shared.Return(filled, clearArray: true);
```

So the next tenant doesn't inherit mine.

Every `Rent` in the codebase carries that decision. The compiler wasn't going to make it for me.

I was the borrow checker.

---

## Two Answers to One Question

A garbage collector and a borrow checker are answers to the same question.

Who is responsible for memory?

Rust answers: the compiler. Ownership, lifetimes, borrow rules — enforced, provable, inescapable.

C# answers: the runtime. The GC sweeps up after you. Allocate freely; the collector sorts it out.

Both are reasonable answers. I've spent my career in the second camp.

But here's what building Nivara taught me.

When the performance genuinely matters — in the hot path of a data engine — the second answer isn't enough. You can't let the GC sort it out, because the GC's sort-out is a pause, and the pause lands inside your loop.

So you step toward the first answer. Not the language. The discipline.

You rent. You return. You clear. Byte by byte, you become your own borrow checker.

That's what `System.Buffers` is, honestly: Rust's ownership model, stripped of the compiler, offered to a managed language as a set of classes and a promise.

You get the control.

You supply the guarantees.

The collection is what you see. The span is the view. The pool is the room behind the view — and in the hot path, the room is yours.

The moment a library starts caring about *where* data lives, it has left the managed world anyway. The GC is still there, holding the door. But in the hot path, you're the one walking through it — and you're the one who has to leave the room the way you found it.

I still haven't needed to learn Rust. Not because I'm not curious. Because C# keeps unwrapping its layers, and every time I think I've reached the bottom, there's a floor underneath that turns out to be a Rust idea I'd read about months earlier.

A garbage collector would tell you that's someone else's job.

A borrow checker would tell you it was never anyone else's job.

Somewhere between the two, you find out which one you are.

*Next: [Shape](https://khurram-uworx.github.io/2026/08/01/Shape.html). The engine I rent that memory for runs on spans — and this is the story of the day the platform took the tensor layer back.*

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
