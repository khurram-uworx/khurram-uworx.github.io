---
title: "Data Pipelines in .NET: Beyond I/O Pipes"
date: 2025-09-15
tags:
- .NET
- Data Pipelines
- TPL

comments: true

---

In .NET, when dealing with data pipelines and task execution, you have several options beyond just using I/O pipes. Here are some key concepts and libraries that can help you implement and manage data pipelines effectively:

1. **Task Parallel Library (TPL)**:
   - The TPL is a set of public types and APIs for parallel programming. It provides a task-based programming model that helps you to write efficient, scalable, and easy-to-read code.
   - You can use `Task` and `Task<TResult>` to represent asynchronous operations.

2. **Dataflow Library (TPL Dataflow)**:
   - The TPL Dataflow Library provides a set of primitives for building data flow and pipeline components. It allows you to create networks of dataflow blocks that can buffer and process data in parallel.
   - Key components include `BufferBlock<T>`, `ActionBlock<T>`, `TransformBlock<TInput, TOutput>`, etc.

3. **Reactive Extensions (Rx)**:
   - Reactive Extensions (Rx) is a library for composing asynchronous and event-based programs using observable sequences and LINQ-style query operators.
   - It's useful for handling streams of data and events in a declarative way.

4. **Channels**:
   - Channels provide a way to transfer data between producers and consumers. They are useful in scenarios where you need to communicate between different parts of an application asynchronously.
   - Channels are part of the `System.Threading.Channels` namespace.

5. **Async/Await Pattern**:
   - The async/await pattern is fundamental for writing asynchronous code in .NET. It allows you to write code that performs asynchronous operations without blocking the main thread.

6. **Concurrent Collections**:
   - .NET provides several thread-safe collections like `ConcurrentDictionary`, `ConcurrentQueue`, and `BlockingCollection` that can be used to manage data in a multi-threaded environment.

7. **Message Queues and Brokers**:
   - For more complex scenarios, you might consider using message queues or brokers like RabbitMQ, Apache Kafka, or Azure Service Bus to handle data pipelines and ensure reliable message delivery.

8. **Custom Pipeline Implementations**:
   - You can implement custom pipelines using producer-consumer patterns, where producers generate data and consumers process it. This can be done using threads, tasks, or other concurrency primitives.

Each of these options has its own strengths and is suitable for different types of problems. The choice depends on the specific requirements of your application, such as the need for parallelism, scalability, and ease of use.

Sure, let's consider a common problem that can be addressed using different concurrency and parallelism techniques in .NET: processing a large collection of data items.

### Problem Statement

Suppose we have a large list of numbers, and we want to perform some computation on each number (e.g., calculating the square of each number) and then summing up all the results. This is a typical map-reduce problem where we can parallelize the computation to improve performance.

### Using Task Parallel Library (TPL)

The Task Parallel Library (TPL) is a set of public types and APIs for parallel programming. It provides a task-based programming model that helps you to write efficient, scalable, and easy-to-read code.

Here's how you might implement this problem using TPL:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Sample data: a large list of numbers
        List<int> numbers = Enumerable.Range(1, 1000000).ToList();

        // Process the numbers in parallel using TPL
        long sumOfSquares = CalculateSumOfSquares(numbers);

        Console.WriteLine($"The sum of squares is: {sumOfSquares}");
    }

    static long CalculateSumOfSquares(List<int> numbers)
    {
        // Use Parallel.ForEach to process the list in parallel
        object lockObject = new object();
        long sum = 0;

        Parallel.ForEach(numbers, () => 0L, (number, state, localSum) =>
        {
            return localSum + number * number;
        },
        localSum =>
        {
            lock (lockObject)
            {
                sum += localSum;
            }
        });

        return sum;
    }
}
```

### Explanation

1. **Parallel.ForEach**: This method is used to iterate over the list of numbers in parallel. It automatically partitions the data and processes it using multiple threads.

2. **Local State**: The `localSum` is used to accumulate the sum of squares for each partition locally. This helps in reducing contention when updating the shared `sum` variable.

3. **Locking**: A lock is used to safely update the shared `sum` variable from multiple threads.

4. **Efficiency**: By processing the data in parallel, we can significantly reduce the time it takes to compute the sum of squares, especially for large datasets.

This example demonstrates how TPL can be used to parallelize a computation-heavy task. In the next steps, we can explore how the same problem can be addressed using other options like TPL Dataflow, Reactive Extensions, etc. Would you like to proceed with another option?

You're right. Using a custom lock is not the most idiomatic way to handle shared state in TPL. Instead, we can use `Interlocked.Add` to safely update the shared `sum` variable without explicit locking. This approach is more efficient and leverages the capabilities of the TPL more effectively.

Here's how you can modify the previous example to use `Interlocked.Add`:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Sample data: a large list of numbers
        List<int> numbers = Enumerable.Range(1, 1000000).ToList();

        // Process the numbers in parallel using TPL
        long sumOfSquares = CalculateSumOfSquares(numbers);

        Console.WriteLine($"The sum of squares is: {sumOfSquares}");
    }

    static long CalculateSumOfSquares(List<int> numbers)
    {
        long sum = 0;

        Parallel.ForEach(numbers, () => 0L, (number, state, localSum) =>
        {
            return localSum + number * number;
        },
        localSum =>
        {
            Interlocked.Add(ref sum, localSum);
        });

        return sum;
    }
}
```

### Using TPL Dataflow

The **TPL Dataflow Library** allows you to compose **blocks** that represent steps in a pipeline. Each block handles a specific stage, and you can link them together to create **producer-consumer flows**.

Here’s how we might implement the same problem:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;

class Program
{
    static async Task Main()
    {
        // Sample data
        List<int> numbers = Enumerable.Range(1, 1000000).ToList();

        // Create blocks
        var transformBlock = new TransformBlock<int, long>(
            number => (long)number * number,
            new ExecutionDataflowBlockOptions
            {
                MaxDegreeOfParallelism = Environment.ProcessorCount
            });

        long sum = 0;

        var actionBlock = new ActionBlock<long>(
            square => { sum += square; },
            new ExecutionDataflowBlockOptions
            {
                MaxDegreeOfParallelism = 1 // single-threaded accumulation
            });

        // Link the blocks
        transformBlock.LinkTo(actionBlock, new DataflowLinkOptions { PropagateCompletion = true });

        // Post data into the pipeline
        foreach (var number in numbers)
        {
            await transformBlock.SendAsync(number);
        }

        transformBlock.Complete();

        // Wait until all work is done
        await actionBlock.Completion;

        Console.WriteLine($"The sum of squares is: {sum}");
    }
}
```

---

### Key Points

1. **Pipeline model**:

   * `TransformBlock<int, long>` handles the squaring of numbers in parallel.
   * `ActionBlock<long>` aggregates the results sequentially.

2. **Backpressure**:
   Dataflow blocks naturally support backpressure (via bounded capacities), preventing producers from overwhelming consumers.

3. **Explicitness**:
   Compared to TPL’s `Parallel.ForEach`, this approach makes the **flow of data** clearer and modular — great when you have multiple transformations or need to throttle, buffer, or broadcast data.

---

Perfect — let’s keep building the comparison. We’ve seen **TPL** (imperative, parallel loops) and **TPL Dataflow** (pipeline blocks). Now let’s reframe the **sum of squares** problem using **Reactive Extensions (Rx)**, which shifts us into a **streaming, declarative mindset**.

---

### Using Reactive Extensions (Rx)

Rx models computations as **observable sequences**. Instead of pushing data into blocks (like in Dataflow), you compose operators (`Select`, `Sum`, `Buffer`, etc.) to describe *what* should happen as the data flows through.

Here’s the same problem:

```csharp
using System;
using System.Linq;
using System.Reactive.Linq;

class Program
{
    static void Main()
    {
        // Sample data
        var numbers = Enumerable.Range(1, 1000000);

        // Create an observable sequence
        var numbersObservable = numbers.ToObservable();

        // Transform and reduce using LINQ-style operators
        var sumOfSquares = numbersObservable
            .Select(n => (long)n * n)
            .Sum()
            .Wait(); // blocks until the observable completes

        Console.WriteLine($"The sum of squares is: {sumOfSquares}");
    }
}
```

---

### Key Points

1. **Declarative pipeline**:

   * `.Select(n => (long)n * n)` is like a `map`.
   * `.Sum()` is a reduction.
   * The code looks like LINQ-to-Objects, but executes asynchronously on observables.

2. **Parallelism?**
   Out of the box, Rx processes events sequentially. To introduce parallelism, you can use `.ObserveOn(Scheduler.Default)` or partition the stream into windows/buffers and process them concurrently. For example:

   ```csharp
   var sumOfSquares = numbersObservable
       .Buffer(10000) // batch items
       .Select(batch => batch.AsParallel().Sum(n => (long)n * n)) // parallel per batch
       .Sum()
       .Wait();
   ```

   This leverages **PLINQ** inside Rx for parallel batch processing.

3. **Strength of Rx**:

   * Rx shines in **continuous or infinite streams** (e.g., UI events, sensor feeds, real-time logs).
   * It composes beautifully with operators like `Throttle`, `Merge`, `Retry`, etc.
   * For one-off “batch” jobs like sum-of-squares, it’s less ergonomic than TPL/Dataflow — but much more powerful for *live pipelines*.

---

✅ Now we’ve covered three flavors:

* **TPL** → great for parallelizing CPU-heavy batch operations.
* **TPL Dataflow** → ideal for modeling multi-stage producer/consumer pipelines.
* **Rx** → excels at event-driven, streaming scenarios with LINQ-like composability.

Lets **push this comparison further** by showing how the same problem looks with **Channels** (from `System.Threading.Channels`), which are closer to Go-style CSP pipelines.

Awesome — let’s complete the tour by looking at **Channels**.
Channels in .NET are lower-level than Dataflow or Rx: they’re essentially **asynchronous producer/consumer queues**, similar to Go’s goroutines + channels or Rust’s async mpsc.

We’ll stick with the same **sum of squares** problem, so we can compare apples-to-apples.

---

### Using Channels

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Channels;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        // Sample data
        List<int> numbers = Enumerable.Range(1, 1000000).ToList();

        // Create an unbounded channel
        var channel = Channel.CreateUnbounded<int>();

        // Producer task: push numbers into the channel
        var producer = Task.Run(async () =>
        {
            foreach (var number in numbers)
            {
                await channel.Writer.WriteAsync(number);
            }
            channel.Writer.Complete();
        });

        // Multiple consumer tasks: read and process numbers
        int consumerCount = Environment.ProcessorCount;
        var consumers = new Task<long>[consumerCount];

        for (int i = 0; i < consumerCount; i++)
        {
            consumers[i] = Task.Run(async () =>
            {
                long localSum = 0;
                await foreach (var number in channel.Reader.ReadAllAsync())
                {
                    localSum += (long)number * number;
                }
                return localSum;
            });
        }

        // Wait for all tasks
        await producer;
        long[] results = await Task.WhenAll(consumers);

        long total = results.Sum();
        Console.WriteLine($"The sum of squares is: {total}");
    }
}
```

---

### Key Points

1. **Producer/Consumer model**

   * The producer writes items (`numbers`) into the channel.
   * Multiple consumers pull from the channel concurrently, each maintaining a local sum.

2. **Parallelism**

   * The channel distributes work naturally across consumers.
   * Work-stealing ensures all consumers stay busy until the channel is drained.

3. **Low-level flexibility**

   * Channels give you the **building blocks** to design custom pipelines (with bounded/unbounded capacity, single or multiple readers/writers).
   * Unlike Dataflow, you don’t get built-in transforms or linkages — you compose them manually.

---

### Quick Recap of Our Journey

* **TPL (`Parallel.ForEach`)** → parallel batch jobs, minimal ceremony.
* **TPL Dataflow** → declarative *pipelines* with built-in blocks, backpressure support.
* **Reactive Extensions (Rx)** → declarative *streams* with LINQ-like operators, great for live/continuous data.
* **Channels** → lower-level async queues, good for custom producer/consumer patterns.
