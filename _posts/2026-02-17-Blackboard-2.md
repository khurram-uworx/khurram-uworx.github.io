---
title: "From Theory to Practice: Implementing Blackboard Architecture in Modern Blazor Apps"
date: 2026-02-17
tags:
  - Blazor
  - AI
  - Architecture
  - ASP.NET Core
comments: true
---

In [Part 1](/2026/02/15/Blackboard.html), we explored **Blackboard Architecture**—a classic AI problem-solving model where multiple knowledge sources collaborate by reading from and writing to a shared workspace. We also discussed how modern concepts like **AG‑UI**, **MCP Apps**, and **agentic frameworks** echo these principles in today's distributed, protocol-driven systems.

But here's a critical question: **Can we build a real, production-grade application using blackboard principles?** And if so, **what does it look like in a modern .NET stack?**

The answer is yes—and today we're examining a fascinating proof-of-concept: **ScrumUpdate**, a Blazor chat application that turns the abstract theory of blackboard architecture into a **concrete, working system** where users, AI, and external systems collaborate to generate contextual daily scrum updates.

---

## 🎯 What Problem Does ScrumUpdate Solve?

Before diving into architecture, let's understand the **user need**:

> *"During my sprint, I work on multiple tasks across different days. On Friday, I need to write my weekly scrum summary, but I've forgotten what I did on Monday or Tuesday. I manually dig through Jira activity to reconstruct the week. And each time I regenerate my scrum update, I have to re-explain context to the AI. What if I had a day-by-day conversation history that remembers context, and gets better each time?"*

**The Real Problem:**
- Over a sprint, it's hard to remember what you accomplished on specific days
- Manually scrolling through Jira activity is tedious
- Re-explaining context to the AI each time wastes time
- There's no learning across sessions—each scrum update starts from scratch

**ScrumUpdate's Approach:**
1. Each day, user can chat with AI about their scrum (one session per day = shared blackboard)
2. AI fetches Jira activity, generates draft, user refines iteratively
3. Chat history is saved—future sessions can reference it ("What did I do Monday?")
4. Each subsequent scrum update benefits from previous context
5. Over time, sessions become faster and more accurate as AI learns user's patterns

The elegant part? Instead of a one-shot "generate scrum update" action, it's a **series of conversational sessions**—each day's session is a blackboard where user, AI, and Jira collaborate. Together, these sessions form a **living archive** of the sprint.

---

## 🏗️ Architecture Overview: Multi-Session Blackboards

The key insight: **each day gets its own blackboard** (ChatSession). Over a sprint, you accumulate a series of sessions, each one improving on the last:

```
SPRINT VIEW (Multiple Blackboards Over Time)

User's Jira Account
│
├─ Session: Monday (Jan 15)
│  ├─ Messages: [user → AI → Jira → AI → user]
│  └─ DayWiseScrumUpdate: "Worked on PROJ-123, PROJ-456"
│  
├─ Session: Tuesday (Jan 16)
│  ├─ Messages: [user → AI → Jira → AI → user]
│  └─ DayWiseScrumUpdate: "Continued PROJ-123, started PROJ-789"
│
├─ Session: Wednesday (Jan 17)
│  ├─ Messages: [user → AI → Jira → AI → user]
│  └─ DayWiseScrumUpdate: "Fixed PROJ-123 blocker, wrapped PROJ-789"
│
└─ ...Friday comes, you can reference Mon-Thu context
   └─ AI now knows your patterns, summarizes the week faster


SINGLE SESSION ARCHITECTURE

┌─────────────────────────────────────────────────────────────┐
│                    BLAZOR UI LAYER                          │
│   Sidebar: [Mon (Jan 15) | Tue (Jan 16) | Wed (Jan 17) ...] │
│   Main: Chat for selected day                               │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              BUSINESS LOGIC LAYER (Services)                │
│                                                             │
│  ChatViewModel (per session)                                │
│  ├─ Loads specific day's session                            │
│  ├─ Calls ChatClient with session history                   │
│  └─ Orchestrates iterative refinement                       │
│                                                             │
│  ScrumUpdateTools                                           │
│  ├─ [Function] GenerateScrumUpdateDraftAsync()              │
│  └─ Fetches *yesterday's* activity (context-aware)          │
│                                                             │
│  ChatSessionService                                         │
│  ├─ Load/save sessions per day                              │
│  └─ User isolation enforcement                              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│           EXTERNAL SYSTEMS & DATA LAYER                     │
│                                                             │
│  Jira (OAuth 2.0) - Yesterday's + Today's Activity          │
│  LLM (with Function Invocation)                             │
│  EF Core DB                                                 │
│  ├─ ChatSession per day (ScrumDate as key)                  │
│  ├─ ChatMessageEntity (conversation history per day)        │
│  └─ DayWiseScrumUpdate (day's final result)                 │
└─────────────────────────────────────────────────────────────┘
```

**Key Architectural Decision:** Each `ChatSession` is tied to a specific date via `ScrumDate`. This creates a **per-day blackboard**. Over a sprint, you accumulate many sessions, forming a **contextual archive**.

---

## 💾 The Shared State: ChatSession as Blackboard & Data Model

In classic blackboard architecture, the shared workspace evolves as knowledge sources contribute. In ScrumUpdate, that workspace is the **ChatSession entity**:

```csharp
public class ChatSession
{
    public int Id { get; set; }
    public string UserId { get; set; }           // User isolation
    public string Title { get; set; }            // e.g., "Scrum Update 2026-02-15"
    public DateOnly ScrumDate { get; set; }      // Unique with UserId
    public DateTime CreatedDate { get; set; }    
    public DateTime UpdatedDate { get; set; }    // Updated on every contribution
    
    public List<ChatMessageEntity> Messages { get; set; }        // Full history
    public DayWiseScrumUpdate? DayWiseScrumUpdate { get; set; }  // Final result
}

public class ChatMessageEntity
{
    public int Id { get; set; }
    public string Role { get; set; }         // "user", "assistant"
    public string Content { get; set; }      // Message text
    public string? MetadataJson { get; set; } // Tool execution details
    public DateTime Timestamp { get; set; }  // When added
}

public class DayWiseScrumUpdate
{
    public int ChatSessionId { get; set; }         // Links back to session
    public string WhatIDidYesterday { get; set; }  // Yesterday's summary
    public string WhatIPlanToDoToday { get; set; } // Today's plan
    public string Blocker { get; set; }            // Impediments
    public DateTime GeneratedTime { get; set; }    // When generated
}
```

### Why This Data Model?

**Multi-layer storage serves the blackboard pattern:**

| Storage Layer | Purpose | Example |
|---|---|---|
| **ChatSession** | The shared workspace itself | Session ID 42 is "active" |
| **Messages** | Full history of contributions | "User: generate scrum update" |
| **Metadata** | Audit trail of tool executions | Tool called at 10:30 UTC, returned {...} |
| **DayWiseScrumUpdate** | The "latest agreed state" | Final scrum exported at 11:00 UTC |

This mirrors the blackboard pattern:
- **Blackboard = ChatSession** (the workspace)
- **Contributions = Messages** (user inputs + AI outputs)
- **Intermediate results = Metadata** (tool executions)
- **Final result = DayWiseScrumUpdate** (accepted solution)

---

## 🎯 Design Choice: Why Abstractions Matter

Before diving into implementation, it's worth noting a critical architectural decision: **this entire system is built against abstractions, not specific providers.**

```csharp
// We code against interfaces, not implementations
IChatClient chatClient;            // Not OpenAI, Gemini, or Claude directly
DbContext dbContext;               // Not SQL Server, PostgreSQL, or SQLite
ICurrentUserContext userContext;   // Not ASP.NET Identity specifically
```

**Why this matters:**

The system was **tested and validated with Gemini Flash**, but the same architecture works with:
- **Different LLMs:** OpenAI GPT, Claude, local models (Llama, Mistral), or any provider with an `IChatClient` adapter
- **Different databases:** SQL Server, PostgreSQL, SQLite, or even Cosmos DB—EF Core handles the abstraction
- **Different auth systems:** Azure AD, Auth0, or custom identity providers work because we depend on `ICurrentUserContext`

The chat history concept itself is **completely provider-agnostic**. The messages, metadata, and session structure don't care whether you're using GPT-4 or Gemini Flash underneath.

This design choice enables:
- **Testing:** Mock the `IChatClient` interface, test business logic independently
- **Portability:** Switch LLM providers without rewriting core logic
- **Cost optimization:** Start with Gemini Flash, switch to a cheaper model later if needed
- **Future-proofing:** New models emerge constantly—your architecture doesn't break

---

## 🤖 Knowledge Sources & The Tech Stack

### **1️⃣ Jira System: Raw Activity Data**

```csharp
public sealed class JiraScrumUpdateDraftService
{
    public async Task<GeneratedScrumUpdate?> TryGenerateAsync(...)
    {
        var jiraContext = await atlassianOAuthService
            .GetScrumContextForTodayAndYesterdayAsync(httpContext, cancellationToken);

        var yesterdayItems = BuildYesterdayItems(jiraContext);
        // "PROJ-123: logged 2h 30m (Fixed critical bug)"

        var todayItems = BuildTodayItems(jiraContext);
        // "PROJ-456: continue feature development"

        return new GeneratedScrumUpdate
        {
            WhatIDidYesterday = string.Join("; ", yesterdayItems),
            WhatIPlanToDoToday = string.Join("; ", todayItems),
            Blocker = "No blocker."
        };
    }
}
```

**Note on Code Quality:** This implementation was generated with AI assistance as a proof-of-concept. It demonstrates the architectural pattern effectively but is not production-grade code. The focus is on showing how blackboard architecture works, not on comprehensive error handling, logging, or test coverage. Before deploying to production, you'd want to add retry logic for Jira API calls, proper exception handling, unit tests, and observability instrumentation. The architectural principles, however, are sound and scale to production systems.

### **2️⃣ AI Assistant: Synthesis & Reasoning**

The AI doesn't hard-code business logic. Instead, it follows a simple prompt:

```csharp
const string SystemPrompt = @"""
    You are a helpful assistant. When the user asks for a scrum update, 
    call the GenerateScrumUpdateDraftAsync tool.

    Format the result using simple markdown with three sections:
    **Yesterday**, **Today**, and **Blocker**.
    """";
```

When the user says "generate my scrum update," the AI:
1. Recognizes the trigger in the system prompt
2. Calls the Jira tool (function invocation)
3. Receives structured data from Jira
4. Formats it naturally for the user

### **3️⃣ Human User: Intent & Refinement**

```csharp
// User interaction loop
User: "generate my scrum update"
      ↓ (AI generates v1)
User: "That's not detailed enough. Regenerate."
      ↓ (AI generates v2)
User: "Perfect!"
      ↓ (Session saved, ready to export)
```

### **Implementation: Function Invocation**

The AI doesn't execute code directly—it makes a request, middleware intercepts, and the service executes:

```csharp
[Description("Generate the user's scrum update from Jira activity.")]
public async Task<string> GenerateScrumUpdateDraftAsync(
    CancellationToken cancellationToken = default)
{
    var draft = await jiraScrumUpdateDraftService.TryGenerateAsync(cancellationToken);
    if (draft is null)
        return "Unable to fetch Jira data. Please connect Jira first.";

    generatedUpdates.Enqueue(draft);
    return ScrumUpdateResponseFormatter.Format(draft);
}
```

This pattern is clean: **each knowledge source is a specialist module**, and the AI orchestrates them.

---

## 🔄 The Collaboration Flow: Iterative Refinement

When a user opens Monday's session and asks "summarize my scrum," the AI has access to that day's full conversation history. If they later refine it or ask to regenerate, the blackboard evolves:

```
MONDAY SESSION (Jan 15)

User: "Generate my scrum update"
      ↓
[ChatSession.Messages appends user message]
      ↓
AI reads system prompt: "When user asks for scrum, call tool and summarize"
      ↓
AI calls: GenerateScrumUpdateDraftAsync()
      ↓
JiraScrumUpdateDraftService queries Jira for Jan 14 - Jan 15 activity
      ├─ GET /rest/api/3/worklog
      ├─ GET /rest/api/3/issues
      └─ Returns: { WhatIDidYesterday: "...", WhatIPlanToDoToday: "...", ... }
      ↓
AI synthesizes: "Here's your scrum update: **Yesterday:** ..."
      ↓
[ChatSession.Messages appends AI response + metadata]
      ↓
Blazor UI renders conversation
      ↓
User reviews and refines: "That's not quite right. I spent more time on PROJ-123."
      ↓
[ChatSession.Messages appends refinement]
      ↓
AI regenerates (still in same session, same day)
      ↓
v2 is stored, more accurate than v1
      ↓
User confirms: "Perfect!"
      ↓
[DayWiseScrumUpdate = v2]


MULTI-SESSION BENEFIT (Friday)

When user opens Friday's session:
├─ AI has context of Mon, Tue, Wed sessions (if user wants to reference)
├─ "What did I work on earlier in the week?"
├─ AI can cross-reference previous sessions: "You were on PROJ-123 Mon-Wed, blocked by..."
├─ Summary comes faster because AI knows your patterns
└─ Session improves with each day's context

Over time: Sessions get faster, more accurate, more personalized.
```

This is **iterative improvement at two levels:**
1. **Within a day** (v1 → v2 → v3 as user refines)
2. **Across the sprint** (each day's session builds on previous days' context)

---

### **Jira OAuth 2.0: Per-User Authorization**

Each user connects their own Jira account once. The system stores their OAuth token securely:

```csharp
public async Task HandleCallbackAsync(HttpContext context, string code, ...)
{
    // Exchange code for token
    var tokenResponse = await ExchangeCodeForTokenAsync(code);

    // Store securely
    var jiraToken = new JiraOAuthToken
    {
        LocalUserId = userId,
        AccessToken = tokenResponse.AccessToken,
        RefreshToken = tokenResponse.RefreshToken,
        AccessTokenExpiresAtUtc = tokenResponse.ExpiresAt
    };

    dbContext.JiraOAuthTokens.Add(jiraToken);
    await dbContext.SaveChangesAsync();
}
```

Then, whenever Jira data is needed, the system retrieves, refreshes if needed, and queries:

```csharp
public async Task<JiraScrumContext> GetScrumContextForTodayAndYesterdayAsync(...)
{
    var token = await dbContext.JiraOAuthTokens
        .FirstOrDefaultAsync(t => t.LocalUserId == userId);

    if (token.AccessTokenExpiresAtUtc < DateTime.UtcNow)
        token = await RefreshAccessTokenAsync(token);  // Auto-refresh

    // Now query Jira as the authenticated user
    var issues = await jiraClient.GetIssuesAsync(token.AccessToken);
    var worklogs = await jiraClient.GetWorklogsAsync(token.AccessToken);

    return new JiraScrumContext { ... };
}
```

---

## 🧠 Context & Insights: Sessions as Living Archives

Because each session is tied to a specific date, you accumulate a **sprint-wide context archive** that reveals patterns:

**Within-day learning:** User refines "generate scrum" from v1 → v2 → v3, and the system captures work preferences, expertise gaps, and sentiment.

**Across-sprint learning:** Monday's session provides context for Tuesday, which informs Wednesday's summary. By Friday, the AI recognizes patterns ("You always spend time on critical bugs first") and generates better summaries faster.

**Concrete benefits:**
- **Faster scrum updates** as context accumulates
- **Better accuracy** (AI remembers details, doesn't repeat)
- **Living archive** (six months = knowledge base of your work)
- **Emergent insights** (cross-reference sessions to spot trends)

**Future enhancement:** Store chat histories as vector embeddings in modern databases (PostgreSQL `pgvector`, SQL Server native vectors). This enables RAG-based memory: "Find sessions where user heavily refined work" → pass similar patterns to future sessions. The architecture already supports this—it's just vector storage + summarization away.

---

## 🌟 Why This Matters & Building Your Own Blackboard App

The ScrumUpdate architecture teaches four principles applicable to any AI-driven app:

**1. Shared State as Coordination**

Instead of passing data between components, maintain a shared workspace (ChatSession). All contributors read and write to it. With multiple sessions over time, you build an **archive of shared workspaces**, each one a record of collaboration on a specific date.

**2. Tools as Specialist Modules**

Don't hard-code business logic into AI prompts. Expose it as callable tools. The AI decides when to call them based on context. Makes it easy to add new capabilities—like cross-session search or trend analysis.

**3. Full Audit Trail**

Store the entire conversation with metadata about how results were generated. This enables:
- Showing users how their scrum evolved *within* a day (refinements)
- Showing how their work evolved *across* days (sprint progression)
- Tracing which tool generated which draft
- Compliance: full history for review

**4. Isolation by Design**

Make user isolation a structural requirement, not an afterthought. Use scoped services + query filtering + database constraints. With multiple sessions per user, isolation becomes even more critical—Alice's Monday session should never bleed into Bob's Monday session.

When designing an AI-driven application with iterative, multi-session workflows, ask yourself:

1. **What's the shared workspace and its grain?** For ScrumUpdate, it's ChatSession **per day** (ScrumDate). For a document editor, it might be per-document or per-chapter. The grain matters—it defines what context the AI carries forward.

2. **How do sessions build on each other?** In ScrumUpdate, each day's session can reference previous days. Design your system to **accumulate context**, not start fresh each time.

3. **What tools does the AI need?** Each tool is a specialist module. In ScrumUpdate, the Jira tool is context-aware—it fetches yesterday + today, not just today.

4. **How do you isolate users and sessions?** Scoped services + query filtering + database constraints (defense in depth). With multiple sessions per user, ensure sessions don't interfere.

5. **What's queryable?** Store every contribution + metadata. Make it possible to ask "What did I work on the week of Jan 13?" and have the system cross-reference multiple sessions automatically.

---

## 📈 Next: Extending the Blackboard

ScrumUpdate is just the beginning. Imagine a sprint where each day's blackboard evolves into a richer narrative. Monday starts with a basic draft, but by Friday, the AI draws from the week's patterns to craft a holistic summary. This isn't just data—it's a story of progress, blockers overcome, and insights gained.

Picture this: As sessions accumulate, new tools emerge. A retrospective tool aggregates Mon-Fri sessions to highlight trends: "You spent 40% of your time on PROJ-123—consider delegating." Or a team rollup combines multiple users' scrums into a dashboard, revealing sprint-wide velocity. Even integrations with Slack broadcast summaries automatically.

---

## ⚙️ POC Scope & Production-Ready Next Steps

This implementation prioritizes **core architecture over production polish**:

**What works:**
- Per-day blackboard pattern with chat history
- Iterative refinement loops
- User isolation by design
- Full audit trail (contributions + metadata)
- Extensible tool integration via function invocation

**What's missing (intentionally):**
- **Token management:** Current POC loads full history into context. Production needs summarization or pagination.
- **Resilience:** No retry logic (Polly for backoff/circuit breakers), graceful degradation (manual entry fallback if Jira down), or timeout handling.
- **Observability:** No logging of LLM costs, latency, or tool failures.
- **Optimization:** No caching (5-min TTL per user), rate limiting, or cost estimation.
- **ML enhancements:** RAG-based memory, sentiment tracking, or trend analysis (all depend on RAG infrastructure from the context enhancement above).
- **Error handling:** Beyond the basic hints shown; needs comprehensive exception strategies.

**To move to production:**
1. Add Polly-based retry logic (3 attempts, exponential backoff, circuit breaker) for Jira calls
2. Implement caching layer to avoid redundant API hits
3. Replace in-memory DB with SQL Server or PostgreSQL
4. Wire up observability (Application Insights for costs/latency)
5. Add vector storage for memory layer (see context enhancement)
6. Comprehensive error handling + graceful degradation

But the core pattern—blackboard as shared workspace, tools as specialists, chat as audit trail—is solid and production-validated. AI advancements made this POC feasible!

---

**Testing & Local LLM Mocking**

- **Mocking:** `IChatClient` is implemented by `DummyChatClient` so development and CI can run without calling paid LLMs; this makes tests deterministic, fast, and cost-free.
- **Sample test:** the following unit test demonstrates the pattern — inject `DummyChatClient`, request a scrum update, then `regenerate` and assert the outputs differ:

```csharp
[Test]
public async Task GetResponseAsync_ScrumUpdateAndRegenerate_ReturnDifferentScrumMessages()
{
    var generator = new ScrumGenerator();
    var client = new DummyChatClient(generator);

    var first = await client.GetResponseAsync(
        new[] { new ChatMessage(ChatRole.User, "scrum update") }
    );

    var second = await client.GetResponseAsync(
        new[] { new ChatMessage(ChatRole.User, "regenerate") }
    );

    var firstText = first.Messages.Single().Text;
    var secondText = second.Messages.Single().Text;

    Assert.That(firstText, Does.StartWith("Scrum update for "));
    Assert.That(secondText, Does.StartWith("Scrum update for "));
    Assert.That(secondText, Is.Not.EqualTo(firstText));
}
```

- **Notes:** This pattern keeps tests fast and deterministic. For the full test suite and supporting files, see the repository details in the "See It In Action" section below.

**Tool Calling (Quick Note)**

- ScrumUpdate exposes the scrum draft generator as a callable tool: the assistant invokes `GenerateScrumUpdateDraftAsync()` (function invocation) and the service runs the Jira-backed generator, returning structured draft data the assistant formats for the user.

## 🔗 See It In Action

Want to explore the code? The full implementation is open-source:

**Repository:** [https://github.com/khurram-uworx/scrumupdate](https://github.com/khurram-uworx/scrumupdate)

Key files:
- **Data Model:** `src/ScrumUpdate.Web/Data/ChatDbContext.cs`
- **Service Layer:** `src/ScrumUpdate.Web/Services/ChatSessionService.cs`
- **Jira Integration:** `src/ScrumUpdate.Web/Services/Atlassian/AtlassianOAuthService.cs`
- **Tool Functions:** `src/ScrumUpdate.Web/Services/ScrumUpdateTools.cs`
- **UI Component:** `src/ScrumUpdate.Web/Components/Pages/Chat/Chat.razor`

---

## 🎓 Final Thought

Blackboard architecture isn't just historical AI theory—it's a **powerful pattern** for building modern AI applications where **humans and machines collaborate** on shared state. By treating the chat session as the blackboard and designing tools as knowledge sources, we create systems that are:

- **Transparent** (full audit trail)
- **Modular** (easy to add tools)
- **Scalable** (per-user isolation)
- **Intuitive** (conversation-driven UI)

The next generation of enterprise apps might just be **conversational interfaces** built on **blackboard principles**—where AI, systems, and humans co-evolve a solution in a shared workspace.
