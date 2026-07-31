# .NET Tensors: Platform-Level Support for ML

> How .NET 10 quietly shipped one of the best enterprise-grade tensor implementations.

## The Problem: Fragmented Vector/Matrix Libraries

Before .NET 10, every ML-adjacent library in .NET brought its own vector and matrix types:

```csharp
// ML.NET has its own vector types
// TorchSharp has its own Tensor type (libtorch-based)
// NumSharp has ndarray (last release: 2021)
// Accord.NET has its own Matrix/Vector types
// MathNet.Numerics has its own Vector<T>
// NumFlat has Vec<T>
```

**The consequence**: namespace conflicts, inability to mix libraries, no shared SIMD optimizations, and every library reimplementing the same dot product from scratch. If you wanted to use MathNet's cosine similarity with TorchSharp's embeddings, you were copying data between incompatible representations.

## The Evolution: .NET 8 → .NET 9 → .NET 10

### .NET 8: TensorPrimitives (40 overloads)
- Static class with span-based math operations
- **Only `float` overloads** — no generics
- SIMD-accelerated under the hood
- Useful but limited

### .NET 9: Tensor\<T\> (Experimental)
- New `Tensor<T>` type for multi-dimensional data
- Expanded to ~200 overloads in `TensorPrimitives`
- **Generic support** — works with `float`, `double`, `Half`, `int`, and any `INumber<T>`
- Marked `[Experimental]` — API could still change
- Interop with ML.NET, TorchSharp, ONNX Runtime

### .NET 10: Stable Release (GA)
- **APIs are no longer experimental** — fully reviewed and finalized
- New `IReadOnlyTensor` nongeneric interface for metadata access (Lengths, Strides)
- **Zero-copy slice operations** — slicing no longer copies data
- **C# 14 extension operators** — arithmetic works if `T` supports it via generic math interfaces
- Still requires the `System.Numerics.Tensors` NuGet package

## The Key Types

```csharp
using System.Numerics.Tensors;

// Core types:
Tensor<T>              // Mutable multi-dimensional tensor
TensorSpan<T>          // Span-like view over tensor data
ReadOnlyTensorSpan<T>  // Read-only span-like view
IReadOnlyTensor        // Nongeneric interface for metadata

// Static helper class:
Tensor                  // Factory methods for creation, manipulation
TensorPrimitives        // SIMD-accelerated span operations
```

## Tensor\<T\> in Practice

### Creating and Manipulating Tensors

```csharp
using System.Numerics.Tensors;

// Create a 1×3 tensor
Tensor<int> t0 = Tensor.Create([1, 2, 3], [1, 3]);  // [[1, 2, 3]]

// Reshape to 3×1
Tensor<int> t1 = t0.Reshape(3, 1);  // [[1], [2], [3]]

// Slice: get everything from index 1 onwards
Tensor<int> t2 = t1.Slice(1.., ..);  // [[2], [3]]

// Broadcast: expand to 3×3
Tensor<int> t3 = Tensor.Broadcast<int>(t1, [3, 3]);
// [[1, 1, 1],
//  [2, 2, 2],
//  [3, 3, 3]]
```

### C# 14 Extension Operators

The arithmetic operations are available based on what `T` supports:

```csharp
// This works — int supports addition
Tensor<int> a = Tensor.Create([1, 2, 3], [3]);
Tensor<int> b = Tensor.Create([4, 5, 6], [3]);
Tensor<int> sum = a + b;  // [5, 7, 9]

// This also works — float supports everything
Tensor<float> x = Tensor.Create([1.0f, 2.0f, 3.0f], [3]);
Tensor<float> y = x * 2.0f;  // [2.0f, 4.0f, 6.0f]

// This does NOT compile — bool doesn't support addition
// Tensor<bool> c = Tensor.Create([true, false], [2]);
// Tensor<bool> d = c + c;  // ❌ compilation error
```

## TensorPrimitives: SIMD on Steroids

The `TensorPrimitives` class is where the real performance lives. ~200 overloads operating on `ReadOnlySpan<T>` with automatic SIMD acceleration.

### Cosine Similarity (The Star of the Show)

```csharp
using System.Numerics.Tensors;

ReadOnlySpan<float> vector1 = [1, 2, 3];
ReadOnlySpan<float> vector2 = [4, 5, 6];
Console.WriteLine(TensorPrimitives.CosineSimilarity(vector1, vector2));
// 0.9746318

// Works with double too
ReadOnlySpan<double> v3 = [1, 2, 3];
ReadOnlySpan<double> v4 = [4, 5, 6];
Console.WriteLine(TensorPrimitives.CosineSimilarity(v3, v4));
// 0.9746318461970762
```

### Other Key Operations

```csharp
// Dot product
float dot = TensorPrimitives.Dot(x, y);

// Element-wise operations
TensorPrimitives.Add(span1, span2, destination);
TensorPrimitives.Multiply(span, scalar, destination);
TensorPrimitives.Subtract(span1, span2, destination);

// Activation functions (critical for neural networks)
TensorPrimitives.Softmax(values, destination);
TensorPrimitives.Sigmoid(values, destination);
TensorPrimitives.Tanh(values, destination);

// Normalization
TensorPrimitives.Normalize(values, destination);

// Reductions
float sum = TensorPrimitives.Sum(values);
float max = TensorPrimitives.Max(values);
```

### Why SIMD Matters

Without SIMD, processing 8 floats requires 8 sequential operations. With AVX2, it's 1 operation on 256-bit registers:

```csharp
// This simple line:
TensorPrimitives.CosineSimilarity(queryEmbedding, movieEmbedding);

// Internally uses SIMD to process multiple values per CPU instruction
// On AVX2 hardware: 8 floats processed simultaneously
// On AVX-512 hardware: 16 floats processed simultaneously
// No manual vectorization needed — it's automatic
```

## Real-World Example: Movie Similarity Search

From the dotnetdebrief notebooks — a practical semantic search:

```csharp
using System.Numerics.Tensors;

var movies = new[]
{
    new { Title="The Lion King", Embedding= new [] { 0.10022575f, -0.23998135f } },
    new { Title="Inception", Embedding= new [] { 0.10327095f, 0.2563685f } },
    new { Title="Toy Story", Embedding= new [] { 0.095857024f, -0.201278f } },
    new { Title="Pulp Fiction", Embedding= new [] { 0.106827796f, 0.21676421f } },
    new { Title="Shrek", Embedding= new [] { 0.09568083f, -0.21177962f } }
};

var queryEmbedding = new[] { 0.12217915f, -0.034832448f }; // Looking for something like Toy Story

var top3Movies = movies.Select(movie =>
    (
        movie.Title,
        Similarity: TensorPrimitives.CosineSimilarity(queryEmbedding, movie.Embedding)
    ))
    .OrderByDescending(m => m.Similarity)
    .Take(3);

foreach (var movie in top3Movies)
    Console.WriteLine(movie);
// Output: (Toy Story, 0.99...), (Shrek, 0.98...), (The Lion King, 0.97...)
```

### With Real Embeddings (Sentence Transformers)

```csharp
using SentenceTransformers.MiniLM;
using System.Numerics.Tensors;

var encoder = new SentenceEncoder();
var sentences = new[] { "A man is eating pasta", "A man is riding a horse" };
var embeddings = sentences.Select(s => encoder.Encode([s])[0]).ToArray();

var query = encoder.Encode(["A man is eating food"])[0];

var results = embeddings.Select((emb, i) =>
    (Sentence: sentences[i],
     Similarity: TensorPrimitives.CosineSimilarity(query, emb)))
    .OrderByDescending(r => r.Similarity);

foreach (var r in results)
    Console.WriteLine($"{r.Sentence}: {r.Similarity:F4}");
// "A man is eating pasta": 0.92xx (closest match)
// "A man is riding a horse": 0.78xx
```

## Why This Matters for Enterprise .NET

### 1. First-Class Runtime Support
This isn't a third-party library — it's part of the .NET runtime ecosystem. Same team that maintains `System.Math`, `System.Numerics.Vector<T>`, and the JIT compiler maintains this. It gets the same performance attention as `Span<T>` and `Memory<T>`.

### 2. Automatic SIMD Acceleration
No need to check `Avx2.IsSupported` or write platform-specific code. `TensorPrimitives` uses `Vector<T>`, `Vector128<T>`, `Vector256<T>`, and `Vector512<T>` internally with software fallbacks. Your code runs fast everywhere.

### 3. Generic Math Integration
With .NET 7+'s generic math (`INumber<T>`, `IAdditionOperators<TSelf, TOther, TResult>`, etc.), `Tensor<T>` works with any numeric type that implements the interfaces. Write once, run with `float`, `double`, `Half`, `int`, or even custom numeric types.

### 4. Zero-Copy Interop
The tensor types are designed for zero-copy interop with ML.NET, TorchSharp, and ONNX Runtime. When you load a model and get predictions, the data doesn't need to be copied between representations.

### 5. Span-Based Design
Everything operates on `ReadOnlySpan<T>` and `Span<T>`. This means:
- Stack allocation for small tensors
- No GC pressure for large operations
- Seamless integration with `ArrayPool<T>`, `MemoryPool<T>`, and `IBufferWriter<T>`

### 6. Reduces Library Dependencies
Before: needed MathNet.Numerics for vector math, Accord.NET for ML operations, custom code for cosine similarity.
Now: `System.Numerics.Tensors` covers the fundamentals. Specialized libraries can focus on higher-level algorithms.

## The Journey

```
2018: System.Numerics.Tensors NuGet package (basic)
2023: .NET 8 — TensorPrimitives (40 overloads, float only)
2024: .NET 9 — Tensor<T> (experimental, ~200 overloads, generics)
2025: .NET 10 — Tensor<T> STABLE (zero-copy slices, C# 14 operators, nongeneric interface)
```

## Key Takeaway

.NET 10 didn't just add tensor support — it added **platform-level** tensor support with the same philosophy that brought us `Span<T>`, `Memory<T>`, and generic math. For enterprise .NET developers building ML-powered applications, this means:

- No more "which vector library should I use?" paralysis
- SIMD acceleration without writing intrinsics code
- Type-safe operations that work with any numeric type
- First-class support that will be maintained and optimized for years

The Python ecosystem has NumPy, PyTorch, and TensorFlow. .NET now has `System.Numerics.Tensors` — not as feature-rich, but deeply integrated into the runtime with enterprise-grade stability.

---

*Sources: Microsoft Learn (.NET 10 What's New in Libraries), ML-2.ipynb, ML-3.ipynb, dotnet/runtime GitHub*
