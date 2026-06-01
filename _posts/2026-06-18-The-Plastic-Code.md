---
title: "The Plastic Code"
date: 2026-06-18
tags:
  - AI
  - Software Engineering
  - Leadership
comments: true
---

It starts quietly.

You join a project mid-flight. Lots of code — more than you'd expect for the team's size. The AI multiplier is real, and it's working. Features ship weekly. Velocity is up. Everyone is busy.

Then you attend your first standup.

Everyone reports progress. Cards move across the board. Nothing is blocked. The burndown looks healthy. But something is wrong.

Someone says: "The PR for the payment module is up."

Someone else says: "I'll review it."

Pause.

"Actually, do you know what that module does?"

"No. It was generated last sprint."

"By who?"

"It was in the backlog. AI wrote it. It works. Tests pass."

Another pause.

No one in that room — not a single person — could explain how half the codebase worked.

---

## The Plastic Code

This is not technical debt.

Technical debt assumes someone once understood the code. Someone took a shortcut, made a tradeoff, incurred a cost they planned to repay. There was a human moment of *choice*, even if it was a bad one.

This is different.

This is code that skipped the human phase entirely.

> Generation → repository → production → archaeology

No author. No design discussion. No institutional memory. No one who can say "I chose this approach because..."

It's code that *works* — tests pass, features ship, customers are happy — but was never known by anyone.

I call it **plastic code**.

Like plastic, it's manufactured, not crafted. Durable but culturally dead. It will outlast its creators. It will still be in the repo years later. But it carries no story, no rationale, no human history.

Code that works, but was never known.

---

## The Rockstar Trap

Here's what makes this uncomfortable.

Almost everyone is *excited*.

Engineers feel 10x. Problem solvers who used to ship one feature a week now ship three. Every sprint feels like a victory lap. Promotions are faster. Impact is higher.

On every visible metric, this is a win.

The invisible loss is the hardest one to name — because struggle is how senior engineers are made.

Deep understanding doesn't come from shipping fast. It comes from the moments when you *can't* ship — when you're stuck, when you have to trace through unfamiliar code, when you have to understand a system well enough to change it safely.

Those moments are disappearing.

And everyone is too busy shipping to notice.

Who will be the senior engineers in five years? Not the ones who reviewed AI-generated PRs quickly. The ones who learned by fire. But what happens when there's no more fire — just an endless conveyor belt of working code?

> **The hardest question about the next generation of senior engineers is that no one is asking it yet.**

---

## A Cultural Failure, Not a Technical One

The failure is invisible because it's not in the code.

It's in the space *around* the code.

- The design conversations that never happened
- The tradeoffs no one debated
- The architectural decisions no one remembers making
- The tribal knowledge that never formed

Engineering has always been a social discipline. The code is the artifact, but the *understanding* lives in conversations, arguments, whiteboard sessions, and late-night debugging sessions where someone finally says "oh, that's why it does that."

Those conversations are evaporating. Not because anyone decided they should. Because when code appears fully formed from a prompt, there's nothing to argue about. No reason to gather around a whiteboard. No moment where two engineers disagree and, in the process of resolving it, both understand the system better.

And when stakeholders and customers see the velocity, they adjust expectations. Faster becomes the baseline. The pressure to maintain speed becomes the new normal. No one asks whether the speed comes at a hidden cost, because the cost isn't visible on any dashboard.

---

## The Night It Stops Working

December 23rd. A retailer's checkout pipeline goes dark. Peak season. Last shipping day before Christmas. Orders that don't go through now don't arrive in time. The revenue loss per minute has a dollar figure on a dashboard somewhere. No one looks at it.

The Slack channel lights up.

The team assembles. Someone opens the offending module.

No one in the room wrote it. No one argued about its design. No one remembers why this approach was chosen over another. The system is a black box that was assembled, not built. You can extend it. You can add features to it. You can run tests against it. But you cannot *reason* about it, because no one in the room has the mental model required.

The VP of Engineering joins the call.

"I need a timeline."

Silence.

The fix comes, eventually. Another prompt. Another patch. It holds. The pipeline recovers. Some orders are lost. Some customers won't come back.

Because the code was *never known* by anyone in the first place.

---

The productivity multiplier is real. The excitement is justified. The ships are shipping.

But here's the question that will haunt the quiet moments after the next outage — and there will be a next one:

**What happens when the plastic world breaks — and no one left knows how it was built?**
