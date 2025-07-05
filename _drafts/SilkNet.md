---
published: false
---

We can use C# to access our laptop GPUs for number crunching, but with **limitations** and **workarounds**, as C# does not natively support GPU computation. Here are your main options:

---

### 🔧 1. **Use OpenCL with C#**

Your GPU supports **OpenCL**, and you can access it via C# using a wrapper library like:

* **Cloo** (OpenCL for .NET)
* **OpenCL.Net**
* **Silk.NET** (modern and more maintained)

You’ll write OpenCL kernels and run them from C#.

**Pros:** Works with your GPU
**Cons:** You must write OpenCL kernel code manually

---

### ⚙️ 2. **Use DirectCompute via SharpDX or Vortice**

* **DirectCompute** can leverage GPU compute capabilities via **DirectX**, and there are .NET wrappers like **SharpDX** (now deprecated) or **Vortice.Windows** (actively maintained).
* You’ll need to write HLSL shaders for GPU execution.

---

### 🧠 3. **Use ManagedCUDA or Alea GPU (NVIDIA only)**

**Not compatible with your AMD GPU**, so skip these.

---

### ✅ Recommended Path

Use **C# + Silk.NET (OpenCL)** or **OpenCL.Net** with OpenCL kernels for GPU computation.

Perfect. Let's implement a **Monte Carlo simulation to estimate π** using your AMD GPU via **C# + Silk.NET (OpenCL)** in a functional style.

---

### 🎯 Problem: Estimate π using Monte Carlo

Generate `N` random points in the unit square. Count how many fall inside the unit circle:

```csharp
π ≈ 4 × (points_inside_circle / total_points)
```

---

### ⚠️ Notes

* OpenCL kernels can't generate random numbers easily.
* We'll generate random (x, y) values on the **CPU**, pass them to the **GPU**, and use OpenCL to count how many fall inside the circle.

---

### ✅ C# Functional Code Using Silk.NET.OpenCL

```csharp
using Silk.NET.OpenCL;
using static Silk.NET.OpenCL.CL;

var cl = CL.GetApi();

// 1. Platform/device/context/queue
var platform = cl.GetPlatformIDs()[0];
var device = cl.GetDeviceIDs(platform, DeviceType.DeviceTypeGpu)[0];
var context = cl.CreateContext(null, 1, new[] { device }, null, null, out _);
var queue = cl.CreateCommandQueue(context, device, 0, out _);

// 2. OpenCL Kernel
var kernelSource = """
__kernel void monte_carlo(__global const float* x, __global const float* y, __global int* results)
{
    int i = get_global_id(0);
    float dx = x[i];
    float dy = y[i];
    results[i] = (dx * dx + dy * dy <= 1.0f) ? 1 : 0;
}
""";

var program = cl.CreateProgramWithSource(context, 1, new[] { kernelSource }, null, out _);
cl.BuildProgram(program, 1, new[] { device }, null, null, null);
var kernel = cl.CreateKernel(program, "monte_carlo", out _);

// 3. Host data
int count = 10_000_000;
var rnd = new Random();
var xs = Enumerable.Range(0, count).Select(_ => (float)rnd.NextDouble()).ToArray();
var ys = Enumerable.Range(0, count).Select(_ => (float)rnd.NextDouble()).ToArray();
var results = new int[count];

// 4. Buffers
var sizeF = (nuint)(sizeof(float) * count);
var sizeI = (nuint)(sizeof(int) * count);

var bufX = cl.CreateBuffer(context, MemFlags.MemReadOnly | MemFlags.MemCopyHostPtr, sizeF, xs, out _);
var bufY = cl.CreateBuffer(context, MemFlags.MemReadOnly | MemFlags.MemCopyHostPtr, sizeF, ys, out _);
var bufOut = cl.CreateBuffer(context, MemFlags.MemWriteOnly, sizeI, null, out _);

// 5. Kernel args
cl.SetKernelArg(kernel, 0, bufX);
cl.SetKernelArg(kernel, 1, bufY);
cl.SetKernelArg(kernel, 2, bufOut);

// 6. Run kernel
cl.EnqueueNDRangeKernel(queue, kernel, 1, null, new[] { (nuint)count }, null, 0, null, null);
cl.Finish(queue);

// 7. Read result
cl.EnqueueReadBuffer(queue, bufOut, true, 0, sizeI, results, 0, null, null);

// 8. Estimate Pi
var inside = results.AsSpan().ToArray().AsParallel().Sum();
var piEstimate = 4.0 * inside / count;

Console.WriteLine($"Estimated π ≈ {piEstimate:F6}");

// 9. Cleanup
cl.ReleaseKernel(kernel);
cl.ReleaseProgram(program);
cl.ReleaseMemObject(bufX);
cl.ReleaseMemObject(bufY);
cl.ReleaseMemObject(bufOut);
cl.ReleaseCommandQueue(queue);
cl.ReleaseContext(context);
```

---

### 📦 Install NuGet

```bash
dotnet add package Silk.NET.OpenCL
```
