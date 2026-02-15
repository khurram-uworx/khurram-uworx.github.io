---
title: "ScrumUpdate Deep Dive"
date: 2026-02-18
tags:
  - Blazor
  - AI
  - Architecture
  - Implementation
comments: true
---

This post contains the deep-dive technical details and implementation notes. It documents the architecture, data model, key service implementations, testing approach, and a production-readiness checklist.

**Quick Recap:** In blackboard architecture, multiple independent knowledge sources collaborate by reading from and writing to a shared workspace. ScrumUpdate applies this pattern to daily scrum updates: each `ChatSession` is a per-day blackboard where the user, AI assistant, and Jira integrate iteratively to build an accurate scrum summary.

This is **Part 3** of our **Blackboard Architecture** series:

- **[Part 1: Blackboard Architecture: A Classic AI Model for Collaborative Problem Solving](/2026/02/15/Blackboard.html)** — Introduction to blackboard concepts, historical context, and modern agentic AI connections
- **[Part 2: From Theory to Practice: Implementing Blackboard Architecture in Modern Blazor Apps](/2026/02/17/Blackboard-2.html)** — Problem statement, multi-session architecture, and design principles
- **Part 3: ScrumUpdate Deep Dive** — This post (technical deep dive)

---

## Architecture Overview

High level:

- UI: Blazor (interactive server rendering)
- Business: `ChatViewModel` per session, `ChatSessionService`, `ScrumUpdateTools`
- External: Jira (OAuth 2.0), LLM via `IChatClient`
- Persistence: EF Core with `ChatSession` and related entities

Key idea: each `ChatSession` is a per-day blackboard (grain = `ScrumDate`). The UI shows a sidebar of sessions and a chat view for the selected day; the `ChatViewModel` loads the session history and calls into `IChatClient` with the session as context.

## Data Model

Core entities used in the POC:

```csharp
public class ChatSession
{
    public int Id { get; set; }
    public string UserId { get; set; }
    public string Title { get; set; }
    public DateOnly ScrumDate { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime UpdatedDate { get; set; }

    public List<ChatMessageEntity> Messages { get; set; } = new();
    public DayWiseScrumUpdate? DayWiseScrumUpdate { get; set; }
}

public class ChatMessageEntity
{
    public int Id { get; set; }
    public string Role { get; set; }    // "user" | "assistant"
    public string Content { get; set; }
    public string? MetadataJson { get; set; }
    public DateTime Timestamp { get; set; }
}

public class DayWiseScrumUpdate
{
    public int ChatSessionId { get; set; }
    public string WhatIDidYesterday { get; set; } = string.Empty;
    public string WhatIPlanToDoToday { get; set; } = string.Empty;
    public string Blocker { get; set; } = string.Empty;
    public DateTime GeneratedTime { get; set; }
}
```

Design rationale:

- `ScrumDate` makes session discovery intuitive and enforces the per-day grain.
- `Messages` keep a full audit trail (user + assistant messages). `MetadataJson` stores tool outputs and execution traces.
- `DayWiseScrumUpdate` holds the current "accepted" summary for quick display or export.

## Important Services & Flows

1) `JiraScrumUpdateDraftService`

- Responsible for building a structured draft from Jira activity (yesterday + today).
- Calls Atlassian APIs using per-user OAuth tokens stored in the DB and auto-refreshes tokens when needed.

Simplified pseudo-flow:

```csharp
var jiraContext = await atlassianOAuthService.GetScrumContextForTodayAndYesterdayAsync(userId);
var yesterdayItems = BuildYesterdayItems(jiraContext);
var todayItems = BuildTodayItems(jiraContext);
return new GeneratedScrumUpdate { WhatIDidYesterday = ..., WhatIPlanToDoToday = ..., Blocker = ... };
```

2) Tool invocation: `GenerateScrumUpdateDraftAsync`

- Exposed as a callable tool that the assistant can invoke. It runs the Jira generator and returns a formatted result string (or structured response when supported).

3) `IChatClient` abstraction

- All LLM interactions go through `IChatClient`. The repo includes `DummyChatClient` for fast local tests.
- This abstraction enables swapping providers without changing business logic.

## Function Invocation Pattern

- System prompt defines when the assistant should call the tool (e.g., on "generate my scrum update").
- Assistant calls `GenerateScrumUpdateDraftAsync()`.
- Service executes the Jira-backed generator, returns structured data, assistant formats as markdown.

```csharp
[Description("Generate the user's scrum update from Jira activity.")]
public async Task<string> GenerateScrumUpdateDraftAsync(CancellationToken ct = default) {
    var draft = await jiraScrumUpdateDraftService.TryGenerateAsync(ct);
    if (draft == null) return "Unable to fetch Jira data. Please connect Jira.";
    return ScrumUpdateResponseFormatter.Format(draft);
}
```

## Sequence Diagram

UI (Blazor) -> ChatViewModel -> ScrumUpdateTools -> Jira
                                                                            \-> LLM (IChatClient)
                                                                            \-> Database

Detailed flow:

    UI (Blazor)
        |
        | Open session / "generate my scrum"
        v
    ChatViewModel
        |
        | Call GenerateScrumUpdateDraftAsync()
        v
    ScrumUpdateTools
        |
        | Query Jira: worklogs / issues (yesterday + today)
        v
    Jira API
        |
        | Return activity
        v
    ScrumUpdateTools
        |
        | (optional) Ask LLM to summarize/format draft
        v
    LLM (IChatClient)
        |
        | Return draft result
        v
    ScrumUpdateTools
        |
        | Return structured draft + execution metadata
        v
    ChatViewModel
        |
        | Send session + draft to LLM for assistant formatting
        v
    LLM (Assistant)
        |
        | Return assistant message
        v
    ChatViewModel
        |
        | Persist messages + MetadataJson
        v
    Database
        |
        | Persisted
        v
    ChatViewModel -> UI: Render response


## MetadataJson: audit trail and generation detection

Each `ChatMessageEntity.MetadataJson` stores a compact audit record produced whenever a tool or external call contributes to the session. We keep this for three reasons:

- **Audit & reproducibility:** store the exact inputs/outputs, timestamps, request IDs, and provider info so any result can be traced back and re-run.
- **Explainability:** surface which tool produced which fragment of the message and include raw tool output for debugging and user transparency.
- **Generation detection & provenance:** reliably detect whether a message (or part of it) was generated by a tool/LLM, and which provider/model produced it. This is useful for UI indicators ("generated by AI"), compliance, or downstream processing (e.g., preventing automated exports without user confirmation).

Sample `MetadataJson` schema (stored as JSON string):

```json
{
    "tool": "GenerateScrumUpdateDraft",
    "toolVersion": "1.0",
    "provider": "JiraApi",
    "requestId": "abc-123",
    "startedAt": "2026-02-18T10:30:00Z",
    "completedAt": "2026-02-18T10:30:02Z",
    "durationMs": 2000,
    "inputs": { "from": "2026-02-17", "to": "2026-02-18" },
    "outputSummary": "3 worklog entries; 2 issues touched",
    "raw": { "issues": [...], "worklogs": [...] },
    "generated": false
}
```

For LLM-produced messages the record will include provider/model details and the raw assistant text; set `generated: true` to mark content produced by an LLM. When the assistant later synthesizes or the user edits content, we append new `MetadataJson` entries so the message history contains a provenance chain.

Detection heuristics and UX:

- The UI can show a small badge when a message's `MetadataJson.generated == true` to indicate an AI-generated draft.
- Exports (copy/paste, Slack post, Jira comment) can require a confirmation step if the latest accepted scrum contains generated content.
- Security and compliance teams can audit all `raw` fields to verify which API calls happened and what data was used.

## Multi-provider AI support (Gemini / Claude / OpenAI + custom endpoints)

ScrumUpdate is designed to be provider-agnostic. The selection logic lives in `Program.cs` and chooses a client in this order:

- `Gemini` (if configured)
- `Claude` (Anthropic) (if configured)
- `OpenAI` (if configured)
- `DummyChatClient` fallback for local testing

Notably, the `OpenAI` configuration supports a customizable `ApiUrl` — this means you can point the app at any OpenAI-compatible host (self-hosted inference servers or third-party AI hosters that expose an OpenAI-compatible API). See `src/ScrumUpdate.Web/Program.cs` for the exact selection logic and configuration options.

This design lets teams swap providers or use private/self-hosted endpoints without changing business code. Typical configuration keys are `Gemini:ApiKey`, `Claude:ApiKey`, `OpenAI:ApiKey`, `OpenAI:ApiUrl`, and `OpenAI:Model` (see `src/ScrumUpdate.Web/Program.cs`).

## Testing Strategy

- Use `DummyChatClient` to simulate LLM responses deterministically.
- Unit tests cover `JiraScrumUpdateDraftService`, `ChatSessionService`, and `ChatViewModel` behaviors with mocked `ICurrentUserContext` and in-memory EF Core provider.
- Integration tests use an in-memory DB fixture and fake Jira responses.

Example test from the POC (asserts regenerate changes):

```csharp
[Test]
public async Task GetResponseAsync_ScrumUpdateAndRegenerate_ReturnDifferentScrumMessages() {
    var generator = new ScrumGenerator();
    var client = new DummyChatClient(generator);
    var first = await client.GetResponseAsync(new[] { new ChatMessage(ChatRole.User, "scrum update") });
    var second = await client.GetResponseAsync(new[] { new ChatMessage(ChatRole.User, "regenerate") });
    Assert.AreNotEqual(first.Messages.Single().Text, second.Messages.Single().Text);
}
```

## POC Caveats & Production Checklist

The POC prioritizes clarity and architecture over production hardening. Key items before shipping:

- Token & secrets management (secure storage, encryption-at-rest).
- Retry/backoff/circuit-breaker for external APIs (Polly).
- Summarization/paging of long histories to avoid token explosion.
- Observability: logging, metrics, tracing (LLM cost/latency tracking).
- Rate limiting and caching for Jira queries.
- Vector store + RAG strategy for long-term memory and similarity search.
- Comprehensive testing and security review for OAuth flows.

## Suggested Enhancements

- Add a summarization layer to compress long session history before sending to LLM.
- Persist embeddings for sessions and implement cross-session search.
- Add a retrospective aggregator tool that composes weekly summaries from multiple sessions.

---

## 🔗 See It In Action

Want to explore the code? The full implementation is open-source:

**Repository:** [https://github.com/khurram-uworx/scrumupdate](https://github.com/khurram-uworx/scrumupdate)

Key files:
- **Data Model:** `src/ScrumUpdate.Web/Data/ChatDbContext.cs`
- **Service Layer:** `src/ScrumUpdate.Web/Services/ChatSessionService.cs`
- **Jira Integration:** `src/ScrumUpdate.Web/Services/Atlassian/AtlassianOAuthService.cs`
- **Tool Functions:** `src/ScrumUpdate.Web/Services/ScrumUpdateTools.cs`
- **UI Component:** `src/ScrumUpdate.Web/Components/Pages/Chat/Chat.razor`
