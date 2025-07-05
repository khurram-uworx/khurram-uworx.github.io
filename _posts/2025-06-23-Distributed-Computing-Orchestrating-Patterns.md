---
title: "Distributed Computing Orchestrating Patterns"
date: 2025-06-23
tags:
- "Distributed Computing"
---


Patterns for orchestrating messaging and reliability in distributed systems have matured and gained formal recognition. Lets take two examples to understand their implementation using RabbitMQ a famous Message Broker.

---

### 1. **Delayed Retry Pattern**

**Formal Name:** **Delayed Retry / Scheduled Retry / Time-based Redelivery**

**Description:**
Instead of immediately retrying a failed message, it's placed in a delay queue (dead-letter queue with TTL and dead-letter exchange configured), and reprocessed after a specified time.

**Common Implementation (RabbitMQ):**

* Use a **delay queue** with a **message TTL** (Time-To-Live).
* Set a **Dead Letter Exchange (DLX)** so that messages are routed back to the main queue after the delay expires.

**Purpose:**

* Avoid overwhelming the consumer when a transient failure occurs.
* Give time for external systems to recover (e.g., database or HTTP service).

**Alternative RabbitMQ Features:**

* RabbitMQ’s **delayed message plugin** can also be used (more flexible than TTL + DLX pattern).

---

### 2. **Throttling or Load-leveling Pattern**

**Formal Name:** **Load Leveling / Buffer Queue / Message Throttling**

**Description:**
Messages are held temporarily in an intermediate (buffer) queue before being forwarded to the processing queue, effectively smoothing out spikes in traffic.

**Common Implementation (RabbitMQ):**

* Introduce a **buffer queue**.
* Consumer processes messages from the buffer at a fixed or rate-limited pace.
* Optionally use **message TTL** and **dead-lettering** to move messages forward.

**Purpose:**

* Protect downstream services from being overwhelmed.
* Handle traffic spikes gracefully without dropping messages.

**Alternative Techniques:**

* Use of **token bucket** or **leaky bucket** algorithms for rate control on the consumer side.
* Combine with delayed messages or message prioritization if needed.

---

### 3. **Dead Letter Queue (DLQ) Pattern**

**Formal Name:** **Dead Letter Queue / Dead Lettering**

**Description:**
Messages that cannot be processed (due to consumer rejection or TTL expiry) are routed to a separate queue for later inspection or reprocessing.

**Common Implementation (RabbitMQ):**

* Set up a **DLX (Dead Letter Exchange)** on the main queue.
* Configure **dead-letter routing key** and a **DLQ** to receive failed messages.

**Purpose:**

* Prevent message loss and identify issues causing repeated failures.
* Enable manual inspection or separate processing logic for bad messages.

**Alternative Techniques:**

* Use DLQs with retry counters or tagging for automated reprocessing.

---

### 4. **Circuit Breaker Pattern**

**Formal Name:** **Circuit Breaker / Fault Isolation**

**Description:**
Prevents a service from trying to access a failing dependency repeatedly by “opening the circuit” temporarily after a threshold of failures.

**Common Implementation (RabbitMQ):**

* Implement at the **consumer level** or in **middleware**.
* Stop consuming messages temporarily if failure rate crosses a threshold.
* Use a **cool-off timeout** before resuming processing.

**Purpose:**

* Avoid cascading failures and allow external services to recover.
* Improve system resilience by isolating faults.

**Alternative Techniques:**

* Combine with retry logic, DLQs, and monitoring systems.

---

### 5. **Compensating Transaction Pattern**

**Formal Name:** **Undo via Compensating Actions**

**Description:**
When a long-running or distributed operation fails partway, previous successful actions are explicitly reversed via compensating logic.

**Common Implementation (RabbitMQ):**

* Publish **compensating events** to a specific queue.
* Consumers of those events trigger actions to revert earlier changes.

**Purpose:**

* Provide consistency in distributed workflows without needing 2PC.
* Allow rollback logic when full transactionality isn't possible.

**Alternative Techniques:**

* Use with **Saga Pattern** to orchestrate compensation.

---

### 6. **Saga Pattern**

**Formal Name:** **Saga / Distributed Transaction Coordination**

**Description:**
Coordinates multiple distributed operations as a sequence of local transactions, each with a corresponding compensating action.

**Common Implementation (RabbitMQ):**

* **Choreography:** Services react to events and trigger next steps.
* **Orchestration:** A central orchestrator publishes events and handles compensations.

**Purpose:**

* Manage distributed, long-running transactions without locking or 2PC.
* Enable scalable, failure-tolerant business processes.

**Alternative Techniques:**

* Use message headers to carry correlation IDs for tracking transaction context.

---

### 7. **Message Filtering Pattern**

**Formal Name:** **Selective Consumer / Content-Based Filtering**

**Description:**
Only deliver relevant messages to specific consumers, reducing unnecessary processing.

**Common Implementation (RabbitMQ):**

* Use **headers exchange** or **topic exchange** with routing keys.
* Consumers bind with specific routing patterns or filter by headers.

**Purpose:**

* Improve efficiency by routing only relevant messages.
* Enable logical separation of consumers without needing multiple queues.

**Alternative Techniques:**

* Use header-based selectors or routing conventions with topic naming.

---

### 8. **Message Deduplication Pattern**

**Formal Name:** **Idempotent Consumer / Duplicate Detection**

**Description:**
Ensure the same message isn't processed more than once, especially in at-least-once delivery scenarios.

**Common Implementation (RabbitMQ):**

* Assign a **unique message ID**.
* Track processed IDs in a **cache** or **database**.
* Discard duplicates during processing.

**Purpose:**

* Prevent side effects from processing the same message multiple times.
* Ensure idempotency across retries or redeliveries.

**Alternative Techniques:**

* Use Redis or in-memory cache with TTL for tracking recent message IDs.

---

### 9. **Priority Queueing Pattern**

**Formal Name:** **Message Prioritization / Priority Queue**

**Description:**
Higher priority messages are processed before lower priority ones, regardless of arrival order.

**Common Implementation (RabbitMQ):**

* Declare a queue with **x-max-priority** argument.
* Publish messages with different **priority levels**.

**Purpose:**

* Ensure critical or time-sensitive messages are handled faster.
* Improve responsiveness for high-priority tasks.

**Alternative Techniques:**

* Use separate queues for different priorities with dedicated consumers.

---

### 10. **Request-Reply / RPC over Messaging Pattern**

**Formal Name:** **Request-Reply / RPC Messaging**

**Description:**
Emulates synchronous communication over asynchronous channels using correlation IDs and response queues.

**Common Implementation (RabbitMQ):**

* Producer sets **reply-to** queue and **correlation ID**.
* Consumer sends reply message with the same correlation ID.
* Producer waits on reply queue for response.

**Purpose:**

* Enable synchronous workflows over RabbitMQ.
* Allow request-response-style interactions in async systems.

**Alternative Techniques:**

* Use temporary queues for replies, or shared queues with filters.

---

### 11. **Fan-out / Publish-Subscribe Pattern**

**Formal Name:** **Pub/Sub / Broadcast Messaging**

**Description:**
Send a single message to multiple consumers simultaneously.

**Common Implementation (RabbitMQ):**

* Use a **fanout exchange**.
* Bind multiple queues to the exchange.
* All queues receive a copy of the message.

**Purpose:**

* Broadcast events to many services (e.g., audit, notification).
* Decouple publishers from subscribers.

**Alternative Techniques:**

* Use topic exchanges for more control or filtering.

---

### 12. **Content-Based Routing Pattern**

**Formal Name:** **Dynamic Routing / Conditional Routing**

**Description:**
Messages are routed based on their content or metadata.

**Common Implementation (RabbitMQ):**

* Use **headers exchange** or **topic exchange** with message metadata.
* Define complex bindings using header values or routing key patterns.

**Purpose:**

* Direct messages to the appropriate service based on business logic.
* Achieve flexible routing without hardcoding queues.

**Alternative Techniques:**

* Combine with filtering or enrichment patterns.

---

### 13. **Aggregator / Scatter-Gather Pattern**

**Formal Name:** **Scatter-Gather / Result Aggregator**

**Description:**
Distribute a request to multiple services (scatter), then aggregate their responses (gather).

**Common Implementation (RabbitMQ):**

* Send parallel messages to multiple queues/services.
* Use **correlation ID** and temporary queue to collect responses.
* Aggregator waits and combines results once all replies are received.

**Purpose:**

* Combine results from parallel processing units.
* Enable multi-service coordination and response shaping.

**Alternative Techniques:**

* Use a timeout and partial aggregation to tolerate missing responses.

---

### 14. **Store and Forward Pattern**

**Formal Name:** **Reliable Messaging / Message Persistence**

**Description:**
Messages are stored in persistent queues and retried later if immediate delivery fails.

**Common Implementation (RabbitMQ):**

* Use **durable queues** and **persistent messages**.
* If a consumer is unavailable, messages stay in queue until it returns.
* Combine with **retry logic** on failure.

**Purpose:**

* Ensure reliable delivery even during transient outages.
* Prevent message loss on consumer failure or restart.

**Alternative Techniques:**

* Use mirrored queues or persistent storage with acknowledgments.

---

### 15. **Outbox Pattern**

**Formal Name:** **Transactional Outbox / Reliable Event Publishing**

**Description:**
Ensure messages are reliably published by saving them to the database within the same transaction as business data changes.

**Common Implementation (RabbitMQ):**

* Store outgoing events in an **outbox table** in the database.
* A **background publisher** reads from the outbox and sends to RabbitMQ.
* Mark events as sent after successful delivery.

**Purpose:**

* Guarantee atomicity between database changes and message sending.
* Prevent message loss or duplication in distributed systems.

**Alternative Techniques:**

* Use CDC (Change Data Capture) tools like Debezium to publish from DB logs.

---
