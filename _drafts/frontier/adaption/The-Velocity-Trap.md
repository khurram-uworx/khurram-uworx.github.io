---
title: "The Velocity Trap"
date: 2026-07-16
series:
  - "Frontier Engineering"
tags:
  - AI
  - Software Engineering
  - Leadership
comments: true
---

The velocity graph looks great. It always looks great.

The team is shipping faster than anyone expected. Features that used to take a week take two days. The sprint board moves. The burndown is clean. Stakeholders are happy. The dashboard everyone checks first thing in the morning tells the story everyone wants to hear: we are faster.

Then a PR comes in.

Bug fix. The diff is 200 lines. It touches five files. The original bug was three lines of broken logic in a single function.

No one questions it. The velocity graph still looks great.

---

## The Relentless Fixer

Here is what coding agents do: they fix the thing you ask them to fix. They do it fast. They do it with total commitment. The code appears. It is well-formatted. It passes linters. Tests pass.

Here is what they do not do: they do not think about what else might break.

A human engineer making the same fix pauses — even if only for a moment. *If I touch this function, does the payment module care? Does the retry logic downstream assume this return type? What else in the system depends on this being the way it is?*

That pause is slow. It is also the thing that keeps systems from collapsing three sprints later.

The agent does not skip that thinking out of laziness. It was never capable of it. It optimizes for the asked thing. The ripples are not in its vocabulary. The future consequence of a present change — the fragility it introduces at B while fixing A — is invisible to it. Not hidden. *Absent.*

And we have decided that is fine. Because it is fast.

---

## The Cascade

This is where velocity becomes a trap — not just a speed, but a self-reinforcing cycle that degrades everything around it.

A bug is found. A ticket comes back. The agent writes the fix. But the fix is not a clean three-line change. The agent touched the original module, a helper function, a config file, and — while it was there — refactored a related utility that looked suboptimal. The diff is 200 lines for a problem that should have been four.

The reviewer sees the sprawl. They are already behind — the sprint is packed with three times the features from two years ago, and the review queue does not care about your velocity graph. They scan the diff. It looks clean. The agent's code always looks clean. They approve.

The tester runs the regression suite. It passes. But the regression suite was written for the old code. It does not cover the new patterns, the new edge cases, the new interactions introduced by the agent's changes across five files instead of one. The ticket is closed.

Next sprint, something else breaks. Not in the original module. In the helper function the agent touched. The one no one asked it to touch.

The agent fixes that too. The diff is bigger this time. The reviewer is more tired. The tester is more behind. The regression suite is the same. Ship it.

Each pass through the cycle makes the next one worse. The diffs grow. The review pressure grows. The testing gap grows. The velocity graph stays green.

**The trap is not that we are moving too fast. It is that going fast makes every subsequent cycle harder to slow down — and the gates that should catch the problems are the first things to erode under the pressure.**

---

## The 3x Perception

There is a deeper problem, and it lives in the way humans process change.

When you used to ship one feature a week and now you ship three, your brain recalibrates. Three becomes normal. The quality that existed at one-per-week — the thinking, the questioning, the moment where you stop and ask *wait, should we even be building it this way?* — that quality does not scale with velocity. But your perception of *normal* does.

Expectations rise. The sprint that shipped three features becomes the baseline for the next sprint. The stakeholder who saw the velocity graph adjusts their timeline. The manager who reported the numbers up now has to maintain them.

And the quality? The quality is still there — in the metrics no one is tracking. In the bugs that escape and get filed as new tickets. In the rework that gets absorbed into "feature work." In the review depth that shrank from careful scrutiny to a glance at a clean diff.

The friction that used to force careful thinking has been removed. Not because anyone decided to remove it. Because the tool made it feel optional.

You need resistance to walk. Your feet push against the ground, and the ground pushes back, and that mutual resistance is what moves you forward with control. Remove the friction from your feet and you do not glide. You fall.

The velocity trap's quiet genius is this: it removes the resistance that would make you notice the trap.

---

## Metrics Were Important. Now They're Existential.

Before AI, metrics were important but forgiving. The pace of change was human. You had time to notice problems. The feedback loops were slow enough to catch things in a retro, in a conversation, in the quiet moment after a deploy where someone says *that felt wrong* and the team pauses to look.

Now the feedback loops run at machine speed. Code is generated in minutes. Fixes touch five files instead of one. The cascade runs faster than any human can review, test, or reason about.

If you are not measuring quality in real time, you are not measuring it at all. By the time you notice the problem in a retrospective, three cascades have passed and the gates are already degraded.

The measurement framework is not a nice-to-have dashboard. It is the early warning system. Not the productivity metrics — everyone tracks those. The ones that cost something to track:

- **Diff size per fix.** Is the average fix getting bigger? That is the cascade made visible.
- **Review depth over time.** Are reviews getting shorter as output increases? That is the pressure curve.
- **Testing coverage relative to change.** Not overall coverage — coverage *per diff*. If code grows 40% and test coverage is flat, the gap is where bugs live.
- **Rework rate on agent-generated output.** If AI-written code is coming back more often than human-written code, the tool is not ready for this system.
- **Time to diagnose production failures.** If it is increasing, the team is losing its mental model of the system.

These metrics existed before. They were best practice. A responsible engineering organization tracked them and felt good about having them on a dashboard somewhere.

They are no longer best practice. They are the difference between catching the problem and discovering it six months later when something critical breaks and nobody can explain why.

---

## The Capacity Problem

Here is the catch, and it is cruel.

The raised expectations from velocity have consumed the capacity that measurement requires. The sprint is packed with three times the features. The review queue is longer. The testing backlog grows. Where does the time for deeper review, exploratory testing, root cause analysis come from?

It does not come from anywhere. Unless you make room for it. Which means deliberately slowing down. Which feels like falling behind. Which is the trap completing itself.

The velocity trap's final move is this: it removes the capacity to detect itself.

The team that most needs to track quality metrics is the team least able to afford the time to do it. The cascade demands more attention with every cycle. The gates demand more rigor with every PR. And the sprint demands that you ship, because the velocity graph says you should be shipping, and the expectations it created will not adjust themselves downward.

This is not a resource problem. It is a priorities problem. And the priority that is being sacrificed — every sprint, without anyone deciding to sacrifice it — is the one that keeps the system from eating itself.

---

## The Ruthless Root Cause

When things slip — and they will — you need a process that does not flinch.

Not a retro where everyone nods and moves on. Not a post-mortem that concludes "we should have caught this." A ruthless, specific, uncomfortable root cause analysis that follows the chain all the way back:

**What broke?** Not the symptom. The actual failure.

**Where did the fix come from?** Human or AI. If AI, what did it touch?

**What did the fix touch?** Not just the target module. Everything the diff touched. The sprawl. The files no one asked for.

**Was the spread reviewed with depth?** Not "was it approved." Was it actually understood by the person who approved it?

**Was the spread tested beyond regression?** Did anyone test the new patterns, the new edge cases, the new interactions introduced across the sprawl?

**Is this a pattern or an incident?** One bad fix is an incident. Three in a sprint is a pattern. A pattern is the cascade. The cascade will not stop on its own.

And then the hardest question — the one that most teams are not prepared to ask:

**If AI is the root cause, what do you do about it?**

Three options, escalating in seriousness:

**Control it.** Constrain AI to specific, well-understood domains. Surgical use only. No "fix everything while you're in there." The diff stays small. The sprawl stays contained. The review stays meaningful.

**Manage it.** Mandatory human-in-the-loop for any AI-generated change. Slower. Deliberately slower. But it preserves the gates. It keeps review as review, not as approval theater.

**Stop using it.** For this system, in this context, the tool is causing more harm than it prevents. The codebase is too complex, the institutional knowledge too deep, the cost of subtle failure too high. Not forever. Not everywhere. Here. Now.

The willingness to say *not here, not now* is the ultimate quality gate. If you cannot say it — if the velocity graph has too much momentum, if the expectations are too high, if the organization has committed too publicly to the AI narrative — then you have already lost the ability to manage the risk. The tool is managing you.

---

The velocity graph still looks great.

But there is a second thing now — a root cause log, short and specific. Most entries say "insufficient review depth under deadline pressure." One says: "AI-generated fix introduced regression in payment module. Root cause: agent touched unrelated code. Diff was 200 lines for a 3-line bug. Action: restricted AI use in payment domain."

The log is not a comfortable document. It is the record of every moment the team chose velocity over understanding, and the cost that followed.

The velocity trap closes when you stop looking. It opens when you choose to — even when what you find means slowing down, tightening process, or admitting the tool you adopted is not ready for the system you are running it on.

The graph will still go up. The question is whether anyone in the room can say what the graph does not show: **whether the system is getting stronger, or just faster — and whether those are still the same thing.**

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
