---
title: "Recommended Approaches for Idempotent Consumers"
date: 2025-07-09
tags:
- DDD
- "Distributed Computing"
comments: true
---

Deduplication is critical in messaging systems to ensure exactly-once processing and prevent duplicate side effects. Here are effective approaches based on different infrastructures:

## Redis-Based Deduplication

1. **Message ID Tracking**
   - Store message IDs with expiration: `SET message_id 1 EX 86400 NX`
   - Use Redis' atomic SETNX operation to check for duplicates

2. **Bloom Filters**
   - Use RedisBloom module for probabilistic deduplication
   - Space-efficient for high-volume systems with acceptable false positives

3. **Sorted Sets with Timestamps**
   - Store messages with scores as timestamps
   - Remove old entries with `ZREMRANGEBYSCORE` to prevent memory bloat

## RabbitMQ-Based Deduplication

1. **Message Deduplication Plugin**
   - Use RabbitMQ's `rabbitmq-message-deduplication` plugin
   - Works by examining message headers for dedupe keys

2. **Consumer-Side Deduplication**
   - Implement idempotent consumers that track processed message IDs
   - Use RabbitMQ's message headers to store unique identifiers

3. **Quorum Queues**
   - Provide exactly-once semantics through deduplication
   - Configure with `x-message-deduplication: true`

## Cloud-Based Solutions

**AWS SQS:**
- Enable content-based deduplication with `MessageDeduplicationId`
- Configure deduplication window (5 minutes to 24 hours)

**Azure Service Bus:**
- Use `MessageId` property for duplicate detection
- Configure duplicate detection history window (up to 7 days)

**Google Pub/Sub:**
- Implement client-side deduplication using message IDs
- Store processed message IDs in Cloud Memorystore (Redis) or Firestore

## Hybrid Approaches

1. **Two-Phase Deduplication**
   - First line: Fast in-memory check (Redis)
   - Second line: Persistent storage check (DB) for critical messages

2. **Windowed Deduplication**
   - Only deduplicate within a specific time window
   - Useful for systems where duplicates are only problematic for short periods

3. **Content-Based Hashing**
   - Generate hash of message content as deduplication key
   - Store hash in Redis/DB with TTL matching your deduplication window

## Implementation Considerations

- **Performance vs. Accuracy**: Choose between probabilistic (faster) and exact (slower) deduplication
- **TTL Management**: Set appropriate expiration times based on your message processing SLA
- **Scalability**: Ensure your deduplication storage can handle the message volume
- **Failure Handling**: Plan for deduplication storage failures (fail open or closed)

# Idempotent consumers in C#

Implementing idempotent consumers in C# requires careful design to ensure message processing remains consistent even when duplicates arrive. Here are the most effective approaches:

## 1. Database-Based Deduplication

**Using a Processed Messages Table:**
```csharp
public async Task<bool> IsMessageProcessed(Guid messageId)
{
    await using var connection = new SqlConnection(_connectionString);
    return await connection.ExecuteScalarAsync<bool>(
        "SELECT 1 FROM ProcessedMessages WHERE MessageId = @messageId",
        new { messageId });
}

public async Task MarkMessageAsProcessed(Guid messageId)
{
    await using var connection = new SqlConnection(_connectionString);
    await connection.ExecuteAsync(
        "INSERT INTO ProcessedMessages (MessageId, ProcessedAt) VALUES (@messageId, GETUTCDATE())",
        new { messageId });
}
```

**Optimized with Stored Procedure:**
```sql
CREATE PROCEDURE MarkMessageIfNotProcessed
    @MessageId UNIQUEIDENTIFIER
AS
BEGIN
    IF NOT EXISTS (SELECT 1 FROM ProcessedMessages WHERE MessageId = @MessageId)
    BEGIN
        INSERT INTO ProcessedMessages (MessageId, ProcessedAt)
        VALUES (@MessageId, GETUTCDATE())
        RETURN 1 -- Not processed before
    END
    RETURN 0 -- Already processed
END
```

## 2. Distributed Cache Approach (Redis)

**Using Redis with StackExchange.Redis:**
```csharp
public class RedisIdempotencyService
{
    private readonly IDatabase _redis;
    private readonly TimeSpan _expiration;
    
    public RedisIdempotencyService(IConnectionMultiplexer redis, TimeSpan expiration)
    {
        _redis = redis.GetDatabase();
        _expiration = expiration;
    }

    public async Task<bool> TryProcessMessage(string messageId)
    {
        return await _redis.StringSetAsync(
            $"msg:{messageId}", 
            "1", 
            _expiration, 
            When.NotExists);
    }
}
```

## 3. Framework-Specific Solutions

**MassTransit Approach:**
```csharp
public class OrderConsumer : IConsumer<OrderPlaced>
{
    public async Task Consume(ConsumeContext<OrderPlaced> context)
    {
        // MassTransit handles deduplication via MessageId by default
        await ProcessOrder(context.Message);
    }
}
```

**Azure Service Bus:**
```csharp
ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxAutoLockRenewalDuration = TimeSpan.FromMinutes(30)
});

processor.ProcessMessageAsync += async args =>
{
    try 
    {
        // Service Bus handles deduplication if enabled
        await ProcessMessage(args.Message);
        await args.CompleteMessageAsync(args.Message);
    }
    catch {}
};
```

## 4. Idempotency Middleware

**Generic Message Filter:**
```csharp
public class IdempotentConsumerFilter<T> : IFilter<ConsumeContext<T>>
    where T : class
{
    public async Task Send(ConsumeContext<T> context, IPipe<ConsumeContext<T>> next)
    {
        var messageId = context.MessageId.ToString();
        
        if (await IsDuplicate(messageId))
        {
            context.LogDuplicate();
            return;
        }

        await next.Send(context);
        await MarkAsProcessed(messageId);
    }
}
```

## 5. Optimistic Concurrency with ETags

**For RESTful Services:**
```csharp
[HttpPut("orders/{id}")]
public async Task<IActionResult> UpdateOrder(Guid id, [FromHeader(Name = "If-Match")] string eTag)
{
    var currentETag = await _repository.GetOrderETag(id);
    
    if (eTag != currentETag)
    {
        return Conflict("Order was modified by another request");
    }
    
    // Process update
}
```

## Key Implementation Considerations

1. **Storage Selection**:
   - For high throughput: Redis (in-memory)
   - For guaranteed persistence: SQL database
   - For cloud-native: Azure Cosmos DB or AWS DynamoDB

2. **Cleanup Strategy**:
   ```csharp
   // Periodic cleanup job
   public async Task CleanupProcessedMessages()
   {
       await using var connection = new SqlConnection(_connectionString);
       await connection.ExecuteAsync(
           "DELETE FROM ProcessedMessages WHERE ProcessedAt < @cutoff",
           new { cutoff = DateTime.UtcNow.AddDays(-7) });
   }
   ```

3. **Message Identification**:
   - Natural keys (order number, user ID + timestamp)
   - Generated GUIDs (from producer)
   - Content hashing (for identical payload detection)

4. **Error Handling**:
   ```csharp
   try
   {
       if (!await _idempotencyService.TryProcessMessage(messageId))
           return;
       
       await _businessService.Process(message);
   }
   catch
   {
       await _idempotencyService.RevertProcessing(messageId);
       throw;
   }
   ```

## Advanced Patterns

**Outbox Pattern Integration:**
```csharp
public async Task ProcessWithOutbox(Order order)
{
    await using var transaction = await _dbContext.Database.BeginTransactionAsync();
    
    try
    {
        // 1. Check idempotency
        if (await _dbContext.ProcessedMessages.AnyAsync(m => m.MessageId == order.Id))
            return;
            
        // 2. Process business logic
        _dbContext.Orders.Add(order);
        
        // 3. Record outbox message
        _dbContext.OutboxMessages.Add(new OutboxMessage(order));
        
        // 4. Mark as processed
        _dbContext.ProcessedMessages.Add(new ProcessedMessage(order.Id));
        
        await _dbContext.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```
