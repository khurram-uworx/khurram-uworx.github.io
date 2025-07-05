---
title: "DDD - Subdomains"
date: 2025-07-25
tags:
- DDD
comments: true
---

Subdomains are at the heart of Domain-Driven Design (DDD), and understanding them helps unlock the clarity and modularity that DDD brings to complex systems. Let’s break it down.

---

### 🧩 What Are Subdomains in DDD?

In DDD, a **subdomain** is a distinct area of the larger problem space that your software is addressing. Think of the entire business domain as a puzzle—each subdomain is a piece with its own rules, workflows, terminology, and challenges.

Subdomains are typically categorized into:

| Type of Subdomain     | Description                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| **Core Subdomain**    | Where your business's competitive advantage lies; requires deep modeling.   |
| **Supporting Subdomain** | Necessary for operations, but not central to differentiation. Often off-the-shelf solutions. |
| **Generic Subdomain** | Broadly applicable to many businesses; commodity functionality (e.g., logging, notifications). |

---

### 🧠 Why Subdomains Exist

They exist because *real-world domains are messy*, and splitting the domain into meaningful parts allows for:

- **Focused modeling**: Each subdomain can have its own language and logic.
- **Team specialization**: DDD encourages aligning teams to subdomains for deeper domain knowledge.
- **Architecture alignment**: Subdomains help inform boundaries of microservices, modules, and Bounded Contexts.
- **Complexity management**: Isolates changes and protects the core logic from ripple effects.

---

### 📦 How Subdomains Help in Design

- **Clear Separation of Concerns**: You avoid blending unrelated functionality (e.g., order fulfillment logic doesn't mix with payment rules).
- **Better Ubiquitous Language**: Each team speaks precisely within the scope they own.
- **Contextual Models**: You tailor models to suit the unique behaviors of each subdomain.
- **Strategic Design**: You can decide where to invest effort (e.g., custom-built vs third-party).

---

### 🛒 Example: *Sales Order Processing*

Imagine you're building a system for an e-commerce platform. Here’s how subdomains might break down:

| Subdomain               | Type              | Role in Sales Order Processing                                       |
|------------------------|-------------------|----------------------------------------------------------------------|
| **Order Management**   | Core              | Handles order lifecycles, cancellations, amendments.                |
| **Inventory Management** | Supporting      | Keeps track of stock availability and reservations.                 |
| **Shipping & Logistics** | Supporting      | Manages carriers, delivery timelines, and tracking.                 |
| **Tax Calculation**     | Generic           | Computes taxes based on rules—might use a third-party service.      |
| **Payment Gateway**     | Generic           | Processes credit card and digital payments—usually outsourced.      |

Each of these subdomains would ideally live in its own **Bounded Context**, ensuring that logic, language, and dependencies remain clean and maintainable.

---

If you’re modeling this system using DDD, you'd likely invest deeply in **Order Management** (your Core Subdomain), ensuring the domain experts and developers collaborate closely and that the model is rich and expressive.

---
In **Domain-Driven Design (DDD)**, **Subdomain** and **Bounded Context** are related but serve different purposes:

---

### **Subdomain**

* **What it is**: A logical part of the overall domain.
* **Purpose**: Helps model and understand the business domain by breaking it into smaller, manageable pieces.
* **Types**:

  * **Core Domain**: The heart of the business and competitive advantage.
  * **Supporting Subdomain**: Supports the core but is not unique to the business.
  * **Generic Subdomain**: Common solutions not specific to the business.

> Think of a subdomain as a **business concept**.

---

### **Bounded Context**

* **What it is**: A **design boundary** within which a model is defined and consistent.
* **Purpose**: Defines the **technical and linguistic boundary** where terms, logic, and rules apply consistently.
* **Encapsulates**: Models, code, database schema, language, and team understanding.

> Think of a bounded context as a **technical boundary** for implementing a model.

---

### Relationship

* A **subdomain** is a **problem space** concept.
* A **bounded context** is a **solution space** construct.
* One subdomain can map to **one or multiple bounded contexts**, depending on complexity and team organization.

---

**Example**:

* Subdomain: **Payments**
* Bounded Contexts:

  * **Invoice Management**
  * **Transaction Processing**
  * **Fraud Detection**

Each of these might deal with “Payments” but implement their logic and terminology differently.

---

Let's turn the concept of subdomains into something concrete, using **Sales Order Processing** as our working example. We’ll model one subdomain—**Order Management**, which we’ve identified as a **Core Subdomain**—into a well-defined **Bounded Context** and then sketch out how it improves design clarity.

---

### 🧠 Step 1: Define the Bounded Context — *Order Management*

This context owns the order lifecycle: creation, status updates, cancellation, etc. To isolate responsibilities, it doesn’t handle payments or shipping—it just *manages orders*.

---

### 📦 Core Concepts in Code

We'll express this in a clean, domain-first manner using aggregates, value objects, and domain events.

#### ✅ Aggregate Root: `Order`

```csharp
public class Order
{
    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }
    public List<OrderLine> Items { get; private set; }
    public DateTime CreatedAt { get; private set; }

    public void Cancel() {
        if (Status == OrderStatus.Shipped)
            throw new InvalidOperationException("Cannot cancel a shipped order.");
        Status = OrderStatus.Cancelled;
        // Raise domain event
    }

    public void AddItem(Product product, int quantity) {
        Items.Add(new OrderLine(product, quantity));
        // Possibly raise domain event
    }

    // Other logic (Amend, Confirm, etc.)
}
```

#### 🎯 Value Object: `OrderLine`

```csharp
public class OrderLine
{
    public Product Product { get; }
    public int Quantity { get; }

    public OrderLine(Product product, int quantity) {
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be greater than 0.");
        Product = product;
        Quantity = quantity;
    }
}
```

#### 📣 Domain Event (Optional)

```csharp
public class OrderCancelledEvent : IDomainEvent
{
    public Guid OrderId { get; }
    public DateTime CancelledAt { get; }

    public OrderCancelledEvent(Guid orderId) {
        OrderId = orderId;
        CancelledAt = DateTime.UtcNow;
    }
}
```

---

### 🚀 How This Helps

| Benefit                            | How it Shows Up in Code                                                      |
|------------------------------------|------------------------------------------------------------------------------|
| **Separation of Concerns**         | Shipping, inventory, and payments are not inside this class—pure focus.     |
| **Ubiquitous Language**            | "Order", "Cancel", "Amend" mirror real business terminology.                |
| **Clear Boundaries**              | Other subdomains interact via interfaces/events—not direct class access.    |
| **Expressive Modeling**            | Business rules (e.g. cancel restrictions) live inside domain logic.         |
| **Resilience to Change**           | If tax rules change, this model doesn’t break—it’s outside this context.    |

---

### 🎨 Sketching the Big Picture

Visually, you could imagine each subdomain as a hexagon in a **Context Map**:

```
+--------------------+     +---------------------+     +---------------------+
|   Order Management | --> | Inventory Management| <-- |  Payment Gateway    |
+--------------------+     +---------------------+     +---------------------+
         ^                       ^                             ^
         |                       |                             |
         v                       v                             v
     Domain Events         REST APIs / Messaging       External Services
```

The arrows represent communication channels—event-driven or via APIs—but each domain models its own world, **free of unnecessary coupling**.

---

Let’s explore how the **Shipping & Logistics** subdomain would be modeled within its own **Bounded Context**. This subdomain supports Order Management but has its own complexities—carrier selection, route planning, delivery windows, and status tracking.

---

### 📦 Shipping as a Bounded Context

Shipping handles everything from preparing a parcel to ensuring it's delivered. It doesn’t decide **what** is ordered or **how** it's paid for—just **how** it’s shipped.

---

### 🧠 Key Concepts

In this context, we’d typically model:

| Concept              | Role in Shipping                                                   |
|----------------------|---------------------------------------------------------------------|
| `Shipment`           | Aggregate representing the delivery journey                         |
| `Carrier`            | Entity or external system that actually performs the shipping       |
| `DeliveryWindow`     | Value object specifying expected delivery time range                |
| `ShippingStatus`     | Enum or value object tracking shipment progress                     |
| `ShippingLabel`      | Value object with address, barcode, carrier instructions            |
| `ShipmentRequestedEvent` | Domain event signaling shipment creation                        |

---

### 🧾 Sample Code Sketch

#### 🚚 Aggregate Root: `Shipment`

```csharp
public class Shipment
{
    public Guid Id { get; private set; }
    public Guid OrderId { get; private set; }
    public Address Destination { get; private set; }
    public DeliveryWindow DeliveryEstimate { get; private set; }
    public ShippingStatus Status { get; private set; }
    public Carrier AssignedCarrier { get; private set; }

    public Shipment(Guid orderId, Address destination, DeliveryWindow window, Carrier carrier) {
        Id = Guid.NewGuid();
        OrderId = orderId;
        Destination = destination;
        DeliveryEstimate = window;
        AssignedCarrier = carrier;
        Status = ShippingStatus.Pending;
        // Raise ShipmentRequestedEvent
    }

    public void MarkInTransit() => Status = ShippingStatus.InTransit;

    public void MarkDelivered() => Status = ShippingStatus.Delivered;
}
```

#### 📅 Value Object: `DeliveryWindow`

```csharp
public class DeliveryWindow
{
    public DateTime Earliest { get; }
    public DateTime Latest { get; }

    public DeliveryWindow(DateTime earliest, DateTime latest) {
        if (earliest >= latest)
            throw new ArgumentException("Earliest must be before latest.");
        Earliest = earliest;
        Latest = latest;
    }

    public bool IsWithinWindow(DateTime deliveryTime) =>
        deliveryTime >= Earliest && deliveryTime <= Latest;
}
```

---

### 🤖 How This Improves Design

| Benefit                          | In Practice                                                        |
|----------------------------------|---------------------------------------------------------------------|
| **Autonomy**                     | Shipment rules evolve independently of order logic.                |
| **Adaptability**                | Easier to swap carriers or integrate new tracking providers.       |
| **Decoupling via Events**       | Shipping reacts to `OrderConfirmedEvent`, not to Order directly.   |
| **Tailored Ubiquitous Language**| Terms like “in transit” and “carrier” are domain-specific.         |
| **Focused Validation & Logic**  | Shipping constraints (delivery windows, addresses) live here.      |

---

### ✨ Big Picture Communication

Shipping would typically be triggered by an event from the Order Management context:

```csharp
public class OrderConfirmedEvent : IDomainEvent
{
    public Guid OrderId { get; }
    public Address ShipTo { get; }
    // Other data needed for shipment
}
```

The Shipping context listens and reacts by creating a `Shipment`. This asynchronous, event-driven approach ensures **loose coupling** while preserving **business cohesion**.

---

Domain events are like signals of *significant business moments* within a bounded context, and when you treat them as first-class citizens, they become powerful building blocks for event-driven architecture (EDA). Let’s unwrap this step-by-step:

---

### 📣 What Are Domain Events, Really?

A **domain event** represents something meaningful that has happened in your domain—for example:  
> *An order was confirmed.*  
> *A shipment was dispatched.*  
> *An item went out of stock.*

Unlike technical events or logs, **domain events capture business semantics** and can trigger meaningful system reactions.

---

### 🔄 How Domain Events Promote Event-Driven Architecture

EDA thrives on loose coupling, responsiveness, and real-time collaboration between components. Domain events help build this by:

#### 1. **Decoupling Contexts**
- Each **Bounded Context** (e.g., Order Management, Shipping) publishes and listens for domain events.
- No direct class or method calls across contexts—just well-defined event contracts.

```plaintext
OrderConfirmedEvent --> ShippingContext reacts --> creates Shipment
```

#### 2. **Asynchronous Communication**
- Events can be published to a message bus or event stream.
- Subscribers react **when** and **if** needed, improving scalability and fault tolerance.

#### 3. **Scalable Reactions**
- Multiple consumers can respond to the same event without affecting each other:
  - Shipping creates a Shipment.
  - Notification service sends email.
  - Analytics updates sales charts.

#### 4. **Clear Intent + Traceability**
- Events act as audit records.
- You can trace what happened and why—ideal for debugging or analytics.

---

### 💡 Design Improvements via Domain Events

| Design Benefit                   | Explanation                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|
| **Loose Coupling**               | Contexts don't rely on each other's internals—changes are isolated.        |
| **High Cohesion**                | Each context owns its logic and emits events as a byproduct of its work.   |
| **Resilience to Change**         | You can add/remove event subscribers without changing the core domain.     |
| **Testability**                  | Events allow clear assertions in tests (e.g., "Was OrderConfirmedEvent raised?") |
| **Extendability**                | New features just "listen" for events—no need to refactor existing code.   |

---

### 🛠 Real-World Analogy: *Sales Order Processing*

Let’s revisit our example:

```csharp
public class Order
{
    public void Confirm() {
        Status = OrderStatus.Confirmed;
        DomainEvents.Add(new OrderConfirmedEvent(this.Id, this.CustomerId));
    }
}
```

Now, in **Shipping**:

```csharp
public class ShippingEventHandler
{
    public void Handle(OrderConfirmedEvent e) {
        var shipment = new Shipment(e.OrderId, /* address, etc. */);
        shipmentRepository.Save(shipment);
    }
}
```

🎯 No direct call from Order → Shipment. Instead, the event flows through a bus or event handler—making the system clean and extensible.

---

### ⚙️ Integrating with Infrastructure

You can wire this up using an event dispatcher internally, or integrate with message brokers (Kafka, RabbitMQ, Azure Service Bus) for true distributed systems.

> Domain events are emitted by your rich domain model  
> Infrastructure events route them to interested listeners  
> Each listener acts as an independent module in the business orchestra 🎻

---
