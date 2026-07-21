# Java Thread Pools, Atomic Classes, and Synchronization

# Thread Pools in Java

A thread pool is a collection of reusable threads managed by the JVM/application.

Instead of creating a new thread for every task:

```java
new Thread(() -> {
    process();
}).start();
```

we reuse existing threads.

Benefits:

- Reduces thread creation overhead
- Controls number of concurrent tasks
- Improves resource management
- Prevents too many threads

Java provides thread pools through:

```java
ExecutorService
```

from:

```java
java.util.concurrent
```

# ExecutorService Example

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);


executor.submit(() -> {
    System.out.println(
        "Task running"
    );
});


executor.shutdown();
```

Flow:

```
Task
 |
 v
ExecutorService
 |
 v
Thread Pool
 |
 v
Worker Thread
```

# Types of Thread Pools

## 1. Fixed Thread Pool

Creates a fixed number of threads.

Example:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);
```

Internal:

```
Thread Pool

Thread-1
Thread-2
Thread-3
...
Thread-10
```

If 100 tasks arrive:

```
10 threads execute
90 tasks wait in queue
```

Use cases:

- Database operations
- API calls
- Controlled workloads

## 2. Cached Thread Pool

Creates threads dynamically.

Example:

```java
ExecutorService executor =
    Executors.newCachedThreadPool();
```

Behavior:

```
Task arrives
    |
    v
Create new thread if needed
```

Idle threads are removed after timeout.

Good for:

- Short-lived asynchronous tasks

Risk:

- Can create too many threads

## 3. Single Thread Executor

Uses only one worker thread.

Example:

```java
ExecutorService executor =
    Executors.newSingleThreadExecutor();
```

Guarantees:

```
Task 1
 |
Task 2
 |
Task 3
```

Execution order is maintained.

Use cases:

- Sequential processing
- File writing
- Logging

## 4. Scheduled Thread Pool

Runs tasks after a delay or periodically.

Example:

```java
ScheduledExecutorService executor =
    Executors.newScheduledThreadPool(3);


executor.scheduleAtFixedRate(
    () -> refreshCache(),
    0,
    10,
    TimeUnit.SECONDS
);
```

Use cases:

- Scheduled jobs
- Cache refresh
- Monitoring

## 5. Work Stealing Pool

Uses multiple queues.

Example:

```java
ExecutorService executor =
    Executors.newWorkStealingPool();
```

Uses:

```
ForkJoinPool
```

Concept:

```
Thread-1 Queue
    |
    |
Thread-2 Queue
    |
    |
Steal tasks from other queues
```

Good for:

- CPU-intensive parallel tasks

# ThreadPoolExecutor

Most thread pools internally use:

```java
ThreadPoolExecutor
```

Constructor:

```java
ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue
)
```

Example:

```java
ThreadPoolExecutor executor =
new ThreadPoolExecutor(
    5,
    10,
    60,
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>()
);
```

Parameters:

## corePoolSize

Minimum number of threads kept alive.

## maximumPoolSize

Maximum allowed threads.

## keepAliveTime

How long extra threads stay alive.

## workQueue

Stores waiting tasks.

Example:

```
Tasks
 |
 v
Queue
 |
 v
Worker Threads
```

# Atomic Classes in Java

Atomic classes provide thread-safe operations without explicit locking.

Package:

```java
java.util.concurrent.atomic
```

Examples:

- AtomicInteger
- AtomicLong
- AtomicBoolean
- AtomicReference

# Problem Without Atomic Classes

Example:

```java
class Counter {

    int count = 0;


    void increment() {
        count++;
    }
}
```

`count++` is not one operation.

Internally:

```
Read count
     |
Increment
     |
Write count
```

Two threads:

```
Thread 1 reads 0
Thread 2 reads 0

Thread 1 writes 1
Thread 2 writes 1
```

Expected:

```
2
```

Actual:

```
1
```

This is a race condition.

# AtomicInteger Example

```java
AtomicInteger count =
    new AtomicInteger(0);


count.incrementAndGet();
```

Internally uses:

```
CAS
(Compare And Swap)
```

# Compare And Swap (CAS)

CAS performs:

```
Check current value
        |
        v
If unchanged
        |
        v
Update value
```

Example:

```
Current value = 10

Thread wants:
Change 10 -> 11

Check:
Is value still 10?

Yes:
Update to 11
```

No lock required.

# Common Atomic Methods

## AtomicInteger

```java
AtomicInteger count =
    new AtomicInteger();


count.incrementAndGet();

count.decrementAndGet();

count.getAndIncrement();

count.compareAndSet(
    10,
    20
);
```

## AtomicReference

Used for objects.

Example:

```java
AtomicReference<User> user =
    new AtomicReference<>();


user.set(new User());

user.compareAndSet(
    oldUser,
    newUser
);
```

# Atomic Classes vs synchronized

| Feature | Atomic | synchronized |
|-|-|-|
| Lock | No | Yes |
| Mechanism | CAS | Monitor lock |
| Performance | Faster for simple operations | Better for complex logic |
| Multiple operations | Limited | Supported |

# Synchronization in Java

Synchronization controls access to shared resources.

It ensures:

```
Only one thread executes critical section at a time
```

# synchronized Method

Example:

```java
public synchronized void increment() {

    count++;

}
```

Equivalent:

```java
synchronized(this) {

    count++;

}
```

# synchronized Block

More efficient because only required code is locked.

Example:

```java
public void increment() {


    synchronized(this) {

        count++;

    }

}
```

# Object Monitor

Every Java object has an intrinsic lock.

Example:

```java
synchronized(lockObject) {

}
```

Flow:

```
Thread-1
    |
Acquire Lock
    |
Execute Code
    |
Release Lock


Thread-2
    |
Wait
```

# Class Level Synchronization

Static methods:

```java
public static synchronized void process(){

}
```

Lock is:

```
Class object
```

Example:

```java
Employee.class
```

# ReentrantLock

Alternative to synchronized.

Example:

```java
Lock lock =
    new ReentrantLock();


lock.lock();

try {

    process();

}
finally {

    lock.unlock();

}
```

Advantages:

- Try lock
- Timeout
- Interruptible lock

Example:

```java
if(lock.tryLock()) {

    process();

}
```

# ReadWriteLock

Useful when:

- Many readers
- Few writers

Example:

```java
ReadWriteLock lock =
    new ReentrantReadWriteLock();
```

Locks:

```
Read Lock

Multiple readers allowed


Write Lock

Only one writer
```

# Volatile Keyword

`volatile` provides visibility.

Example:

```java
volatile boolean running = true;
```

Thread 1:

```java
running = false;
```

Thread 2 immediately sees:

```
false
```

But:

```
volatile does not provide atomicity
```

This is unsafe:

```java
volatile int count;

count++;
```

# Synchronization vs Atomic vs Volatile

| Feature | synchronized | Atomic | volatile |
|-|-|-|-|
| Visibility | Yes | Yes | Yes |
| Atomic operations | Yes | Yes | No |
| Locking | Yes | No | No |
| Complex logic | Yes | No | No |
| Performance | Slower | Faster | Fastest |

# Deadlock

Deadlock occurs when threads wait forever for each other.

Example:

```
Thread 1:

Lock A
Waiting for Lock B


Thread 2:

Lock B
Waiting for Lock A
```

Solution:

- Lock ordering
- Timeout locks
- Avoid nested locks

# Spring Boot Thread Pool Example

Using Async:

Configuration:

```java
@Configuration
@EnableAsync
public class AsyncConfig {


@Bean
public Executor taskExecutor() {

    return Executors
        .newFixedThreadPool(10);

}

}
```

Service:

```java
@Service
public class EmailService {


@Async
public void sendEmail(){

    // runs in another thread

}

}
```

Flow:

```
Controller Thread

      |
      v

Async Executor

      |
      v

Worker Thread

      |
      v

sendEmail()
```

# Interview Summary

```
Thread pools manage reusable threads using ExecutorService.

Common pools:
- FixedThreadPool
- CachedThreadPool
- SingleThreadExecutor
- ScheduledThreadPool
- WorkStealingPool

Atomic classes provide lock-free thread-safe operations using CAS.

Synchronization provides mutual exclusion using locks.

Use:
- Atomic classes for simple counters/state updates.
- synchronized/ReentrantLock for complex critical sections.
- volatile for visibility-only requirements.
```

## One-Line Interview Answers

**Thread Pool:**

> A thread pool manages a set of reusable threads to execute tasks efficiently without creating a new thread for every request.

**Atomic Class:**

> Atomic classes provide lock-free thread-safe operations using CAS (Compare-And-Swap).

**Synchronization:**

> Synchronization ensures that only one thread accesses a critical section at a time, preventing race conditions.

# CompletableFuture in Java

## What is CompletableFuture?

`CompletableFuture` is a Java class used for **asynchronous and non-blocking programming**.

It allows you to:

- Run tasks asynchronously
- Chain multiple async operations
- Combine results from multiple tasks
- Handle exceptions
- Avoid blocking threads

Package:

```java
java.util.concurrent.CompletableFuture
```

Introduced in:

```
Java 8
```

## Future vs CompletableFuture

## Future

Traditional `Future` allows you to submit a task and get the result later.

Example:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(2);


Future<String> future =
    executor.submit(() -> {

        Thread.sleep(2000);

        return "Result";

    });


String result = future.get();
```

Problem:

```java
future.get();
```

is blocking.

Thread waits until the result is available.

## CompletableFuture

`CompletableFuture` allows non-blocking chaining.

Example:

```java
CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(result -> result + " World")
    .thenAccept(System.out::println);
```

Output:

```
Hello World
```

Flow:

```
Task 1
 |
 v
Transform Result
 |
 v
Consume Result
```

# Creating CompletableFuture

## 1. completedFuture()

Creates an already completed future.

```java
CompletableFuture<String> future =
    CompletableFuture.completedFuture(
        "Completed"
    );
```

Result is immediately available.

## 2. runAsync()

Used when there is no return value.

Example:

```java
CompletableFuture<Void> future =
    CompletableFuture.runAsync(() -> {

        sendEmail();

    });
```

Equivalent:

```
Runnable task
       |
       v
Async execution
```

## 3. supplyAsync()

Used when a result needs to be returned.

Example:

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> {

        return fetchData();

    });
```

Equivalent:

```
Supplier<T>
     |
     v
CompletableFuture<T>
```

# Default Thread Pool

By default:

```java
supplyAsync()
```

uses:

```
ForkJoinPool.commonPool()
```

Example:

```java
CompletableFuture
    .supplyAsync(() -> {

        System.out.println(
            Thread.currentThread().getName()
        );

        return "data";

    });
```

Output:

```
ForkJoinPool.commonPool-worker-1
```

# Using Custom Executor

Recommended in Spring Boot applications.

Example:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);


CompletableFuture<String> future =
    CompletableFuture.supplyAsync(
        () -> fetchData(),
        executor
    );
```

Flow:

```
Task
 |
 v
Custom Thread Pool
 |
 v
Worker Thread
```

# CompletableFuture Operations

## thenApply()

Transforms the result.

Example:

```java
CompletableFuture<String> future =
    CompletableFuture
        .supplyAsync(() -> "john")
        .thenApply(
            name -> name.toUpperCase()
        );
```

Flow:

```
"john"

   |
   v

"JOHN"
```

Equivalent:

```java
Function<T,R>
```

## thenAccept()

Consumes result but returns nothing.

Example:

```java
CompletableFuture
    .supplyAsync(() -> "Data")
    .thenAccept(
        data -> System.out.println(data)
    );
```

Equivalent:

```java
Consumer<T>
```

## thenRun()

Runs after completion but does not use result.

Example:

```java
CompletableFuture
    .supplyAsync(() -> "Data")
    .thenRun(
        () -> cleanup()
    );
```

Flow:

```
Task Completed

      |
      v

Run cleanup
```

# Chaining CompletableFuture

Example:

```java
CompletableFuture
    .supplyAsync(() -> getUserId())
    .thenApply(id -> getUser(id))
    .thenApply(user -> user.getName())
    .thenAccept(
        name -> System.out.println(name)
    );
```

Flow:

```
Get User ID
      |
      v
Fetch User
      |
      v
Get Name
      |
      v
Print Name
```

# Combining CompletableFutures

## thenCompose()

Used when the second async operation depends on the first result.

Example:

```java
CompletableFuture<User> future =
    getUserId()
    .thenCompose(
        id -> getUser(id)
    );
```

Without `thenCompose`:

```
Future<Future<User>>
```

With `thenCompose`:

```
Future<User>
```

Example:

```
Get ID
 |
 v
Get User using ID
```

# thenCombine()

Used when two independent tasks need to be combined.

Example:

```java
CompletableFuture<String> user =
    CompletableFuture.supplyAsync(
        () -> getUser()
    );


CompletableFuture<String> order =
    CompletableFuture.supplyAsync(
        () -> getOrder()
    );


user.thenCombine(
    order,
    (u, o) -> u + o
);
```

Flow:

```
Task 1
   \
    \
     Combine
    /
Task 2
```

# Waiting for Multiple Futures

## allOf()

Waits for all tasks.

Example:

```java
CompletableFuture<Void> result =
    CompletableFuture.allOf(
        future1,
        future2,
        future3
    );


result.join();
```

Flow:

```
Task 1
Task 2
Task 3
   |
   v
all completed
```

## anyOf()

Completes when any task completes.

Example:

```java
CompletableFuture<Object> result =
    CompletableFuture.anyOf(
        future1,
        future2
    );
```

Use cases:

- First available response
- Multiple service fallback

# Exception Handling

## exceptionally()

Handles errors and provides fallback.

Example:

```java
CompletableFuture
    .supplyAsync(
        () -> callService()
    )
    .exceptionally(
        ex -> "Default Response"
    );
```

Flow:

```
Success
 |
 v
Return result


Failure
 |
 v
Fallback value
```

## handle()

Handles both success and failure.

Example:

```java
future.handle(
    (result, exception) -> {

        if(exception != null) {
            return "Error";
        }

        return result;

    }
);
```

## whenComplete()

Used for logging/side effects.

Example:

```java
future.whenComplete(
    (result, exception) -> {

        log(result);

    }
);
```

# CompletableFuture in Spring Boot

## Enable Async

```java
@Configuration
@EnableAsync
public class AsyncConfig {

}
```

## Service Example

```java
@Service
public class UserService {


@Async
public CompletableFuture<User> getUser() {

    User user = repository.findUser();

    return CompletableFuture
            .completedFuture(user);
}

}
```

Controller:

```java
@GetMapping("/user")
public CompletableFuture<User> getUser(){

    return service.getUser();

}
```

Spring returns immediately:

```
Request Thread

      |
      v

Async Worker Thread

      |
      v

Database Call
```

# CompletableFuture with Thread Pool

Recommended configuration:

```java
@Configuration
@EnableAsync
public class AsyncConfig {


@Bean
public Executor taskExecutor(){

    ThreadPoolTaskExecutor executor =
        new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(100);

    executor.initialize();

    return executor;
}

}
```

Use:

```java
CompletableFuture
    .supplyAsync(
        () -> serviceCall(),
        executor
    );
```

# CompletableFuture vs Parallel Streams

| Feature | CompletableFuture | Parallel Stream |
|-|-|-|
| Async workflows | Yes | Limited |
| Combine APIs | Yes | No |
| Custom executor | Yes | Difficult |
| Exception handling | Better | Limited |
| Best for | I/O tasks | CPU operations |

# CompletableFuture vs Thread

| Feature | Thread | CompletableFuture |
|-|-|-|
| Result handling | Manual | Built-in |
| Chaining | No | Yes |
| Combining tasks | Manual | Built-in |
| Exception handling | Manual | Built-in |

# Real-Time Example: Calling Multiple APIs

Scenario:

Need:

- User details
- Order details
- Payment details

Instead of:

```
Call User API
(wait)

Call Order API
(wait)

Call Payment API
(wait)
```

Use:

```java
CompletableFuture<User> user =
    getUserAsync();


CompletableFuture<Order> order =
    getOrderAsync();


CompletableFuture<Payment> payment =
    getPaymentAsync();


CompletableFuture
    .allOf(
        user,
        order,
        payment
    )
    .join();
```

Execution:

```
User API       ------>
Order API      ------>  Parallel
Payment API    ------>

        |
        v

Combine Result
```

# Important Interview Questions

## 1. Difference between thenApply and thenCompose?

```
thenApply:
- Transform result
- One-to-one mapping

thenCompose:
- Chain another async operation
- Avoid Future<Future<T>>
```

## 2. Difference between join() and get()?

| join() | get() |
|-|-|
| Does not require checked exception handling | Throws checked exceptions |
| Throws CompletionException | Throws ExecutionException |

## 3. Which thread executes thenApply?

Depends:

```java
thenApply()
```

may execute in the thread completing the previous stage.

For separate thread:

```java
thenApplyAsync()
```

uses executor.

# Interview Summary

```
CompletableFuture provides asynchronous programming in Java.

supplyAsync() runs tasks asynchronously and returns results.
runAsync() runs tasks without returning results.

Important methods:

thenApply()     -> transform result
thenAccept()    -> consume result
thenRun()       -> execute next action
thenCompose()   -> chain dependent async calls
thenCombine()   -> combine independent async calls
allOf()         -> wait for all tasks
anyOf()         -> wait for first task

In Spring Boot, CompletableFuture is commonly used with custom thread
pools for parallel API calls, background processing, and improving
application responsiveness.
```

## One-Line Interview Answer

> CompletableFuture is Java's framework for asynchronous and non-blocking programming. It allows creating, chaining, combining, and handling results of asynchronous tasks using methods like supplyAsync, thenApply, thenCompose, and allOf.

# Spring Boot TaskExecutor

Spring does not create raw Java `Thread` objects directly for most asynchronous operations. It uses **task executor abstractions** built on top of Java's concurrency APIs.

The main Spring classes are:

```
TaskExecutor
      |
      +-- ThreadPoolTaskExecutor
      |
      +-- SimpleAsyncTaskExecutor
      |
      +-- ConcurrentTaskExecutor
```

# 1. TaskExecutor Interface

Spring's abstraction for executing tasks.

Package:

```java
org.springframework.core.task.TaskExecutor
```

Interface:

```java
public interface TaskExecutor 
        extends Executor {

    void execute(Runnable task);

}
```

Example:

```java
taskExecutor.execute(() -> {

    process();

});
```

Spring code generally works with `TaskExecutor` instead of directly depending on thread pools.

# 2. ThreadPoolTaskExecutor (Most Common)

In Spring Boot applications, this is the most commonly used executor.

Class:

```java
org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor
```

Internally uses:

```java
java.util.concurrent.ThreadPoolExecutor
```

Architecture:

```
Spring Application

        |
        v

ThreadPoolTaskExecutor

        |
        v

ThreadPoolExecutor

        |
        v

Worker Threads
```

Example configuration:

```java
@Configuration
@EnableAsync
public class AsyncConfig {


    @Bean
    public Executor taskExecutor() {

        ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(5);

        executor.setMaxPoolSize(10);

        executor.setQueueCapacity(100);

        executor.setThreadNamePrefix(
            "async-worker-"
        );

        executor.initialize();

        return executor;
    }
}
```

Created threads:

```
async-worker-1
async-worker-2
async-worker-3
```

# 3. SimpleAsyncTaskExecutor

Class:

```java
org.springframework.core.task.SimpleAsyncTaskExecutor
```

Creates a new thread for every task.

Example:

```java
@Bean
TaskExecutor executor(){

    return new SimpleAsyncTaskExecutor();

}
```

Behavior:

```
Task 1 --> Thread 1

Task 2 --> Thread 2

Task 3 --> Thread 3
```

Important:

It does not reuse threads.

Not recommended for high traffic applications.

# 4. ConcurrentTaskExecutor

Adapter around Java's Executor.

Class:

```java
org.springframework.scheduling.concurrent.ConcurrentTaskExecutor
```

Example:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);


TaskExecutor taskExecutor =
    new ConcurrentTaskExecutor(
        executor
    );
```

Useful when integrating existing Java executors.

# @Async and Threads in Spring

When using:

```java
@Async
public void sendEmail(){

}
```

Spring creates a proxy.

Flow:

```
Controller Thread

        |
        v

Spring Async Proxy

        |
        v

TaskExecutor

        |
        v

Worker Thread

        |
        v

sendEmail()
```

Example:

```java
@Service
public class EmailService {


    @Async
    public void sendEmail(){

        System.out.println(
            Thread.currentThread()
                  .getName()
        );

    }
}
```

Output:

```
async-worker-1
```

# Default Executor for @Async

If you do not configure an executor:

```java
@EnableAsync
```

Spring uses:

```
SimpleAsyncTaskExecutor
```

depending on Spring configuration/version.

It is recommended to configure your own:

```java
ThreadPoolTaskExecutor
```

for production systems.

# Spring MVC Request Threads

For normal REST APIs, Spring MVC uses the embedded server thread pool.

Example:

```
Tomcat Thread Pool

http-nio-8080-exec-1
http-nio-8080-exec-2
http-nio-8080-exec-3
```

Flow:

```
HTTP Request

      |
      v

Tomcat Worker Thread

      |
      v

Controller Method
```

This is different from `@Async`.

# Spring Boot + Tomcat Thread Pool

Embedded Tomcat uses:

```java
org.apache.tomcat.util.threads.ThreadPoolExecutor
```

Configuration:

```properties
server.tomcat.threads.max=200
server.tomcat.threads.min-spare=10
```

Flow:

```
Client Request

      |
      v

Tomcat Thread Pool

      |
      v

DispatcherServlet

      |
      v

Controller
```

# Spring Scheduling Threads

For:

```java
@Scheduled
public void refreshCache(){

}
```

Spring uses:

```java
TaskScheduler
```

Common implementation:

```java
ThreadPoolTaskScheduler
```

Example:

```java
@Bean
public TaskScheduler scheduler(){

    ThreadPoolTaskScheduler scheduler =
        new ThreadPoolTaskScheduler();

    scheduler.setPoolSize(5);

    return scheduler;

}
```

Flow:

```
Scheduler

   |
   v

Scheduled Thread

   |
   v

@Scheduled Method
```

# Spring Classes Summary

| Use Case | Spring Class | Internal Implementation |
|-|-|-|
| @Async | ThreadPoolTaskExecutor | ThreadPoolExecutor |
| Scheduling | ThreadPoolTaskScheduler | ScheduledThreadPoolExecutor |
| REST requests | Tomcat Thread Pool | Tomcat Executor |
| Generic tasks | TaskExecutor | Executor |
| Simple async | SimpleAsyncTaskExecutor | New Thread |

# ThreadPoolTaskExecutor Important Properties

## Core Pool Size

Minimum threads:

```java
executor.setCorePoolSize(5);
```

## Max Pool Size

Maximum threads:

```java
executor.setMaxPoolSize(20);
```

## Queue Capacity

Waiting tasks:

```java
executor.setQueueCapacity(100);
```

Example:

```
5 active threads

100 waiting tasks

Then create extra threads up to max
```

## Rejection Policy

When pool and queue are full:

```java
executor.setRejectedExecutionHandler(
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

Options:

```
AbortPolicy
CallerRunsPolicy
DiscardPolicy
DiscardOldestPolicy
```

# Interview Answer

**Q: Which class does Spring use for threads?**

Answer:

> Spring provides the TaskExecutor abstraction for thread management. In Spring Boot applications, the most commonly used implementation is ThreadPoolTaskExecutor, which internally uses Java's ThreadPoolExecutor. For @Async methods, Spring delegates execution to a configured TaskExecutor. For scheduled tasks, Spring uses TaskScheduler implementations like ThreadPoolTaskScheduler.


