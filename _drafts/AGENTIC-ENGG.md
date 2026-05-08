# The Tiny Team Thesis

This is one of the most interesting structural shifts happening in software right now, and I think you're largely right — but the story is more nuanced than it first appears.

The communication channel math is real. Metcalfe's law applied to team overhead: n(n-1)/2 channels. At 6 people that's 15 relationships to maintain, align, and keep unblocked. Every standup, every PR review cycle, every "who owns this?" conversation is friction that scales super-linearly. Small teams sidestep this almost entirely.

Let me visualize both the core problem and where AI fits:Here's where I'd push back, agree, and add nuance:

**Where you're spot on**

The communication overhead argument is airtight. Brooks' Law from 1975 still holds — adding people to a late software project makes it later. The coordination tax is real. A 2-person team with agents doing the work of 6 is genuinely more efficient because the human-to-human surface stays tiny. That's not speculation anymore; we're seeing it empirically at companies like Cursor, Vercel, Linear — all tiny teams, outsized output.

**The harder insight you're pointing at**

The real unlock isn't just *replacing headcount*. It's changing what humans are responsible for. In a tiny AI-augmented team, the engineer's job shifts from "write code" to "define intent clearly enough that agents can act on it without ambiguity." That's a fundamentally different skill. Context engineering — how you structure system prompts, manage agent memory, design task decomposition — becomes the critical path. Bad context engineering in a 2-person team is worse than unclear tickets in a 10-person team.

**Where I'd add friction to the optimism**

A few things the "tiny teams" thesis underweights:

- **Verification overhead grows.** Agents are fast but not always right. With 6 engineers, bugs get caught in review. With 2 humans and 6 agents, you need robust automated testing and eval pipelines — otherwise the humans become the bottleneck reviewing agent output, which is a *new* kind of coordination problem.
- **Domain complexity doesn't compress.** For CRUD apps and greenfield projects — tiny teams + agents work beautifully. For highly regulated domains (finance, medical devices), distributed systems with complex failure modes, or orgs with massive legacy codebases, the context window of an agent and its ability to reason about subtle constraints hits a ceiling humans don't.
- **Organizational trust lags.** Most companies deploying software aren't startups. Getting a CTO at an enterprise to sign off on an autonomous agent merging PRs to production is a change management problem that won't be solved by better prompts.

**The genuine shift**

The model I think actually plays out: the *floor* for what a small team can build rises dramatically. A 3-person startup can now build what a 20-person team could build in 2020. But the *ceiling* for what requires humans doesn't disappear — it just moves up. The question is how fast that ceiling climbs relative to agent capability. Right now, agents are excellent at narrow, well-specified tasks. The autonomous SDLC — planning, architecting, testing, deploying, monitoring with minimal human intervention — is close for simple domains and still far for complex ones.

What's your take on the verification problem? That feels like the unsolved piece of the tiny team thesis to me.

# The verification problem

Ha — "mentally prepare myself" is doing a lot of work in that sentence. That's not a technical problem anymore, that's a psychological one.

You've stumbled into what I think is the central irony of the current agent moment: **the agent removed the coding bottleneck and immediately exposed the review bottleneck**. You didn't save time, you deferred it — and now it's compounding.

A few honest observations:

**The 8,000-line review is probably unworkable as stated.** Nobody meaningfully reviews 8,000 lines of unfamiliar code in one go. What you'd actually be doing is skimming with occasional deep dives, which is a false sense of security more than actual verification. The mental model of "I need to review all of it" might itself be the wrong frame.

**What actually works instead:**

Rather than reviewing code, review *behavior*. Does it do what you specified? Does it fail in expected ways? Does it handle edge cases you care about? Tests, logs, and integration runs tell you more than reading 8,000 lines. The code is almost implementation detail if the observable behavior is correct and the architecture isn't a disaster.

Then you spot-check architecture decisions — not every line, but the structural choices: How is state managed? Where are the external dependencies? What does error handling look like? 20 minutes on that is worth more than 3 hours of linear reading.

**The real lesson for your next sessions:**

The agent didn't just generate code — it generated *unreviewed debt* faster than you can process it. The fix isn't to slow the agent down. It's to change the loop:

- Smaller sessions with review checkpoints baked in, not bolted on at the end
- Agent writes tests *alongside* code so you're reviewing behavior contracts, not just implementation
- Explicit "explain what you just built and why" prompts before moving to the next chunk — forces the agent to surface architectural decisions in plain language you can audit in 5 minutes rather than reverse-engineering from code

**The uncomfortable truth**

Right now you have two codebases where you're the only human who understands the intent but not the implementation. That's an unusual and somewhat precarious position — not catastrophic, but worth taking seriously. If either project has any complexity around security, data integrity, or production reliability, that's where I'd prioritize the review time, not the feature logic.

What are these projects? Greenfield tools, client work, production systems? That changes the risk calculus a lot.

# The legacy whirlpool

Yes — and this is the *actual* hard problem that almost nobody is talking about clearly yet.

Everyone's celebrating greenfield agent productivity. The ugly unsolved problem is the 15-year-old Rails monolith with 400k lines, no tests, tribal knowledge baked into variable names, and three people who "kind of know" how the payment module works.

**The greenfield vs legacy asymmetry is brutal**

On greenfield you control the context. You can design for agents from day one — small modules, explicit contracts, clear separation of concerns, documented intent. The agent operates in a well-lit room.

Legacy codebases are the opposite. Implicit dependencies everywhere. Business logic hiding in database triggers. Functions that do seven things named `processData`. An agent let loose in there isn't just ineffective — it's confidently wrong in ways that are hard to detect. It'll refactor something that looks dead but is called by a cron job nobody documented.

**The modernization problem has three layers you're essentially identifying:**

*Making the codebase agent-readable* — this is almost archaeological work. Surfacing implicit knowledge: why does this exist, what constraints drove this decision, what breaks if you touch it. READMEs, ADRs, inline comments aren't just documentation anymore, they're agent context. The codebase needs to explain itself.

*Carving out agent-safe zones* — not the whole codebase at once. Identifying the seams: what's genuinely isolated, what has clean inputs and outputs, what has test coverage already. Starting there. The strangler fig pattern but for agent-assisted modernization — let the agent work on the edges while humans hold the core.

*Designing the new pieces agent-first* — when you're adding to legacy, the new modules get designed differently. Explicit interfaces. Contract tests. No side effects smuggled through global state. Not because it's elegant but because it's the only way to give an agent a reliable work surface.

**The thing nobody has figured out yet**

There's no good playbook for "how do you brief an agent on a legacy system without either lying to it or exceeding its context window." You can feed it code but not the *history* of the code — the abandoned approaches, the constraints that no longer exist but left their shape behind, the reason that seemingly redundant check is actually load-bearing.

That institutional memory problem is genuinely unsolved. Some teams are experimenting with long-form "codebase diaries" — essentially a running document of decisions and their reasoning — but it's early and expensive to maintain.

What's your legacy situation — is this client work you're being asked to modernize, or your own accumulated technical debt?