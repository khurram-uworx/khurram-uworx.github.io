---
title: "Fine Tuning SLMs and LLMs: Techniques, Tools, and When to Use Them"
date: 2025-06-12
tags:
- GenAI
---

We **can fine-tune** both **Small Language Models (SLMs)** and **Large Language Models (LLMs)**, and **LoRA** and **PEFT** are among the most popular techniques.

Here’s an ordered list of the most **popular fine-tuning techniques**, from most to less commonly used, with brief explanations:

---

### 🔥 1. **LoRA (Low-Rank Adaptation)**

* **Popularity**: ⭐⭐⭐⭐⭐ (Very widely used)
* **What it does**: Injects small, trainable low-rank matrices into the model’s weight layers (usually attention layers).
* **How it works**: Instead of updating the full weights, it updates only the added low-rank matrices — making it memory-efficient and fast.
* **Use case**: Works well for fine-tuning large models on new tasks or domains using very little compute.

---

### 🔥 2. **PEFT (Parameter-Efficient Fine-Tuning)**

* **Popularity**: ⭐⭐⭐⭐⭐ (Framework level popularity)
* **What it is**: Not a technique by itself, but a **family of approaches** (including LoRA, Prefix Tuning, etc.).
* **How it works**: Offers a unified way to apply parameter-efficient methods using libraries like HuggingFace `peft`.
* **Use case**: Choose LoRA, Prompt Tuning, etc., under one toolkit for your fine-tuning task.

---

### 🚀 3. **QLoRA (Quantized LoRA)**

* **Popularity**: ⭐⭐⭐⭐
* **What it does**: Combines LoRA with **4-bit quantization** of model weights.
* **How it works**: Keeps the base model in quantized (compressed) form to reduce memory usage while fine-tuning only LoRA adapters.
* **Use case**: Enables fine-tuning **very large models (e.g., 65B)** on a single GPU with minimal resources.

---

### 🧠 4. **Full Fine-tuning**

* **Popularity**: ⭐⭐⭐
* **What it does**: Updates **all** model weights.
* **How it works**: Standard gradient descent across all parameters.
* **Use case**: Best when you have **lots of compute and data**; needed for deep domain specialization.

---

### 📌 5. **Prefix Tuning**

* **Popularity**: ⭐⭐
* **What it does**: Adds trainable "prefix tokens" to the input of each transformer layer.
* **How it works**: These tokens steer model behavior without changing its core weights.
* **Use case**: Parameter-efficient and effective for task-specific tuning.

---

### 🧷 6. **Prompt Tuning / Soft Prompts**

* **Popularity**: ⭐⭐
* **What it does**: Learns embeddings for special "soft" prompts (not actual text).
* **How it works**: Only the prompt embeddings are trained while the model is frozen.
* **Use case**: Good for small tasks, but less expressive than LoRA.

---

### 📦 7. **Adapter Tuning**

* **Popularity**: ⭐
* **What it does**: Adds small trainable layers (adapters) between transformer layers.
* **How it works**: Original model weights stay frozen; only adapters are updated.
* **Use case**: Similar to LoRA but with heavier memory use.

---

### Summary Table:

| Technique        | Updates Params | Efficient? | Needs Full Model? | Best For               |
| ---------------- | -------------- | ---------- | ----------------- | ---------------------- |
| LoRA             | ⬛ Part         | ✅ Yes      | ✅ Yes             | Most tasks             |
| PEFT             | ⬛ Part (meta)  | ✅ Yes      | ✅ Yes             | Flexible approach      |
| QLoRA            | ⬛ Part + Quant | ✅✅ Very    | ✅ Yes             | Low-memory fine-tuning |
| Full Fine-tuning | 🟥 All         | ❌ No       | ✅ Yes             | Large data + compute   |
| Prefix Tuning    | ⬛ Part         | ✅ Yes      | ✅ Yes             | Lightweight tasks      |
| Prompt Tuning    | ⬛ Part         | ✅ Yes      | ✅ Yes             | Simple tasks           |
| Adapter Tuning   | ⬛ Part         | ✅ Yes      | ✅ Yes             | Similar to LoRA        |

---

To fine-tune SLMs or LLMs, you’ll typically use a stack of tools and libraries that help with:

* Model loading
* Data processing
* Fine-tuning (full or parameter-efficient)
* Experiment tracking
* Model evaluation & deployment

Here’s a breakdown of **key tools** used in fine-tuning and how they **compare** by benefits:

---

## 🔧 Core Tool Categories

| Category            | Popular Tools                                                              | Purpose                               |
| ------------------- | -------------------------------------------------------------------------- | ------------------------------------- |
| Model Framework     | **Transformers (Hugging Face)**, **trl**, **DeepSpeed**                    | Load and manipulate pretrained models |
| PEFT Library        | **PEFT (Hugging Face)**, **LoRA**, **QLoRA**                               | Enable efficient fine-tuning          |
| Quantization        | **BitsAndBytes**, **GPTQ**, **AutoGPTQ**                                   | Reduce model size (for QLoRA etc.)    |
| Trainer/Accelerator | **Accelerate**, **DeepSpeed**, **Ray**, **Lightning**                      | Optimize and parallelize training     |
| Data Processing     | **Datasets (Hugging Face)**, **Pandas**, **OpenAI Whisper**, **LangChain** | Prepare, clean, tokenize data         |
| Experiment Tracking | **Weights & Biases**, **TensorBoard**, **MLflow**                          | Log and visualize training metrics    |
| Deployment Tools    | **Optimum**, **ONNX**, **vLLM**, **FastAPI**                               | Serve or export fine-tuned models     |

---

## 🔍 Tool Comparison by Benefits

### 🧠 **Transformers (Hugging Face)**

* **Pros**: Easy model access, well-documented, huge ecosystem
* **Use**: Load base models, apply PEFT, inference
* **Rating**: ⭐⭐⭐⭐⭐ (Most essential)

---

### 🧪 **PEFT (Hugging Face)**

* **Pros**: Unified interface for LoRA, prefix tuning, etc.
* **Use**: Parameter-efficient tuning (LoRA, QLoRA)
* **Rating**: ⭐⭐⭐⭐⭐ (Best for low-resource fine-tuning)

---

### 🧬 **Accelerate (Hugging Face)**

* **Pros**: Abstracts device and distributed training
* **Use**: Easily scale training to multi-GPU, mixed precision
* **Rating**: ⭐⭐⭐⭐ (Simple + powerful)

---

### ⚙️ **DeepSpeed**

* **Pros**: Very efficient for large models, ZeRO optimization
* **Use**: Train LLMs with huge batch sizes and low memory
* **Rating**: ⭐⭐⭐⭐ (Powerful, more complex)

---

### 🧱 **BitsAndBytes**

* **Pros**: Enables 8-bit or 4-bit quantization
* **Use**: Load LLMs in reduced memory space (QLoRA)
* **Rating**: ⭐⭐⭐⭐⭐ (Essential for big models on limited GPUs)

---

### 📦 **Datasets (Hugging Face)**

* **Pros**: Easy dataset loading and pre-processing
* **Use**: Ready-made NLP datasets, streaming, caching
* **Rating**: ⭐⭐⭐⭐⭐ (Best for text fine-tuning)

---

### 📊 **Weights & Biases (wandb)**

* **Pros**: Beautiful dashboards, easy integration
* **Use**: Monitor metrics, log artifacts, track experiments
* **Rating**: ⭐⭐⭐⭐ (Great for research and team work)

---

### 🧪 **Optimum**

* **Pros**: Export models to ONNX, TensorRT, etc.
* **Use**: Optimize for inference on CPU/GPU/Edge
* **Rating**: ⭐⭐⭐ (Nice-to-have for deployment)

---

## ✅ Tooling Stack Examples

### For Small-Scale (SLMs, 1 GPU):

* Transformers + PEFT + Accelerate + Datasets + wandb

### For Large Models (LLMs, multi-GPU):

* Transformers + PEFT/QLoRA + DeepSpeed + BitsAndBytes + Datasets + wandb

### For Easy PEFT:

```bash
pip install transformers peft accelerate datasets bitsandbytes
```

---

Great question. While **larger context windows** make models more capable of handling longer, richer prompts or chat histories, there’s a point where **context stuffing becomes inefficient** or insufficient — and that’s when **fine-tuning** starts to make more sense.

---

## ✅ **When You Should Start Considering Fine-Tuning**

### 1. **Repetitive Instructions in Prompts or Chat History**

If you find yourself **repeating the same context, rules, tone, or examples** in every interaction (e.g., instructions, formatting styles, brand voice), that’s a strong sign.

> 💡 Fine-tuning can make this implicit and save tokens + costs.

---

### 2. **You Hit Token or Context Window Limits**

Even with 128k+ token windows, eventually you’ll hit limits:

* Long documents
* Complex conversations
* Multimodal inputs

> 💡 Fine-tuning bakes the core knowledge into the model instead of relying on runtime memory.

---

### 3. **Performance Is Inconsistent with Prompting Alone**

When you:

* Use chain-of-thought or few-shot prompting, and it works **sometimes**
* But it’s brittle or hard to scale

> 💡 Fine-tuning adds **task stability and generalization**.

---

### 4. **You Have Labeled or High-Quality Task Data**

Got examples of inputs + ideal outputs?

* Customer support logs
* Domain-specific Q\&A
* Code completions or translations

> 💡 Perfect for fine-tuning (especially LoRA or QLoRA).

---

### 5. **Latency, Cost, or Speed Constraints**

Stuffing context:

* Increases token usage (input/output costs)
* Slows down response time

> 💡 Fine-tuning reduces the dependency on long prompts = **cheaper + faster** inference.

---

### 6. **Need for Custom Behavior or Domain Knowledge**

Prompting alone may not:

* Capture subtle brand tone
* Follow strict reasoning paths
* Understand niche or proprietary terminology

> 💡 Fine-tuning is best for **deep customization**.

---

### 7. **Multi-Turn Agents or Tools with Memory**

In agentic systems:

* It’s inefficient to store every instruction in memory
* Models should "just know" how to behave

> 💡 Fine-tuning sets a strong behavioral prior.

---

## 🔁 When Prompting or RAG is Still Better

* Small tasks or few users
* Constantly changing information (use RAG instead)
* No labeled data for training
* Fast iteration or experimentation

---

## 🧠 Rule of Thumb

| Situation                        | Best Approach      |
| -------------------------------- | ------------------ |
| Small app, general tasks         | Prompting          |
| Domain-specific knowledge        | RAG or Fine-tuning |
| Behavioral tuning, style control | Fine-tuning        |
| Cost-sensitive use               | Fine-tuning        |
| Just need facts from documents   | RAG                |

