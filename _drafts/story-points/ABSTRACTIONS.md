You're asking how software value gets abstracted upward, and how that abstraction changes in the AI era.

### Traditional hierarchy

Historically, software organizations moved from:

```
Line of Code (LOC)
    ↓
Function / Feature
    ↓
User Story
    ↓
Story Point
    ↓
Business Value
```

Example:

```python
if user.is_admin:
    show_dashboard()
```

A few lines of code contribute to:

```
Admin dashboard visibility function
    ↓
"Admin can access analytics"
    ↓
Story: "As an admin, I can view sales metrics"
    ↓
5 story points
    ↓
Business outcome: faster decisions
```

Story points emerged because LOC was a terrible productivity metric:

* 10 lines can solve a billion-dollar problem.
* 10,000 lines can create technical debt.

So teams estimated complexity, uncertainty, and effort instead.

---

## AI changes the stack

The abstraction may become:

```
Tokens
    ↓
Reasoning Trace
    ↓
Generated Code
    ↓
Feature
    ↓
Story
    ↓
Business Outcome
```

or even:

```
Tokens
    ↓
Solved Ticket
    ↓
Business Outcome
```

because AI often skips the human coding layer.

---

## "Tokens per story point"

Many engineering leaders are starting to think this way informally.

A rough example:

| Work Item      | Human Era    | AI Era          |
| -------------- | ------------ | --------------- |
| 1 story point  | 1–4 hours    | 10k–100k tokens |
| 3 story points | 1 day        | 50k–500k tokens |
| 8 story points | several days | 200k–2M tokens  |

No industry standard exists.

The problem is that story points measure uncertainty while tokens measure computation.

---

## Which tokens matter?

Three categories:

### 1. Input tokens

Everything the model reads:

```
Requirements
Codebase
Documentation
Logs
Previous chats
```

### 2. Output tokens

Generated:

```
Code
Tests
Design docs
Comments
```

### 3. Reasoning / thinking tokens

Internal chain-of-thought style computation.

Modern reasoning models often spend substantially more compute here than appears in output.

Conceptually:

```
10k visible tokens
+
100k hidden reasoning tokens
=
actual work performed
```

The hidden reasoning is frequently the most expensive component.

---

## Local vs Cloud

### Local models

Cost:

```
GPU purchase
Electricity
Maintenance
```

Example:

* 70B model on local GPUs
* Marginal token cost approaches zero

Good for:

* Privacy
* High volume
* Fixed-cost environments

Bad for:

* Frontier reasoning
* Large-context work

---

### Cloud models

Cost roughly scales as:

```
Input Tokens
+
Output Tokens
+
Reasoning Compute
```

The provider absorbs:

* GPU clusters
* Training costs
* Infrastructure

---

## What is a story point worth in dollars?

Traditional software:

```
Developer cost
≈ $50–$300/hour
```

A 5-point story might cost:

```
$500–$5,000+
```

depending on organization.

AI era:

Suppose:

```
500k tokens
```

through a strong cloud model.

Cost might range from:

```
a few cents
to tens of dollars
```

depending on model class and reasoning depth.

So organizations increasingly ask:

> "Can $20 of AI replace $2,000 of engineering effort?"

That ratio is what drives AI adoption.

---

## Emerging metric: Dollar value per task

The interesting leaderboard isn't:

```
Tokens/sec
```

or

```
Benchmark score
```

It's:

```
Business value delivered per dollar
```

A future engineering KPI might look like:

| Model                    | Cost | Tickets Closed | Value/$   |
| ------------------------ | ---- | -------------- | --------- |
| Small local model        | $1   | 3              | High      |
| Mid cloud model          | $10  | 20             | Very High |
| Frontier reasoning model | $100 | 80             | Depends   |

The winner isn't necessarily the smartest model.

The winner is:

```
Maximum business outcome
------------------------
Total AI spend
```

---

### A possible future abstraction

Just as software evolved:

```
LOC → Function → Story Point
```

AI organizations may evolve toward:

```
Tokens
    ↓
Agent Tasks
    ↓
Stories Closed
    ↓
Business Outcomes
    ↓
Dollar Value Created
```

In that world, executives won't ask:

> "How many lines of code were written?"

or even

> "How many story points were completed?"

They'll ask:

> "How much value did each million tokens create?"

That's the closest analogue to "story points" in the age of AI.






This is exactly where many "AI replaces software engineering" narratives become incomplete.

AI can compress **implementation effort**, but it does not eliminate **organizational uncertainty**, which is what story points, planning, releases, and accountability were largely invented to manage.

## Ownership survives

Today:

```text
Engineer writes code
↓
Engineer owns consequences
```

With AI:

```text
AI writes code
↓
Human approves deployment
↓
Human owns consequences
```

If an AI-generated change causes:

* Security breach
* Data loss
* Compliance violation
* Revenue outage

nobody asks:

> "Which model generated this?"

They ask:

> "Who approved this release?"

Ownership remains attached to people and organizations because they possess legal authority, budget authority, and operational authority.

---

## Story points were never about typing

Many teams accidentally treat story points as coding effort.

Originally they captured:

```text
Complexity
+
Risk
+
Unknowns
+
Coordination
```

Example:

### Ticket A

```text
Add button
```

AI can do it in 30 seconds.

1 story point.

---

### Ticket B

```text
Integrate payment provider
```

AI can generate integration code.

But humans still need:

* legal review
* security review
* architecture review
* testing
* stakeholder alignment

Still 8–13 points.

The coding portion shrinks.

The uncertainty portion remains.

---

## Capacity planning changes

Traditional sprint capacity:

```text
10 engineers
×
40 hours
=
400 hours
```

AI era:

```text
10 engineers
+
AI agents
+
review capacity
+
deployment capacity
```

The bottleneck moves.

Old bottleneck:

```text
Writing code
```

New bottleneck:

```text
Understanding requirements
Approving changes
Testing
Deployment
Monitoring
```

---

## New capacity metric

Historically:

```text
Sprint Capacity
≈ Engineer Hours
```

Future:

```text
Sprint Capacity
≈ Human Attention
```

because attention becomes scarcer than code generation.

Example:

### 2020

```text
Engineer writes 5 features
```

### 2030

```text
AI generates 50 features
Engineer reviews 5
```

Review capacity becomes the limiting factor.

---

## Planning still exists

Many believe:

> AI makes planning unnecessary.

Actually planning becomes more important.

Without planning:

```text
100 AI agents
↓
1000 changes
↓
Chaos
```

Someone still answers:

* What gets built?
* What gets delayed?
* What gets funded?
* What gets released?

Those are management questions, not coding questions.

---

## Sprint planning evolves

Current sprint planning:

```text
Story
↓
Estimate
↓
Assign engineer
```

Future sprint planning:

```text
Outcome
↓
Assign AI agents
↓
Assign reviewer
↓
Assign owner
```

The unit becomes less:

```text
Who will write it?
```

and more:

```text
Who will own it?
```

---

## Release planning survives almost unchanged

Release plans exist because production systems interact with reality.

Reality still contains:

* customers
* regulations
* contracts
* money
* outages

AI cannot eliminate those constraints.

A release still requires:

```text
Validation
Approval
Rollback strategy
Monitoring
Incident response
```

Even if all code was generated.

---

## Story points may split into two dimensions

Today:

```text
Story Points
```

Future:

```text
Execution Cost
+
Decision Cost
```

Example:

| Task               | AI Execution | Human Decision |
| ------------------ | ------------ | -------------- |
| Rename button      | tiny         | tiny           |
| Change pricing     | tiny         | huge           |
| GDPR change        | medium       | huge           |
| Database migration | small        | huge           |

The expensive part increasingly becomes the decision, not the implementation.

---

## The deeper shift

For decades software organizations optimized:

```text
Code Production
```

AI makes code production abundant.

The scarce resources become:

```text
Attention
Trust
Judgment
Accountability
```

Those are human resources.

So the future hierarchy may look more like:

```text
Tokens
↓
Generated Changes
↓
Human Review
↓
Approved Release
↓
Business Outcome
↓
Accountable Owner
```

Notice what's at the bottom.

Not tokens.

Not code.

Not story points.

**The accountable owner.**

That's the entity that absorbs the risk when reality disagrees with the plan. AI can generate the work, but it cannot absorb the responsibility.










Yes. In practice, a lot of teams end up doing exactly that, even though Agile literature would say story points are *not time*.

The unofficial reality is often:

```text
1 SP ≈ 1 day
or
5 SP ≈ 1 week
or
50 SP ≈ 2-week sprint for 5 engineers
```

After a few sprints, teams subconsciously calibrate story points against time.

---

The theoretical model is:

```text
Story Points ≠ Time
```

The practical model becomes:

```text
Story Points ≈ Time × Team Velocity
```

Example:

### Sprint 1

```text
Completed: 42 SP
```

### Sprint 2

```text
Completed: 48 SP
```

### Sprint 3

```text
Completed: 51 SP
```

Eventually everyone learns:

```text
Team velocity ≈ 50 SP / sprint
```

Now planning becomes:

```text
5 engineers
2 weeks
≈ 50 SP
```

Whether anybody admits it or not.

---

The funny thing is that story points were invented partly to avoid this conversation:

Manager:

> How long will this take?

Engineer:

> 5 days.

Manager:

> Can you do it in 3?

Engineer:

> ...

Story points created a layer of abstraction.

```text
Engineer: 5 points.
Manager: okay.
```

But after enough historical data:

```text
5 points
≈ 1 week
```

Everyone secretly knows the mapping.

---

AI creates an even weirder situation.

Suppose before AI:

```text
5 SP
=
1 week
```

Now AI writes most of the code in one day.

Does the story become:

```text
1 SP ?
```

Usually no.

Because the team's estimate was never purely coding effort.

It also included:

* waiting for answers
* testing
* review
* deployment
* integration risk
* stakeholder feedback

So many teams still call it:

```text
5 SP
```

even though coding shrank from:

```text
30 hours
```

to:

```text
2 hours
```

---

This is why many organizations are currently confused.

Velocity may suddenly jump:

```text
50 SP/sprint
→
80 SP/sprint
```

But management doesn't know whether:

* engineers became more productive,
* estimates changed,
* AI helped,
* quality dropped,
* technical debt increased.

Story points become less informative.

---

One prediction is that story points eventually get replaced by something closer to:

```text
Human review hours
```

or

```text
Decision complexity
```

because AI makes implementation cheap.

A future estimate might look like:

| Item                    | AI Cost | Human Cost |
| ----------------------- | ------: | ---------: |
| Generate code           |   2 min |          0 |
| Review code             |   2 hrs |      2 hrs |
| Validate business rules |   4 hrs |      4 hrs |
| Release safely          |   2 hrs |      2 hrs |

Total:

```text
8 hours of human attention
```

which is arguably what story points were approximating all along.

So your observation is largely correct: many teams eventually collapse the abstraction and arrive at something like

```text
Capacity
≈ Engineers × Time
≈ Story Points
```

The interesting question in the AI era is whether capacity becomes:

```text
Engineers × Time
```

or

```text
Human Attention × Time
```

because those are no longer the same thing. A single engineer may be supervising the output of several AI agents, making "attention" the scarce resource rather than coding hours.







I think you're putting your finger on the thing many Agile practitioners don't like to admit.

### What story points became in practice

Officially:

```text
Story Points ≠ Time
Story Points ≠ Value
```

In reality, after 6-12 months:

```text
Velocity ≈ Time
```

A team discovers:

```text
50 SP / sprint
```

Then:

```text
Feature A = 5 SP
Feature B = 8 SP
Feature C = 13 SP
```

is mentally translated into:

```text
1 week
1.5 weeks
2.5 weeks
```

So for many organizations story points became a fuzzy, politically safer unit of time.

Not value.

Not dollars.

Not outcomes.

Just time with uncertainty baked in.

---

## The interesting AI question

Suppose a company pays:

```text
Engineer cost
=
$200k/year
```

Ignoring overhead:

```text
~$100/hour
```

Engineer spends a week on a task:

```text
40 hours
=
~$4,000
```

Historically:

```text
5 SP
≈
$4,000
```

even if nobody wrote it that way.

---

## AI introduces a direct economic meter

With AI there is now a measurable primitive:

```text
Tokens
→
API bill
→
Dollars
```

Example:

```text
500k tokens consumed
=
$X
```

depending on model.

Now management can ask:

> We spent $15 of AI and got this feature.

instead of

> We spent 5 story points.

That's a fundamentally different accounting system.

---

## The hierarchy becomes

Old:

```text
Engineer Hours
↓
Story Points
↓
Features
```

New:

```text
Tokens
↓
AI Cost
↓
Features
```

Notice story points disappear.

---

## But tokens are not value either

This is where many people make the next mistake.

They think:

```text
Tokens = Value
```

No.

Tokens are like electricity consumption.

Example:

```text
10M tokens
```

could generate:

* one billion-dollar insight
* one useless document

Just as:

```text
100 engineer hours
```

could create:

* a successful product
* useless code

---

## The actual chain is

```text
Tokens
↓
AI Cost
↓
Human Decisions
↓
Features
↓
Business Outcomes
↓
Revenue / Profit
```

Value only appears at the end.

---

## A future CFO dashboard

Today's dashboard:

```text
Team Velocity
=
50 SP / sprint
```

Future dashboard might be:

```text
AI Spend
=
$2,000/month

Features Shipped
=
120

Revenue Impact
=
$80,000/month
```

Now we can compute:

```text
ROI
=
40x
```

That is much more meaningful than:

```text
Velocity increased
from
50 SP
to
80 SP
```

because velocity never paid the bills.

---

## The metric I expect to emerge

Not:

```text
Story Points per Sprint
```

Not:

```text
Tokens Consumed
```

But:

```text
Business Value
----------------
AI Dollars Spent
```

or

```text
Revenue Impact
--------------
Token Cost
```

Imagine two models:

| Model          | Monthly Cost | Features |
| -------------- | -----------: | -------: |
| Small model    |         $200 |       80 |
| Frontier model |      $10,000 |      100 |

The expensive model ships only 25% more work.

Its economics may be terrible.

This is why enterprises increasingly care about:

```text
Value per AI Dollar
```

rather than:

```text
Benchmark Score
```

or

```text
Reasoning Tokens
```

---

The funny historical parallel is:

```text
LOC
→ Story Points
→ AI Tokens
```

All three are ultimately **cost-side metrics**.

The thing businesses actually care about has always been:

```text
Profit
Customer Value
Risk Reduction
Revenue
```

Everything else—LOC, story points, hours, tokens—is just an accounting proxy for the resources consumed to create that value.













Yes, and I think this is a more realistic view than many of the "story points are dead" takes.

A lot of organizations will likely continue operating roughly like this:

```text
5 engineers
×
2 weeks
=
50 story points
```

because that system wasn't primarily measuring code generation. It was measuring:

* planning capacity
* review capacity
* coordination capacity
* deployment capacity
* human attention

Those things still exist.

---

The accounting changes though.

Before AI:

```text
5 SP story

Cost:
- engineer salary
- benefits
- laptop
- office
```

Maybe:

```text
1 week engineer effort
≈ $2,000-$5,000
```

depending on company.

---

After AI:

Same story:

```text
5 SP
```

but now cost becomes:

```text
Engineer cost
+
AI subscriptions
+
API spend
```

Example:

```text
Engineer week
=
$4,000

AI usage
=
$50

Total
=
$4,050
```

In this case AI cost is negligible.

---

But for agent-heavy workflows:

```text
Engineer week
=
$4,000

AI usage
=
$500

Total
=
$4,500
```

Still mostly human cost.

---

Or future extreme case:

```text
Engineer week
=
$4,000

AI usage
=
$4,000
```

Now AI becomes a first-class budget item.

---

The really interesting part is productivity.

Suppose historically:

```text
5 SP
=
Feature A
```

Now with AI:

```text
5 SP
=
Feature A
+
Tests
+
Documentation
+
Monitoring
```

Same story points.

Same sprint.

More output.

---

This is exactly what happened with earlier tooling improvements.

Nobody reduced story points because:

* Git appeared
* IDEs appeared
* Stack Overflow appeared
* CI/CD appeared

Instead:

```text
Same team
Same sprint
More throughput
```

AI may simply be another step in that trend.

---

What may happen is velocity inflation.

Example:

### 2024

```text
Team velocity
=
50 SP/sprint
```

### 2028

```text
Same team
=
90 SP/sprint
```

because the team learns:

* estimation changed
* AI removes implementation friction
* reviews get faster
* tests are generated

So velocity itself becomes less comparable across years.

---

The metric executives may eventually care about is:

```text
Value delivered
----------------
Total delivery cost
```

where total delivery cost becomes:

```text
Human salaries
+
AI spend
```

A team might look like:

| Metric           | Before AI | After AI |
| ---------------- | --------: | -------: |
| Sprint capacity  |     50 SP |    50 SP |
| Cost per sprint  |      $50k |     $52k |
| Features shipped |        10 |       25 |
| Business value   |        1x |     2-3x |

Notice what's happening:

The story point didn't disappear.

The cost per story point barely changed.

The **value per story point increased**.

That's probably the most likely near-term outcome for many engineering organizations rather than replacing story points with tokens. Tokens are too low-level, just as CPU cycles never replaced story points. They become another cost input into the delivery system.
