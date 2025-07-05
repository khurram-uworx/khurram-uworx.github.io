---
title: "Understanding Base vs Instruct Models in LLMs"
date: 2025-06-02
tags:
- GenAI
---

When discussing large language models (LLMs) like **Qwen** or **Llama**, it's crucial to understand the distinction between **"base models"** and **"instruct models."** Here's a breakdown:

---

## Base Models

### 🔹 Foundation
- These are the foundational LLMs, pre-trained on massive datasets of raw text.
- Their primary objective is to learn the statistical patterns and structure of language.
- They excel at generating coherent and contextually relevant text.

### 🔹 Capabilities
- Base models are versatile and can be adapted for various natural language processing (NLP) tasks.
- Often used for tasks like text completion or generating text based on a given prompt.
- They are the starting point for further model development.

### 🔹 Limitations
- While they produce fluent text, they may struggle with following specific instructions or performing complex reasoning without additional training.
- They can produce outputs that are not aligned with user intent, as they're trained only to predict the next token in a sequence.

---

## Instruct Models

### 🔹 Fine-Tuning
- Instruct models are derived from base models but undergo further training (fine-tuning).
- This involves using datasets containing instructions and their corresponding desired outputs.
- This process teaches the model to interpret and follow user commands accurately.

### 🔹 Capabilities
- Instruct models excel at tasks like:
  - Summarization  
  - Translation  
  - Question answering  
  - Following complex prompts
- Designed to be more reliable and consistent in responding to specific instructions.

### 🔹 Advantages
- Better suited for applications requiring targeted functionalities.
- Tend to produce more predictable and user-friendly outputs.

---

## In Essence

- **Base models** are the raw, foundational language models.  
- **Instruct models** are refined versions, optimized to follow instructions effectively.  

> When you see models like `Qwen-Instruct`, it signifies that those models have undergone instruction fine-tuning.

---

## Using Qwen2.5-7B-Instruct with Ollama and Python

### 1. Pulling and Running Qwen2.5-7B-Instruct with Ollama

#### • Install Ollama
- Follow the installation instructions from the [Ollama website](https://ollama.com).

#### • Pull the Model
```bash
ollama pull qwen2.5-7b-instruct
```

#### • Run the Model
```bash
ollama run qwen2.5-7b-instruct
```

---

### 2. Python Chat Completion with Ollama

#### • Install the Ollama Python Library
```bash
pip install ollama
```

#### • Python Code Example
```python
import ollama

response = ollama.chat(
  model='qwen2.5-7b-instruct',
  messages=[
    {
      'role': 'user',
      'content': 'Summarize the following article: [Paste your article here]'
    },
  ],
)

print(response['message']['content'])
```

---

### 3. Prompt Engineering for Instruct Models

#### • Clear and Specific Instructions
- Avoid ambiguity.
- Example:  
  - ❌ "Tell me about cats."  
  - ✅ "Explain the different breeds of domestic cats and their common characteristics."

#### • Use Task-Oriented Language
- Start prompts with action verbs:  
  `Summarize`, `Translate`, `Explain`, `Generate`, `Write`, `Answer`

#### • Provide Context
- Include background information or examples to guide the model.

#### • Specify Desired Output Format
- Example:  
  `"Write a short summary in three bullet points."`

#### • Use System Messages (Optional)
```python
messages = [
  {'role': 'system', 'content': 'You are a helpful and concise assistant.'},
  {'role': 'user', 'content': 'Explain quantum physics.'},
]
```

#### • Few-Shot Examples (Optional)
- Provide input-output pairs to demonstrate the task.

#### • Example Prompts
- "Write a short story about a cat that goes on an adventure. The story must be no longer than 200 words."
- "Given this paragraph, extract the names of all of the people mentioned, and put them into a Python list: [paragraph]"
- "What are the pros and cons of electric cars? Answer in a table format."

---

## Key Considerations

- **Model Limitations**: Even instruct models may produce inaccurate or nonsensical outputs.
- **Resource Usage**: Running LLMs is resource-intensive.
- **Prompt Iteration**: Try multiple prompt formats to find what works best.

---

# Enforcing JSON with Instruct Models

Base models (like Claude) often fail to consistently adhere to specific output formats like JSON.

---

## Pipeline Design

| Step                      | Description |
|---------------------------|-------------|
| **1. Base Model Generation (Claude)** | - Send prompt to Claude requesting JSON.<br>- Capture the response. |
| **2. JSON Validation and Correction (Qwen)** | - Feed Claude’s output into Qwen.<br>- Prompt Qwen to extract info and return valid JSON. |
| **3. Output Processing** | - Parse the validated JSON from Qwen. |

---

## Python Implementation (Conceptual)

```python
import ollama
import json

def generate_json_with_qwen(claude_response, desired_schema):
    """
    Uses Qwen to ensure a response is in the specified JSON format.
    """
    prompt = f"""
    Here is some text that should be converted into JSON, adhering to the following schema:
    {json.dumps(desired_schema, indent=2)}

    Text:
    {claude_response}

    Please generate the JSON output, and only the JSON output. If the provided text is not valid JSON, create valid JSON using the information contained within the text.
    """

    response = ollama.chat(
        model='qwen2.5-7b-instruct',
        messages=[{'role': 'user', 'content': prompt}],
    )

    try:
        return json.loads(response['message']['content'])
    except json.JSONDecodeError:
        print("Qwen did not return valid JSON.")
        return None

# Example Usage
claude_response = "Here is some information, name:John, age:30, city:New York"
desired_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"},
        "city": {"type": "string"},
    },
    "required": ["name", "age", "city"],
}

qwen_json = generate_json_with_qwen(claude_response, desired_schema)

if qwen_json:
    print(json.dumps(qwen_json, indent=2))
```

---

## Why This Is a Good Use of Instruct Models

- ✅ **Instruction Following**: Trained to follow specific formatting rules.
- ✅ **Robustness**: Handles imperfect or non-JSON inputs from base models.
- ✅ **Consistency**: Delivers structured, machine-readable output.
- ✅ **Error Handling**: Recovers from base model mistakes (e.g., missing fields or invalid JSON).

---

## Important Considerations

- **Prompt Engineering**: Must be clear, concise, and unambiguous.
- **Schema Definition**: Ensure the desired schema is accurately defined.
- **Performance**: Running two LLMs can be intensive—optimize your usage.

---

By implementing this pipeline, you can significantly improve the **reliability** and **consistency** of your generative AI applications.

Continue to [Base and Instruct Versions of Large Language Models: A Comparative Analysis of Qwen and Llama](/llm-researches/instruct.md)
