# Frontier Engineering — Series Reference

> Reference for future sessions creating or editing posts in the "Frontier Engineering" series.
> Last updated: 2026-07-18

---

## Series Overview

A long-form blog series by Khurram exploring what "frontier engineering" means in the AI era — defining it, mapping how to pursue it, and warning how the current AI adoption wave threatens to hollow out the very discipline that frontier work depends on.

**Series tag (in front matter):** `"Frontier Engineering"`
**Folder for published posts:** `_posts/`
**Folder for working materials & drafts:** `_drafts/frontier/`

---

## Series Arc

The series has a two-phase progression:

### Phase 1 (2025): Definition & Aspiration
Optimistic, definitional, motivational. The author carves out a category and encourages readers to move toward it.

- **Post 1** (2025-07-23) — *More Frontier Engineers* — Defines what a "Frontier Engineer" is and why the label is needed in the 2020s. Commoditization of engineering, the explosion of abstractions, new domains (AI, quantum, space, bio), the dilution of the "engineer" title. Where AI will hit hardest and what survives.
- **Post 2** (2025-07-24) — *Becoming a Frontier Engineer: A Roadmap to Innovation* — A practical roadmap for transitioning from industrial software engineer to frontier engineer. 7 strategic steps, plus three 3-month detailed transition plans (frontend, backend, QA/test engineers) with weekly breakdowns.
- **Post 3** (2025-08-04) — *The Evolution of Data Science: From Frontier to Infrastructure and the Path Forward* — Argues data science has transitioned from frontier work to infrastructure. 6 reasons the hype cooled. The frontier has shifted to MLOps, generative AI, ethics/bias, and AI-first product design.

### Phase 2 (2026): Warning & Consequence
Serious, literary, urgent. Examines what happens when AI coding agents are adopted without honesty about their limits.

- **Post 4** (2026-05-09) — *The Productivity Illusion* — Part 1 of "The Agentic Engineering Team" sub-series. The measurement problem (output vs. outcome), the hidden value of friction, the greenfield bias, rushing what needed time. What to actually measure.
- **Post 5** (2026-05-10) — *The Structural Problems* — Part 2 of "The Agentic Engineering Team." The review bottleneck, the codebase scale cliff, the enterprise dark matter problem, the seniority paradox. What engineering leaders should actually do.
- **Post 6** (2026-06-18) — *The Plastic Code* — The narrative culmination. Introduces "plastic code" — AI-generated code that was never known by any human. The rockstar trap. A cultural failure, not a technical one. The Christmas-season outage scene.
- **Post 7** (2026-07-18) — *The Two Algorithms* — The fork in the road. Every org adopting AI is running one of two algorithms: augmentation (engineer is the product, AI is the lever) or replacement (output is the product, engineer is the input being optimized away). Both produce the same dashboard. The divergence only shows when something breaks — and only one organization can fix it.

---

## Writing Style

### Tone
- **Early posts (1-3):** Conversational, slightly informal. Emoji section headers, heavy bullets, comparison tables, rhetorical questions. Feels like a tech blog or internal engineering talk.
- **Later posts (4-7):** Literary, essayistic, serious. Reads like an essay by a senior engineering leader who has lived these problems. Longer flowing paragraphs, minimal bullets, narrative scenes, dramatic closings. The voice is authoritative but not academic.

### What the series is NOT
- **No code blocks, no code examples** — even though topics are engineering-related, depth comes from argument quality, not code snippets.
- **No images, no diagrams** — text-only.
- **Not prescriptive** — posts diagnose and argue, they don't hand out checklists (Post 2 is the exception — it's a practical roadmap).
- **Not vendor/case-study content** — no tool reviews, no benchmarks, no product names beyond passing references.

### Recurring stylistic devices
- **Bold thesis lead-ins** at the start of sections or paragraphs
- **Rhetorical questions** as transitions between ideas
- **`---` horizontal rule dividers** between every section (liberally)
- **Italic emphasis** for precise concepts (*absent*, *never*, *why* vs. *how*)
- **Narrative openings** for the later posts (Post 6's standup scene, Post 7's org announcement)
- **Dramatic closing questions** — each post ends with a haunting, open question
  - Post 6: "What happens when the plastic world breaks — and no one left knows how it was built?"
  - Post 7: "Which algorithm is your organization actually running — and would you know if the answer was the one you didn't intend?"

### Closing question pattern
Every post ends with a question that doesn't answer itself. The series trusts the reader to sit with the discomfort. Do not resolve the tension. Do not offer a neat summary. End on the question.

---

## Front Matter Format

All posts use this exact format:

```yaml
---
title: "Title String"
date: YYYY-MM-DD
series:
  - "Frontier Engineering"
tags:
  - Tag1
  - Tag2
comments: true
---
```

**Notes:**
- `series` is always a YAML list with a single entry: `"Frontier Engineering"`. This powers the series navigation widget.
- `tags` vary by post. Recent posts (6, 7) use: `AI`, `Software Engineering`, `Leadership`.
- `comments: true` is present in every post.
- Tag capitalization is inconsistent across the series (`Engineering` vs `software engineering` vs `Software Engineering`). Recent posts normalize to Title Case. Follow the recent convention.
- No `author`, `description`, `image`, `layout`, or `categories` fields.

---

## Series Navigation Widget

Every post ends with this Liquid template (identical across all posts — copy verbatim):

```liquid
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
```

---

## Core Themes & Recurring Motifs

1. **Frontier vs. industrialization** — the central tension. Something starts as frontier, becomes mainstream, then commoditized. Data science went through this. Software engineering is going through this now.

2. **Commoditization lifecycle** — Posts 1, 3, and the later posts all describe the cycle: frontier → mainstream → commoditized.

3. **"Friction is valuable"** — difficulty, slowness, and struggle are features, not bugs. Post 4 makes this analytically ("friction was doing hidden work"). Post 6 makes it narratively ("struggle is how senior engineers are made"). Post 7 makes it structurally (removing friction is how organizations slip from augmentation to replacement).

4. **The human phase matters** — engineering is a social discipline. The code is the artifact; the *understanding* lives in conversations, arguments, whiteboard sessions, debugging sessions. When code skips the human phase (Post 6) or the human is optimized away (Post 7), the understanding disappears.

5. **What AI can't do is what matters most** — complex systems, institutional knowledge, judgment under pressure, frontier work. The capability being eroded is exactly the capability that's hardest to rebuild.

6. **The dashboard lies** — velocity, output, and adoption metrics look great while the thing that matters (understanding, judgment, capability) silently disappears.

---

## Ground Already Covered (Avoid Repeating)

When writing a new post, do NOT re-argue these points — reference them as established:

| Theme | Covered in | Do not re-argue |
|---|---|---|
| Output vs. outcome measurement | Post 4 | State as known; build on it |
| Friction does hidden work | Post 4, 6 | Callback in one line, not a section |
| Greenfield bias in productivity claims | Post 4 | Referenced, not re-explained |
| Review bottleneck (static) | Post 5 | Extend dynamically, don't re-describe |
| Codebase scale cliff | Post 5 | Established |
| Enterprise dark matter (agents not trained on internal code) | Post 5 | Established |
| Seniority paradox (junior growth loop broken) | Post 5 | Established |
| Plastic code (code never known) | Post 6 | Established |
| Rockstar trap (10x feeling ≠ deep understanding) | Post 6 | Established |
| Cultural failure: design conversations evaporating | Post 6 | Established |
| Augmentation vs. replacement as org choice | Post 7 | Established |
| The rollout roadmap (and how Algorithm 2 organizations skip it) | Post 7 | Established |
| The equation that reveals which algorithm you're running | Post 7 | Established |
| Metrics as confession (what you measure reveals your algorithm) | Post 7 | Established |
| Terminal condition of replacement (compounding capability loss) | Post 7 | Established |

---

## How Posts Connect

- **Posts 4 & 5** form an explicit 2-part sub-series ("The Agentic Engineering Team: Part 1 of 2 / Part 2 of 2") with direct textual cross-references.
- **Post 6** takes a word from Post 1 ("software is infinitely malleable" → "plastic") and turns it into the central metaphor.
- **Post 7** reframes the series' frontier-vs-industrialization tension (Post 1) as an organizational choice: augmentation keeps the frontier alive, replacement industrializes it.
- **Implicit thematic throughline:** Posts 4-7 each diagnose a different facet of the same erosion. Post 4 = measurement. Post 5 = structure. Post 6 = code artifact. Post 7 = organizational choice. Post 8 (if written) = individual trust calibration. The series trusts the reader to see the pattern.

---

## Research & Working Materials

Working materials live in `_drafts/frontier/next/`. Historic drafts and abandoned attempts live in `_drafts/frontier/adaption/`:

| File | Content | Status |
|---|---|---|
| `next/Business.md` | Economic models — ROI framing with variables (team size X, value Y, AI cost Z), three scenarios (productivity increase, team reduction, full economic model) | Partially used in Post 7 (Scenarios 1 & 2 concepts). Scenario 3 + executive framing unused — available for future posts. |
| `next/Metrics.md` | 4-layer measurement framework (Adoption, Productivity, Quality, Engineering Excellence), maturity levels, skill matrices, ROI metrics, behavioral metrics, balanced scorecard, rollout roadmap | Partially used in Post 7 (behavioral metrics + rollout roadmap). Capability metrics, skill matrix, ROI metrics, balanced scorecard unused — available for future posts. |
| `next/Risks.md` | 8 core AI risk categories (malicious use, misalignment, capability overshoot, economic disruption, information integrity, autonomous systems, power concentration, existential) + "treat AI as critical infrastructure" mindset shift | Unused. Could feed a future post or a separate series. |
| `next/NEXT.md` | Backlog of next-post ideas | Living document — update when planning a new post |
| `adaption/Two-Algorithms.md` | Early draft of Post 7 ("The Two Algorithms") — published version in `_posts/2026-07-18-Two-Algorithms.md` | Historic record |
| `adaption/The-Velocity-Trap.md` | Abandoned first draft — velocity-as-trap concept, later subsumed into Post 7's argument. Decision made to pivot to the two-algorithms framing instead. | Abandoned draft — not revisited (velocity/trap ground already well covered in Posts 4-7) |

---

## Next Post Ideas (Backlog)

See `_drafts/frontier/next/NEXT.md` for the full backlog. Current candidates:

| Post idea | Source material | Series fit |
|---|---|---|
| "The Ceiling" | Personal story — compiler PR rejection despite agent review. NOTES.md points 1, 2, 5. | Extends Post 7's org-level argument to the individual engineer's trust calibration. Strongest candidate — personal, specific, series-fitting. |
| "The Unbalanced Force" | NOTES.md point 3 — physics analogy (push force increased, resistance unchanged) | Best used as a 2-3 sentence metaphor inside another post, not standalone. Could strengthen Post 7's equation section retroactively. |
| "The Craft We're Skipping" | NOTES.md point 2 — juniors not learning foundational habits | Extension of Post 5's seniority paradox from the training side. Risk: overlaps. |
| "The Rhythm" | NOTES.md point 1 — senior engineers losing instinct | Personal, identity-level. Works better as a scene within a larger post. |
| "The Trial Trap" | NOTES.md point 5 — AI companies' free models and lock-in | Risk of feeling like vendor criticism. Better as a scene. |

---

## Editorial Guidance for Future Posts

1. **Before writing a new post:** read the previous 2-3 posts. The series is a continuous argument. A new post extends it, it doesn't restart it.

2. **Check the "Ground Already Covered" table above.** If a section re-argues an established point, compress it to a one-line callback or cut it.

3. **Match the recent tone (Posts 4-7):** essayistic, literary, no emoji headers, minimal bullets, narrative openings where appropriate, dramatic closing questions.

4. **Maintain the format:** the front matter block, the `---` dividers, the closing series nav widget (copied verbatim), the absence of code/images.

5. **End on a question, not an answer.** The series is a diagnosis, not a prescription. Trust the reader to sit with the discomfort.

6. **Naming convention:** "The [Noun] [Noun]" — *The Plastic Code*, *The Structural Problems*, *The Two Algorithms*. Clean, parallel, memorable.

7. **Length target:** ~1,500 words (Post 6 = 134 lines, Post 7 = 149 lines). Long enough to build an argument, short enough to read in one sitting.

8. **The closing question is the post's sting.** It should stay with the reader after they close the tab. It should not be answerable from the dashboard, the sprint board, or the all-hands deck.

9. **Post 7 optional revision:** The physics analogy (NOTES.md point 3) could strengthen the equation section with a 2-3 sentence metaphor about unbalanced force — push force increased, resistance unchanged. The post works without it, but it would sharpen the structural argument. Low priority.