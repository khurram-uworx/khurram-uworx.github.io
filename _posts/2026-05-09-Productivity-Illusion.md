---
title: "The Productivity Illusion"
date: 2026-05-09
series:
  - "Frontier Engineering"
tags:
    - software engineering
    - coding agents
comments: true
---

There is a version of the AI-in-engineering story that is true. Agents write code faster than humans. Certain categories of work that took days now take hours. Some teams are shipping more with fewer people. That part is real.

# The Agentic Engineering Team: What the Productivity Numbers Aren't Telling You
## Part 1 of 2 — The Productivity Illusion

What is not real is the conclusion being drawn from it — that we have found a productivity multiplier that applies broadly, scales cleanly, and can be managed with the same frameworks we have always used. We have not. What we have found is a tool that is genuinely powerful in a narrow band of conditions, and genuinely dangerous when those conditions are assumed rather than verified.

Engineering leaders who do not make this distinction clearly, and soon, will pay for it later. Quietly, slowly, and expensively.

---

## The Measurement Problem

Most productivity claims in this space share a common flaw: they measure output, not outcome.

Lines of code produced. Tickets closed. Features shipped per sprint. These are the numbers going up, and they are the numbers being cited in blog posts, board decks, and vendor case studies. They are also, largely, the wrong numbers.

Output metrics were always a rough proxy for value. In the agentic era they become actively misleading. Agents are extraordinarily good at producing output. They are not inherently good at producing the right output, the maintainable output, or the output that will still make sense to your team in eighteen months. The surface signal of productivity — volume, velocity, visible activity — has been decoupled from the underlying signal of engineering quality. Measuring one and calling it the other is a category error with consequences.

The numbers that matter — incident rates, time to diagnose failures, onboarding cost as the team turns over, architectural coherence over time — are slower to appear and harder to attribute. By the time they surface, the causal link to today's decisions will be obscure. That is not an accident of measurement. It is the nature of technical debt, which is what a significant portion of this "productivity gain" actually is.

---

## The Friction That Was Doing Hidden Work

When implementing a feature took two weeks, the cost of that two weeks was a forcing function. Teams thought carefully about whether the feature was worth building. They designed deliberately because rework was expensive. They prioritized ruthlessly because the queue was long and the queue was real.

That friction was not waste. It was embedded judgment. The slowness was the organization asking "are we sure?" without needing to schedule a meeting to do it.

Remove the friction and you remove the question.

When something feels cheap, the psychological relationship to it changes. Engineers stop thinking like architects and start thinking like orderers. The cognitive mode shifts from "what should exist in this system" to "what can I get done today." These are not the same question. Historically the cost of building enforced the former. Reduce the cost far enough and you reliably get the latter.

This is not speculation about human nature. It is a well-documented dynamic across engineering, design, and product contexts. Constraints produce better outcomes than unconstrained freedom, not because struggle is virtuous but because constraints force prioritization and prioritization forces clarity. The clarity was the valuable thing. The constraint was the mechanism that produced it.

Cheaper execution has not made teams better at deciding what to execute. In many cases it has made them worse, because the natural pressure to decide carefully has been removed.

---

## The Greenfield Bias

The productivity numbers that circulate are almost exclusively drawn from the easiest conditions: greenfield projects, well-understood problem domains, standard architectures, clean requirements, modern stacks.

This is where agents genuinely shine. These conditions closely resemble the publicly available code on which agents were trained. Standard REST APIs, typical authentication flows, conventional data models — agents have pattern-matched against millions of examples of these. The output is fast, clean, and often correct. The 10x claim is not entirely wrong in this context.

The problem is the extrapolation. Greenfield on a standard stack is not representative of most engineering work in most organizations. It is the easiest slice of the problem space. Claiming broad productivity gains from performance in that slice is like claiming a race car is a practical commuter vehicle because it performed well on a track.

The curve drops sharply as conditions diverge from the ideal. Complex existing systems, novel problem domains, ambiguous requirements, non-standard architectures — in each of these the gains shrink and the risks grow. Nobody is publishing those benchmarks. The vendors are not running those case studies. The leaders making decisions based on the published numbers are, in many cases, making decisions based on a systematically biased sample.

---

## Rushing What Needed Time

There is a category of engineering problem that genuinely requires time. Not because engineers are slow, but because understanding the problem takes time, and building the wrong thing quickly is worse than building the right thing slowly.

Hard problems require sustained engagement with ambiguity. The temptation to resolve that ambiguity prematurely — to converge on a solution before the problem is fully understood — is always present. Good engineering discipline resists it. The cost of implementation has historically reinforced that resistance: if building the thing is expensive, you had better be sure you understand the thing.

Cheap implementation removes that reinforcement. Teams now jump to execution before the problem is understood, because execution is no longer the scarce resource. But the problem understanding was the actual scarce resource. It still is. Making execution cheap did not make understanding cheaper. It just made it easier to skip.

The result is a proliferation of half-right things. Solutions that work in some cases and fail in others. Features built before the use case was validated. Architectural decisions made at speed that will constrain the system for years. None of this shows up in the sprint velocity. All of it shows up eventually.

---

## What to Actually Measure

If you are leading an engineering organization through this transition, the question is not "how much faster are we shipping." The question is:

- Are our incident rates stable or rising as agent-generated code reaches production?
- Can engineers who did not write a piece of code explain why it works the way it does?
- Is our time-to-diagnose for production failures increasing?
- Are we making more architectural decisions, or fewer, than before — and are those decisions being made deliberately?
- What is our rework rate on agent-generated output?

These are harder to track. They are also the numbers that reflect whether the productivity gain is real or whether it is being borrowed from future stability.

The wand is real. It does something. But a wand pointed without judgment does not produce magic. It produces volume. Volume and value are not the same thing, and the distance between them is where the reckoning lives.

---

*Part 2 examines the review bottleneck, the codebase scale cliff, and why the most important software in most large organizations is precisely the software these agents are least equipped to touch.*

{% if page.series %}
{% for s in page.series %}
{% assign series_posts = site.posts | where_exp: "post", "post.series contains s" | sort: 'date' %}
<div class="series-nav">
<h4>More from {{ s }}</h4>
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
{% endfor %}
{% endif %}
