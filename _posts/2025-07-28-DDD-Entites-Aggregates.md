---
title: "DDD - Entities and Aggregates"
date: 2025-07-28
tags:
- DDD
comments: true
---

In context of Domain Driven Design, Entities and Aggregate confuses lot of people. Let's unravel the knot:

---

### 🧩 **What Is an Entity in DDD?**
An Entity is an object defined by its **identity**, not just its attributes. Even if its properties change, it's still considered the same entity.

- 🔑 Has a unique ID (like a GUID or key)
- 🚶 Has a lifecycle and can evolve over time
- 🧠 Behaves like a real-world concept with meaningful operations

**Example**: A `Customer` with an ID of `12345` is still the same person whether they change their name or address.

---

### 🗂️ **What Is an Aggregate?**
An Aggregate is a **cluster of related entities** and value objects treated as a single unit for data changes.

- 🏛️ One entity inside the aggregate is the **Aggregate Root**
- 🔐 All access and modifications go through the root
- 🔄 Guarantees consistency and business rules within the aggregate boundary

**Example**: An `Order` aggregate might contain `OrderItems`, `ShippingAddress`, and `PaymentInfo`. The `Order` is the Aggregate Root—you don’t update `OrderItem` directly; you ask `Order` to do it.

---

### 📐 **Difference Between Entity and Aggregate**

| Concept        | Entity                            | Aggregate                                  |
|----------------|-----------------------------------|---------------------------------------------|
| Focus          | Identity of individual object     | Consistency and encapsulation of group      |
| Access         | Direct                           | Only via Aggregate Root                     |
| Lifecycle      | Independent                      | Managed by Aggregate Root                   |
| Scope          | Can exist standalone              | Defines a transaction boundary              |

---

### 🧭 **Relationship with Bounded Context**

A **Bounded Context** defines a clear boundary where specific models, rules, and language apply consistently. Within it:

- Aggregates organize **business logic**
- Entities live and act within the rules of that context
- Each Bounded Context may model the **same concept differently** (e.g., "Customer" in Accounting vs. CRM)

**Think of it like this**:  
> In DDD, the **Bounded Context is the city**,  
> an **Aggregate is a neighborhood**,  
> and an **Entity is a house** with its own unique address and personality.

So the design question shifts from “What is a customer?” to “How does the customer behave _within this context_?”

---

Now we’re stepping into **Bounded Context** implementation territory—the fascinating part where theory meets architecture. Let's walk through how `Customer` might be modeled differently in **CRM** and **Accounting** contexts using DDD principles.

---

## 🎯 Understanding the Two Bounded Contexts

| Context     | Purpose                               | Customer Role                                |
|-------------|----------------------------------------|-----------------------------------------------|
| CRM         | Relationship management, marketing     | Represents potential or active clients        |
| Accounting  | Financial tracking, invoicing          | Represents legal entities involved in transactions |

Even though both use the term “Customer,” the underlying responsibilities and rules are **totally different**.

---

## 🧱 Modeling `Customer` in CRM Context

```csharp
// CRM Bounded Context
public class Customer
{
    public Guid Id { get; private set; }
    public FullName Name { get; private set; }
    public Email Email { get; private set; }
    public CustomerStatus Status { get; private set; } // Lead, Prospect, Active
    public DateTime? LastContacted { get; private set; }

    public void UpdateStatus(CustomerStatus newStatus)
    {
        // Validate and apply status transitions
    }

    public void RecordContact(DateTime contactDate)
    {
        LastContacted = contactDate;
    }
}
```

💡 **Focus**: Interaction history, lifecycle, engagement status.

---

## 📦 Modeling `Customer` in Accounting Context

```csharp
// Accounting Bounded Context
public class Customer
{
    public Guid Id { get; private set; }
    public CompanyName Name { get; private set; }
    public BillingAddress Address { get; private set; }
    public TaxId TaxIdentifier { get; private set; }
    public IReadOnlyList<Invoice> Invoices => _invoices.AsReadOnly();

    private List<Invoice> _invoices = new();

    public void AddInvoice(Invoice invoice)
    {
        // Business rule: No duplicate invoice IDs, etc.
        _invoices.Add(invoice);
    }

    public decimal GetTotalOutstanding()
    {
        return _invoices.Where(i => !i.IsPaid).Sum(i => i.AmountDue);
    }
}
```

💡 **Focus**: Financial identity, payment tracking, regulatory data.

---

## 🔄 Translating Across Contexts

You usually don’t share these `Customer` models directly between contexts. Instead:

- Use **Context Maps** or **Anti-Corruption Layers** to translate and protect boundaries
- CRM might **notify** Accounting when a customer becomes billable
- Accounting may **return status** or warnings to CRM (e.g., suspended due to overdue)

Example interface between the two:

```csharp
public interface IAccountingCustomerGateway
{
    void CreateCustomerAccountingProfile(Guid crmCustomerId, CompanyName name, BillingAddress address);
}
```

---

## 🧠 Design Considerations

- Don’t try to unify these models—you’ll lose precision and introduce chaos.
- Keep terminology **ubiquitous** _within_ a context but allow differences _between_ them.
- Let the domain experts from each context shape what the `Customer` means.

---

Ah, now we’re digging into the orchestration dance between Bounded Contexts—one of the trickiest yet most crucial aspects of DDD in practice. Here's how to keep entity details like `Email` in sync **without violating autonomy or leaking domain concepts**.

---

## 🔄 The Problem at a Glance

The `Customer` exists in both **CRM** and **Accounting**, but with different purposes. When a shared detail like `Email` changes, you need to propagate that change **without coupling** the systems tightly.

---

## 🧩 Solution: Event-Driven Synchronization

This is where **Domain Events** and **Integration Events** shine:

### Step 1: CRM Raises an Event

```csharp
public record CustomerEmailChanged(Guid CustomerId, Email NewEmail);
```

When the customer updates their email in CRM, the system raises this event internally and publishes it externally.

### Step 2: Publish via Message Bus

Use a **Message Broker** (e.g., RabbitMQ, Azure Service Bus, Kafka) to push the event outside the CRM context.

### Step 3: Accounting Subscribes and Handles the Update

The Accounting context subscribes to relevant events and maps them to its own model:

```csharp
public class CustomerEmailChangedHandler
{
    public void Handle(CustomerEmailChanged message)
    {
        var customer = _repo.FindById(message.CustomerId);
        if (customer != null)
        {
            customer.UpdateEmail(message.NewEmail);
            _repo.Save(customer);
        }
    }
}
```

This avoids direct API calls or shared models and keeps the **bounded contexts loosely coupled**.

---

## 🧱 Anti-Corruption Layer (Optional)

If the CRM event uses a different format or terminology, you can insert an **ACL** that maps external events into Accounting’s own language:

```csharp
public class CrmToAccountingTranslator
{
    public CustomerEmailChanged Translate(CrmCustomerEmailUpdated crmEvent)
    {
        return new CustomerEmailChanged(crmEvent.Id, Email.Parse(crmEvent.EmailAddress));
    }
}
```

This layer ensures that context boundaries are respected and translation is controlled.

---

## 🛡️ Safety Considerations

- 🔐 **Authorization**: Only trusted sources should trigger updates.
- 📜 **Idempotency**: Ensure repeated events don’t corrupt the state.
- 🧪 **Event Versioning**: Future-proof changes to event formats.
- 🕒 **Eventual Consistency**: Accept brief windows of desync and design UX around it.

---

### Bonus Thought 🌐

For more complex scenarios—like syncing multiple fields or user preferences—you might consider a **Customer Profile Service** as a shared abstraction or intermediary, _but only if it doesn’t bleed domain logic across boundaries_.

---

A **shared integration event** acts as a signal, allowing each bounded context to interpret and apply it in their own domain language. Let’s break this down and flesh it out with clarity:

---

## 📣 **1. Emit a Shared Integration Event**

When the `Customer` changes their email in CRM, emit an integration event like this:

```csharp
public record CustomerEmailChanged(Guid CustomerId, string NewEmail, DateTime ChangedAt);
```

This event doesn’t carry CRM-specific semantics—just **neutral data** that other contexts can adapt.

---

## 🎧 **2. CRM Listener (Internal Domain Event → Integration Event)**

CRM updates its internal model and publishes externally:

```csharp
public class UpdateCustomerEmailCommandHandler
{
    public Task Handle(UpdateCustomerEmailCommand command)
    {
        var customer = _repo.FindById(command.CustomerId);
        customer.ChangeEmail(command.NewEmail); // domain logic
        _repo.Save(customer);

        _eventBus.Publish(new CustomerEmailChanged(customer.Id, customer.Email, DateTime.UtcNow));
    }
}
```

---

## 🧾 **3. Accounting Listener (Autonomous Interpretation)**

Accounting doesn’t blindly copy CRM's data. Instead, it processes the event per its own domain rules:

```csharp
public class CustomerEmailChangedHandler
{
    public async Task Handle(CustomerEmailChanged evt)
    {
        var customer = await _accountingRepo.FindById(evt.CustomerId);
        if (customer == null) return;

        // Maybe check if email was previously validated or invoice-related conditions apply
        customer.UpdateInvoiceEmail(evt.NewEmail); // Accounting's own model and logic
        await _accountingRepo.Save(customer);
    }
}
```

---

## ⚖️ **Why This Works Well**

- 🧭 Each context owns its domain logic and doesn’t share entities or repositories.
- 🔐 Promotes **loose coupling** while achieving **eventual consistency**.
- 🧠 Allows each context to **apply its own validation**, formatting, or business constraints.
- 🔁 Handles failures gracefully via retries, dead-letter queues, or compensating actions.

---

## 🛠️ Event Bus Choices

- **Lightweight**: MediatR + Message queues for internal/single app
- **Distributed**: Kafka, Azure Service Bus, RabbitMQ for microservices
- **Advanced**: Use outbox pattern with transactional boundaries if reliability is paramount

---

Event-driven architecture (EDA) is powerful—but only when wielded with care. Here’s a curated set of **best practices** to help you build robust, scalable, and maintainable systems:

---

### 🧱 **Design Principles**

- **Define clear event boundaries**  
  Events should represent meaningful business occurrences (e.g. `CustomerEmailChanged`), not low-level technical changes.

- **Use well-structured event schemas**  
  Include metadata like timestamps, source, and version. Prefer formats like JSON or Avro with schema registries for validation.

- **Favor immutability**  
  Events should be immutable. Once published, they’re historical facts—not to be changed or deleted.

- **Design for eventual consistency**  
  Accept that consumers may process events asynchronously. Build with retries, idempotency, and compensating actions in mind.

---

### 🔄 **Event Lifecycle Management**

- **Version your events**  
  Use schema evolution strategies (e.g. backward-compatible changes) and avoid breaking consumers.

- **Document events thoroughly**  
  Maintain an event catalog with descriptions, producers, consumers, and sample payloads.

- **Avoid over-publishing**  
  Don’t emit events for every minor change. Focus on domain-relevant transitions.

---

### 🧭 **Infrastructure & Delivery**

- **Choose the right broker**  
  Kafka, RabbitMQ, Azure Service Bus, etc.—pick based on latency, throughput, durability, and delivery guarantees.

- **Support multiple delivery semantics**  
  Understand and configure for `at-most-once`, `at-least-once`, or `exactly-once` depending on business needs.

- **Implement dead-letter queues**  
  Capture failed messages for diagnostics and recovery.

- **Centralize observability**  
  Use tools like Grafana, Splunk, or OpenTelemetry for tracing, logging, and metrics across producers and consumers.

---

### 🧠 **Organizational Practices**

- **Decouple teams via event contracts**  
  Producers and consumers should evolve independently. Use schema registries and ACLs to enforce boundaries.

- **Introduce a Cloud Center of Excellence (CCoE)**  
  Standardize non-functional requirements like security, reliability, and monitoring across teams.

- **Balance centralized vs decentralized ownership**  
  Centralized brokers offer consistency; decentralized ones promote agility. Choose based on team maturity and scale.

---

Would you like to explore real-world patterns like **event sourcing**, **CQRS**, or **event mesh** next? Or maybe dive into how to test and validate event flows across microservices?