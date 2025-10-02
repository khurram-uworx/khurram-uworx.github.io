---
published: false
---

Intel Xe Graphics (integrated in many Intel CPUs starting with 11th Gen Core processors and newer) can be used for AI inference via Intel's **OpenVINO™ Toolkit**.

### Key inference engine:

* **OpenVINO Inference Engine** — It supports execution on:

  * **CPU**
  * **Intel GPU (Xe Graphics)**
  * **VPU (like Intel Movidius Myriad)**
  * **Integrated AI accelerators (like Intel NPU in newer chips)**

When using Xe graphics, OpenVINO offloads supported parts of the model to the integrated GPU using the **GPU plugin**.

### Notes:

* OpenVINO optimizes models from popular frameworks like TensorFlow, ONNX, PyTorch, etc.
* For Xe graphics, you’ll typically use the `GPU` device target in OpenVINO’s `IECore` or `Core` API:

  ```python
  compiled_model = core.compile_model(model, device_name="GPU")
  ```
* Ensure your system has the **Intel Graphics Driver** and **OpenCL support**, which are prerequisites for GPU execution.

With **OpenVINO™**, you can optimize and run AI models for **fast, efficient inference** on Intel hardware. It’s designed for deployment (not training) and supports a wide range of use cases across edge, desktop, and cloud environments.

### 🔧 Core Capabilities

* **Model Optimization**

  * Convert models from TensorFlow, PyTorch, ONNX, etc.
  * Reduce precision (FP32 → FP16 / INT8) for speed and size.
* **Inference Execution**

  * Run models on CPU, Intel Xe GPU, VPU (Myriad X), NPU, or hybrid combinations.
* **Asynchronous & parallel inference**

  * Use streams, batching, and pipelining for performance.

### 🧠 Use Cases

* **Computer Vision**

  * Object detection (YOLO, SSD, Faster R-CNN)
  * Image classification (ResNet, MobileNet)
  * Pose estimation (OpenPose)
  * Semantic segmentation (UNet, DeepLab)

* **Natural Language Processing**

  * Text classification
  * Named entity recognition
  * Question answering (e.g., BERT-based models)

* **Audio & Speech**

  * Voice activity detection
  * Keyword spotting

* **Multi-modal Pipelines**

  * Combine vision + text + audio
  * Process video streams with multiple models in real time

* **Edge/IoT Deployment**

  * AI apps on edge devices (drones, robots, security cameras)

### ⚙️ Hardware Support

* **Intel CPUs** – highly optimized inference
* **Intel Xe Graphics** – acceleration via OpenCL
* **Intel VPUs (e.g., Myriad X)** – low-power, dedicated AI chips
* **Intel NPUs (AI Boost)** – in newer Core Ultra CPUs

Here's a **minimal C# example** for using **OpenVINO** to load a model and perform inference:

> ✅ No explanation, functional style, no Regex.

### ✅ Prerequisites:

* Install [OpenVINO Runtime](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/download.html)
* Use .NET 6+ or compatible
* Add OpenVINO .NET bindings (`Intel.OpenVINO`)

### ✅ `Program.cs`

```csharp
using System;
using OpenVino;

class Program
{
    static void Main()
    {
        var core = new Core();
        var model = core.ReadModel("model.xml");
        var compiledModel = core.CompileModel(model, "CPU");
        var inferRequest = compiledModel.CreateInferRequest();

        var inputTensor = Tensor.CreateFromArray(new float[1, 3, 224, 224]);
        inferRequest.SetInputTensor(inputTensor);

        inferRequest.Infer();

        var output = inferRequest.GetOutputTensor();
        var result = output.GetData<float>();

        foreach (var value in result)
            Console.WriteLine(value);
    }
}
```

---

You can get `model.xml` by converting a pre-trained model using **OpenVINO Model Optimizer** or downloading it from the **Open Model Zoo**.

---

### ✅ Option 1: **Download from Open Model Zoo**

OpenVINO provides pre-converted, ready-to-use models.

#### Steps:

1. Install OpenVINO
2. Run model downloader:

   ```bash
   omz_downloader --name resnet-50
   omz_converter --name resnet-50
   ```
3. Output:

   ```
   intel/resnet-50/FP16/
   ├── resnet-50.xml
   └── resnet-50.bin
   ```

> Location: `open_model_zoo/models/intel/resnet-50/FP16/resnet-50.xml`

---

### ✅ Option 2: **Convert Your Own Model**

Convert ONNX, TensorFlow, or PyTorch model:

```bash
mo --input_model your_model.onnx
```

This will produce:

* `model.xml`
* `model.bin`

---

You can use **most ONNX models** with OpenVINO, but not **all**. Some ONNX operators or layers may not be supported, especially if they are:

* **Custom ops** or non-standard layers
* From **very new** ONNX versions
* Contain **dynamic shapes** without constraints
* Use unsupported **data types** or control flow

---

### ✅ ONNX Models That Work Well:

* Image classification (ResNet, MobileNet, EfficientNet)
* Object detection (YOLOv5, SSD, Faster R-CNN)
* Semantic segmentation (DeepLabV3, UNet)
* NLP (BERT, DistilBERT, RoBERTa)

---

### ⚠️ What to Check

Before converting:

1. Run model checker:

   ```bash
   mo --input_model model.onnx --silent
   ```
2. If conversion fails, check logs for unsupported ops.

---

### ✅ Tips

* Use ONNX **IR version ≤ 11** for best compatibility
* Convert dynamic shapes using:

  ```bash
  mo --input_model model.onnx --input_shape "[1,3,224,224]"
  ```
* Prefer models from [ONNX Model Zoo](https://github.com/onnx/models)

---

**Ollama** does not use **OpenVINO** primarily because:

---

### 1. **Target Use Case: LLMs (Large Language Models)**

Ollama focuses on running models like LLaMA, Mistral, Phi, Gemma — these are **large transformer-based language models**, not small computer vision models.

* **OpenVINO** is optimized for **vision and lightweight NLP models** (e.g., BERT, MobileBERT), not massive decoder-only LLMs.
* OpenVINO doesn’t (yet) provide efficient runtime support for **causal attention, KV caching, or batching** used in modern LLM inference.

---

### 2. **Better Alternatives Exist for LLMs**

Ollama uses backends like:

* **GGML / GGUF**: Quantized tensor format + runtime for CPUs and GPUs
* **llama.cpp**: Highly optimized C++ inference engine for LLMs
* These are purpose-built for fast LLM inference, with CPU/GPU/NPU support and quantization-aware inference.

---

### 3. **OpenVINO's Focus Is Different**

* **OpenVINO** is ideal for:
  ✅ Edge AI
  ✅ Vision models
  ✅ Some speech/NLP
  ✅ Running on Intel CPUs/GPUs/VPUs
* It’s **not optimized for decoder-only LLMs** used in chat-based apps like Ollama.

---

### Summary:

> 🧠 **Ollama uses optimized LLM runtimes (like llama.cpp) because OpenVINO isn’t built for high-performance transformer-based LLMs.**

Ollama doesn’t use **Intel Xe Graphics** by default because:

---

### 🔧 1. **Lack of Backend Support for Intel GPUs**

Ollama runs models via backends like:

* `llama.cpp`
* `ggml / gguf`
* `cuBLAS` / `cuDNN` (for NVIDIA)
* `Metal` (for Apple M1/M2)
* `DirectML` (experimental, Windows)

These **don’t include a mature backend for Intel Xe GPUs**.

* Xe doesn’t support `CUDA` (NVIDIA-only)
* No solid support for Xe via `OpenCL` or `Level Zero` in LLM inference yet
* No `DirectML` support for Xe in llama.cpp

---

### 🚫 2. **No Stable Intel GPU Backend in llama.cpp**

Ollama relies heavily on `llama.cpp`, and it currently supports:

* CPU (default)
* CUDA (NVIDIA)
* Metal (Apple)
* Vulkan (experimental)
* OpenCL (incomplete / unstable)

Xe GPUs use **OpenCL** or **Level Zero**, but those paths are:

* **Not actively maintained**
* **Lack quantized ops support**
* **Have memory issues with large models**

---

### ✅ 3. **NVIDIA Just Works**

When Ollama detects NVIDIA and `CUDA`, it:

* Uses `cuBLAS` kernels for matmul
* Efficient memory management
* Full quantization support
* Predictable performance

---

### 🔄 Can I Force Ollama to Use Xe Graphics?

**Not easily today.** You’d need:

* Build `llama.cpp` with OpenCL or Vulkan backend
* Patch in Intel GPU support (and deal with memory & performance issues)
* Still unlikely to beat CPU for small Xe GPUs

---

### 🧠 Summary:

> 🧨 **Xe Graphics lack mature LLM inference support**, while NVIDIA has fully optimized CUDA paths. That’s why Ollama picks NVIDIA but ignores Intel GPU for now.

Yes, **DirectML can use Intel Xe Graphics**, **in theory and in practice** — but with some important caveats.

---

### ✅ What is DirectML?

**DirectML** is a low-level, hardware-agnostic machine learning API by Microsoft, built on top of **DirectX 12**.

* Supports **NVIDIA**, **AMD**, and **Intel GPUs** (including Xe Graphics)
* Automatically chooses available D3D12-compatible hardware

---

### ✅ Intel Xe Graphics + DirectML

* **Supported**: Intel Xe GPUs (like Iris Xe, Arc, UHD Xe) do support **D3D12**, which DirectML builds on.
* **Windows 10/11** with latest **Intel GPU drivers** (e.g. 31.x or newer) is required.
* **Works best with FP16 or INT8**, depending on model and GPU capabilities.

---

### ⚠️ Real-World Issues

1. **Lack of LLM support**
   DirectML works well with ONNX models (vision, speech), but **LLMs like LLaMA or Mistral** are not officially supported or optimized through DirectML.

2. **No llama.cpp integration**
   `llama.cpp` (which Ollama uses) does **not** have DirectML support — and would need a full backend to be written for it.

3. **Performance bottlenecks**
   Even if used, Xe graphics (especially iGPUs) may have limited VRAM (\~1–2 GB), which restricts large model inference.

---

### 🧪 You Can Try It With ONNX

You **can run ONNX models** using DirectML on Intel Xe:

```bash
python -m onnxruntime.tools.convert_onnx_models_to_ort --use_gpu --execution_provider directml model.onnx
```

Or:

```csharp
var sessionOptions = new SessionOptions();
sessionOptions.AppendExecutionProvider_DML();
```

---

### 🧠 Summary

> ✅ **Yes**, DirectML **can** use **Intel Xe Graphics**,
> ❌ but **not useful for Ollama or llama.cpp-based LLMs** yet.
