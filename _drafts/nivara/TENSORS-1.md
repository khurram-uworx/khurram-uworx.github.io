# Tensors: The Data Structure Powering ML

> From API calls to understanding what's actually happening underneath.

## What is a Tensor?

In machine learning, "tensor" informally refers to a **multidimensional array** — a structured way to organize data with multiple axes (dimensions). While the mathematical definition involves multilinear transformations over vector spaces, for software engineers the practical definition is simpler:

> A tensor is an N-dimensional array of numbers.

That's it. A spreadsheet is a 2D tensor. A color image is a 3D tensor. A batch of sentences tokenized and embedded is a 4D tensor.

## From Scalars to Tensors

| Concept | Dimensions | Example |
|---------|-----------|---------|
| Scalar | 0D | A single number: `42` |
| Vector | 1D | A list of numbers: `[1, 2, 3]` |
| Matrix | 2D | A grid of numbers: `[[1,2],[3,4]]` |
| Tensor | 3D+ | A cube (or higher) of numbers |

The progression is natural — as our data gets more complex, we need more axes to represent it.

## Why Tensors Matter in ML

Tensors serve four critical roles:

1. **Represent Data** — Text tokens, images, video frames, audio waveforms are all encoded as tensors
2. **Efficient Manipulation** — Operations on millions of numbers need to be fast; tensors enable vectorized computation
3. **Neural Network Weights** — Every learned parameter in a model lives inside a tensor
4. **Intermediate Computation** — During training and inference, activations and gradients flow as tensors

## Real Tensor Shapes

Understanding shapes is key to understanding ML:

```
Grayscale Image:     [Height, Width]                    → 2D tensor
Color Image:         [Height, Width, Channels]           → 3D tensor (e.g., 224×224×3 for RGB)
Batch of Images:     [Batch, Height, Width, Channels]    → 4D tensor (e.g., 32 images at once)
Text Tokens:         [Batch, SequenceLength]             → 2D tensor
Embedded Text:       [Batch, SequenceLength, EmbedDim]   → 3D tensor (e.g., 512-dimensional embeddings)
Video:               [Frames, Height, Width, Channels]   → 4D tensor
```

## Key Tensor Operations

### Element-wise Arithmetic
The simplest operations apply to each element independently:
```
[1, 2, 3] + [4, 5, 6] = [5, 7, 9]
[1, 2, 3] * 2         = [2, 4, 6]
```

### Dot Product
Measures similarity between two vectors — the foundation of cosine similarity:
```
[1, 2, 3] · [4, 5, 6] = 1×4 + 2×5 + 3×6 = 32
```

### Matrix Multiplication
How neural network layers transform data:
```
Weights (2×3) × Input (3×1) = Output (2×1)
```

### Reshape
Changing dimensions without changing data — critical for moving between layers:
```
[1, 2, 3, 4, 5, 6] → reshape(2, 3) → [[1,2,3], [4,5,6]]
```

### Broadcasting
Automatically expanding a smaller tensor to match a larger one:
```
[1, 2, 3] + 10 = [11, 12, 13]
```

## Vectors in ML

Vectors are 1D tensors and the building blocks of everything:

- **Sparse Vectors** — Mostly zeros, only meaningful positions filled (e.g., word presence in a document)
- **Dense Vectors** — All positions have meaningful values (e.g., word embeddings from neural networks)
- **One-Hot Vectors** — Binary vectors with exactly one `1` (e.g., categorical encoding: `Urban=[1,0]`, `Rural=[0,1]`)
- **Embedding Vectors** — Dense, low-dimensional representations learned by models (e.g., Word2Vec, sentence transformers)

```csharp
// Sparse representation: store only non-zeros
// Dense: [0, 0, 3, 0, 0, 7, 0]
// Sparse: indices=[2,5], values=[3,7]

// One-hot encoding for categorical data
// House Type: Urban=[1,0], Rural=[0,1]

// Embedding: learned dense representation
// "king" → [0.25, -0.1, 0.7, 0.3, ...] (e.g., 300 dimensions)
```

## Floating Point Precision in ML

ML models are sensitive to numerical precision. The trade-offs:

| Type | Bytes | Precision | Use Case |
|------|-------|-----------|----------|
| FP8 | 1 | ~2 decimals | Inference acceleration (NVIDIA Tensor Cores) |
| FP16 (Half) | 2 | ~3-5 decimals | Training and inference, memory efficient |
| FP32 (float) | 4 | ~6-9 decimals | Standard training precision |
| FP64 (double) | 8 | ~15-17 decimals | Scientific computing, rarely needed in ML |

**Why normalize to [-1, 1]?**
- Neural networks with sigmoid/tanh activations perform better with small inputs
- IEEE 754 floats have higher precision near zero
- Prevents overflow/underflow in exponentiation operations common in ML

## The Python Tensor Landscape

Before diving into .NET, here's what the Python world looks like:

- **NumPy** (`ndarray`) — The foundation; not explicitly "tensors" but functions as the basic array type
- **TensorFlow** (`tf.Tensor`) — GPU-optimized, object-oriented API, production-focused
- **PyTorch** (`torch.Tensor`) — GPU-optimized, Pythonic API, research-focused
- **JAX** — Functional approach, automatic differentiation, built on NumPy arrays

Each has its own tensor type, its own operations, and its own ecosystem. This fragmentation is exactly what .NET aims to solve with a unified approach.

## Key Takeaway

Tensors aren't magic — they're structured arrays with hardware-accelerated operations. Understanding shapes, operations, and precision is enough to grasp how ML models process data. The real sophistication is in how models learn which operations to apply and which values to store.

---

*Sources: ML-1.ipynb, ML-2.ipynb, ML-3.ipynb, Wikipedia (Tensor - machine learning)*
