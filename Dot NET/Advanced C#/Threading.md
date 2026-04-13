### Introduction to Threading and Modern C# Concurrency

#### 1. Intuition First

Imagine you're a chef in a busy restaurant kitchen. You have multiple orders coming in simultaneously. If you tried to cook each dish from start to finish, one after another, your customers would be waiting an eternity.

Instead, a good chef works concurrently:
*   They might put a pot of water on to boil (a long-running task).
*   While the water is heating, they start chopping vegetables for another dish.
*   Once the water boils, they might add pasta, then immediately turn their attention back to sautéing the vegetables.
*   They might even delegate a task, like asking a prep cook to plate a finished dish.

In this analogy:
*   The **chef** is your CPU.
*   Each **task** (boiling water, chopping, sautéing) is a piece of work.
*   Working on multiple tasks seemingly at the same time is **concurrency**.
*   If you had *multiple chefs* working simultaneously on different dishes, that would be **parallelism**.

In computing, a **thread** is like one of these independent lines of execution within your program. By using multiple threads, your program can perform several tasks concurrently, making it more responsive and efficient, especially on multi-core processors.

#### 2. Core Theory

At its heart, a **thread** is the smallest unit of execution that an operating system can schedule. Every program (or **process**) starts with at least one thread, known as the **main thread**.

*   **Process**: An isolated execution environment. It owns its own memory space, file handles, and other resources. When you launch an application, you're launching a process.
*   **Thread**: A path of execution within a process. Threads within the same process share the process's memory space and resources. This shared memory is both a powerful feature (allowing easy data exchange) and a significant source of complexity (requiring careful synchronization).

**Concurrency vs. Parallelism**:
*   **Concurrency**: The ability to deal with many things at once. It's about structuring your program so that it can make progress on multiple tasks over a period of time, even if only one task is truly executing at any given instant (e.g., on a single-core CPU, tasks are rapidly switched).
*   **Parallelism**: The ability to execute many things at once. This requires multiple execution units (e.g., multiple CPU cores) to truly run different parts of a program simultaneously.

The **Common Language Runtime (CLR)**, which executes your C# code, manages a pool of threads called the **Thread Pool**. Creating new threads is an expensive operation. The thread pool mitigates this by maintaining a collection of pre-created, idle threads that can be reused for tasks. This is a key optimization for modern .NET applications.

When you offload work to another thread, the operating system's scheduler decides which thread runs on which CPU core and for how long. This rapid switching (context switching) gives the illusion of simultaneous execution, even on a single core. On multi-core systems, different threads can genuinely run in parallel on different cores.

#### 3. C# Implementation

Historically, C# offered the `System.Threading.Thread` [[#Lesson 2 Direct Thread Management and Thread Synchronization]] class for direct thread manipulation. While still available, it's generally discouraged for most application-level concurrency due to its low-level nature and the overhead of managing threads manually, as we have to wait the OS to create this thread for us.

The modern, idiomatic approach in C# for managing concurrency, especially for CPU-bound work, is to use the **Task Parallel Library (TPL)**, primarily through the `System.Threading.Tasks.Task` class. `Task` objects represent an asynchronous operation and are designed to leverage the CLR's thread pool efficiently.

Let's look at a simple example of offloading a CPU-bound calculation to a background thread using `Task.Run`.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class ThreadingExample
{
    public static void Main()
    {
        Console.WriteLine($"Main thread started. Thread ID: {Thread.CurrentThread.ManagedThreadId}");

        // Simulate a long-running, CPU-bound operation
        // Task.Run schedules the provided delegate to run on a thread pool thread.
        Task<long> calculationTask = Task.Run(() =>
        {
            Console.WriteLine($"\tCalculation thread started. Thread ID: {Thread.CurrentThread.ManagedThreadId}");
            long sum = 0;
            for (int i = 0; i < 1_000_000_000; i++) // A billion iterations to simulate work
            {
                sum += i;
            }
            Console.WriteLine($"\tCalculation thread finished. Thread ID: {Thread.CurrentThread.ManagedThreadId}");
            return sum; // The result of the calculation
        });

        // The main thread continues its work immediately without waiting.
        Console.WriteLine("Main thread continues doing other work...");

        // We can do other things here while the calculation is running in the background.
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Main thread doing quick task {i}.");
            Thread.Sleep(100); // Simulate some small work
        }

        // To get the result of the background task, we need to 'await' it or access its 'Result' property.
        // Accessing 'Result' on a non-completed task will block the current thread until the task finishes.
        // In a real application, you'd typically use 'await' for non-blocking waits (which we'll cover later).
        Console.WriteLine("Main thread waiting for calculation to complete...");
        long finalSum = calculationTask.Result; // This line blocks until calculationTask is done.

        Console.WriteLine($"Main thread received result: {finalSum}. Thread ID: {Thread.CurrentThread.ManagedThreadId}");
        Console.WriteLine("Main thread finished.");
    }
}
```

#### 4. Step-by-Step Walkthrough

Let's trace the execution of the `ThreadingExample` program:

1.  `Main` method starts on the **main thread**. Its ID is printed.
2.  `Task.Run(() => { ... })` is called.
    *   The lambda expression (the code inside `{ ... }`) is packaged as a work item.
    *   `Task.Run` requests a thread from the CLR's **thread pool**.
    *   A thread pool thread (let's call it **T1**) picks up this work item and starts executing the lambda.
    *   Crucially, `Task.Run` *returns immediately* to the main thread, providing a `Task<long>` object (`calculationTask`) that represents the ongoing background operation. The main thread does *not* wait for T1 to finish.
3.  The main thread prints "Main thread continues doing other work..."
4.  The main thread enters a `for` loop, printing "Main thread doing quick task X" and sleeping for 100ms in each iteration. This demonstrates that the main thread is responsive and performing other work while T1 is busy calculating.
5.  Meanwhile, **T1** (the thread pool thread) starts executing the long-running `for` loop to calculate the sum. It prints its own start and end messages, along with its thread ID.
6.  After the main thread finishes its `for` loop, it prints "Main thread waiting for calculation to complete..."
7.  `long finalSum = calculationTask.Result;` is executed.
    *   The `Result` property of a `Task` is a **blocking call**. If `calculationTask` has not yet completed, the main thread will *pause* its execution at this line and wait indefinitely until T1 finishes its calculation and sets the task's result.
    *   Once T1 completes its work and returns the `sum`, the `calculationTask` is marked as completed, and its `Result` property becomes available.
8.  The main thread retrieves `finalSum`, prints the result, and then prints "Main thread finished."

You'll observe that the thread IDs for the main thread and the calculation thread are different, confirming that the work was indeed offloaded. The output from the main thread's "quick tasks" will likely be interleaved with the calculation thread's messages, demonstrating concurrency.

#### 5. Common Mistakes

1.  **Blocking the UI Thread (in GUI applications)**: A common mistake is performing long-running operations directly on the main thread (which is often the UI thread in desktop applications). This makes the application unresponsive, appearing "frozen." While `Task.Run` helps, directly accessing `Task.Result` on the UI thread will *still* block it.
```csharp
// BAD: In a UI application, this would freeze the UI until calculation is done.
// Even though calculationTask runs on a thread pool thread, accessing .Result blocks the UI thread.
long result = calculationTask.Result;
```
*Correction*: Use `await` (which we'll cover in the next lesson) to non-blockingly wait for the task.

2.  **Not Waiting for Background Tasks**: If your main application exits before background tasks complete, those tasks might be abruptly terminated or not run at all, because the parent thread terminate all the child threads before it terminated.
```csharp
// Potentially BAD: If Main exits before calculationTask finishes, the calculation might be lost.
Task.Run(() => { /* long running work */ });
// ... Main method ends here without waiting ...
```
*Correction*: Ensure you wait for critical background tasks to complete before the application exits, using `Task.Wait()`, `Task.Result`, or `await Task.WhenAll()`.

3.  **Assuming Immediate Execution**: `Task.Run` schedules work on the thread pool. There's no guarantee that the work will start immediately; it depends on thread pool availability and scheduling.
```csharp
// Don't assume this will run instantly. It's scheduled.
Task.Run(() => Console.WriteLine("I might run a bit later."));
```

4.  **Race Conditions (without synchronization)** [[More About Threading `async` and `await`]]: This is a more advanced mistake we'll delve into later, but it's crucial to be aware of. When multiple threads access and modify shared data without proper coordination, the outcome can be unpredictable and incorrect.
```csharp
int counter = 0;
// BAD: If two threads increment 'counter' simultaneously, you can lose updates.
Task.Run(() => { for (int i = 0; i < 1000000; i++) counter++; });
Task.Run(() => { for (int i = 0; i < 1000000; i++) counter++; });
// The final value of counter will likely not be 2,000,000.
```
*Correction*: Use synchronization primitives like `lock`, `Monitor`, `SemaphoreSlim`, or `Interlocked` operations.

#### 6. Advanced Insights

*   **Thread Pool Efficiency**: The CLR's thread pool is highly optimized. It dynamically adjusts the number of threads based on workload, minimizing the overhead of creating and destroying threads. For most general-purpose background work, especially CPU-bound tasks, `Task.Run` (which uses the thread pool) is the preferred mechanism.
*   **CPU-Bound vs. I/O-Bound**:
    *   **CPU-Bound**: Tasks that spend most of their time performing computations (e.g., complex calculations, image processing, data compression). These benefit from running on separate threads to utilize multiple CPU cores. `Task.Run` is excellent for these.
    *   **I/O-Bound**: Tasks that spend most of their time waiting for an external resource (e.g., reading from a file, making a network request, querying a database). While you *can* use `Task.Run` for these, the more efficient and modern approach is `async/await` with I/O-bound operations that are inherently asynchronous (e.g., `HttpClient.GetAsync`, `StreamReader.ReadAsync`). This allows the thread to be returned to the thread pool while waiting, rather than blocking it. We will cover `async/await` [[More About Threading `async` and `await`]] in detail in a future lesson.
*   **Overhead of Threading**: While beneficial, threading isn't free. There's overhead associated with:
    *   **Thread creation**: Though mitigated by the thread pool, it's still a cost.
    *   **Context switching**: The OS switching between threads incurs a performance penalty.
    *   **Synchronization**: Protecting shared data adds complexity and can introduce contention, potentially negating performance gains if not managed carefully.
*   **Stack vs. Heap**: Each thread has its own call stack for local variables and method calls. However, all threads within a process share the same heap memory. This shared heap is where objects are allocated, making careful synchronization essential when multiple threads interact with the same objects.

#### 7. Practice Problems

1.  **Simple Background Calculation**:
    Create a console application. In the `Main` method, start a `Task.Run` that calculates the factorial of a large number (e.g., 20). While the factorial is being calculated in the background, the main thread should print a "Loading..." message every 200ms for 5 seconds. Finally, the main thread should wait for the factorial task to complete and print the result.
    *Hint: Factorial of 20 is 2,432,902,008,176,640,000. Use `long`.*

2.  **Simulating Multiple Concurrent Operations**:
    Modify the previous example. Instead of one factorial, start three separate `Task.Run` operations, each calculating the factorial of a different number (e.g., 15, 18, 20). The main thread should continue printing "Processing..." messages. After all three tasks are initiated, the main thread should wait for *all* of them to complete and then print all three results.
    *Hint: Look into `Task.WhenAll()` for waiting on multiple tasks.*

3.  **Introducing a Delay**:
    Extend problem 2. Make each factorial calculation include a `Thread.Sleep()` call *inside* the task's lambda, but *before* the calculation starts. For example, the factorial of 15 waits 3 seconds, 18 waits 1 second, and 20 waits 5 seconds. Observe how the main thread remains responsive and how the results are printed once each task completes.

#### 8. Connection to Bigger Picture

Threading is the bedrock of **concurrency** and **parallelism** in modern applications. Understanding it is essential for:

*   **Responsive User Interfaces**: Preventing UI freezes by offloading long-running operations.
*   **Scalability**: Utilizing multiple CPU cores to process more work simultaneously, improving throughput.
*   **Performance**: Speeding up CPU-bound computations by distributing them across cores.
*   **Asynchronous Programming**: Threading concepts are foundational to understanding `async/await`, which is C#'s primary mechanism for efficient I/O-bound concurrency.
*   **Distributed Systems**: While threads are within a single process, the principles of managing concurrent operations and shared state extend to distributed systems where multiple processes or machines need to coordinate.

---

This concludes our first lesson on Threading. You now have a foundational understanding of what threads are, why they're important, and how to initiate basic concurrent operations using `Task.Run` in modern C#.

For our next lesson, we will dive deeper into **`async` and `await`**, which provide a much more elegant and efficient way to handle asynchronous operations, especially I/O-bound ones, without explicitly managing threads. This will build directly on our understanding of `Task` and the thread pool.

Do you have any initial questions about the concepts we've covered today before you tackle the practice problems?


### Lesson 2: Direct Thread Management and Thread Synchronization

#### 1. Intuition First

Let's return to our busy restaurant kitchen.

**Scenario 1: Direct Thread Management (`Thread` class)**
Imagine you're the head chef, and you need a very specific, long-running, isolated task done – perhaps preparing a complex, multi-stage dessert that requires its own dedicated workstation and focus, separate from the main line. Instead of pulling a general-purpose prep cook from the pool (like `Task.Run` would do), you specifically hire a "Dessert Specialist" chef. This specialist has their own dedicated space, and you manage their entire lifecycle: when they start, when they finish, and what they do. This is akin to creating a `new Thread()`. It's powerful but requires more direct management.

**Scenario 2: Thread Synchronization (Race Conditions and Locks)**
Now, imagine you have multiple chefs (threads) working in the kitchen. There's a single, shared ingredient pantry (a shared resource, like a `List<T>` or an `int` counter).

*   **The Problem (Race Condition)**: If two chefs try to grab the last bag of flour *at the exact same time* without coordination, one might think they got it, but the other might also grab it, leading to confusion, or worse, both thinking they have it when only one actually does. Or, if two chefs are updating the "flour remaining" count, they might overwrite each other's updates, leading to an incorrect inventory. This is a **race condition**: the outcome depends on the unpredictable timing of multiple threads.

*   **The Solution (Locks/Semaphores)**: To prevent this, you establish rules:
    *   **`lock` (or `Monitor`)**: "Only one chef can be in the pantry at a time." When a chef enters, they lock the door. No other chef can enter until the first one leaves and unlocks it. This ensures **mutual exclusion** for that critical section.
    *   **`SemaphoreSlim`**: "Only three chefs can use the deep fryers at a time." You have a limited number of deep fryers. Chefs wait in line if all fryers are busy, and when one becomes free, the next chef in line can use it. This limits **concurrent access** to a resource.
    *   **`Interlocked`**: "When updating the 'number of completed orders' display, just use the special 'atomic increment' button." This button guarantees that even if two chefs press it simultaneously, the display will correctly increment by two, not just one. It's for very specific, simple, atomic operations.

#### 2. Core Theory

##### The `System.Threading.Thread` Class

The `System.Threading.Thread` class allows you to create and manage threads directly. When you create a `new Thread()`, you're asking the operating system to allocate a new execution path. This thread starts with its own call stack and can execute code independently.

**Key characteristics of `Thread`:**
*   **Manual Management**: You are responsible for starting, joining (waiting for it to finish), and potentially aborting (though `Abort` is deprecated and dangerous) the thread.
*   **Dedicated Thread**: Unlike `Task.Run` which uses the thread pool, `new Thread()` typically creates a new OS thread. This can be more resource-intensive if you create many short-lived threads.
*   **Background vs. Foreground**: By default, threads created with `new Thread()` are foreground threads. This means the application (process) will not terminate until all foreground threads have completed. You can set `IsBackground = true` to make it a background thread, which will be terminated automatically when all foreground threads (including the main thread) exit.

While `Thread` offers fine-grained control, `Task` (especially with `Task.Run`) is generally preferred for most application-level concurrency because it leverages the thread pool, reducing overhead and simplifying management. `Thread` is typically reserved for very specific scenarios, such as creating a long-running, dedicated background service thread that needs to be explicitly managed and kept alive.

##### Race Conditions and Critical Sections

A **race condition** occurs when the correctness of a program depends on the relative timing or interleaving of multiple threads. This typically happens when multiple threads try to access and modify shared data concurrently.

A **critical section** is a segment of code that accesses shared resources (data structures, variables, files, etc.) and must not be executed by more than one thread at a time. The goal of synchronization primitives is to protect these critical sections.

##### Synchronization Primitives

1.  **`lock` Keyword (Mutual Exclusion)**
    The `lock` keyword in C# is syntactic sugar for `System.Threading.Monitor`. It ensures that only one thread can execute a specific block of code at a time. It achieves **mutual exclusion**.

    *   **Mechanism**: When a thread encounters a `lock` statement, it attempts to acquire an exclusive lock on the provided object. If the lock is available, the thread acquires it and enters the protected code block. If the lock is already held by another thread, the current thread blocks (waits) until the lock is released. When the thread exits the `lock` block (either normally or via an exception), the lock is automatically released.
    *   **Lock Object**: The object used in the `lock` statement (`lock (myLockObject)`) must be a reference type (e.g., `object`, `string`, an instance of a class). It serves as the "key" for the lock. It's crucial to lock on a private, static, readonly object to prevent other code from accidentally locking on the same object, which could lead to deadlocks or unexpected behavior.

2.  **`System.Threading.Monitor` Class (Explicit Mutual Exclusion and Signaling)**
    `Monitor` provides the underlying functionality for the `lock` keyword. It offers more advanced features like `Wait`, `Pulse`, and `PulseAll` for inter-thread communication (signaling), which are essential for implementing patterns like producer-consumer.

    *   `Monitor.Enter(object obj)`: Acquires an exclusive lock on `obj`.
    *   `Monitor.Exit(object obj)`: Releases the exclusive lock on `obj`.
    *   `Monitor.Wait(object obj)`: Releases the lock on `obj` and blocks the current thread until another thread calls `Pulse` or `PulseAll` on `obj`. When the thread wakes up, it re-acquires the lock.
    *   `Monitor.Pulse(object obj)`: Notifies a single waiting thread that the state of `obj` has changed.
    *   `Monitor.PulseAll(object obj)`: Notifies all waiting threads.

3.  **`System.Threading.SemaphoreSlim` (Resource Limiting)**
    A `Semaphore` (or `SemaphoreSlim` for in-process use) is a synchronization primitive that limits the number of threads that can concurrently access a resource or a pool of resources. Unlike `lock` which allows only one, a semaphore allows `N` threads.

    *   **Mechanism**: A semaphore maintains a count. Threads call `Wait()` (or `WaitAsync()`) to decrement the count and acquire access. If the count is zero, threads block until another thread calls `Release()` to increment the count.
    *   **`Semaphore` vs. `SemaphoreSlim`**: `Semaphore` is a system-wide synchronization primitive, meaning it can be used to synchronize threads across different processes. `SemaphoreSlim` is a lighter-weight, in-process version, generally preferred for synchronizing threads within the same application.

4.  **`System.Threading.Interlocked` Class (Atomic Operations)**
    The `Interlocked` class provides atomic operations for variables that are shared by multiple threads. An operation is **atomic** if it appears to occur instantaneously and indivisibly from the perspective of other threads. This means it cannot be interrupted in the middle of its execution.

    *   **Mechanism**: `Interlocked` methods (like `Increment`, `Decrement`, `Add`, `Exchange`, `CompareExchange`) use special CPU instructions to perform operations without requiring a full lock. This is highly efficient for simple operations on primitive types (like `int`, `long`).
    *   **Use Case**: Ideal for simple counters or flags where you need to guarantee that an update is not lost due to concurrent access.

#### 3. C# Implementation

Let's demonstrate these concepts with code.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks; // We'll still use Task.Run for convenience in starting threads

public class SynchronizationExample
{
    // Shared resource that will be subject to race conditions
    private static int _sharedCounter = 0;

    // Lock object for mutual exclusion
    private static readonly object _lockObject = new object();

    // SemaphoreSlim to limit concurrent access to a resource
    private static SemaphoreSlim _resourceSemaphore = new SemaphoreSlim(initialCount: 3); // Allows 3 threads concurrently

    // Shared list to demonstrate Monitor.Wait/Pulse (Producer-Consumer)
    private static readonly Queue<int> _sharedQueue = new Queue<int>();
    private static readonly object _queueLock = new object();
    private const int MaxQueueSize = 5;

    public static void Main()
    {
        Console.WriteLine($"Main thread ID: {Thread.CurrentThread.ManagedThreadId}\n");

        // --- Part 1: Demonstrating Race Condition ---
        Console.WriteLine("--- Part 1: Race Condition (without lock) ---");
        _sharedCounter = 0; // Reset counter
        List<Thread> threads = new List<Thread>();
        for (int i = 0; i < 5; i++)
        {
            // Using the Thread class directly
            Thread t = new Thread(IncrementCounterWithoutLock);
            t.Name = $"Thread_NoLock_{i}";
            threads.Add(t);
            t.Start();
        }

        foreach (Thread t in threads)
        {
            t.Join(); // Wait for all threads to finish
        }
        Console.WriteLine($"Final counter value (without lock): {_sharedCounter} (Expected: 5000000)\n");
        // Expected value is 5 * 1,000,000 = 5,000,000.
        // You will almost certainly see a value less than this due to race conditions.

        // --- Part 2: Fixing Race Condition with 'lock' ---
        Console.WriteLine("--- Part 2: Fixing Race Condition with 'lock' ---");
        _sharedCounter = 0; // Reset counter
        threads.Clear();
        for (int i = 0; i < 5; i++)
        {
            Thread t = new Thread(IncrementCounterWithLock);
            t.Name = $"Thread_WithLock_{i}";
            threads.Add(t);
            t.Start();
        }

        foreach (Thread t in threads)
        {
            t.Join();
        }
        Console.WriteLine($"Final counter value (with lock): {_sharedCounter} (Expected: 5000000)\n");
        // This time, the value should be correct.

        // --- Part 3: Using Interlocked for Atomic Operations ---
        Console.WriteLine("--- Part 3: Using Interlocked for Atomic Operations ---");
        _sharedCounter = 0; // Reset counter
        threads.Clear();
        for (int i = 0; i < 5; i++)
        {
            Thread t = new Thread(IncrementCounterWithInterlocked);
            t.Name = $"Thread_Interlocked_{i}";
            threads.Add(t);
            t.Start();
        }

        foreach (Thread t in threads)
        {
            t.Join();
        }
        Console.WriteLine($"Final counter value (with Interlocked): {_sharedCounter} (Expected: 5000000)\n");
        // This will also be correct and often more performant for simple increments.

        // --- Part 4: Using SemaphoreSlim for Resource Limiting ---
        Console.WriteLine("--- Part 4: Using SemaphoreSlim for Resource Limiting ---");
        // We'll use Task.Run here as it's more common for this pattern
        List<Task> tasks = new List<Task>();
        for (int i = 0; i < 10; i++) // 10 tasks, but only 3 can run concurrently
        {
            int taskId = i; // Capture loop variable
            tasks.Add(Task.Run(() => AccessLimitedResource(taskId)));
        }
        Task.WhenAll(tasks).Wait(); // Wait for all tasks to complete
        Console.WriteLine("All tasks finished accessing limited resource.\n");

        // --- Part 5: Producer-Consumer with Monitor.Wait/Pulse ---
        Console.WriteLine("--- Part 5: Producer-Consumer with Monitor.Wait/Pulse ---");
        Task producerTask = Task.Run(ProduceItems);
        Task consumerTask1 = Task.Run(() => ConsumeItems("Consumer 1"));
        Task consumerTask2 = Task.Run(() => ConsumeItems("Consumer 2"));

        Task.WhenAll(producerTask, consumerTask1, consumerTask2).Wait();
        Console.WriteLine("Producer-Consumer example finished.\n");
    }

    // Method to be executed by threads without synchronization
    static void IncrementCounterWithoutLock()
    {
        Console.WriteLine($"\t{Thread.CurrentThread.Name} started (no lock).");
        for (int i = 0; i < 1_000_000; i++)
        {
            _sharedCounter++; // This is a read, modify, write operation, not atomic
        }
        Console.WriteLine($"\t{Thread.CurrentThread.Name} finished (no lock). Counter: {_sharedCounter}");
    }

    // Method to be executed by threads with 'lock' synchronization
    static void IncrementCounterWithLock()
    {
        Console.WriteLine($"\t{Thread.CurrentThread.Name} started (with lock).");
        for (int i = 0; i < 1_000_000; i++)
        {
            // Acquire lock on _lockObject before entering critical section
            lock (_lockObject)
            {
                _sharedCounter++; // Critical section: only one thread can execute this at a time
            }
            // Lock is released automatically when exiting the 'lock' block
        }
        Console.WriteLine($"\t{Thread.CurrentThread.Name} finished (with lock). Counter: {_sharedCounter}");
    }

    // Method to be executed by threads with 'Interlocked' synchronization
    static void IncrementCounterWithInterlocked()
    {
        Console.WriteLine($"\t{Thread.CurrentThread.Name} started (with Interlocked).");
        for (int i = 0; i < 1_000_000; i++)
        {
            // Atomically increments _sharedCounter
            Interlocked.Increment(ref _sharedCounter);
        }
        Console.WriteLine($"\t{Thread.CurrentThread.Name} finished (with Interlocked). Counter: {_sharedCounter}");
    }

    // Method to simulate accessing a resource with limited concurrency
    static void AccessLimitedResource(int id)
    {
        Console.WriteLine($"\tTask {id} waiting to access resource...");
        _resourceSemaphore.Wait(); // Acquire a slot in the semaphore
        try
        {
            Console.WriteLine($"\tTask {id} acquired resource. Working for 1 second...");
            Thread.Sleep(1000); // Simulate work
            Console.WriteLine($"\tTask {id} finished working with resource.");
        }
        finally
        {
            _resourceSemaphore.Release(); // Release the slot, allowing another thread to enter
        }
    }

    // Producer method for Monitor.Wait/Pulse example
    static void ProduceItems()
    {
        for (int i = 0; i < 10; i++)
        {
            lock (_queueLock)
            {
                // Wait if the queue is full
                while (_sharedQueue.Count == MaxQueueSize)
                {
                    Console.WriteLine($"\tProducer: Queue full, waiting... (Count: {_sharedQueue.Count})");
                    Monitor.Wait(_queueLock); // Release lock and wait
                }

                _sharedQueue.Enqueue(i);
                Console.WriteLine($"\tProducer: Produced item {i}. Queue size: {_sharedQueue.Count}");
                Monitor.PulseAll(_queueLock); // Notify waiting consumers
            }
            Thread.Sleep(200); // Simulate time to produce next item
        }
        Console.WriteLine("Producer finished producing items.");
    }

    // Consumer method for Monitor.Wait/Pulse example
    static void ConsumeItems(string consumerName)
    {
        for (int i = 0; i < 5; i++) // Each consumer tries to consume 5 items
        {
            int item;
            lock (_queueLock)
            {
                // Wait if the queue is empty
                while (_sharedQueue.Count == 0)
                {
                    Console.WriteLine($"\t{consumerName}: Queue empty, waiting... (Count: {_sharedQueue.Count})");
                    Monitor.Wait(_queueLock); // Release lock and wait
                }

                item = _sharedQueue.Dequeue();
                Console.WriteLine($"\t{consumerName}: Consumed item {item}. Queue size: {_sharedQueue.Count}");
                Monitor.PulseAll(_queueLock); // Notify waiting producers/other consumers
            }
            Thread.Sleep(500); // Simulate time to process item
        }
        Console.WriteLine($"{consumerName} finished consuming items.");
    }
}
```

#### 4. Step-by-Step Walkthrough

Let's focus on the `Race Condition` and `lock` sections:

**Part 1: Race Condition (without lock)**

1.  `Main` thread starts 5 new `Thread` instances, each running `IncrementCounterWithoutLock`.
2.  Each thread (e.g., `Thread_NoLock_0`, `Thread_NoLock_1`, etc.) starts its `for` loop, attempting to increment `_sharedCounter` 1,000,000 times.
3.  The operation `_sharedCounter++` is not atomic. It typically involves three CPU instructions:
    *   Read the current value of `_sharedCounter` into a register.
    *   Increment the value in the register.
    *   Write the new value back to `_sharedCounter` in memory.
4.  **The Race**: Imagine `Thread A` reads `_sharedCounter` (value 10). Before `Thread A` can write back the incremented value (11), `Thread B` also reads `_sharedCounter` (it also sees 10). `Thread B` increments its local copy to 11 and writes it back. Then `Thread A` writes its local copy (11) back. One increment is lost! The counter should be 12, but it's 11.
5.  This happens millions of times across the 5 threads, leading to a final `_sharedCounter` value significantly less than 5,000,000.
6.  `Main` thread uses `t.Join()` to wait for each of the 5 threads to complete before printing the final, incorrect result.

**Part 2: Fixing Race Condition with 'lock'**

1.  `Main` thread starts 5 new `Thread` instances, each running `IncrementCounterWithLock`.
2.  Each thread enters its `for` loop. When it reaches `lock (_lockObject)`, it attempts to acquire the lock.
3.  **Mutual Exclusion**:
    *   The *first* thread to reach `lock (_lockObject)` successfully acquires the lock. It then enters the critical section (`_sharedCounter++`).
    *   Any *other* thread that reaches `lock (_lockObject)` while the first thread holds the lock will immediately **block** (pause execution) and wait until the lock is released.
4.  The thread holding the lock performs `_sharedCounter++`. Because it's the *only* thread allowed in that critical section, its read-modify-write operation is guaranteed to complete without interruption from other threads trying to modify `_sharedCounter`.
5.  When the thread exits the `lock` block, the lock is automatically released.
6.  One of the waiting threads (if any) is then unblocked, acquires the lock, and enters the critical section.
7.  This process repeats. Although threads are now executing sequentially within the critical section, the `_sharedCounter` is always updated correctly.
8.  The final `_sharedCounter` value will be the correct 5,000,000.

**Part 4: Using SemaphoreSlim**

1.  `Main` thread initiates 10 tasks, each calling `AccessLimitedResource`.
2.  `_resourceSemaphore` is initialized with a count of 3, meaning only 3 threads can acquire it simultaneously.
3.  The first 3 tasks to call `_resourceSemaphore.Wait()` will successfully acquire a slot, decrement the semaphore's internal count to 0, and proceed into the `try` block. They print "acquired resource" and simulate work.
4.  The remaining 7 tasks will call `_resourceSemaphore.Wait()` but will find the count is 0. They will **block** and wait.
5.  When one of the first 3 tasks finishes its work, it calls `_resourceSemaphore.Release()`. This increments the semaphore's count, and one of the waiting tasks is unblocked, acquires the slot, and proceeds.
6.  This continues until all 10 tasks have had their turn, ensuring that no more than 3 tasks are ever "working with the resource" concurrently.

#### 5. Common Mistakes

1.  **Deadlocks**: This is perhaps the most dangerous and common mistake. A deadlock occurs when two or more threads are blocked indefinitely, waiting for each other to release a resource.
    ```csharp
    object lockA = new object();
    object lockB = new object();

    // Thread 1
    lock (lockA)
    {
        Thread.Sleep(100); // Simulate work
        lock (lockB) { /* Access shared resources */ }
    }

    // Thread 2
    lock (lockB) // Thread 2 tries to acquire lockB first
    {
        Thread.Sleep(100); // Simulate work
        lock (lockA) { /* Access shared resources */ } // Thread 2 tries to acquire lockA
    }
    // If Thread 1 acquires lockA and Thread 2 acquires lockB,
    // then Thread 1 waits for lockB (held by Thread 2),
    // and Thread 2 waits for lockA (held by Thread 1). Both are stuck.
    ```
    *Correction*: Always acquire locks in a consistent order across all threads. Avoid nested locks if possible. Use `Monitor.TryEnter` with a timeout for more robust deadlock detection/avoidance.

2.  **Locking on `this`, `typeof(T)`, or string literals**:
    *   `lock (this)`: If the object instance is publicly accessible, other code might lock on the same instance, leading to unexpected blocking or deadlocks.
    *   `lock (typeof(MyClass))`: Locks on the `Type` object, which is shared across all instances and potentially across different parts of your application. This can lead to contention or deadlocks if other code also locks on the same `Type`.
    *   `lock ("some string")`: String literals are interned by the CLR, meaning all identical string literals refer to the *same* object in memory. Locking on a string literal means you're locking on a global, shared object, which is highly dangerous.
    *Correction*: Always use a `private static readonly object` for your lock object.

3.  **Not Locking Consistently**: If you protect a shared resource with a lock in one place but forget to do so in another, you still have a race condition. Every access (read or write) to shared mutable state must be protected by the *same* lock.

4.  **Over-locking**: Locking too broadly or for too long can serialize your code, negating the benefits of concurrency. Only the absolute critical section should be protected.
    ```csharp
    lock (_lockObject)
    {
        // BAD: Doing non-critical, long-running work inside the lock
        _sharedCounter++;
        Thread.Sleep(5000); // This holds the lock for 5 seconds!
        Console.WriteLine("Done with critical section.");
    }
    ```
    *Correction*: Minimize the code within a `lock` block. Perform non-critical work outside.

5.  **Starvation**: A thread might repeatedly lose the race to acquire a lock or resource, leading to it never getting to execute its critical section. This is less common with `lock` but can happen with more complex scheduling.

6.  **Using `Semaphore` instead of `SemaphoreSlim`**: For in-process synchronization, `SemaphoreSlim` is generally more efficient and lighter-weight. `Semaphore` is for inter-process synchronization.

#### 6. Advanced Insights

*   **Performance of Synchronization**:
    *   `Interlocked` operations are the fastest because they often map directly to single CPU instructions and avoid context switching. Use them for simple atomic increments/decrements/exchanges.
    *   `lock` (and `Monitor`) involves more overhead. When a thread blocks, it typically involves a context switch to the operating system kernel, which is expensive. However, modern `Monitor` implementations often use "spin locks" for a short period before blocking, trying to acquire the lock without a context switch if it's released quickly.
    *   `SemaphoreSlim` also involves blocking and context switching when its count reaches zero.
    *   **Contention**: The more threads that contend for the same lock, the higher the overhead and the greater the performance degradation. High contention can make a multi-threaded application slower than its single-threaded counterpart.

*   **`ReaderWriterLockSlim`**: For scenarios where a resource is read much more frequently than it's written, `ReaderWriterLockSlim` is a specialized synchronization primitive. It allows multiple threads to read concurrently but requires exclusive access for writing. This can significantly improve performance in read-heavy situations compared to a simple `lock`.

*   **Concurrent Collections**: The `System.Collections.Concurrent` namespace provides thread-safe collection types like `ConcurrentDictionary<TKey, TValue>`, `ConcurrentQueue<T>`, `ConcurrentStack<T>`, and `ConcurrentBag<T>`. These collections manage their own internal synchronization, often using fine-grained locking or lock-free algorithms, making them highly efficient and generally preferable to manually locking around standard collections.

*   **`CancellationTokenSource` and Cooperative Cancellation**: While not a locking mechanism, `CancellationTokenSource` is crucial for gracefully stopping long-running threads or tasks. Instead of forcefully aborting a thread (which is dangerous), you signal a `CancellationToken`, and the running thread periodically checks if cancellation has been requested and exits cooperatively.

*   **Volatile Keyword**: The `volatile` keyword ensures that a field's value is always read from main memory and that writes are immediately flushed to main memory, preventing certain compiler and CPU optimizations that might reorder memory accesses. It's rarely needed with `lock` (as `lock` provides memory barriers), but it's important for fields accessed by multiple threads without explicit locking (e.g., a flag indicating a thread should stop). However, `volatile` *does not* guarantee atomicity for read-modify-write operations.

#### 7. Practice Problems

1.  **Thread-Safe Bank Account**:
    Create a `BankAccount` class with a `Balance` property and `Deposit(decimal amount)` and `Withdraw(decimal amount)` methods.
    In your `Main` method, create a single `BankAccount` instance with an initial balance.
    Start 10 `Thread` instances. Each thread should perform 1000 random deposits (e.g., $1-$100) and 1000 random withdrawals (e.g., $1-$50).
    Demonstrate the race condition if `Deposit` and `Withdraw` are not synchronized. Then, modify the `BankAccount` methods to use a `lock` to ensure thread safety, and verify the final balance is correct (initial balance + total deposits - total withdrawals).

2.  **Limited Printer Pool**:
    Simulate a scenario where you have 5 "printers" (represented by a `SemaphoreSlim` with an initial count of 5).
    Create 15 tasks (using `Task.Run`). Each task represents a "print job."
    Each print job should:
    *   Wait to acquire a printer from the semaphore.
    *   Print a message like "Job X acquired printer. Printing for Y seconds..."
    *   Simulate printing for a random duration (e.g., 1-3 seconds) using `Thread.Sleep()`.
    *   Print "Job X finished printing."
    *   Release the printer.
    Observe that no more than 5 jobs are printing concurrently.

3.  **Producer-Consumer with Bounded Buffer (using `Monitor`)**:
    Expand on the `Producer-Consumer` example from the lesson.
    Modify the `ProduceItems` and `ConsumeItems` methods to ensure that:
    *   The producer waits if the queue is full (`MaxQueueSize`).
    *   The consumer waits if the queue is empty.
    *   Use `Monitor.Wait()` and `Monitor.PulseAll()` (or `Pulse`) to signal between producers and consumers when the queue state changes.
    *   Add a second producer and a third consumer to observe how they interact.

#### 8. Connection to Bigger Picture

Thread synchronization is not just a C# concept; it's a fundamental challenge in all concurrent programming paradigms, whether you're dealing with threads, processes, or distributed systems.

*   **Operating Systems**: The concepts of mutexes, semaphores, and critical sections are core to how operating systems manage concurrent access to hardware and software resources.
*   **Database Systems**: Transaction isolation levels and locking mechanisms in databases are direct applications of synchronization principles to ensure data integrity when multiple users access the same data.
*   **Distributed Systems**: When you move beyond threads in a single process to multiple processes or machines, you encounter similar problems of shared state and coordination, leading to concepts like distributed locks, consensus algorithms (e.g., Paxos, Raft), and message queues.
*   **Concurrent Data Structures**: Understanding synchronization is crucial for designing and implementing efficient, thread-safe data structures, or for appreciating the design of the `System.Collections.Concurrent` types.
*   **Debugging Complexity**: Race conditions and deadlocks are notoriously difficult to debug because they are often non-deterministic and depend on timing. A solid understanding of synchronization helps prevent these issues proactively.

---

This concludes our deep dive into the `Thread` class and the essential synchronization primitives. Mastering these concepts is vital for writing robust, performant, and correct multi-threaded applications.

Do you have any questions about `Thread` class usage, race conditions, or any of the synchronization mechanisms we've discussed before we move on?