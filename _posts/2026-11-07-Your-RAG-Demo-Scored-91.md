---
title: "Better RAG without the math panic"
date: 2026-11-07
series:
  - "Better RAG without the math panic"
  - "Nivara"
tags:
  - RAG
  - Retrieval
  - Embeddings
  - Nivara
  - .NET
  - Search
comments: true
---

Your RAG demo scored 91%. Why did it pick the wrong answer anyway?

You shipped a chatbot.

So did every other team with a `docs/` folder and a README.

Twenty questions, written by you, answered by the bot, graded by you. Then staging, then a quiet win, then three weeks of nobody looking at it.

A reader asked where the null mask lives.

The bot sent them to `docs/ROADMAP-SUGGESTION.md`.

The chunk that mentioned "null masks" also carried the word budget, and the single embedding averaged both into 0.40. Another chunk, less useful, scored 0.41. The retrieval stack did everything it was asked to do. Given one number to score the chunk with, it had no honest way to decide.

That is the whole failure. Not the model. The shape of the thing you asked the model to see.

This post is about that shape, and three moves that change it. No new embedding model. No fine-tuning. No reranker purchase. The embedding stays exactly where it is.

## One number per chunk

Start with the most common case: a company has a `docs/` tree of Markdown, a README, and a chatbot.

A hundred pages. A couple of thousand section-scoped chunks, capped at 500 characters each. One sentence model, `all-MiniLM-L6-v2` — 22 million parameters, 384 floats out, distilled from a BERT encoder that is much larger than this job needs.

The query "How do I use BLAS or MKL?" should land on `docs/BLAS-MKL.md`.

The baseline embeds every chunk once, embeds the query, takes cosine similarity, sorts, keeps the top ten. It is honest. It is measurable. It is what most articles ship.

It is also wrong in a subtler way than a wrong model. It is the wrong document shape, three chapters before you get to the model.

The fix is not more vectors. It is better vectors: four of them per chunk, each one embedding a different part of the same document.

| View | What gets embedded | Missing when |
|---|---|---|
| Content | prose and fenced blocks, as written | almost never |
| Title | the document title, constant per file | never, with a file-name fallback |
| Section | the heading breadcrumb, `H1 > H2 > H3` | never, because depth stops at three |
| Code | only the fence bodies inside the chunk | whenever the chunk has no fenced block |

Look at the last column.

`docs/STREAMING.md` is mostly prose. Many of its chunks contain no fence at all. Code is the conditional view. The other three are always there.

That asymmetry — three constant signals, one that can drop out — is the thing every later block has to account for. It is also the reason the fourth view is worth having. If all four were always present, you could pretend they were one vector and nobody would notice for a year.

Here is the moment that convinced me.

A chunk's prose says "don't be clever." Its fence says `throw new Exception`. Collapse both into one embedding and the model builds a single interesting vector whose geometry sits somewhere between the two signals.

Ask it "how do I throw an error" and the chunk scores high. Ask it whether the containment policy chapter should be surfaced and the same chunk scores higher than it deserves.

The chunk won. The reader lost.

Now score Content and Code independently and the same hit carries a second number.

*Code 0.94. Content 0.71. The fence matched, not the prose.*

That sentence is worth more retrieval authority than any score column, and it did not exist until the views were split. A score tells you what won. The sentence tells you why.

A junior engineer on my team asked the obvious question when I showed her the table.

"So which view is right?"

Whichever won. But that is not the useful part.

The useful part is that we can name it.

## A chunk is a section

If the four views are going to mean anything, they have to be cut from a unit that has all four parts.

Our chunk is a section-scoped run: one section's prose plus its whole fenced blocks, up to 500 characters. Fences are never split. A long paragraph is split at whitespace. The unit matches the way a section is actually written — paragraphs and complete fences — instead of spreading each fence across the wrong granularity.

The design decision I want you to notice is the boring one.

One chunk is a section-sized unit, not one fence.

If a chunk were a single fence, Content and Code would be the same text embedded twice. The winning-view column would always say Code when a fence matched. The fused score would be cosine, counted twice, wearing a hat.

Chunking by section keeps the two distinct. The prose is one thing. The fence inside it is another. The model gets to disagree with itself, and the disagreement is where the information lives.

There is a second benefit, and it is the kind that pays for the whole design.

Title and Section text repeat heavily. The title of a file is the same string for every chunk in that file. Section breadcrumbs repeat too. So the view texts are deduplicated per document before embedding.

In the run I am quoting, 2,389 chunks produce 9,556 view slots. Deduplicated, that is roughly 3,400 unique texts. We embed 3,400 things instead of 9,556 and reuse each result for every chunk that needs it.

A 2.8× reduction in embedding work, for the same retrieval.

That is not a machine-learning win. That is a cache.

## The mask

Almost every RAG sample does something like this:

```text
finalScore = 0.4 * contentScore + 0.2 * titleScore
```

The almost-always-missing piece is the mask.

When Code has no vector, it has no score. Feed a zero into the sum and you have just punished a chunk for a view it never had. Drop the term instead and the remaining weights still add up to less than one, so the chunk is punished again, this time by arithmetic rather than by zero.

Both failure modes look like a ranking. Neither is a measurement.

The fix is a per-row normalization over the views that actually exist.

```text
fused = ((S ⊙ M) @ w) / (M @ w)

S: cosine scores, shape [rows, 4]
M: presence mask, 1 when the view exists, 0 otherwise
w: softmax(theta), always positive
```

`⊙` is elementwise multiply. `@` is a matrix-vector product. A missing view disappears from the numerator and the denominator at the same time, so a chunk without code is scored against the views it has. Weights are `softmax(theta)`, which keeps them positive and summing to one, so the denominator can never be zero while at least one view is present. A row with no scored view at all is dropped and counted, not allowed to explode.

That is the entire math. It fits in three lines and you can explain it to a product manager.

The other half of the design is what travels with each hit. Every row carries the view that won, the gap to the top hit, and the divergence between Content and the other views.

| Band | Condition | What to read it as |
|---|---|---|
| `Near` | gap ≤ 0.02 | the views mostly agreed |
| `Distinct` | gap ≤ 0.08 | the views differ but coexist |
| `Weak` | gap > 0.08 | one view spiked and dragged the result |
| `InsufficientViews` | fewer than two scored views | agreement statistics are meaningless here |

The winning view is whichever score is highest, with ties broken in declaration order: Content, then Title, then Section, then Code. Divergence uses Content as the reference and buckets the result: `Agree` at 0.05 or less, `Mild` at 0.15 or less, `Strong` above that.

## What normal looks like

The sample lives in `samples/NivaraChat` in the [Nivara repository](https://github.com/khurram-uworx/Nivara). Nivara is a .NET dataframe library; the retrieval modes and the fusion arithmetic in `MultiViewScoring.cs` are sample code, not library code. Run it with `--rag-multiview` and it prints a table per query: fused score, rank band, winning view, the four raw scores, the maximum divergence, and the chunk.

The first query I ran was "How does a symbol citation resolve?"

The top three hits were the same ADR, three sections apart. All `Near`. All won by Content. Divergences of 0.037, 0.037, 0.032.

That table is the retrieval saying something a single score cannot: this document is the answer, and these are three depths of the same answer. Nothing was wrong. The ranking was merely flat, and flat is what you want to see when a document is genuinely on topic.

Then the corpus-wide view. `--explore` buckets every top hit by how far Content sits from the other views. That is the closest thing to a sanity check a retrieval engineer owns: not "is the score high" but "do the representations agree about what this document is."

On the early evaluation run, a hundred queries graded by an offline judge, fused retrieval scored 0.698 nDCG@10 against 0.596 for content-only. Sixty-eight wins, thirty-one losses, one tie. Sign test p = 0.0003, Cohen's d 0.422.

I want to be careful with that number, because it is easy to over-read.

Across the thousand top hits that run logged, a hundred queries at ten hits each, the winning view was Title 557 times, Content 348, Section 69, Code 26. The agreement buckets were 134 `Agree`, 354 `Mild`, 512 `Strong`.

And when individual views were dropped, fused nDCG landed between 0.688 and 0.693. Dropping Content alone took it to 0.596.

So the honest reading is this. Content carries most of the score. Title, Section, and Code add a small amount of ranking and a large amount of explanation. If your only goal is nDCG, four views are a modest improvement over one good content embedding. If your goal is to know why a chunk was retrieved, four views are the difference between a number and a sentence.

The five hundred and twelve `Strong` rows are the part I keep thinking about.

Strong divergence means the title, the heading, the prose, and the code of the same document disagree about what the document is about. That is not a retrieval bug. That is a documentation bug, surfaced by retrieval.

Your RAG is measuring your docs.

Which means the owner of retrieval quality is usually not the ML team. It is the team that writes the headings, names the files, and leaves the stale example in the fence.

## The part that is still dark

By the end of this, you have four view embeddings per chunk in one frame, a masked late fusion that refuses to punish a missing view, a dial that names the winner and the disagreement on every hit, and a way to compare tuning head-to-head before anyone buys a reranker.

The bot can now say the fence matched.

A system that can name what it noticed is doing something a score cannot. It is not just ranking. It is reporting.

Which leaves the four numbers.

`0.5`, `0.2`, `0.2`, `0.1`.

Content, Title, Section, Code, in that order, in a file, because someone wrote them there.

I looked at that line for a while. I could not tell you who chose it, or what measurement it came from. It might be a good guess. It might be a guess that has been copied into a thousand repositories.

That is not a foundation. That is an opinion with a softmax wrapper.

*Next: Learning the fusion. The formula does not change. The four numbers stop being constants, and the first thing the model has to prove is that it has not moved at all.*

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
