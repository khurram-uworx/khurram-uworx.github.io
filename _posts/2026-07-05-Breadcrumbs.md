---
title: "Breadcrumbs"
date: 2026-07-05
tags:
  - Traceability
  - Software Engineering
  - Brownfield
  - Mentoring
  - Leadership
comments: true
---

I asked ChatGPT a stupid question a few months ago.

"There's a story," I typed, "where children leave a trail of something behind them so they can find their way back. What's it called?"

Hansel and Gretel, it said.

Of course.

I'd read it as a child. Forgotten even the name. But the image stayed — breadcrumbs as a trail, as a way back — sitting somewhere in the back of my mind.

I think that's why I always enjoyed the ritual of it. Meaningful commit messages. Branches that tell a story. PRs linked to tickets. I wasn't being pedantic. I was seeing breadcrumbs in the process before I had the word for it.

---

I spent most of my career in greenfield. Building things for the first time, sitting with the same small team for years. You live in the codebase together — learn it, discuss it, apply it, revisit it. You know each other's systems because you were there when they were born.

If you wanted to know why something existed, you asked the person who wrote it. They were still in the building.

Traceability wasn't a concept. It was just Tuesday.

---

Then I joined a modern software house. A customer had acquired a product. The founders were still there, consulting — but they'd spent years fielding the same shallow questions from outsourced engineers, so they'd stopped volunteering *why*. The knowledge was in the room, but the bridge was broken.

I stared at hundreds of files and realized the only way forward was backward.

I needed breadcrumbs.

I found a trick. Pick the area you care about. Pull the recent tickets. Find their PRs. Read the diffs — not just the code, but the conversation. The rejected alternatives. The edge case someone discovered at 4 PM on a Friday. Trace forward: ticket → PR → diff → deploy. Trace backward: line of code → commit → PR → ticket → requirement.

Patterns emerge. You stop seeing individual changes and start seeing a system's history. You understand not just *what* things are, but *why*.

This is traceability. Not a process mandate. Archaeology. Survival.

Follow the trail far enough — ticket, PR, commit, intent — and you arrive at a consultant with a question that isn't *what* but *why*. They stop being bored. The breadcrumbs didn't just answer the question. They rebuilt the bridge.

The breadcrumbs were always there. I just had to learn to follow them.

---

I am training fresh engineers now.

They arrive with sharp minds and fast hands. They write good code — because the tools write it with them. Correct, clean, well-structured, generated in seconds. A prompt away.

The skill we spent years building — shaping clean code by hand — is no longer the bottleneck. The gate is lower. That's progress. But it means the hard thing shifted.

What they didn't learn in university is how to *understand* a codebase. Not just write code that works — but look at a system and ask: what does this do? Why does it do it this way? Will I still understand this in six months?

Their entire education trained them for assignments that last two weeks. A final year project that spans a semester. Code that gets written, submitted, graded, and forgotten. No one taught them that *this* codebase will outlive their tenure — that six months from now, someone else will need to understand not just *what* was written, but *why*. And that understanding doesn't come from the code alone. It comes from the trail left behind it.

This is the part I want you — the engineer who's been doing this for five, eight, ten years — to sit with.

You've seen the other side. You've spent three hours tracing a regression back to a commit whose message says nothing. You've inherited a module where no one remembers why a design choice was made.

The junior isn't lazy. They're operating with the skills they were taught — produce correct code on a short deadline. The skill they need is the one no one taught them: establishing understanding of a codebase they didn't build and won't abandon.

So here's what I've found works. Next time you review their work, don't start with the checklist. Don't ask for a ticket link. Walk them through a different exercise. Pull a file from a module that's been through a dozen engineers. Pick a line and trace it backward — commit to PR to ticket to the person who wrote it. Let them see the trail when it's clear, and let them feel what happens when it goes cold.

That's the lesson. Not the rule. The feeling of a trail gone cold stays with them longer than any PR template.

Your job isn't to enforce the process. It's to make them understand why the process exists.

The ticket isn't bureaucracy. It's a breadcrumb.

We spent twenty years building codebases that we could hold in our heads. But that world is gone.

Today's codebases are built by dozens, hundreds of people over years, decades. No single person holds the map. The map *is* the trail of tickets, PRs, commit messages, and review discussions that trace back through every decision ever made.

This is traceability. The honest answer to the question every engineer in a modern codebase eventually asks:

> Why does this exist?

---

I thought I understood what that question meant. Then I watched an agent scaffold an entire module in an afternoon.

We don't write infrastructure code anymore. A few sentences of intent, and the agent generates thousands of lines — well-structured, working, deployable. The first day of a new project, you already have code you didn't write, conventions you didn't choose, decisions the model won't remember tomorrow.

Day one greenfield. Day one brownfield.

And if that code arrives without a trail — no linked requirements, no commit message explaining *why* — you've just accelerated the creation of untraceable brownfield. More code, faster, with less context attached. That's not progress. It's a bigger forest to get lost in.

Here's the part that still stops me. The coding agents I use now — before they write code, they scan the git history. They trace commits → diffs → patterns → suggestions. The same chain I was following through tickets and PRs, the agent follows through git log and blame.

That's traceability. Running as inference.

If the breadcrumbs are missing — commits that say nothing, PRs unlinked from tickets — the agent generates code that doesn't fit. Wrong patterns. Wrong assumptions. It fails the same way a human does: because it couldn't answer what the trail was supposed to answer.

> Why does this exist?

And that's the thing. This trick — trace tickets, read PRs, learn patterns — it was never just for brownfield survival. It's how any intelligent system makes sense of a codebase it didn't build. Human or machine.

---

The same chain — requirement → design → code → test → deploy → log → review — is what ISO 27001 formalizes as controls. I recognized it the first time I read a clause. Not because I'd studied compliance. Because I'd already been tracing that same chain through tickets and PRs every day.

ISO doesn't demand traceability because of compliance. It demands it because, at scale, breadcrumbs are the only reliable thing. When a system must survive auditors, regulators, and engineers who come and go over a decade, the question is never "can we prove this was reviewed?" It's always "can the next person understand why this exists?"

The controls only feel like overhead when traceability hasn't been internalized. When a team gets it — really gets it, not just follows the checklist — the same controls feel obvious. Not easier. Just obvious. Because everyone already knows why they exist. The ticket link isn't a burden. It's how you find your way back.

Coding agents are writing code faster than we can leave trails for it. The forest isn't just dense anymore — it's growing while we sleep. Every prompt generates a dozen files. Every session spins up conventions no human chose. Day one greenfield is day one brownfield, and the compounding is faster than any human team can document.

There's no tool coming to save us from this. No agent that will retroactively trace what another agent generated without context. The only thing that scales is the same thing that's always worked: the discipline of leaving breadcrumbs while you're still at the table.

This is where traceability stops being a practice and becomes a survival instinct.

Be pragmatic. Not every line needs a ticket — but every meaningful decision does. The agent generated a thousand lines overnight? It also needs to link them back to intent.

Be agile. Trace forward, not backward. Don't write the essay after the fact. Link the context while it's fresh, while the decision is still warm. The best breadcrumb is the one you drop while you're still walking the path.

Be vigilant. When an agent generates code, read it. Then ask the agent to leave the same trail you would. Watch where AI strengthens the breadcrumb and where it kicks dust over it. The agent can write the commit message. The agent can link the ticket. The agent can explain why. Make it.

Be structural. Put the design context in the repo. ADRs, requirements, architecture decisions — not in a wiki, not in a shared drive. In the repo, next to the code. That's how the agent finds them — through the same git log and blame it already reads. But we're still learning the patterns: how to link them, tag them, weave them into the commit trail so they surface when the agent traces history. A breadcrumb is only useful if it's on the path.

The process works when the intent is clear. Human. Machine. Both.

You don't do it for the auditor.

You don't do it for the process.

You do it because someone will come after you, lost in the forest of a codebase they didn't build.

And they'll be looking for breadcrumbs.
