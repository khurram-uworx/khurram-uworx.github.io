---
title: "The Story Point That Was Never About Time"
date: 2026-06-21
tags:
  - Story Points
  - Agile
  - Software Engineering
  - AI
  - Leadership
comments: true
---

I opened Jira.

The board alone looked like it required onboarding.

A Product Owner. A Scrum Master. Five engineers. Ten tickets. One clear mission:

"Before this meeting ends, we need estimates for everything we're starting on Monday."

The first ticket appeared.

Questions.
Assumptions.
Clarifications.
More questions.

Twenty minutes later we were finally discussing the estimate.

"Okay everyone, Poker."

Cards appeared.

3, 5, 8, 13, 13.

"Why 13?"
"Unknowns."

"Why 3?"
"Looks straightforward."

"Why 8?"
"Somewhere in the middle."

The debate continued.

Then the next ticket.
And the next.
And the next.

An hour later we had numbers. Whether we had certainty was another question.

---

This was my first backlog meeting at a modern software house.

I had spent the previous twenty years on a team of five engineers — the newest guy had already been with us for over ten years. We never played planning poker. We sat together, talked constantly, knew each other's systems almost as well as our own.

Estimation wasn't a ceremony.

It was:

"How long?"

"About a week."

"Make it two. Something always goes wrong."

"Fair."

Done.

Not because we were better engineers. Because after twenty years together, we already knew what card everyone was holding.

Those engineers in that first Jira meeting were trying to discover, in sixty minutes, what my previous team had learned about each other over two decades.

---

What struck me wasn't that the process was wrong.

It was that it was solving a problem my old team never had.

They didn't need story points. They had trust.

And that's the thing about story points that nobody says out loud: they are a substitute for shared context. When you don't know each other well enough to say "about a week" and have it mean something, you need a more structured language.

Story points became that language.

A way to say "I don't know" without saying it.

---

I bought the books.

They were surprisingly reasonable.

Story points, the books said, were not time. They were complexity. Uncertainty. Effort. Risk. A five-point story wasn't five hours or five days. It simply meant "more than a three, less than an eight."

Interesting.

Quite elegant, actually.

Then I went back to work.

"We have five engineers."

"Two-week sprint."

"We need around fifty points."

Wait.

Fifty points?

If points aren't time, why are we calculating capacity with them?

The next sprint came. Then another. Soon I could predict the conversations.

"We delivered 47 points."

"We usually deliver 52."

"We should commit to 50."

The books kept saying points were not time.

The spreadsheets disagreed.
The velocity chart disagreed.
The BI dashboard disagreed.
The budget forecasts disagreed with extraordinary confidence.

Everywhere I looked, story points had quietly become a unit of time while everyone continued insisting they were not.

A strange collective agreement.

Like calling a duck a complexity metric while measuring its speed, forecasting its migration patterns, and budgeting next year's ducks based on historical duck throughput.

Nobody was being dishonest. The people running releases wanted predictability. The people funding projects wanted forecasts. The people building dashboards wanted numbers.

Complexity does not fit neatly into charts.

Time does.

So complexity slowly became time, while retaining its original name.

---

## Two Ideas of What We're Doing

Here is the tension that story points were invented to manage:

| What the engineer is thinking | What the number person needs |
|---|---|
| This is uncertain | I need a date |
| It depends on things I don't know yet | I need to tell someone else a number |
| 13 points means "I'm uncomfortable" | 13 points means "two weeks" |
| The estimate is a guess with a distribution | The estimate is a commitment |

Story points were a buffer between these two realities. A linguistic fog. Thick enough that both sides could pretend they were talking about the same thing.

Engineers got to say "it's not a date."
Management got to build a forecast.
Both sides got what they needed.

It worked because the abstraction held.

Then AI showed up.

---

## AI Breaks the Hall of Mirrors

Before AI:

```
5 SP = one ticket, a week of work, some uncertainty
```

Now:

```
AI generates the code in one day
Same ticket
Same 5 SP
```

Does the story become 1 SP?

Usually no. Because the estimate was never purely coding effort. It also included waiting for answers, testing, review, deployment, integration risk, stakeholder feedback.

So many teams still call it 5 SP even though coding shrank from thirty hours to two.

But the team's velocity suddenly jumps. 50 SP per sprint becomes 80 SP per sprint. And now both sides are confused for different reasons.

**The engineer**: "The code writes itself, but I still don't know when it'll be done. The uncertainty didn't go anywhere — it just moved."

**The number person**: "If AI writes the code, why aren't we going faster? What are these points even measuring anymore?"

The old truce is broken. Both sides realize the abstraction they were leaning on just got a lot wobblier.

---

## The New Language

Management has always wanted a number that feels real. Lines of code was the first one — terrible, but measurable. Story points were the second — slightly better, but fuzzy.

Now there's a third:

Tokens.

Every AI interaction has a measurable cost. Input tokens. Output tokens. Reasoning tokens. API bills. Dollars.

For the first time, the input cost of software is directly visible to finance.

| Work Item | Human Era | AI Era |
|---|---|---|
| 1 story point | 1–4 hours | 10k–100k tokens |
| 3 story points | ~1 day | 50k–500k tokens |
| 8 story points | several days | 200k–2M tokens |

The question that now gets asked in boardrooms:

"Can $20 of AI replace $2,000 of engineering effort?"

That ratio is what drives adoption. And tokens are the new language for having that conversation — bypassing story points entirely.

The hierarchy becomes:

**Old:**
```
Engineer Hours → Story Points → Features
```

**New:**
```
Tokens → AI Cost → Features
```

Notice story points disappeared.

---

## But Tokens Are Not Value Either

This is where the next mistake happens.

Tokens are like electricity consumption. Ten million tokens could generate a billion-dollar insight or a useless document. Same as one hundred engineer hours could create a successful product or worthless code.

The actual chain is:

```
Tokens → AI Cost → Human Decisions → Features → Business Outcomes
```

Value only appears at the end.

And human decisions are still measured in the same squishy, uncertain units they always were.

---

## The Tension Didn't Go Anywhere

Here is what I keep coming back to.

The engineer/number-person tension wasn't created by story points. Story points were just the current vocabulary for having the argument.

Before that, it was lines of code.

Before that, it was man-months.

The argument itself is much older than any of these.

> Engineer: "I don't know exactly when this will be done."
> Manager: "I need a date."
> Engineer: "I can give you a range."
> Manager: "I need a date."

That conversation has been happening since the first person was paid to write software. Every generation invents a new abstraction layer to have the same conversation in slightly different words.

AI doesn't resolve this tension. It gives both sides new ammunition.

Management gets tokens — a measurable, finance-friendly unit. "We spent $47 of AI and got this feature. That's an objective fact."

Engineers get the same old uncertainty, now wrapped in a faster delivery system. "The code appeared in two hours, but I still spent three days reviewing it, finding the edge cases the model missed, and fixing the thing that broke staging."

The tension didn't resolve. It just found new ground to fight on.

---

## What Changes

One thing does change.

The bottleneck moves.

Traditional capacity planning:

```
5 engineers × 40 hours = 200 hours
```

AI era capacity:

```
5 engineers + AI agents + review capacity + deployment capacity
```

The old bottleneck was writing code. The new bottleneck is everything around it: understanding requirements, approving changes, testing, deployment, monitoring.

A single engineer may now supervise the output of several AI agents. Coding hours are abundant. Human attention is scarce.

| Task | AI Execution | Human Decision |
|---|---|---|
| Rename a button | tiny | tiny |
| Change a pricing model | tiny | huge |
| GDPR compliance change | medium | huge |
| Database migration | small | huge |

The expensive part is increasingly the decision, not the implementation.

Story points were never about typing. They captured complexity, risk, unknowns, coordination. AI compresses implementation. It does not compress ownership.

Someone still has to answer:

- What gets built?
- What gets delayed?
- What gets funded?
- What gets released?
- Who owns it when it breaks?

Those are management questions, not coding questions. AI doesn't answer them. It just makes the coding part faster, which means you have to answer them more often.

---

## The Future Abstraction

I expect story points to split into two dimensions:

```
Execution Cost + Decision Cost
```

| Item | AI Cost | Human Cost |
|---|---|---|
| Generate code | 2 minutes | 0 |
| Review code | 2 hours | 2 hours |
| Validate business rules | 4 hours | 4 hours |
| Release safely | 2 hours | 2 hours |

Total: eight hours of human attention.

Which is arguably what story points were approximating all along — just with less honesty about it.

---

## What I Stopped Arguing About

At some point in a career, many engineers become fascinated by process. Then architecture. Then strategy. Then organizational design.

And then, if they are lucky, they come full circle.

Back to code.
Back to solving problems.
Back to that quiet satisfaction of making a machine do exactly what you intended.

I stopped arguing about story points a while ago. Not because I agreed with how they were used. Because I realized the velocity charts would continue, the dashboards would continue, the quarterly metrics would continue.

They always had.
They always will.

Meanwhile, I have a compiler to satisfy. And unlike the dashboards, it is refreshingly honest.

But I do keep thinking about that first backlog meeting. The five engineers. The cards. The 3, 5, 8, 13, 13. The hour of debate.

They were all trying to answer the same question my old team answered in ten seconds:

"How sure are we?"

AI doesn't make that question easier to answer. It just changes what we're guessing about.

The tension between the people who build and the people who count was never going to be resolved by a better estimation framework. It is a permanent feature of organizations that build things with uncertain outcomes.

Story points were just the latest language for that conversation.

The next language is already arriving.

And it will have the same argument, just with different words.
