---
title: "The Structural Problems"
date: 2026-05-10
series: "Frontier Engineering"
tags:
    - software engineering
    - coding agents

comments: true
---

Part 1 addressed the productivity illusion — the measurement bias, the hidden value of friction, the greenfield conditions that make the numbers look good. This part addresses the structural problems that emerge at scale: the review bottleneck that agents make dramatically worse, the hard limits of agent capability on real codebases, and the vast category of enterprise software that the productivity conversation has ignored almost entirely.

# The Agentic Engineering Team: The Problems That Don't Show Up in the Demo
## Part 2 of 2 — The Structural Problems

These are not edge cases. For most large engineering organizations, they are the center of the problem.

---

## The Review Bottleneck

Human review was already a constraint before agents. It is now a crisis in the making.

The math is simple and the implications are not being taken seriously. Before agents, review was roughly proportional to output — one engineer writes code, one reviews it, the ratio holds across the team. Agents break that ratio. One engineer prompts and receives ten times the output. Review capacity did not scale. The same human, the same cognitive bandwidth, the same hours.

Something has to give. Either you review everything and become the bottleneck, negating the velocity gain. Or you review less and accept that trust degrades, bugs accumulate, and security exposure widens. Or you review differently — which is the right answer, and one that the industry has not yet worked out at the level the problem demands.

What makes this harder than the pre-agent review problem is the nature of agent-generated code. It looks correct. It is well-formatted, passes linters, includes documentation, follows conventions. The surface signals of quality are present. The failures hide deeper: in logic, in edge case handling, in subtle misreadings of the specification. A reviewer scanning for obvious problems will miss them. The review mode that worked for human-authored code does not transfer cleanly.

There is also a volume effect on judgment. Reviewing five hundred lines of clean agent output is cognitively different from reviewing five hundred lines of a colleague's code. With a colleague you have context — you can ask why they made a choice, you understand their tendencies, you have a shared history with the system. With agent output you are often reverse-engineering intent from result, at volume, under time pressure. The judgment degrades in ways that are hard to detect and easy to rationalize.

The proposed solutions are real but limited. AI reviewing AI is promising and creates a genuine false sense of security — reviewer models share blind spots with generator models in ways that are not fully understood. Better test coverage helps until you encounter a failure mode you did not think to test for. Sampling-based review works until the thing you did not sample is the critical security flaw. Formal verification is genuinely useful in narrow domains and does not scale to general application code.

The harder truth is that review was never only bug-catching. It was knowledge transfer. It was shared ownership. It was the mechanism by which the team maintained a distributed understanding of why the system is the way it is. When review becomes a throughput problem to be optimized, that function disappears. Months later nobody knows. The codebase becomes an artifact that everyone uses and nobody fully understands. That is not a codebase. It is a liability.

---

## The Codebase Scale Cliff

Agent capability does not degrade linearly with codebase size. It falls off a cliff.

Under ten thousand lines of code, on a clean architecture, agents are genuinely impressive. The context is manageable, the patterns are recognizable, the scope of any change is bounded. This is where the demos live.

At twenty to thirty thousand lines the coherence problems emerge. The agent is making locally correct changes that are globally inconsistent. It is introducing duplication it cannot see because the duplicate is outside its effective context. It is making assumptions about how components interact that are plausible but wrong.

At the scale of real enterprise systems — hundreds of thousands of lines, multiple services, years of accumulated decisions — the agent is effectively working blind across significant portions of the system. It will produce confident output. That confidence is not grounded in actual comprehension of the system. It is pattern completion at a scope the model cannot actually hold.

This is not primarily a context window problem, though that is part of it. Retrieval and comprehension are different capabilities. Presenting a model with one hundred thousand tokens of code and expecting coherent reasoning across all of it is not the same as a senior engineer who has worked in that system for two years. The engineer has a mental model. The agent has tokens. They are not equivalent.

Existing codebases compound this in specific ways. Greenfield agents get to make all the assumptions. Existing systems have decisions made years ago for reasons nobody remembers, workarounds for bugs in dependencies that no longer exist, implicit contracts between components that are nowhere documented, and business logic so organizationally specific that it was never worth abstracting. An agent in that environment does not know what it does not know. It will fix the workaround without understanding why it existed. It will refactor the thing that looks messy but is messy for a reason. It will do this confidently and the review process, if it is operating at the throughput levels agents make tempting, will not catch it.

---

## The Enterprise Dark Matter Problem

This is the issue that the productivity conversation has almost entirely avoided, and it is arguably the most important one for large organizations.

Everything agents learned from was public. Open source repositories, Stack Overflow, technical documentation, blog posts. That sounds comprehensive. It is not. What lives in public repositories is not representative of what runs the world.

The systems that process payroll, clear financial transactions, route insurance claims, manage hospital records, schedule infrastructure, and execute the operational core of most large organizations — almost none of that code has ever been public. It was built internally, for internal purposes, under conditions of confidentiality. It reflects decades of organizational-specific decisions, regulatory constraints, domain modeling that is unique to that organization, and technical choices made around hardware and software environments that no longer exist.

Agents were not trained on this code. They have no pattern recognition for these systems. They do not know the conventions, the naming schemes, the internal frameworks built before external frameworks existed, the architectural patterns invented to solve problems that no public project had faced in quite the same way.

When an agent encounters this code it is not operating with reduced capability. It is operating without the foundational recognition that makes it useful in the greenfield context. It sees tokens. It does not see a system.

The undocumented institutional knowledge problem is equally severe. In mature enterprise environments the documentation is the people. The engineer who has been there for twenty years and knows why the reconciliation process runs at three in the morning specifically. The team that inherited a system from a vendor that went bankrupt and reverse-engineered half of it over five years. The architect whose comments in a 2009 commit are the only record of a decision that still shapes the system today. The oral tradition of warnings that exist only in the memory of people who have been there long enough to have been burned.

Agents have none of this. And unlike greenfield development, the cost of getting it wrong in these environments is not a bug ticket. It is regulatory exposure, financial error, operational failure, or data integrity damage at a scale that reflects the criticality of the systems involved.

---

## The Seniority Paradox

One structural consequence that is not being discussed enough: the engineers best equipped to navigate this transition are senior engineers who already have the judgment that agents lack. They will apply that judgment even when the tooling does not demand it. They will recognize the workaround that should not be touched, the architectural decision that deserves more time, the agent output that is locally correct and globally wrong.

Junior engineers do not have this yet. They are developing it, through the same slow accumulation of experience and consequence that every generation of engineers has gone through. That process depends on a feedback loop: make a decision, observe the outcome, update your model of the system and of good engineering.

Agents disrupt that loop. When code is generated rather than written, the junior engineer is not building an understanding of why it works. When review is optimized for throughput, they are not building an understanding of what good looks like. When friction is removed from execution, they are not building the judgment that friction was developing.

The result is a generation of engineers who are fast and productive on paper and underdeveloped in ways that will not be visible until the moment it matters most — when the agent fails, or when the system fails, and someone needs to understand what is actually happening well enough to fix it.

---

## What Engineering Leaders Should Actually Do

None of this argues against using agents. It argues against using them without honesty about what they are and where they work.

**Set the context boundary explicitly.** Agents are tools for well-understood work in well-understood systems. They are not tools for your thirty-year-old core platform, your undocumented integration layer, or any system where the cost of a subtle error is significant. Draw that line clearly and defend it when the pressure to expand it comes, because the pressure will come.

**Protect the review function.** Not as a throughput gate but as a knowledge transfer mechanism. The team's distributed understanding of the system is not a nice-to-have. It is structural resilience. Optimize it away in the name of velocity and you will pay for it in fragility.

**Invest upstream, not just downstream.** The leverage in an agentic workflow is in the specification, the constraint-setting, the architectural guardrails — the things defined before the agent runs. This is where human judgment concentrates now. It requires more rigor than most teams currently apply to it, not less.

**Measure what degrades slowly.** Incident rates, diagnosis time, architectural coherence, rework rates. These are the numbers that will tell you whether the productivity gain is real. They are also the numbers that will surface the problem early enough to do something about it, if you are tracking them.

**Be honest about the dark matter in your organization.** Know which of your systems falls outside the conditions where agents are useful. Know where the institutional knowledge lives and who holds it. Treat that knowledge as the organizational asset it is, not as a problem to be automated away.

The organizations that navigate this well will be the ones that used agents deliberately — as force multipliers on the work where they genuinely help — while maintaining the human capability and judgment required for the work where they do not. That is a harder management problem than "adopt agents and measure velocity." It is also the actual problem.

The ones that do not will discover, slowly and expensively, that they traded long-term resilience for short-term output metrics. And they will make that discovery at the worst possible time — when something critical breaks and nobody in the organization understands it well enough to fix it.

{% if page.series %}
  {% assign series_posts = site.posts | where: "series", page.series | sort: 'date' %}
  <div class="series-nav">
    <h3>More from {{ page.series }}</h3>
    <ul>
      {% for post in series_posts %}
        <li>
          {% if post.url == page.url %}
            <strong>{{ post.title }} (Current)</strong>
          {% else %}
            <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
          {% endif %}
        </li>
      {% endfor %}
    </ul>
  </div>
{% endif %}
