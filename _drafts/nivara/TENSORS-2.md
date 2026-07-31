# The .NET ML Ecosystem: Libraries, Tensors, and Interoperability

> Before Tensor<T>, how did .NET developers actually do ML? And why does a unified tensor type matter?

## The .NET ML Landscape

.NET has three major ML-related libraries, each solving different problems:

| Library | Purpose | Tensor Type | Status |
|---------|---------|-------------|--------|
| **ML.NET** | Traditional ML (classification, regression, clustering) | Internal (`VBuffer<T>`) | Stable, Microsoft-supported |
| **TorchSharp** | Deep learning (PyTorch port) | `torch.Tensor` | Active, libtorch-based |
| **ONNX Runtime** | Model inference (production deployment) | `OrtValue` | Stable, cross-platform |

Each brought its own data representations. If you wanted to use all three in one application, you were copying data between incompatible types.

## ML.NET: The Enterprise Workhorse

ML.NET is Microsoft's official ML framework for .NET. It's designed for **traditional ML tasks** — classification, regression, clustering, recommendation — not deep learning.

### How ML.NET Works

```csharp
using Microsoft.ML;

// MLContext is your entry point (like DbContext in EF Core)
var mlContext = new MLContext();

// Define a pipeline of estimators
var pipeline = mlContext.Transforms.Text
    .FeaturizeText("Features", nameof(SentimentData.Text))
    .Append(mlContext.BinaryClassification.Trainers
        .SdcaLogisticRegression(labelColumnName: "Label", featureColumnName: "Features"));

// Train the model
var model = pipeline.Fit(trainingData);

// Make predictions
var predictor = mlContext.Model.CreatePredictionEngine<SentimentData, SentimentPrediction>(model);
var result = predictor.Predict(new SentimentData { Text = "This movie is great!" });
Console.WriteLine($"Prediction: {result.Prediction}, Probability: {result.Probability:F4}");
```

### ML.NET and TensorFlow/ONNX Integration

ML.NET can load and use pre-trained TensorFlow and ONNX models:

```csharp
// Load a TensorFlow model into ML.NET
var pipeline = mlContext.Transforms.ScoreTensorFlowModel(
    modelLocation: "./inception",
    outputColumnNames: new[] { "softmax2_pre_activation" },
    inputColumnNames: new[] { "input" });

// Or load an ONNX model
var pipeline = mlContext.Transforms.ApplyOnnxModel(
    modelFile: "./model.onnx",
    outputColumnNames: new[] { "output" },
    inputColumnNames: new[] { "input" });
```

**Key insight**: ML.NET wraps TensorFlow and ONNX Runtime under the hood. The `VBuffer<T>` type is ML.NET's internal representation for feature vectors — it handles sparse/dense efficiently but doesn't expose raw tensor operations.

### ML.NET's TorchSharp Integration (Experimental)

Starting with ML.NET 4.0, there's experimental support for TorchSharp-based models:

```csharp
// Experimental: Using TorchSharp models in ML.NET pipelines
// Llama, Phi family of models
// CausalLM pipeline APIs
```

This bridges the gap between ML.NET's traditional ML and deep learning.

## TorchSharp: PyTorch for .NET

TorchSharp is a .NET wrapper around libtorch (PyTorch's C++ backend). If you know PyTorch, you know TorchSharp.

### Core Tensor Operations

```csharp
using TorchSharp;
using static TorchSharp.torch;

// Create tensors (just like PyTorch)
var t1 = torch.tensor(new float[] { 1, 2, 3 });  // 1D tensor
var t2 = torch.tensor(new float[,] { { 1, 2 }, { 3, 4 } });  // 2D tensor

// Operations
var result = t1 + t1;           // Element-wise addition
var matmul = torch.matmul(t1, t2);  // Matrix multiplication
var softmax = torch.softmax(t1, dim: 0);

// GPU support (if CUDA available)
var gpuTensor = t1.cuda();
var cpuTensor = gpuTensor.cpu();
```

### Building Neural Networks

```csharp
using TorchSharp;
using static TorchSharp.torch.nn;

// Define a model (like PyTorch's nn.Module)
var model = Sequential(
    ("linear1", Linear(784, 256)),
    ("relu1", ReLU()),
    ("dropout1", Dropout(0.2)),
    ("linear2", Linear(256, 128)),
    ("relu2", ReLU()),
    ("output", Linear(128, 10))
);

// Training loop
var optimizer = torch.optim.Adam(model.parameters(), lr: 0.001);
var loss = torch.nn.CrossEntropyLoss();

for (int epoch = 0; epoch < 10; epoch++)
{
    foreach (var (images, labels) in trainLoader)
    {
        optimizer.zero_grad();
        var predictions = model.forward(images);
        var lossValue = loss.forward(predictions, labels);
        lossValue.backward();
        optimizer.step();
    }
}
```

### TorchSharp's Tensor Type

`torch.Tensor` is TorchSharp's tensor type. It's backed by libtorch and supports:
- Multi-dimensional arrays (any rank)
- Automatic differentiation (autograd)
- GPU acceleration (CUDA)
- In-place and out-of-place operations

**The problem**: `torch.Tensor` is specific to TorchSharp. If you want to use ML.NET's vector operations or System.Numerics.Tensors, you need to copy data.

## ONNX Runtime: Production Inference

ONNX Runtime is the go-to for deploying ML models in production. It's cross-platform, hardware-accelerated, and supports models from PyTorch, TensorFlow, scikit-learn, and more.

### Loading and Running ONNX Models

```csharp
using Microsoft.ML.OnnxRuntime;
using Microsoft.ML.OnnxRuntime.Tensors;

// Create an inference session
using var session = new InferenceSession("model.onnx");

// Prepare input (ONNX Runtime's tensor type)
var inputTensor = new DenseTensor<float>(
    new[] { 1, 3, 224, 224 },  // Batch, Channels, Height, Width
    false);

// Fill with image data...
// (normalize pixels, etc.)

// Create input metadata
var inputs = new List<NamedOnnxValue>
{
    NamedOnnxValue.CreateFromTensor("input", inputTensor)
};

// Run inference
using var results = session.Run(inputs);

// Get output
var output = results.First().AsTensor<float>().ToArray();
```

### ONNX Runtime's Tensor Type

`Microsoft.ML.OnnxRuntime.Tensors.DenseTensor<T>` is ONNX Runtime's tensor type. It's a standard multi-dimensional array implementation but doesn't have the rich operations of TorchSharp or the SIMD acceleration of TensorPrimitives.

## The Interoperability Problem

Before `System.Numerics.Tensors`, each library had its own tensor type:

```
ML.NET:         VBuffer<float>    → feature vectors
TorchSharp:     torch.Tensor      → neural network tensors
ONNX Runtime:   DenseTensor<float> → inference inputs/outputs
System.Numerics: Vector<T>        → SIMD-accelerated vectors (1D only)
```

### The Copy-Paste Tax

```csharp
// Scenario: Use TorchSharp to train, ONNX Runtime to serve

// 1. Train in TorchSharp
var model = new MyModel();
var trainingData = torch.tensor(new float[,] { ... });
var predictions = model.forward(trainingData);

// 2. Export to ONNX
torch.jit.export(model, "model.onnx");

// 3. Load in ONNX Runtime (different tensor type!)
using var session = new InferenceSession("model.onnx");
var inputTensor = new DenseTensor<float>(new[] { 1, 784 });  // Different type!
// Copy data from torch.Tensor to DenseTensor<float>
for (int i = 0; i < 784; i++)
    inputTensor[0, i] = predictions[0, i].item<float>();  // Element-by-element copy!

// 4. Run inference
var results = session.Run(inputs);
```

### What Was Missing

1. **No shared tensor type** — every library reinvented the wheel
2. **No SIMD acceleration for multi-dimensional data** — Vector<T> was 1D only
3. **No zero-copy interop** — data had to be copied between representations
4. **No generic math** — each library specialized for float/double only
5. **No common operations** — cosine similarity, dot product, normalization reimplemented everywhere

## What .NET 10 Changes

`Tensor<T>` (now stable in .NET 10) solves these problems:

### 1. Common Representation

```csharp
using System.Numerics.Tensors;

// This works with ML.NET, TorchSharp, and ONNX Runtime
Tensor<float> shared = tensor;
```

### 2. Zero-Copy Interop

```csharp
// TorchSharp → Tensor<T> (zero-copy)
Tensor<float> fromTorch = torchTensor.ToTensor();

// ONNX Runtime → Tensor<T> (zero-copy)
Tensor<float> fromOnnx = ortTensor.ToTensor();

// Tensor<T> → ML.NET
var mlVector = shared.AsVBuffer();
```

### 3. SIMD Acceleration

```csharp
// Before: manual SIMD for each operation
Vector<float> a = new Vector<float>(span1);
Vector<float> b = new Vector<float>(span2);
Vector<float> result = a + b;  // 1D only

// After: TensorPrimitives handles multi-dimensional SIMD
TensorPrimitives.Add(tensor1.Span, tensor2.Span, destination.Span);
```

### 4. Generic Math

```csharp
// Before: separate implementations for float, double, int
float CosineSimilarityFloat(float[] a, float[] b) { ... }
double CosineSimilarityDouble(double[] a, double[] b) { ... }

// After: one implementation for all numeric types
T CosineSimilarity<T>(ReadOnlySpan<T> a, ReadOnlySpan<T> b) 
    where T : INumber<T> { ... }

// Or just use TensorPrimitives
TensorPrimitives.CosineSimilarity(vector1, vector2);
```

## Real-World Interop Scenarios

### Scenario 1: ML.NET + ONNX Model

```csharp
// Load a pre-trained ONNX model for image classification
var pipeline = mlContext.Transforms
    .ResizeImages(outputColumnName: "input", imageWidth: 224, imageHeight: 224)
    .Append(mlContext.Transforms.ExtractPixels(outputColumnName: "input"))
    .Append(mlContext.Transforms.ApplyOnnxModel(
        modelFile: "resnet50.onnx",
        outputColumnNames: new[] { "output" },
        inputColumnNames: new[] { "input" }));

// Internally: ML.NET handles tensor conversion
// No manual data copying needed
```

### Scenario 2: TorchSharp Training → ONNX Serving

```csharp
// Train with TorchSharp
var model = new NeuralNetwork();
// ... training code ...

// Export to ONNX (TorchSharp handles this)
torch.jit.export(model, "model.onnx");

// Deploy with ONNX Runtime (production-ready)
using var session = new InferenceSession("model.onnx");
// ONNX Runtime optimizes for target hardware (CPU, GPU, NPU)
```

### Scenario 3: Sentence Embeddings + Cosine Similarity

```csharp
// From ML-2.ipynb - practical semantic search
using SentenceTransformers.MiniLM;
using System.Numerics.Tensors;

var encoder = new SentenceEncoder();
var sentences = new[] { "A man is eating pasta", "A man is riding a horse" };

// Get embeddings (returns float[])
var embeddings = sentences
    .Select(s => encoder.Encode([s])[0])
    .ToArray();

// Use TensorPrimitives for similarity (SIMD-accelerated)
var query = encoder.Encode(["A man is eating food"])[0];
var similarities = embeddings
    .Select((emb, i) => (
        Sentence: sentences[i],
        Similarity: TensorPrimitives.CosineSimilarity(query, emb)  // One line!
    ))
    .OrderByDescending(r => r.Similarity);

// Before TensorPrimitives: you'd implement cosine similarity manually
// or use a third-party library
```

## The Evolution Timeline

```
2018: System.Numerics.Tensors NuGet (basic, not used much)
2019: TorchSharp emerges (libtorch wrapper)
2020: ML.NET 1.x (VBuffer<T>, no tensor interop)
2021: NumSharp stagnates (last release)
2022: ONNX Runtime matures (cross-platform, hardware acceleration)
2023: .NET 8 — TensorPrimitives (40 overloads, float only)
2024: .NET 9 — Tensor<T> experimental (~200 overloads, generics)
2025: .NET 10 — Tensor<T> STABLE (zero-copy, C# 14 operators, nongeneric interface)
```

## Why This Matters

### For Enterprise Developers
- **Reduced dependencies** — no need for MathNet.Numerics, Accord.NET, or custom vector math
- **Lower learning curve** — one tensor type across all ML libraries
- **Better performance** — SIMD acceleration without manual optimization
- **Production-ready** — ONNX Runtime + Tensor<T> = deploy anywhere

### For ML Practitioners in .NET
- **Seamless workflows** — train in TorchSharp, serve with ONNX Runtime, use Tensor<T> as the common currency
- **Interoperability** — mix libraries without copy-paste overhead
- **Future-proof** — backed by Microsoft, part of the runtime

### For the Ecosystem
- **Standardization** — libraries can target Tensor<T> instead of custom types
- **Performance improvements** — runtime can optimize Tensor<T> operations globally
- **Cross-platform consistency** — same code on Windows, Linux, macOS

## Key Takeaway

The .NET ML ecosystem was fragmented — ML.NET, TorchSharp, and ONNX Runtime each had their own tensor types, requiring data copying between libraries. `Tensor<T>` (stable in .NET 10) provides the common representation that was missing, with zero-copy interop, SIMD acceleration, and generic math support. This doesn't replace the libraries — it makes them work together seamlessly.

---

*Sources: ML-2.ipynb, ML-3.ipynb, Microsoft Learn (ML.NET tutorials, ONNX Runtime, .NET 9/10 What's New)*
