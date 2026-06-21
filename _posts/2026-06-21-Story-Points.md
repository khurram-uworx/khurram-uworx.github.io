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

## What Story Points Really Were

Here is the tension they were invented to manage:

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

But over time, story points became something more specific. They became a currency.

```
5 SP × team velocity = ~1 week of effort
1 week of effort = ~$X of salary
$X of effort → a feature
```

Every team I watched eventually collapsed the abstraction. Five points cost about this much. A sprint cost about that much. A feature's story points mapped to a dollar figure on a budget line.

The books said story points weren't time.

The finance department knew better.

Because when you are planning a quarter, forecasting a release, or justifying headcount, you are not counting complexity. You are counting money. And story points were the unit you did it in.

It worked because the abstraction held.

Then AI showed up.

---

## The Hall of Mirrors Breaks

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

Does the story become 1 SP? Usually no. The estimate was never purely coding effort. It also included waiting for answers, testing, review, deployment, integration risk, stakeholder feedback.

So many teams still call it 5 SP even though coding shrank from thirty hours to two.

But the team's velocity jumps. 50 SP per sprint becomes 80 SP per sprint.

The engineer: "The code writes itself, but I still don't know when it'll be done."

The number person: "If AI writes the code, why aren't we going faster? What are these points even measuring?"

The old truce is broken. Both sides realize the abstraction they were leaning on just got a lot wobblier.

---

## The New Language Arrives

Management has always wanted a number that feels real. Lines of code was the first — terrible, but measurable. Story points were the second — slightly better, but fuzzy.

Now there is a third.

Tokens.

Every AI interaction has a measurable cost. Input tokens. Output tokens. Reasoning tokens. API bills. Dollars. For the first time, the input cost of software is directly visible to finance.

The rough mapping looks something like this — illustrative, not standard:

| Work Item | Human Era | AI Era |
|---|---|---|
| 1 story point | 1–4 hours | 10k–100k tokens |
| 3 story points | ~1 day | 50k–500k tokens |
| 8 story points | several days | 200k–2M tokens |

No industry standard exists. But now management has a new toy.

And you know what happens when someone gets a new toy. They start measuring everything with it.

"Team X delivered 50 story points and spent $100 in AI costs. Team Y delivered 50 story points and spent $200. Why?"

"Team X spent $1000 on AI last month. Team Y spent $500. Should we change compensation models?"

"Team Z isn't using AI at all. Why not? Put it in their objectives."

"Let's study what Team X is doing with AI. Can our AI provider do that? Let's build a dashboard — ask our AI provider to provide everyone's chat histories, see how they're prompting."

The conversations write themselves.

Some make sense. Most don't. Because the person asking the questions is treating tokens like a production cost — raw materials per unit — when what they're really looking at is a mix of experimentation, iteration, dead ends, genuine productivity, and a whole lot of people figuring out what works.

Lines of code had the same problem. Story points had the same problem. Now tokens get the same treatment.

The question that hides behind all of them:

> "Can $200 of AI replace $2,000 of engineering effort?"

That ratio is what drives adoption. And tokens are the new language for having that conversation.

The hierarchy shifts.

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

## The Bottleneck Moves

But tokens are not value either. Ten million tokens can generate a billion-dollar insight or a useless document — same as a hundred engineer hours can create a successful product or worthless code.

Value still appears at the end of the chain, after human decisions.

And those decisions are still measured in squishy, uncertain units.

What does change is where the constraint sits.

Traditional capacity:

```
5 engineers × 40 hours = 200 hours of coding
```

AI era capacity:

```
5 engineers + AI agents = ? hours of coding
                        = limited by review, judgment, attention
```

Coding becomes abundant. Human attention becomes scarce.

| Task | AI Execution | Human Decision |
|---|---|---|
| Rename a button | tiny | tiny |
| Change a pricing model | tiny | huge |
| GDPR compliance change | medium | huge |
| Database migration | small | huge |

The expensive part is increasingly the decision, not the implementation.

Story points were never about typing. They captured complexity, risk, unknowns, coordination. AI compresses implementation. It does not compress ownership. Someone still has to answer what gets built, what gets delayed, what gets funded, and who owns it when it breaks.

---

## The Question Nobody Has Answered Yet

These are the questions the industry is going to have to sit with.

Not because anyone wants to defend story points.

Because no one knows what to replace them with.

The questions go like this:

"Does a five-point story stay five points if AI does half the work?"

"If velocity goes from 50 to 90, is that real productivity or just inflated estimates?"

"How do we justify the AI cost against the story points? Is there a ratio?"

"We need to tell the board something. They're asking why velocity went up but headcount didn't change."

Nobody has answers. Just more questions.

What is a story point when a machine writes most of the code? Do we re-baseline velocity? Do we split estimates into AI execution cost and human decision cost? Does the dollar value per story point change when $50 of tokens replaces $2,000 of salary?

I don't know either.

But here is what I keep thinking.

The engineer/number-person tension wasn't created by story points. It predates them — lines of code, man-months, function points. Every generation invents a new abstraction layer to have the same argument in slightly different words.

Story points were the latest language for that conversation.

AI doesn't resolve the tension. It gives both sides new ammunition.

Management gets tokens: measurable, finance-friendly, objective.

Engineers get the same old uncertainty: "The code appeared in two hours, but I still spent three days reviewing it, finding the edge cases, and fixing what broke staging."

The conversation continues. Just with different words.

And the questions — about what a story point means now, whether velocity will inflate, how to surface and justify AI costs — don't have settled answers yet.

But they are the right questions to be asking.

Because when a number that everyone pretended wasn't time suddenly breaks, you get to decide what comes next.

And whatever it is, the tension will still be there.

Just with a new name.
