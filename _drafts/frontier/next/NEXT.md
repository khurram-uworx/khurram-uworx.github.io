# Next Post Ideas — Frontier Engineering Series

## Current State

- **Post 7**: "The Two Algorithms" (2026-07-18) — published in `_posts/`
- **Unused material in `next/`**: `Business.md` (Scenario 3 + executive framing unused), `Metrics.md` (capability metrics, skill matrix, ROI metrics, balanced scorecard unused), `Risks.md` (all 8 risk categories, untouched)
- **Working notes**: `_drafts/frontier/NOTES.md` — 5 raw points captured 2026-07-18

---

## Proposed Post 8: "The Ceiling"

**Core question**: We don't know the upper bound of these models' failure modes. Trusting them on daily tasks — which were never rocket science — creates a false sense of capability that collapses at the ceiling.

### Source material

Personal story: Contributed to a compiler project (not popular, recent, small). Found a genuine feature gap, implemented it (4-5 files). Before opening the PR upstream, used multiple coding agents (Opus, GLM 5.2) to review from security, performance, and design angles. Agents spent 30-40 minutes burning tokens, came back saying everything looked fine.

PR was rejected upstream for security issues. The maintainers outlined the problem clearly. Tried giving agents clues — they failed. Gave them the complete problem — they prepared the fix. Updated the PR. Two more security issues found. PR closed by maintainers seeing a pattern.

### Argument structure

1. **Opening**: The scene — building the feature, running agents for review, the confidence. Then the rejection.
2. **The verification illusion**: You used agents to verify your work. They spent 40 minutes. They missed everything. Not partially — completely. The tools you trusted to catch what you missed missed exactly what they were supposed to catch.
3. **The ceiling we can't see**: Daily tasks are not rocket science. Enterprise CRUD, test scaffolding, API wiring — AI does these reliably. That reliability trains trust. But we don't know where the ceiling is. The compiler project was a small, well-defined contribution. If agents fail there, where else do they fail silently?
4. **The rot**: Every task AI handles well makes you less likely to question it on the next task. The trust compounds faster than the capability. You stop reading the output as carefully. You stop building the skill that would let you catch the failure. The ceiling is real, but you can't see it because you've stopped climbing.
5. **Closing**: The question — what is the thing you're trusting AI with today that you won't know was wrong until it fails publicly?

### Series fit

Extends Post 7's organizational argument to the individual engineer's trust calibration. Post 7 says orgs choose between augmentation and replacement. Post 8 asks: what happens when the engineer choosing augmentation discovers the tool has a ceiling they couldn't have known about?

### Tone

Personal, essayistic. Opens with a specific scene (the compiler PR). The story carries the argument — not a framework post. Closer to Breadcrumbs (Post 8 in the personalization arc) than to Two Algorithms.

---

## Alternative Post Ideas (backlog)

### "The Unbalanced Force" (physics analogy)
- Uses: NOTES.md point 3 — AI turbo-charged the car but we didn't change the tyres/roads
- Argument: Push force increased (AI capability), resistance unchanged (QA, Scrum Master, Process). Teams talk about AI as push force without recognizing the system is under more stress, not less.
- Risk: Could overlap with Post 7's equation section. Best used as a 2-3 sentence metaphor inside another post, not a standalone.

### "The Rhythm" (senior engineers losing instinct)
- Uses: NOTES.md point 1 — senior engineers losing rhythm by using AI
- Argument: Personal story of losing the intuitive flow — navigating code, debugging, reasoning through systems. The muscle memory that took years to build atrophies when you hand it off. Not a productivity problem. An identity problem.
- Risk: Very personal, may not generalize. Works better as a scene within a larger post than as a standalone argument.

### "The Craft We're Skipping" (junior engineers and foundational habits)
- Uses: NOTES.md point 2 — juniors not learning IDE tricks, refactoring, code structure
- Argument: AI skips the foundational habits that make someone an engineer. Format code, editor configs, changing variable names, grouping private/protected/public, separating static/instance. These aren't chores — they're how taste develops.
- Risk: Overlaps with Post 5's seniority paradox and Post 6's cultural failure. Could be the personal-story version of those arguments.

### "The Trial Trap" (AI companies' free models and lock-in)
- Uses: NOTES.md point 5 — AI companies luring with unlimited trials and free models
- Argument: When trials end and free models vanish, we'll be so dependent we can't perform under constraints (quotas). The sugar rush ends. The dependency remains.
- Risk: Could feel like vendor criticism. Better as a scene within a larger post than a standalone.

### "The Capability We're Losing"
- Uses: Metrics.md maturity levels, Post 5's seniority paradox
- Argument: Individual engineer capability is declining because the environment no longer develops them
- Risk: Overlaps with Posts 5 and 6

### "Treat It Like Infrastructure"
- Uses: Risks.md (autonomous systems risk, rapid capability gain), mindset shift
- Argument: AI agents should be treated as critical infrastructure, not productivity tools
- Risk: Pulls away from software-eng focus

### "The Honest ROI"
- Uses: Business.md full economic model, Metrics.md ROI metrics
- Argument: Economics of AI adoption done honestly
- Risk: Could feel spreadsheet-y

### "Risks We're Not Measuring"
- Uses: Risks.md (economic disruption, information integrity, autonomous systems)
- Argument: Engineering risks go beyond code quality
- Risk: Too broad for series scope

---

## NOTES.md Theme Mapping

| NOTES.md point | Best home | Notes |
|---|---|---|
| 1. Senior engineers losing rhythm | "The Rhythm" or scene within Post 8 | Personal, identity-level. Not org-level. |
| 2. Fresh engineers not learning craft | "The Craft We're Skipping" or scene within Post 8 | Extension of Post 5's seniority paradox but from the training side. |
| 3. AI as unbalanced force (physics) | 2-3 sentence metaphor in Post 7's equation section OR standalone "The Unbalanced Force" | Best as metaphor, not standalone argument. |
| 4. Economic reckoning | Already covered in Post 7 (equation section) | Don't re-argue. |
| 5. Trial trap / brighter rainbows | "The Trial Trap" or scene within Post 8 | Risk of feeling like vendor criticism. |

---

## Post 7 Revision Note

The physics analogy (NOTES.md point 3) could strengthen Post 7's equation section with a 2-3 sentence metaphor about unbalanced force. The post currently treats replacement as a budget decision; the analogy adds that it's also a systemic imbalance. Optional — the post works without it, but it would sharpen the structural argument.

---

## Risks.md Usage Plan (`next/Risks.md`)

Risks.md hasn't been used yet. Options:

1. Fold relevant risks into a future post (economic disruption, autonomous systems)
2. Standalone post later — but would need heavy filtering to stay in series scope
3. Skip for this series — global/existential risks may be better for a different context

Recommendation: Fold the most relevant risks into whichever post naturally extends from them. Leave global/existential risks for a different context.
