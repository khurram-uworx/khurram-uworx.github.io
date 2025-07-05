---
title: "Prompt Engineering"
date: 2025-06-11
tags:
- GenAI
---

In generative AI and prompt engineering, **Few-shot**, **System**, and **Instruction** refer to different ways of guiding or controlling a model’s behavior. Here’s what each term means:

---

### **Few-shot**

* **Definition**: A prompt that includes a few examples to help the model understand the task.
* **Usage**: You provide 1–5 input-output pairs (shots) before the actual input.
* **Purpose**: Helps the model generalize from examples.

**Example**:

```
Translate English to French:
English: Hello
French: Bonjour

English: Thank you
French: Merci

English: Good night
French:
```

---

### **System**

* **Definition**: A message or prompt that sets the **global behavior** or **persona** of the model.
* **Usage**: Usually used at the beginning of a chat session to define the assistant's role or style.
* **Purpose**: Establishes tone, domain, constraints, etc.

**Example**:

```
You are a helpful assistant that explains complex legal concepts in simple terms.
```

---

### **Instruction**

* **Definition**: A direct command or instruction to the model within the user prompt.
* **Usage**: Often used in zero-shot or one-shot formats, without extensive context.
* **Purpose**: Tells the model exactly what to do.

**Example**:

```
Summarize the following paragraph in two sentences.
```

---

### Summary:

| Term        | Purpose                           | Style         | Example Use                     |
| ----------- | --------------------------------- | ------------- | ------------------------------- |
| Few-shot    | Teach by example                  | Sample-driven | "Translate: Hello → Bonjour..." |
| System      | Set behavior/policies globally    | Meta-message  | "You are a math tutor."         |
| Instruction | Command a specific action or task | Direct prompt | "Summarize this text..."        |


In **Retrieval-Augmented Generation (RAG)** scenarios, **Query Rewriting** and **Query Summarization** refer to techniques used to improve the effectiveness of information retrieval before generation. Here's what they mean:

---

### 🔁 **Query Rewriting**

* **Definition**: Modifying the original user query to make it more effective for retrieval.
* **Goal**: Make ambiguous, incomplete, or conversational queries more specific and search-friendly.

**Example**:
User query:

> "What about Tesla in 2023?"

Rewritten query:

> "What were the major developments related to Tesla Inc. in 2023?"

**Why**: Retrieval systems work better with precise, well-formed queries.

---

### 📝 **Query Summarization**

* **Definition**: Condensing a long, complex query or multi-turn dialogue into a concise summary query.
* **Goal**: Focus the retrieval on the core question or topic, especially in chat-like interactions.

**Example**:
Conversation:

* User: "Earlier you said Tesla had major events in 2023. Can you remind me what they were?"
* Assistant: \[retrieves and responds]

Summarized query:

> "List of major events related to Tesla in 2023."

**Why**: Helps retrieve the most relevant chunks without all the conversational baggage.

---

### 🔍 Why They Matter in RAG

RAG systems rely on retrieving **relevant documents** before generating an answer. Poor queries → poor retrieval → weak answers.
So:

* **Rewriting** improves *clarity* and *specificity*
* **Summarization** improves *focus* and *relevance*

Both are forms of **prompt engineering for retrieval**, improving the model’s grounding in accurate information.


In **Retrieval-Augmented Generation (RAG)**, **Pre-generated answers** refer to **answers that are generated and stored in advance**, rather than generated dynamically at runtime. Here's what that means in detail:

---

### 🔹 What Are Pre-generated Answers?

They are **precomputed responses** associated with common or expected queries. Instead of generating an answer on the fly using retrieved documents, the system **retrieves a ready-made answer**.

---

### 🔹 Why Use Them?

* ⚡ **Speed**: No need for generation; just retrieve and return.
* 🎯 **Consistency**: Responses are curated and controlled.
* 💾 **Efficiency**: Reduced computation load during runtime.

---

### 🔹 Where They Fit in RAG

In a typical RAG pipeline:

1. User submits a query.
2. The system retrieves relevant documents.
3. A generation model creates a response based on those docs.

With pre-generated answers:

* Step 3 is replaced by **fetching a prewritten answer** tied to the retrieved context or query.

---

### 🔹 Use Cases

* FAQs
* Customer support bots
* Knowledge base with stable answers
* High-traffic or repetitive queries

---

### 🔹 Example

| Query                         | Response Type                                                              |
| ----------------------------- | -------------------------------------------------------------------------- |
| "What is your refund policy?" | Pre-generated answer: "Our refund policy allows returns within 30 days..." |
| "Explain quantum mechanics"   | Generated dynamically from retrieved sources                               |

---

So in summary:
**Pre-generated answers** in RAG are optimized, stored responses for known queries—used to boost speed, accuracy, and scalability.
