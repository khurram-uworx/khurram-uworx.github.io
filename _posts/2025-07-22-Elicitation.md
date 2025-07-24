---
title: "Elicitation in GenAI Applications"
date: 2025-07-22
tags:
- GenAI
comments: true
---

In the context of a **model context protocol**, *elicitation* typically refers to the process of **gathering, extracting, or soliciting relevant information** from a source (often a user, domain expert, or dataset) to inform or populate the context that a model will use during inference or interaction.

This is crucial in systems that use large language models (LLMs), where the **context window** (e.g. prompt or input buffer) is limited. Elicitation ensures that the **most relevant knowledge, constraints, goals, or preferences** are surfaced and included in that context.

In practice, elicitation might involve:

* Asking targeted questions to the user.
* Inferring user intent from prior inputs.
* Selecting relevant documents or facts from a database.
* Extracting key details from ongoing conversations.

It's often part of a **retrieval-augmented generation** pipeline or any protocol where dynamic and efficient context construction is critical for model performance.

You're right — when building a bot within a third-party chat application like **Microsoft Teams**, **Slack**, or **GitHub Copilot Chat**, you're bound by the capabilities of their **SDKs**, **APIs**, and **message formatting systems**. However, even within these constraints, you can still implement *elicitation* effectively as part of your **Model Context Protocol (MCP)**.

### 🔧 How to Implement Elicitation in an MCP-Based Bot

Here’s a functional breakdown of how elicitation could work:

---

### 1. **User-Initiated Elicitation**

Let the user explicitly or implicitly provide information via chat:

* **Example**:
  **User**: “Can you help me generate a quarterly sales report?”
  Your bot elicits:

  > “Sure — to do that, I’ll need:
  > ① The region,
  > ② The time frame (e.g., Q2 2025),
  > ③ The data source (CRM, spreadsheet, etc.).
  > Can you provide those?”

This is a simple **step-by-step clarification** using chat messages, following a structured elicitation flow.

---

### 2. **Implicit Elicitation via Prior Context**

Use the chat history or prior interactions stored (if permitted) to infer missing parameters.

* Use Teams/Slack message threads or app state to pre-fill context for the LLM.
* Cache user preferences or entity values.

Example:

> “Last time you asked for sales data from the Northeast region — use that again?”

---

### 3. **Bot-Initiated Elicitation Workflow**

Define **dialog flows** (via state machines or decision trees) in your bot service to elicit needed info progressively before sending to the LLM.

Each step:

* Collects one missing parameter.
* Stores it (in memory or conversation state).
* Proceeds only when sufficient context is gathered.

**Slack SDK** and **Teams Bot Framework** both allow storing temporary conversation state to support this.

---

### 4. **Elicitation Through Adaptive Cards or Modals (UI Elicitation)**

If SDK permits rich UI (e.g., Adaptive Cards in Teams, modals in Slack), use them to ask for structured input:

* Dropdowns, checkboxes, date pickers, etc.
* Upon submit, you collect structured values to pass into the LLM.

Example card (Teams):

```json
{
  "type": "AdaptiveCard",
  "body": [
    { "type": "Input.ChoiceSet", "id": "region", "choices": [...] },
    { "type": "Input.Date", "id": "dateRange" }
  ],
  "actions": [{ "type": "Action.Submit", "title": "Submit" }]
}
```

---

### 5. **Elicitation Logic Before LLM Invocation**

Your **bot service acts as an MCP engine**, deciding:

* Is enough context collected?
* If not, what to elicit next?
* When context is complete, construct the prompt and call the LLM.

This can be done using a context controller layer that sits before calling the LLM.

---

### 🧠 What the Model Context Protocol (MCP) Manages

* Tracks required inputs for the task.
* Maintains state of collected inputs.
* Chooses when to call the model.
* Formats the final context to send to the model.

---

### ✅ Summary of Elicitation Options in Constrained SDKs

| Approach                      | SDK Requirement             | Good For                 |
| ----------------------------- | --------------------------- | ------------------------ |
| Text-based clarifications     | Basic messaging             | All chat SDKs            |
| Contextual inference          | Message history/state cache | Slack, Teams with memory |
| UI-based input collection     | Adaptive Cards / Modals     | Teams, Slack             |
| Dialog flow-based elicitation | Bot state & routing logic   | All SDKs                 |

---

Here’s a **succinct C# example** of a **Teams Bot** using **Semantic Kernel** where we implement **Elicitation** for a `Reporting` MCP (Model Context Protocol). The bot asks for missing report parameters before invoking the LLM.

Assumptions:

* Using **Bot Framework SDK** for Teams.
* Using **Semantic Kernel** to call the LLM.
* Required parameters: `region`, `timePeriod`, `dataSource`.

---

### 💡 C# Code – Teams Bot (Text-based Elicitation)

```csharp
public class ReportingBot : ActivityHandler
{
    private readonly Dictionary<string, Dictionary<string, string>> _userState = new();

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        var userId = turnContext.Activity.From.Id;
        if (!_userState.ContainsKey(userId))
            _userState[userId] = new();

        var state = _userState[userId];
        var text = turnContext.Activity.Text?.Trim().ToLowerInvariant();

        if (text.Contains("report"))
        {
            await ElicitMissingParamsAsync(turnContext, state, cancellationToken);
            if (IsReady(state))
            {
                var result = await GenerateReportAsync(state);
                await turnContext.SendActivityAsync(MessageFactory.Text(result), cancellationToken);
                _userState.Remove(userId);
            }
        }
        else
        {
            await UpdateStateFromMessageAsync(turnContext, state, cancellationToken);
            await ElicitMissingParamsAsync(turnContext, state, cancellationToken);
        }
    }

    private bool IsReady(Dictionary<string, string> state) =>
        state.ContainsKey("region") && state.ContainsKey("timePeriod") && state.ContainsKey("dataSource");

    private async Task ElicitMissingParamsAsync(ITurnContext turnContext, Dictionary<string, string> state, CancellationToken ct)
    {
        if (!state.ContainsKey("region"))
            await turnContext.SendActivityAsync("What region is this report for?", ct);
        else if (!state.ContainsKey("timePeriod"))
            await turnContext.SendActivityAsync("Which time period should I use (e.g., Q2 2025)?", ct);
        else if (!state.ContainsKey("dataSource"))
            await turnContext.SendActivityAsync("What data source should I use (CRM, Excel, etc.)?", ct);
    }

    private async Task UpdateStateFromMessageAsync(ITurnContext turnContext, Dictionary<string, string> state, CancellationToken ct)
    {
        var input = turnContext.Activity.Text?.ToLowerInvariant();

        if (!state.ContainsKey("region") && input.Contains("region"))
            state["region"] = ExtractValue(input, "region");

        else if (!state.ContainsKey("timePeriod") && input.Contains("q"))
            state["timePeriod"] = input;

        else if (!state.ContainsKey("dataSource") && (input.Contains("crm") || input.Contains("excel")))
            state["dataSource"] = input;
    }

    private async Task<string> GenerateReportAsync(Dictionary<string, string> state)
    {
        var kernel = new KernelBuilder().Build();
        var prompt = $"""
        Generate a report for:
        - Region: {state["region"]}
        - Time Period: {state["timePeriod"]}
        - Data Source: {state["dataSource"]}
        """;

        var func = kernel.CreateSemanticFunction("{{$input}}", new() { MaxTokens = 1000 });
        return await func.InvokeAsync(prompt);
    }

    private string ExtractValue(string input, string keyword)
    {
        var parts = input.Split(' ');
        var idx = Array.FindIndex(parts, p => p.Contains(keyword));
        return idx >= 0 && idx < parts.Length - 1 ? parts[idx + 1] : "unspecified";
    }
}
```

---

This bot:

* Tracks missing parameters using in-memory state per user.
* Prompts only for what’s missing.
* Uses Semantic Kernel to generate the report once all inputs are collected.

Absolutely — Adaptive Cards are ideal in **Teams bots** for structured elicitation. Below is a concise C# example where the bot sends an **Adaptive Card** to collect the `region`, `timePeriod`, and `dataSource`, then passes the result to **Semantic Kernel**.

---

### 🧩 Step 1: Adaptive Card JSON (you can keep this as a file or inline)

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "Please provide report parameters",
      "weight": "Bolder",
      "size": "Medium"
    },
    {
      "type": "Input.ChoiceSet",
      "id": "region",
      "style": "compact",
      "label": "Region",
      "choices": [
        { "title": "North America", "value": "North America" },
        { "title": "Europe", "value": "Europe" },
        { "title": "Asia", "value": "Asia" }
      ]
    },
    {
      "type": "Input.Text",
      "id": "timePeriod",
      "label": "Time Period (e.g., Q2 2025)"
    },
    {
      "type": "Input.ChoiceSet",
      "id": "dataSource",
      "style": "compact",
      "label": "Data Source",
      "choices": [
        { "title": "CRM", "value": "CRM" },
        { "title": "Excel", "value": "Excel" },
        { "title": "Database", "value": "Database" }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Generate Report"
    }
  ]
}
```

---

### 🧠 Step 2: Bot Logic in C\#

```csharp
public class ReportingBot : TeamsActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        var card = File.ReadAllText("Cards/reporting-request.json");
        var attachment = new Attachment
        {
            ContentType = "application/vnd.microsoft.card.adaptive",
            Content = JsonConvert.DeserializeObject(card)
        };

        await turnContext.SendActivityAsync(MessageFactory.Attachment(attachment), cancellationToken);
    }

    protected override async Task OnTeamsMessagingExtensionSubmitActionDispatchAsync(
        ITurnContext<IInvokeActivity> turnContext,
        CancellationToken cancellationToken)
    {
        var values = ((JObject)turnContext.Activity.Value).ToObject<Dictionary<string, string>>();
        if (values != null &&
            values.TryGetValue("region", out var region) &&
            values.TryGetValue("timePeriod", out var timePeriod) &&
            values.TryGetValue("dataSource", out var dataSource))
        {
            var result = await GenerateReportAsync(region, timePeriod, dataSource);
            await turnContext.SendActivityAsync(MessageFactory.Text(result), cancellationToken);
        }
    }

    private async Task<string> GenerateReportAsync(string region, string timePeriod, string dataSource)
    {
        var kernel = new KernelBuilder().Build();
        var prompt = $"""
        Generate a report for:
        - Region: {region}
        - Time Period: {timePeriod}
        - Data Source: {dataSource}
        """;

        var func = kernel.CreateSemanticFunction("{{$input}}", new() { MaxTokens = 1000 });
        return await func.InvokeAsync(prompt);
    }
}
```

---

### ✅ What This Does:

* Sends an **Adaptive Card** when user says anything.
* Collects structured input via **Action.Submit**.
* Calls **Semantic Kernel** once all parameters are submitted.

Here’s a simple **dummy interaction** that shows how a Teams user would interact with the bot using **Adaptive Cards** for elicitation in your **Reporting MCP** scenario:

---

### 👤 **User**:

> I need a report

---

### 🤖 **Bot** (sends Adaptive Card):

```
[Adaptive Card UI rendered in Teams]

📝 Please provide report parameters

📍 Region:
   ☐ North America  
   ☐ Europe  
   ☐ Asia  

📅 Time Period:
   [ Q2 2025 ] ← (text input)

📊 Data Source:
   ☐ CRM  
   ☐ Excel  
   ☐ Database  

[Generate Report ✅]
```

---

### 👤 **User selects**:

* Region: Europe
* Time Period: Q2 2025
* Data Source: Excel
  (Clicks **Generate Report**)

---

### 🤖 **Bot**:

> 📄 Here's your report for **Europe**, for **Q2 2025**, using **Excel**:
>
> *"The total sales in Europe for Q2 2025 showed a 12% increase over the previous quarter, driven primarily by strong performance in Germany and France..."*

---

This is how *elicitation* looks in a clean, professional Teams flow using Adaptive Cards — clear UI, guided inputs, and a sharp response once context is complete.

When building an **MCP Server** using the [official MCP SDK for C#](https://github.com/modelcontextprotocol/csharp-sdk), there are several architectural and design considerations to keep in mind — especially when incorporating **elicitation** logic for parameter gathering (like your `report` example).

Let’s walk through:

---

## ✅ Key Considerations for Building an MCP Server (with Elicitation)

### 1. **MCP-Compliant Contract Structure**

You need to define:

* **Capabilities**: What can this skill do? (e.g., `generate-report`)
* **Parameters**: What inputs are required?
* **Context**: What’s currently known or still missing?

Use the SDK to define these in a structured, declarative way.

---

### 2. **Elicitation Protocol (Core to MCP)**

The server must:

* Determine what parameters are **missing or ambiguous**.
* Return structured **"elicitation requests"** to the calling client (e.g., your Teams bot).
* Track **progressively fulfilled inputs** (via `ContextObject` and `PartialContext`).

---

## 🧱 Example: `ReportGenerationSkill` with Elicitation

### ✳️ Step 1: Define the `report` method skill

```csharp
public class ReportSkill : ISkill
{
    public SkillManifest Manifest => new SkillManifest
    {
        Name = "report",
        Description = "Generate business reports from context",
        Parameters = new List<ParameterDefinition>
        {
            new("region", "Region of the report", isRequired: true),
            new("timePeriod", "Time period (e.g., Q2 2025)", isRequired: true),
            new("dataSource", "Data source to use", isRequired: true)
        }
    };

    public async Task<SkillResult> ExecuteAsync(ContextObject context, CancellationToken ct)
    {
        var region = context.Get("region");
        var time = context.Get("timePeriod");
        var source = context.Get("dataSource");

        if (region == null || time == null || source == null)
        {
            return SkillResult.MissingParameters(new[] { "region", "timePeriod", "dataSource" }
                .Where(p => context.Get(p) == null)
                .ToArray());
        }

        var report = await GenerateReportWithSemanticKernel(region, time, source);
        return SkillResult.Completed(report);
    }

    private async Task<string> GenerateReportWithSemanticKernel(string region, string time, string source)
    {
        var kernel = new KernelBuilder().Build();
        var input = $"""
        Generate a business report for:
        - Region: {region}
        - Time Period: {time}
        - Data Source: {source}
        """;

        var func = kernel.CreateSemanticFunction("{{$input}}");
        return await func.InvokeAsync(input);
    }
}
```

---

### ✳️ Step 2: Elicitation Flow (from client perspective)

When you call the MCP Server with partial context:

```json
{
  "skill": "report",
  "context": {
    "region": "Europe"
  }
}
```

The server will respond:

```json
{
  "status": "MissingParameters",
  "missing": ["timePeriod", "dataSource"]
}
```

Your bot (client) then uses this to generate a UI (Adaptive Card) or follow-up prompt to collect missing values.

---

### ✳️ Step 3: Complete Round

Once all fields are submitted via UI or conversation:

```json
{
  "skill": "report",
  "context": {
    "region": "Europe",
    "timePeriod": "Q2 2025",
    "dataSource": "Excel"
  }
}
```

The MCP Server will respond:

```json
{
  "status": "Completed",
  "output": "The Q2 2025 report for Europe shows a 12% increase in revenue based on Excel data..."
}
```

---

## 🧠 Summary – Building a Robust MCP Server

| Consideration                     | Recommendation                                                                           |
| --------------------------------- | ---------------------------------------------------------------------------------------- |
| 🧩 Skill Parameter Modeling       | Use `SkillManifest` with `ParameterDefinition`                                           |
| 🤖 Elicitation Handling           | Return `SkillResult.MissingParameters()` if inputs missing                               |
| 📦 Progressive Context Completion | Support partial context from clients                                                     |
| 🪝 Integration with UI/Client     | Design clients to respond to `MissingParameters` and resubmit                            |
| 🧠 Backend Intelligence           | Consider optional inference using Semantic Kernel or memory                              |
| 🧪 Test with `mcp-cli`            | Try your skills locally using the [MCP CLI](https://github.com/modelcontextprotocol/cli) |

---

This approach ensures your MCP Server is **flexible, extensible**, and **compliant** with the Model Context Protocol, while also providing a robust elicitation mechanism for gathering necessary parameters dynamically.