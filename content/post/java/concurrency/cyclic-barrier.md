---
title: "CyclicBarrier in Java"
date: 2024-06-19T13:15:18Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Understanding CyclicBarrier Usage in Java**

The `CyclicBarrier` class in Java is a synchronization aid that allows a set of threads to wait for each 
other to reach a common barrier point. This is useful in scenarios where you want multiple 
threads to start processing concurrently after reaching a particular state.

Unlike `CountDownLatch`, a `CyclicBarrier` can be reused after the waiting threads are released, 
making it suitable for cyclic or iterative operations.

**Key Features of CyclicBarrier**

- `Reusable`: After all waiting threads are released, the barrier can be reused.
- `Optional Runnable`: You can specify an optional Runnable task that will be executed once all threads have reached the barrier.

**Basic Usage of CyclicBarrier**

Here’s a simple example to demonstrate the usage of `CyclicBarrier`:
```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await();  // Wait at the barrier
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

In this example:

- **Barrier Initialization**: The `CyclicBarrier` is initialized with the number of threads (`NUMBER_OF_THREADS`) that need to wait at the barrier and an optional `Runnable` task that runs once all threads reach the barrier.
- **Task Implementation**: The Task class implements `Runnable` and calls `barrier.await()`. Each thread waits at the barrier until all threads have reached it.
- **Thread Execution**: Three threads are started, and each waits at the barrier. 
Once all threads have arrived at the barrier, the barrier action is executed, and then the threads proceed.

**Advanced Usage of CyclicBarrier**

1. **Handling Broken Barrier**: If one of the waiting threads is interrupted or times out, the `CyclicBarrier` is considered broken. 
You can check if the barrier is broken using `barrier.isBroken()` and handle it accordingly.
2. **Timeouts**: You can specify a timeout for the `await()` method to avoid waiting indefinitely.
    ```java
    barrier.await(5, TimeUnit.SECONDS);
    ```
3. **Resetting the Barrier**: You can reset the barrier to its initial state using `barrier.reset()`, 
which can be useful if you want to reuse the barrier in a different phase or iteration.

**Example with Timeout and Reset**

Here’s an example that includes handling timeout and resetting the barrier:
```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class CyclicBarrierAdvancedExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await(5, TimeUnit.SECONDS);  // Wait at the barrier with timeout
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException | TimeoutException e) {
                e.printStackTrace();
                if (barrier.isBroken()) {
                    System.out.println("Barrier is broken, resetting...");
                    barrier.reset();  // Reset the barrier if it's broken
                }
            }
        }
    }
}
```

**In this advanced example**:

- **Timeout Handling**: Each thread waits at the barrier with a timeout of 5 seconds. If the timeout is exceeded, a `TimeoutException` is thrown.
- **Broken Barrier Check**: If the barrier is broken, the thread prints a message and resets the barrier using `barrier.reset()`.

**Summary**

The `CyclicBarrier` class in Java provides a powerful mechanism for synchronizing a fixed number of threads at a common barrier point. 
Its reusability and optional barrier action make it a flexible tool for coordinating threads in cyclic or iterative tasks. 
By understanding and leveraging `CyclicBarrier`, developers can build robust multithreaded applications that require synchronization among multiple threads.
