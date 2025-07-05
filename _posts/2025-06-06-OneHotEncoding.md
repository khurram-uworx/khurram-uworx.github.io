---
title: "One-Hot Encoding vs Multi-Hot Encoding"
date: 2025-06-06
tags:
- ML
---

> If I am creating one hot embedding vector for say food, let's say those have keys, pizza, burger, sushi and drink
> If I am encoding an order that has burger and drink, my encoding vector will be 0 1 0 1?

Yes, that's correct.

Given the keys in this order:
`[pizza, burger, sushi, drink]`

An order with **burger** and **drink** would be encoded as:
`[0, 1, 0, 1]`
→ 0 for pizza, 1 for burger, 0 for sushi, 1 for drink.

Each index corresponds to whether that item is present (`1`) or not (`0`) in the order.

No, this is **not** strictly **one-hot encoding** — it's actually called a **multi-hot encoding** (or **multi-label binary encoding**).

### Difference:

* **One-hot encoding**:
  Used when there's **only one possible category** per instance.
  Example:
  If an order could be *only one* of `pizza`, `burger`, `sushi`, or `drink`, then:

  * `burger` → `[0, 1, 0, 0]`
  * `sushi` → `[0, 0, 1, 0]`

* **Multi-hot encoding**:
  Used when **multiple categories can apply simultaneously**, like in your case with an order containing both `burger` and `drink`.

  * `burger + drink` → `[0, 1, 0, 1]`

So your example is **multi-hot encoding**, not one-hot.

> When we should use one hot encoding and when we can use multi hot, in context of tokenization

Great question — especially in the context of **tokenization** and **NLP/ML pipelines**, the choice between **one-hot** and **multi-hot** encoding depends on the nature of your data and task:

---

### ✅ **Use One-Hot Encoding When:**

* You have **a single label/token per instance** (e.g., classification tasks).
* Your vocabulary is **mutually exclusive** — each input can only map to one category at a time.
* You're encoding **individual tokens** in NLP — common in early NLP models or toy examples.

📌 **Example use cases:**

* Classifying a sentence sentiment: `[positive, neutral, negative]`
* Encoding tokens in a sequence for RNNs:
  `word = "pizza"` → `[0, 0, 1, 0, ..., 0]`

---

### ✅ **Use Multi-Hot Encoding When:**

* You have **multiple possible labels per instance**.
* You need to represent **sets of tokens** or features — not just individual ones.
* You're encoding **entire documents or inputs** as a presence/absence vector over a vocabulary.

📌 **Example use cases:**

* Text classification where documents can belong to multiple categories.
* Representing a bag-of-words or bag-of-items (like your food order).
* Building inputs for simple neural nets without sequence models.

---

### ⚠️ In Tokenization:

Most modern tokenizers (like BPE or WordPiece) **don't use one-hot or multi-hot encodings directly**. Instead:

* Tokens are mapped to **integers (IDs)**.
* These IDs are then passed through **embedding layers**, which learn dense vector representations (instead of sparse one-hot vectors).

So:

* **One-hot/multi-hot** are mostly used in simpler, classical ML models.
* For modern NLP, use **token IDs + embedding layers** instead.
