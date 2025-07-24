---
title: "DDD - Bounded Context"
date: 2025-07-26
tags:
- DDD
comments: true
---

Let’s break down the **Bounded Context** concept in **Domain-Driven Design (DDD)** for someone who's just getting familiar with it.

---

### 📘 **Domain-Driven Design Solution Space**

```markdown
                          +---------------------+
                          |   Generic           |
                          |   Bounded Context   |
                          +---------------------+
                                     |
                                     v
                          +---------------------+
                          |   Core              |
                          |   Bounded Context   |
                          +---------------------+
                            ^       |       ^
                            |       v       |
        +----------------------+   +---------------------+   +---------------------+
        |  Supporting          |   |   Bounded Context   |   |  Legacy Context     |
        |  Bounded Context     |   +---------------------+   |  (Support Role)     |
        +----------------------+                            +---------------------+
            ^           ^                                          ^
            |           |                                          |
+-------------------+   |                                          |
|   Bounded Context |---+                                          |
+-------------------+                                              |
            |                                                      |
            v                                                      v
+-------------------+                                     +---------------------+
|   Supporting      |                                     |     Generic         |
|   Bounded Context |                                     |     Bounded Context |
+-------------------+                                     +---------------------+


     +------------------------+                   +------------------------------+
     |  Domain Knowledge      |                   |  Domain Experts + Dev Team   |
     +------------------------+                   +------------------------------+
                     |                                         |
                     v                                         v
       +----------------------------------+       +-----------------------------+
       |   Domain-Driven Design Space     | <---> |  Model-Driven Design        |
       +----------------------------------+       |  (Core/Complex via Models)  |
                                                  +-----------------------------+
```

This diagram presents the **solution space** of Domain-Driven Design, showing how different types of **bounded contexts** fit together within a full system and interact with other DDD elements.

---

### 🧩 Key Concepts Illustrated

* **Core Domain**
  This is where the **most business-critical logic** lives. It must be modeled with deep insight and precision—often gets the **most attention and modeling effort**.

* **Supporting Subdomains / Contexts**
  These help the core but are not themselves central. They follow simpler or less strategic models.

* **Generic Subdomains**
  Reusable, non-differentiating functionality (e.g., logging, identity, email) often handled with off-the-shelf solutions.

* **Legacy Context**
  Older systems that are still in play. DDD allows them to coexist within the broader model landscape.

* **Ubiquitous Language**
  Forms around each bounded context and is used consistently by domain experts and developers within it.

* **Domain Experts & Dev Teams**
  Collaborate to refine models and create shared understanding in each context.

* **Model-Driven Design**
  The practice of using rich models (especially in the core) to drive software behavior, structure, and interaction.

---

### 🔄 Flow of Insight

1. **Domain Knowledge** ➝ Shapes DDD efforts.
2. **Bounded Contexts** ➝ Segment models according to business reality.
3. **Model-Driven Design** ➝ Helps build expressive, purposeful implementations.
4. **Teams & Language** ➝ Collaborate in context using Ubiquitous Language.

---

### ✅ Summary

This diagram is a conceptual **map of the DDD landscape**, where each type of **bounded context** serves a purpose, and where **domain knowledge and team collaboration** drive the creation of effective, model-rich software.

### 🔹 What is a Bounded Context?

A **Bounded Context** is a boundary around a particular domain model. Inside that boundary, **everything has a specific, well-defined meaning**. Outside of it, the same words or concepts might mean something different.

Think of it as a **clear bubble** where a specific **ubiquitous language** is used consistently, and the rules and logic are tailored to a particular part of your business domain.

---

### 🔸 Why is it useful?

In real-world applications, large domains can be **complex and messy**. Different departments or teams might use the same terms but mean **entirely different things**. For example:

* "Order" in a **sales** context means a customer's purchase.
* "Order" in a **warehouse** context refers to a list of items to pick and ship.

A **Bounded Context** keeps these meanings and business logic **separate and clear**, reducing misunderstanding and bugs.

---

### 🔹 How does it work in practice?

* You **divide your system** into multiple bounded contexts.
* Each one has its **own model**, language, and logic.
* You define **how they interact** (e.g., through APIs, messages, or translation layers).
* The boundaries are **explicit**, not blurry.

---

### 🔸 Common Characteristics of a Bounded Context:

* One context per **microservice** (in many cases).
* Teams are often organized around them.
* Code, database schemas, and APIs don't cross the boundary.

---

### 🔹 Example

Let's say you’re building an **e-commerce system**:

1. **Customer Context**
   Deals with customer registration, profiles, and preferences.

2. **Ordering Context**
   Handles placing, updating, and canceling orders.

3. **Shipping Context**
   Manages deliveries, tracking, and logistics.

Each of these contexts can use the term "Order", but they **define and process it differently**. That's why **isolating them in Bounded Contexts** is crucial.

---

### Summary

* A Bounded Context is a **boundary** where a particular domain model is **valid and consistent**.
* It helps break down complexity by **separating concerns**.
* Enables teams to **work independently** and reduces coupling.
* It's essential for **scalability**, **clarity**, and **maintainability** in complex systems.


### 📦 E-commerce Application

```markdown
+-------------------+      +-------------------+      +-------------------+      +-------------------+
|  Shipping         |      |  Loyalty          |      |  Promotion        |      |  Allocation       |
|  Subdomain        |      |  Subdomain        |      |  Subdomain        |      |  Subdomain        |
|                   |      |                   |      |                   |      |                   |
| +---------------+ |      | +---------------+ |      | +---------------+ |      | +---------------+ |
| | Pricing       | |      | | Loyalty       | |      | | Promotion     | |      | | Allocation    | |
| | Bounded       | |      | | Bounded       | |      | | Bounded       | |      | | Bounded       | |
| | Context       | |      | | Context       | |      | | Context       | |      | | Context       | |
| |               | |      | |               | |      | |               | |      | |               | |
| | Presentation  | |      | | Presentation  | |      | | Presentation  | |      | | Presentation  | |
| | Domain Logic  | |      | | Domain Logic  | |      | | Domain Logic  | |      | | Domain Logic  | |
| | Persistence   | |      | | Product       | |      | | Product       | |      | | Product       | |
| +---------------+ |      | | Persistence   | |      | | Persistence   | |      | | Persistence   | |
|                   |      | +---------------+ |      | +---------------+ |      | +---------------+ |
| +---------------+ |                                   
| | Booking       | |
| | Bounded       | |
| | Context       | |
| |               | |
| | Presentation  | |
| | Domain Logic  | |
| | Persistence   | |
| +---------------+ |
+-------------------+      +-------------------+      +-------------------+      +-------------------+

        [Database]              [Database]               [Database]               [Database]
```

---

### 🧠 Notes:

* Each **bounded context** (e.g. `Pricing`, `Booking`, `Loyalty`, `Promotion`, `Allocation`) is encapsulated and contains its own:

  * **Presentation**
  * **Domain Logic**
  * **Persistence**
  * (Sometimes additional domain-specific layers like `Product`)
* Each **context has its own database**, reinforcing **independent evolution** and **data ownership**.
* The structure illustrates **layered architecture per bounded context**, **not** per entire application.

This diagram is a great illustration of **Bounded Contexts** in action within the scope of **Domain-Driven Design (DDD)**.

---

### 🔍 **Review in DDD Context**

In DDD, a **Bounded Context** defines the **logical boundary** within which a specific **domain model is valid**. This diagram exemplifies that idea by showing **separate models, logic, and persistence layers** for each distinct part of an e-commerce system.

---

### 🧩 **Key Highlights**

* **Multiple Bounded Contexts**
  Each subdomain—like **Shipping**, **Loyalty**, **Promotion**, and **Allocation**—contains one or more **Bounded Contexts** (e.g., *Pricing*, *Booking*, *Loyalty*).
  Each context handles its own interpretation of the domain independently.

* **Isolated Models & Layers**
  Inside each context, you see clear separation into:

  * `Presentation`
  * `Domain Logic`
  * `Persistence`
    This keeps models **cohesive and context-specific**, avoiding shared or ambiguous definitions across modules.

* **Independent Databases**
  Every context has its own database, reinforcing the **autonomy** of each boundary. This aligns with the DDD principle that contexts should **not share persistent models or storage**.

* **No Leaky Boundaries**
  The architecture encourages **communication between contexts** through **well-defined interfaces**, rather than sharing internal logic or databases.

---

### ✅ **Why This Fits DDD Well**

* It avoids the classic trap of a **"big ball of mud"** by separating logic clearly.
* Enables teams to **work independently** within their own context.
* Supports **scalability**, **flexibility**, and **clear language boundaries** across the system.

---

### 🧠 Takeaway

This layered-per-context design directly supports the DDD principle of **Bounded Context**, making complex domains manageable by breaking them down into focused, independent units—each with its **own model, rules, and language**.

Now lets try to understand how one class can be defined differently in two bounded contexts:

---

### 📦 E-commerce Application – Product Class in Different Contexts

```markdown
+-------------------+      +-------------------+      +-------------------+      +-------------------+
|  Shipping         |      |  Loyalty          |      |  Promotion        |      |  Allocation       |
|  Subdomain        |      |  Subdomain        |      |  Subdomain        |      |  Subdomain        |
|                   |      |                   |      |                   |      |                   |
|                   |      | +---------------+ |      | +---------------+ |      |                   |
|                   |      | | Loyalty       | |      | | Promotion     | |      |                   |
|                   |      | | Bounded       | |      | | Bounded       | |      |                   |
|                   |      | | Context       | |      | | Context       | |      |                   |
|                   |      | |               | |      | |               | |      |                   |
|                   |      | | Presentation  | |      | | Presentation  | |      |                   |
|                   |      | | Domain Logic  | |      | | Domain Logic  | |      |                   |
|                   |      | |  +--------+   | |      | |  +--------+   | |      |                   |
|                   |      | |  |Product |<-------------|  |Product |   | |      |                   |
|                   |      | |  +--------+   | |      | |  +--------+   | |      |                   |
|                   |      | | Persistence   | |      | | Persistence   | |      |                   |
|                   |      | +---------------+ |      | +---------------+ |      |                   |
+-------------------+      +-------------------+      +-------------------+      +-------------------+

[Database]              [Database]               [Database]               [Database]

```

### 🧾 Class Representations in Code (per context)

```csharp
// Loyalty Bounded Context
namespace Loyalty
{
    public class Product
    {
        public Points Earns() { }
    }
}

// Promotion Bounded Context
namespace Promotion
{
    public class Product
    {
        public void IncludeIn(Offer offer) { }
    }
}
```

---

We illustrated how the **`Product` class** is defined differently in two bounded contexts: `Loyalty` and `Promotion`.

---

### 🧠 Review in Context of Domain-Driven Design (DDD)

This diagram demonstrates one of the most **critical DDD insights**:

> 📌 **Same term, different meaning per Bounded Context.**

---

### 🔍 Key Takeaways

* Both `Loyalty` and `Promotion` contexts define a `Product` class.
* But:

  * In **Loyalty**, `Product` is about earning **points**.
  * In **Promotion**, `Product` relates to **offers and deals**.
* The name is the same, but the **behavior, properties, and meaning** are tailored to the **business needs** of each context.

---

### ✅ Why This Matters in DDD

* This avoids a **one-size-fits-all model**, which leads to **leaky abstractions**.
* Each team can evolve their model independently.
* It promotes clarity and autonomy—each context owns and defines its **ubiquitous language** precisely.