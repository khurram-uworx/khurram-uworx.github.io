---
title: "Learning the fusion"
date: 2026-11-08
series:
  - "Better RAG without the math panic"
  - "Nivara"
tags:
  - RAG
  - Retrieval
  - Learning to Rank
  - Nivara
  - .NET
  - Machine Learning
comments: true
---

Learning the fusion: how to move 0.50/0.20/0.20/0.10 without changing the fusion formula

```csharp
public static FusionWeights Default { get; } = new(0.5f, 0.2f, 0.2f, 0.1f);
```

Four constants.

A retrieval policy, written in one line, reviewed by nobody in particular.

Fixed weights look like a confession. They describe an opinion, and like all opinions they survive without evidence until a reading of the eval un-blinds you.

The last post was about the four views and the mask. The mask was the interesting part. The weights were the part we walked past.

This post goes back for them.

## The same formula, with a gradient

The formula does not change. `w = softmax(theta)`, the same presence mask, the same per-row normalization. The only new piece is that `theta` is a variable instead of a literal.

```text
theta = [ln 0.5, ln 0.2, ln 0.2, ln 0.1]
softmax(theta) = (0.5, 0.2, 0.2, 0.1)
```

The defaults sum to exactly 1.0. That coincidence is the engine room.

Step 0 is numerically identical to the fixed `--rag-multiview` run, so every difference reported afterward is a change from what you can already compute, not a difference in framework. The mode prints `max |learned - fixed|` and targets about `1e-7`. It refuses to credit a delta as improvement if those numbers are not that close.

Without that line, I could blame every delta on a different formula. With it, every delta is learning.

The gate has the same trick. Its input weights start at zero and its bias starts at the log of the defaults, so at initialization the gate ignores the query and reproduces the constant. It has to earn the right to depend on the input.

The math, gently: `softmax` keeps the weights positive and summing to one, so the optimizer never needs a projection step to stay feasible. It can move freely.

The gate itself is small on purpose. Per query, per view, it takes the mean and the max of that view's scores across the candidate pool. Eight numbers in, four logits out. Eight times four plus four is 36 parameters.

Why mean and max? It is the minimum viable version of "how attentive is each expert to this query." It routes on the catalog of views that wake up, not on a deep semantic projection of the query. With a hundred queries, that is about all the signal you can afford to fit.

`--gate embed` replaces it with `Linear<384, 4>`, 1,540 parameters over the raw MiniLM vector. `--gate none` skips the gate entirely. We will come back to that comparison, because it did not go the way I expected.

The loss is listwise cross-entropy against a `2^grade` target, normalized per query.

```text
p_i    = 2^(g_i) / Σ 2^(g_c)
loss   = -Σ p_i * logsoftmax(fused)_i
```

One query is one graph. The implementation multiplies the target vector by `LogSoftmax` and sums; there is no row-gather operation in the gradient path, which keeps the whole thing inside the operations Nivara already has.

`--loss pairwise` uses the other gather-free path: a `+/-1` selector matrix and `BCEWithLogitsLoss`. Same model, same pool, same error bar.

## The ground truth is the hard part

None of that is the hard part. Loss functions are solved. Ground truth is not, and this is the part of learning-to-rank that actually decides whether the result means anything.

Four boring questions, in order.

Where do the grades come from?

| Source | What it is | When it runs |
|---|---|---|
| Judge | `relevance.tsv`, produced by `--rag-eval`, graded 0/1/2 per chunk over several stochastic runs | default, when the cache exists |
| Fixture | `relevance-fixture.tsv`, hand-authored per document | offline fallback, committed |
| Auto | heading-derived pseudo-queries from `queries.tsv` | opt-in with `--labels auto`, diagnostic only |

The judge is the headline. It is an LLM run offline; it never sits in the query path. The fixture is the thing you can run on a plane. Auto is the thing that will lie to you in a way that looks like a result, which is why it prints a leakage warning and never becomes a baseline of its own.

Why does the fixture key by document path instead of chunk id?

The judge cache names rows like `docs/X.md#N`. `N` is the chunk ordinal. Edit one section of `docs/SILK.md` three weeks later and every ordinal after the edit shifts by one. The cache still loads. It just grades the wrong chunk. I have watched that kill a benchmark run-off.

The fixture stores `(query, document path)` and expands one grade to every chunk of that document. In the run I am quoting, 88 labeled rows, 88 matched, 100 percent coverage against 2,389 chunks.

Why refuse to train below 50 percent coverage?

Because a leaky label file makes every ranker's recall look bad at the same time. The experiment you thought was measuring ranking was measuring a shrunken relevance set. The check costs one set subtraction per run. Own it.

How does the candidate pool avoid flattering the baseline?

Per query, the pool is the union of each view's own top-K plus every labeled positive. Every ranker reorders the same pool. If you build the pool from the fixed fusion's top-K, the baseline chooses its own competitors and every challenger runs on the baseline's shortlist. Rows with no scored view are counted and dropped; they cannot be a legal fusion row and they cannot produce a gradient.

And the target itself. `2^grade`, normalized. Grade 2 is four units, grade 1 is two, grade 0 is one, so a grader commits less when a chunk is merely relevant. A query whose rows are all zero still gets a uniform target so the run does not crash, but it is marked unusable. No positive means no signal.

How deterministic is the split?

Eighty, ten, ten. `string.GetHashCode()` is randomized per process, so the same line of code can produce a different partition in every run, and the nDCG tables stop being comparable. The split hashes each query with FNV-1a over its UTF-8 bytes and mixes in the seed. Same seed, same queries, same partition, every time.

A split that moves on every relabel releases a news cycle of noise.

## One query, one step

The loop is as small as it sounds. One epoch, one query, one step. Three hundred and forty-one train queries, three hundred and forty-one Adam steps per epoch, two hundred epochs for the headline run. Learning rate 0.05, weight decay 0.01.

A checkpoint is written after an epoch's validation pass, not after a training step. Validation tells you how well the model generalizes. Training loss tells you how well it fit, which is not the same question and is usually the less interesting one.

Selection uses validation nDCG@10 with patience 20. At the end, the best checkpoint is loaded back with strict state matching. On a cached MiniLM, the whole run is about two minutes on CPU.

Ridiculously small by modern standards. Four logits and a 36-parameter gate. No backpropagation through the encoder blocks.

That is not a limitation I am apologizing for. It is the point. The only thing that can move is the four numbers, so any movement is the fusion learning, not some other layer quietly compensating.

Now the gate comparison, which is where the honesty starts to cost something.

With 1,540 parameters and a 384-d input, the embedding gate does not move a 50-query held-out set. The mode reports negative 0.028 nDCG with p = 0.88. Not significant. Not a regression either.

The report's own line is "acceptance is a fair comparison, not a guarantee."

I left it in because the lesson is worth more than a clean scoreboard. A bigger gate did not buy anything on this data, and saying so is more useful than hiding a line that did not cooperate.

## Anatomy of one query

Here is what one query looks like end to end.

The query becomes 384 floats. `MultiViewScoring.ScoreAll` returns a `[rows, 4]` block of view scores and a mask. `CandidatePool.Select` builds the candidate set: about ten per view from that view's own ranking, plus every positive. `ForwardMasked` computes the weights, either from `theta` or from the gate, and applies the masked fusion. The listwise path takes `logsoftmax` over the fused scores, multiplies elementwise by the target, and sums. The pairwise path builds the `+/-1` selector and calls `BCEWithLogitsLoss`. Backprop, Adam step, zero grad.

That is one training example. Not one chunk. One query and its whole pool.

The first run I did was on the fixture, because the fixture is small enough to read.

Forty queries, 88 rows, split 34 train, 4 validation, 2 test. One epoch. The learned weights moved Content from 0.50 to 0.106 and Title to 0.527. On the two test queries, the learned model won one and tied one. p = 1.0.

A learned model with a perfect-looking nDCG on two queries.

I could have put that on a slide. I put it in a report instead.

The headline run uses the judge: 452 queries, 21,092 graded rows, 451 usable, split into 341 train, 60 validation, 50 test. That is the number I trust more, and it is also the one that carries a problem I need to say out loud.

The 452 queries were derived from the documents' own titles.

Read that again with the view census from the last post in mind: Title won 557 of the thousand top hits. That may not be the retriever discovering that titles matter. It may be the query generator asking questions the titles already answer.

The fixture tells the same story from the other direction. Its grades are attached to documents, not chunks, and the learned weights moved toward Title there too. When every chunk of the right document is a positive, the view that identifies the document is the view that wins. The model learned the shape of its labels. That is not a bug in the trainer. It is the label set describing itself.

Which is exactly why an evaluation set is a requirements document. It needs an owner, a version, and a review. And it is why the next step has to start with a better eval, not a better head.

One more number, because it keeps me honest. A sign test needs six non-tied queries before it can say anything. The fixture has two. It claims nothing. The 50-query held-out split is thin, and the report says so. That is what an honest small test looks like. So is p = 0.88. So is p = 1.0.

## What keeps the experiment clean

What kept the experiment clean was not cleverness. It was a short list of rules, each one paid for by a way the experiment could have lied.

| Rule | What it prevents |
|---|---|
| Step-0 parity | blaming a delta on a different formula |
| Gate invariance at init | a gate that "helps" before it has seen a query |
| Weight-agnostic pool | the baseline choosing its own competitors |
| Coverage refusal at 50% | training on a shrinking relevance set |
| Validation-only checkpointing | tuning on the test split |
| Six-query sign-test minimum | claiming significance from two |
| "Fixed vs learned" report language | calling any delta an improvement |
| Auto labels as diagnostic | a metric that measures its own generator |
| Gitignored reports | experiment output in the commit history |

None of that is machine learning. It is the discipline of a versioned schema and a reproducible build, applied to an opinion about relevance.

The four numbers are gone as constants. There is a checkpoint, a report, and a parameter vector that starts exactly where the old pipeline stood.

If you remember one number from this series, remember the init parity: `max |learned - fixed| ≈ 1e-7`.

Not because the model learned nothing at step zero. Because it proved it had learned nothing else.

But it learned once, offline. It cannot say "I don't know." It has never seen the reader.

The mask taught it to ignore a view it did not have. It never taught it to refuse an answer it did not believe.

The next one freezes the understanding layer, adds a typed decision that can choose, score, or abstain with a calibrated confidence, and lets feedback move it after deployment. It is specified in the open issues, not invented here, and it will take a few weeks because it has to be built rather than described.

You can read the blueprint in [issue #916](https://github.com/khurram-uworx/Nivara/issues/916).

The four numbers are no longer the interesting part. What they learned is. What they still cannot say is next.

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
