---
title: "Reliable Timeout Handling"
date: 2025-07-09
tags:
- DDD
- "Distributed Computing"
comments: true
---

The recommended approach uses RabbitMQ's Dead Letter Exchange with TTL (Option 1) for reliable timeout handling, where expired messages automatically gets routed. Alternative solutions include delayed messages (if plugin available) or an external scheduler. Always include correlation IDs and ensure idempotency in timeout handling.

# Implementing Timeout-Based Signaling with RabbitMQ in C#

For your distributed system where Component B needs to complete work within a fixed time or trigger Component C, here are several robust approaches:

## Option 1: Dead Letter Queue with TTL (Recommended)

```csharp
// Component A - Publisher
var factory = new ConnectionFactory() { HostName = "localhost" };
using (var connection = factory.CreateConnection())
using (var channel = connection.CreateModel())
{
    // Main queue for Component B
    channel.QueueDeclare("work_queue", durable: true, exclusive: false, autoDelete: false);
    
    // DLX setup for timeout
    var args = new Dictionary<string, object> {
        { "x-dead-letter-exchange", "timeout_exchange" },
        { "x-message-ttl", 30000 } // 30 second timeout
    };
    channel.QueueDeclare("work_queue_with_ttl", durable: true, exclusive: false, autoDelete: false, arguments: args);
    
    // Publish message
    var body = Encoding.UTF8.GetBytes("transaction_data");
    channel.BasicPublish(exchange: "",
                        routingKey: "work_queue_with_ttl",
                        basicProperties: null,
                        body: body);
}

// Component B - Consumer
var consumer = new EventingBasicConsumer(channel);
consumer.Received += (model, ea) => {
    try {
        // Process the message
        // If successful:
        channel.BasicAck(ea.DeliveryTag, false);
    } catch {
        // If failed:
        channel.BasicNack(ea.DeliveryTag, false, true);
    }
};
channel.BasicConsume(queue: "work_queue", autoAck: false, consumer: consumer);

// Component C - Consumer of timeout messages
channel.ExchangeDeclare("timeout_exchange", ExchangeType.Fanout);
var timeoutQueue = channel.QueueDeclare().QueueName;
channel.QueueBind(queue: timeoutQueue, exchange: "timeout_exchange", routingKey: "");
var timeoutConsumer = new EventingBasicConsumer(channel);
timeoutConsumer.Received += (model, ea) => {
    // Handle timeout - notify Component C
    channel.BasicAck(ea.DeliveryTag, false);
};
channel.BasicConsume(queue: timeoutQueue, autoAck: false, consumer: timeoutConsumer);
```

## Option 2: Scheduled/Delayed Messages Plugin

If you have RabbitMQ's delayed message plugin installed:

```csharp
// Component A publishes to both queues
channel.ExchangeDeclare("delayed_exchange", "x-delayed-message", 
    durable: true, autoDelete: false, 
    arguments: new Dictionary<string, object> { { "x-delayed-type", "direct" } });

// Immediate processing queue
channel.BasicPublish("", "work_queue", null, messageBody);

// Delayed message for Component C
var headers = new Dictionary<string, object> { { "x-delay", 30000 } };
var props = channel.CreateBasicProperties();
props.Headers = headers;
channel.BasicPublish("delayed_exchange", "timeout_routing_key", props, messageBody);
```

## Option 3: External Scheduler Service

```csharp
// When Component A publishes:
var messageId = Guid.NewGuid().ToString();
var props = channel.CreateBasicProperties();
props.MessageId = messageId;
channel.BasicPublish("", "work_queue", props, messageBody);

// Schedule timeout check
_scheduler.Schedule(() => {
    if (!_completedTransactions.Contains(messageId))
    {
        channel.BasicPublish("", "component_c_queue", null, messageBody);
    }
}, TimeSpan.FromSeconds(30));
```

## Best Practice Recommendations:

1. **Use Option 1 (DLX with TTL)** for simplicity and reliability if you don't need exact timing
2. **Add idempotency** - Make sure Component C can handle duplicate timeout messages
3. **Include correlation IDs** - Pass a unique ID through all messages for tracking
4. **Consider sagas** - For complex workflows, look into the Saga pattern
5. **Monitor timeouts** - Track metrics on how often timeouts occur
