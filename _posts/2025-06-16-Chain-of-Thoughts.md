---
title: "Chain of Thoughts"
date: 2025-06-16
tags:
- GenAI
---

To implement a chain of thoughts (CoT) using Ollama models like Qwen in C#, you can leverage either the **Microsoft AI extensions** or **Semantic Kernel**. Below is a step-by-step guide to help you approach this:

---

### 1. **Understand Chain of Thought (CoT)**
   - CoT is a technique where the model breaks down a problem into intermediate reasoning steps before arriving at the final answer.
   - For example, instead of directly answering a math problem, the model might first explain the steps to solve it.

---

### 2. **Set Up Ollama and Qwen**
   - Ensure you have access to the Ollama API or the Qwen model.
   - If Ollama provides an API, you’ll need to interact with it via HTTP requests or a client library.
   - If you’re running the model locally, ensure it’s properly set up and accessible.

---

### 3. **Choose Between Microsoft AI Extensions or Semantic Kernel**
   - **Microsoft AI Extensions**: These are typically lightweight and designed for specific AI tasks. If you’re looking for simplicity, this might be a good choice.
   - **Semantic Kernel**: This is a more robust framework for orchestrating AI models and workflows. It’s ideal for complex tasks like CoT, where you need to chain multiple prompts or reasoning steps.

---

### 4. **Implement CoT Using Semantic Kernel (Recommended)**
   Semantic Kernel is well-suited for implementing CoT because it allows you to chain prompts and manage intermediate reasoning steps.

#### Steps:
   - **Install Semantic Kernel**:
     Add the Semantic Kernel NuGet package to your C# project:
     ```bash
     dotnet add package Microsoft.SemanticKernel
     ```

   - **Set Up the Kernel**:
     Initialize the kernel and configure it to use the Ollama model (Qwen). If Ollama provides an API, you can use the `HttpClient` to interact with it.

     ```csharp
     using Microsoft.SemanticKernel;
     using Microsoft.SemanticKernel.Orchestration;

     var kernel = new KernelBuilder()
         .WithOllamaService() // Assuming you have a custom extension for Ollama
         .Build();
     ```

   - **Define Prompts for CoT**:
     Create prompts that guide the model to break down the problem into intermediate steps. For example:
     ```csharp
     string cotPrompt = @"
     You are a helpful assistant. Break down the following problem into intermediate reasoning steps before providing the final answer.

     Problem: {{$input}}

     Steps:
     1. 
     2. 
     3. 

     Final Answer:
     ";
     ```

   - **Create a Semantic Function**:
     Wrap the prompt into a semantic function:
     ```csharp
     var cotFunction = kernel.CreateSemanticFunction(cotPrompt);
     ```

   - **Invoke the Function**:
     Pass the input to the function and execute it:
     ```csharp
     var context = new ContextVariables();
     context.Set("input", "What is 25% of 80?");

     var result = await kernel.RunAsync(context, cotFunction);
     Console.WriteLine(result.ToString());
     ```

   - **Handle Intermediate Steps**:
     You can modify the prompt and function to handle multiple steps or iterative reasoning.

---

### 5. **Alternative: Use Microsoft AI Extensions**
   If you prefer a simpler approach, you can use Microsoft AI extensions to interact with the Ollama API directly.

#### Steps:
   - **Install Microsoft AI Extensions**:
     Add the necessary NuGet packages for AI extensions.

   - **Call the Ollama API**:
     Use `HttpClient` to send requests to the Ollama API and process the responses.
{% raw %}
     ```csharp
     using System.Net.Http;
     using System.Threading.Tasks;

     public async Task<string> GetCoTResponseAsync(string input)
     {
         var client = new HttpClient();
         var request = new HttpRequestMessage
         {
             Method = HttpMethod.Post,
             RequestUri = new Uri("https://api.ollama.com/v1/chat"),
             Content = new StringContent($"{{\"model\": \"qwen\", \"messages\": [{{\"role\": \"user\", \"content\": \"{input}\"}}]}}")
         };

         var response = await client.SendAsync(request);
         response.EnsureSuccessStatusCode();

         var responseBody = await response.Content.ReadAsStringAsync();
         return responseBody;
     }
     ```
{% endraw %}

   - **Parse and Process the Response**:
     Extract the intermediate steps and final answer from the API response.

---

### 6. **Test and Iterate**
   - Test your implementation with various inputs to ensure the model generates coherent intermediate steps.
   - Refine your prompts and logic as needed.

---

### 7. **Advanced: Multi-Step Reasoning**
   If your use case requires multi-step reasoning, you can chain multiple semantic functions in Semantic Kernel or make sequential API calls to the Ollama model.

---

### Example Workflow in Semantic Kernel
```csharp
// Define prompts for each step
string step1Prompt = "Explain the first step to solve: {{$input}}";
string step2Prompt = "Based on the first step, explain the next step: {{$input}}";
string finalAnswerPrompt = "Provide the final answer: {{$input}}";

// Create semantic functions
var step1Function = kernel.CreateSemanticFunction(step1Prompt);
var step2Function = kernel.CreateSemanticFunction(step2Prompt);
var finalAnswerFunction = kernel.CreateSemanticFunction(finalAnswerPrompt);

// Execute the chain
var context = new ContextVariables();
context.Set("input", "What is 25% of 80?");

var step1Result = await kernel.RunAsync(context, step1Function);
context.Set("input", step1Result.ToString());

var step2Result = await kernel.RunAsync(context, step2Function);
context.Set("input", step2Result.ToString());

var finalResult = await kernel.RunAsync(context, finalAnswerFunction);
Console.WriteLine(finalResult.ToString());
```

By following these steps, you can effectively implement a chain of thoughts using Ollama models like Qwen in C#

---

There are several existing libraries and projects on GitHub related to **Chain of Thought (CoT)** reasoning and prompt chaining, which may align with your goal of implementing CoT using Ollama models like Qwen. Below is a summary of relevant projects and resources:

---

### 1. **Chain-of-Thought Hub**
   - **Description**: A comprehensive evaluation suite for measuring the reasoning capabilities of large language models (LLMs). It includes benchmarks for complex reasoning tasks like math, science, and coding, and supports open-source models like LLaMA and Qwen.
   - **Key Features**:
     - Tracks progress of LLMs on multi-step reasoning tasks.
     - Provides datasets and evaluation scripts for benchmarking.
     - Focuses on open-source models and their potential to match proprietary models like GPT-4.
   - **GitHub Link**: [Chain-of-Thought Hub](https://github.com/FranxYao/chain-of-thought-hub) .

---

### 2. **ThoughtSource**
   - **Description**: A central hub for chain-of-thought reasoning data and tools. It provides datasets, tutorials, and tools for generating and evaluating reasoning chains.
   - **Key Features**:
     - Standardized datasets for CoT reasoning.
     - Tools for generating reasoning chains using models like GPT-3.5, GPT-4, and Flan-T5.
     - Supports evaluation of reasoning performance.
   - **GitHub Link**: [ThoughtSource](https://github.com/OpenBioLink/ThoughtSource) .

---

### 3. **Open Thoughts**
   - **Description**: A project focused on curating open reasoning datasets and training state-of-the-art small reasoning models. It includes models like OpenThinker-32B and OpenThinker-7B, which are optimized for math and code reasoning.
   - **Key Features**:
     - Open-source datasets and models.
     - Benchmarks for evaluating reasoning performance.
     - Integration with Ollama for local inference.
   - **GitHub Link**: [Open Thoughts](https://github.com/open-thoughts/open-thoughts) .

---

### 4. **Prompt Chaining with Qwen and Ollama**
   - **Description**: A GitHub gist demonstrating prompt chaining using local reasoning models like Qwen and QwQ with Ollama. It includes examples of chaining prompts for tasks like architecture design and content generation.
   - **Key Features**:
     - Combines powerful reasoning models with base models for extraction.
     - Provides scripts for running prompt chains using Bun, Python, and Ollama.
   - **GitHub Link**: [Prompt Chaining with Qwen](https://gist.github.com/geenet/2b7172e2b07c1d8259091b8028a96639) .

---

### 5. **Open O1 Open Source Chain of Thought Model**
   - **Description**: An open-source CoT model trained on O1-style thought data, compatible with LLaMA and Qwen. It includes setup instructions and sample Q&A for logical reasoning tasks.
   - **Key Features**:
     - Supports few-shot prompting for complex reasoning tasks.
     - Available on Hugging Face for easy integration.
   - **GitHub Link**: [Open O1 Model](https://medium.com/@rajeshmanikumarg/open-o1-open-source-chain-of-thought-model-e01ffed9c9cb) .

---

### 6. **Outlines Library for Chain of Thought**
   - **Description**: A library that applies CoT through structured output using models like Hermes-2-Pro-Llama-3-8B. It supports JSON schema-based reasoning steps and conclusions.
   - **Key Features**:
     - Structured reasoning steps using Pydantic classes.
     - Integration with llama-cpp-python for local inference.
   - **GitHub Link**: [Outlines Library](https://dottxt-ai.github.io/outlines/cookbook/chain_of_thought/) .

---

### 7. **Finetuning Qwen-2.5 for CoT**
   - **Description**: A Medium article detailing the process of fine-tuning Qwen-2.5 for CoT reasoning using reinforcement learning techniques like GRPO (Group Relative Policy Optimization).
   - **Key Features**:
     - Step-by-step guide for fine-tuning Qwen-2.5.
     - Includes setup instructions and training scripts.
   - **GitHub Link**: [Finetune Qwen-2.5](https://medium.com/@mahadir.ahmad/finetune-qwen-2-5-ai-model-for-chain-of-thought-cot-4d1eae3a3aa9) .

---

### Summary
If you're looking to implement CoT using Ollama models like Qwen, you can leverage existing libraries and projects such as **Chain-of-Thought Hub**, **ThoughtSource**, and **Open Thoughts**. These resources provide datasets, evaluation tools, and pre-trained models that can significantly accelerate your development process. Additionally, the **Prompt Chaining with Qwen** gist and **Outlines Library** offer practical examples and frameworks for implementing CoT in your workflow.

For more details, you can explore the provided GitHub links and adapt the tools to your specific use case.

---

**Semantic Kernel** provides a powerful **Agents** framework that can be leveraged to implement **Chain of Thought (CoT)** reasoning more effectively. Agents in Semantic Kernel are designed to handle complex workflows, manage state, and orchestrate multiple steps of reasoning, making them ideal for CoT implementations.

Here’s how you can use Semantic Kernel’s **Agents** to implement CoT effectively:

---

### 1. **What Are Semantic Kernel Agents?**
   - **Agents** in Semantic Kernel are autonomous or semi-autonomous components that can perform tasks, make decisions, and interact with other agents or services.
   - They are designed to handle **multi-step workflows**, making them perfect for breaking down problems into intermediate reasoning steps (CoT).
   - Agents can use **semantic functions**, **native functions**, and **external APIs** to achieve their goals.

---

### 2. **Why Use Agents for CoT?**
   - **Modularity**: Agents allow you to break down the reasoning process into smaller, reusable components.
   - **State Management**: Agents can maintain context and state across multiple steps, which is crucial for CoT.
   - **Orchestration**: Agents can coordinate multiple reasoning steps, ensuring the workflow is executed in the correct order.
   - **Extensibility**: You can easily add new agents or modify existing ones to handle more complex reasoning tasks.

---

### 3. **Steps to Implement CoT Using Semantic Kernel Agents**

#### Step 1: **Install Semantic Kernel**
   Add the Semantic Kernel NuGet package to your C# project:
   ```bash
   dotnet add package Microsoft.SemanticKernel
   ```

#### Step 2: **Define the Reasoning Steps**
   Break down the problem into intermediate reasoning steps. For example, if you’re solving a math problem, the steps might include:
   1. Understanding the problem.
   2. Breaking it into smaller sub-problems.
   3. Solving each sub-problem.
   4. Combining the results to get the final answer.

#### Step 3: **Create Agents for Each Step**
   Create an agent for each reasoning step. Each agent will handle a specific part of the problem.

   ```csharp
   using Microsoft.SemanticKernel;
   using Microsoft.SemanticKernel.Orchestration;

   // Agent 1: Understand the problem
   var understandAgent = new SemanticFunctionAgent(kernel, "Understand the problem: {{$input}}");

   // Agent 2: Break into sub-problems
   var breakAgent = new SemanticFunctionAgent(kernel, "Break the problem into sub-problems: {{$input}}");

   // Agent 3: Solve sub-problems
   var solveAgent = new SemanticFunctionAgent(kernel, "Solve the sub-problem: {{$input}}");

   // Agent 4: Combine results
   var combineAgent = new SemanticFunctionAgent(kernel, "Combine the results to get the final answer: {{$input}}");
   ```

#### Step 4: **Orchestrate the Agents**
   Use Semantic Kernel’s orchestration capabilities to chain the agents together. Each agent’s output will be passed as input to the next agent.

   ```csharp
   var context = new ContextVariables();
   context.Set("input", "What is 25% of 80?");

   // Execute the agents in sequence
   var step1Result = await understandAgent.RunAsync(context);
   context.Set("input", step1Result.ToString());

   var step2Result = await breakAgent.RunAsync(context);
   context.Set("input", step2Result.ToString());

   var step3Result = await solveAgent.RunAsync(context);
   context.Set("input", step3Result.ToString());

   var finalResult = await combineAgent.RunAsync(context);
   Console.WriteLine(finalResult.ToString());
   ```

#### Step 5: **Handle State and Context**
   Agents can maintain state across steps using the `ContextVariables` object. This ensures that the context is preserved throughout the reasoning process.

---

### 4. **Advanced: Multi-Agent Workflows**
   For more complex problems, you can create **multi-agent workflows** where multiple agents collaborate to solve the problem. For example:
   - One agent might focus on extracting key information from the input.
   - Another agent might handle mathematical reasoning.
   - A third agent might validate the results.

   Semantic Kernel’s **planner** can be used to dynamically orchestrate these agents based on the problem.

---

### 5. **Example: Math Problem Solving with Agents**
   Here’s an example of how you can use agents to solve a math problem using CoT:

   ```csharp
   // Define agents
   var understandAgent = new SemanticFunctionAgent(kernel, "Understand the problem: {{$input}}");
   var breakAgent = new SemanticFunctionAgent(kernel, "Break the problem into sub-problems: {{$input}}");
   var solveAgent = new SemanticFunctionAgent(kernel, "Solve the sub-problem: {{$input}}");
   var combineAgent = new SemanticFunctionAgent(kernel, "Combine the results to get the final answer: {{$input}}");

   // Set up context
   var context = new ContextVariables();
   context.Set("input", "What is 25% of 80?");

   // Execute the agents
   var step1Result = await understandAgent.RunAsync(context);
   context.Set("input", step1Result.ToString());

   var step2Result = await breakAgent.RunAsync(context);
   context.Set("input", step2Result.ToString());

   var step3Result = await solveAgent.RunAsync(context);
   context.Set("input", step3Result.ToString());

   var finalResult = await combineAgent.RunAsync(context);
   Console.WriteLine(finalResult.ToString());
   ```

---

### 6. **Benefits of Using Agents for CoT**
   - **Reusability**: Agents can be reused across different workflows.
   - **Scalability**: You can add more agents to handle increasingly complex problems.
   - **Flexibility**: Agents can be combined in different ways to solve a variety of tasks.
   - **Debugging**: It’s easier to debug and test individual agents than a monolithic reasoning process.

---

### 7. **Resources**
   - **Semantic Kernel Documentation**: [Semantic Kernel Docs](https://learn.microsoft.com/en-us/semantic-kernel/)
   - **Agents in Semantic Kernel**: [Agents Overview](https://learn.microsoft.com/en-us/semantic-kernel/concepts/agents)
   - **GitHub Examples**: [Semantic Kernel GitHub](https://github.com/microsoft/semantic-kernel)

---

By using Semantic Kernel’s **Agents**, you can implement CoT reasoning in a modular, scalable, and maintainable way.
