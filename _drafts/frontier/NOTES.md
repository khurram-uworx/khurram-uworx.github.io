# Frontier Engineering — Working Notes

> Raw points to develop into posts. Not polished — source material for future writing.
> Last updated: 2026-07-18

---

## 1. Senior Engineers Losing the Rhythm

Personal story. How senior engineers, by adopting AI tools, start to lose the "rhythm" — that intuitive flow of navigating code, debugging, reasoning through systems. The muscle memory that took years to build starts to atrophy when you hand it off to a tool. Not a productivity problem. An identity problem. When the rhythm goes, so does the edge.

---

## 2. Fresh Engineers Not Learning the Craft

Using AI for junior/fresh engineers is harmful. They are not learning:

- IDE tricks — format code, editor configs, adding disclaimers on top
- Small refactorings — changing variable/function names, structuring code across files and classes
- Code structure within a single file — grouping private/protected/public methods together, knowing where to put them, separating static vs instance variables and functions

These are the foundational habits that make someone an engineer, not just a code producer. If you skip this, you never develop taste.

---

## 3. AI as Push Force Without Changing the Tyres

Everyone is talking about AI. Every team member wants to prove they have understood and are using AI — without realizing what is actually expected from different roles in the SDLC.

Physics analogy: AI has turbo-charged the engineers (the cars), but we haven't changed the tyres and the roads. The push force is increased but the resistance hasn't changed — QA and Scrum Master / Process are still the same resistance. Many times they are even talking about AI as the push force itself: "Hey, see, I am able to use AI to get context from my ticketing system and write test cases — because AI also knows about my testing style (Given/When/Then formats) or testing frameworks (Playwright / Selenium etc)."

The force went up. Everything else stayed the same. The system is under more stress, not less.

---

## 4. Short-Term Rainbows and the Economic Reckoning

Everyone is showing rainbows without realizing the short-term and long-term consequences. Using AI is basically an increased cost of doing business. The economic math is simple: if the left side of the equation number is increased, they will naturally ask to balance it by reducing human capital. And it will have ripple effects — given that juniors and now fewer juniors are already rotted.

---

## 5. Brighter Rainbows — The Trial Trap

We are showing brighter rainbows. AI companies are luring us into their programs — offering unlimited trials and free models. Agent this, agent that, agent here, agent there. When these trials will be over and free models will vanish, and realities will hit, we will be so rotted that we might not be able to perform that well in constraints (quotas).

The sugar rush ends. The dependency remains.

---

## 6. The Ceiling — Compiler PR Story

After Opus (Frontier / Commercial) and GLM 5.2 (Frontier / Open Source) models, started trusting coding agents. Found a compiler project (not popular, recent, small) with a genuine feature gap. Implementation looked doable — 4-5 files. Went ahead and created a PR.

Before opening the PR upstream, asked different coding agents to review the work: security angle, performance angle, design angle. Agents spent 30-40 minutes burning tokens and dollars. Came back saying everything looked nice.

PR got rejected the next day due to security issues. The maintainers outlined the problem clearly. Tried giving agents clues — they failed. When the complete problem was given to them, they prepared the fix. Updated the PR. Two more security issues found. PR closed by maintainers seeing a pattern.

Key insight: We don't know the ceiling of these models. Seeing them perform daily tasks (which were never rocket science anyway / enterprise scenarios) builds trust. They can miserably fail at any time — while rotting our brains.

**Potential post:** "The Ceiling" or "The Confidence Gap" — the distance between "AI can do my daily tasks" and "AI can do the work that actually matters." Maps to NOTES.md points 1, 2, 5.
