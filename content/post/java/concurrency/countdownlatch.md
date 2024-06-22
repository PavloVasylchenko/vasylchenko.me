---
title: "CountDownLatch in Java with examples"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch in Java with examples**

In the world of Java concurrency, synchronizing multiple threads is a common challenge. 
Java’s `CountDownLatch` class is a powerful synchronization aid that helps manage this problem effectively. 
It allows one or more threads to wait until a set of operations being performed in other threads completes.

**What is CountDownLatch?**

Imagine you are organizing a race. You want all runners to start running at the same time. 
For this, you can use a countdown timer. When the countdown reaches zero, the race begins. 
`CountDownLatch` works in a similar way. 
It initializes with a count that represents the number of operations or threads that must complete before proceeding.

**How Does CountDownLatch Work?**

1. **Initialization**: A CountDownLatch is initialized with a count. This count represents the number of times the `countDown()` method must be invoked before the latch opens.
2. **Count Down**: Each call to `countDown()` decrements the count. When the count reaches zero, all waiting threads are released.
3. **Await**: The `await()` method blocks until the count reaches zero.

**Basic Example**

Let’s look at a simple example to see how `CountDownLatch` works:

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            latch.await();  // Main thread waits until latch count reaches zero
            System.out.println("All tasks are completed. Main thread resumes.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Simulate some work with sleep
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                latch.countDown();  // Decrement the count of the latch
            }
        }
    }
}
```

In this example:

- The `CountDownLatch` is initialized with a count of 3.
- Three threads are started, each representing a task.
- Each task simulates work by sleeping for a second and then calls `countDown()` to decrement the latch count.
- The main thread waits at `latch.await()` until all tasks complete, i.e., the count reaches zero.

**Advanced Example with Timeout**

Sometimes, you don’t want to wait indefinitely. You can use the await method with a timeout to avoid this.

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchTimeoutExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            boolean completed = latch.await(5, TimeUnit.SECONDS);  // Main thread waits with timeout
            if (completed) {
                System.out.println("All tasks are completed. Main thread resumes.");
            } else {
                System.out.println("Timeout occurred before all tasks completed.");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Simulate some work with sleep
                Thread.sleep(2000);  // Longer sleep to potentially trigger timeout
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                latch.countDown();  // Decrement the count of the latch
            }
        }
    }
}
```

In this example:

- The await method is used with a timeout of 5 seconds.
- Each task simulates a longer operation by sleeping for 2 seconds.
- If all tasks complete within the timeout period, the main thread resumes normally. Otherwise, it prints a timeout message.

**Real-World Use Cases for CountDownLatch**

1. Starting Multiple Threads at the Same Time: Sometimes, you need to ensure that a set of threads starts processing simultaneously. By using a CountDownLatch initialized with 1, you can make all threads wait until the latch is counted down once, ensuring a synchronized start.
    ```java
    import java.util.concurrent.CountDownLatch;
    
    public class ConcurrentStartExample {
        private static final int NUMBER_OF_THREADS = 5;
        private static final CountDownLatch startLatch = new CountDownLatch(1);
    
        public static void main(String[] args) {
            for (int i = 0; i < NUMBER_OF_THREADS; i++) {
                new Thread(new Worker()).start();
            }
    
            try {
                System.out.println("Ready... Set...");
                Thread.sleep(1000); // Simulate preparation work
                startLatch.countDown();  // Signal all threads to start
                System.out.println("Go!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        static class Worker implements Runnable {
            @Override
            public void run() {
                try {
                    startLatch.await();  // Wait for the start signal
                    System.out.println(Thread.currentThread().getName() + " is working.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    ```
2.	Waiting for Multiple Threads to Complete: You can use CountDownLatch to wait for a set of threads to complete their work before proceeding. This is useful for aggregating results from multiple threads.
    ```java
    import java.util.concurrent.CountDownLatch;
    
    public class WaitForCompletionExample {
        private static final int NUMBER_OF_TASKS = 3;
        private static final CountDownLatch completionLatch = new CountDownLatch(NUMBER_OF_TASKS);
    
        public static void main(String[] args) {
            for (int i = 0; i < NUMBER_OF_TASKS; i++) {
                new Thread(new SubTask()).start();
            }
    
            try {
                completionLatch.await();  // Main thread waits for all tasks to complete
                System.out.println("All subtasks completed. Aggregating results.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        static class SubTask implements Runnable {
            @Override
            public void run() {
                try {
                    // Simulate some work with sleep
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName() + " subtask completed.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    completionLatch.countDown();  // Decrement the count of the latch
                }
            }
        }
    }
    ```

**Summary**

The `CountDownLatch` class in Java provides a robust mechanism for synchronizing multiple threads, ensuring tasks are completed in a coordinated manner. 
Whether you’re managing the start of multiple threads, waiting for them to finish, or dividing a task into subtasks, 
`CountDownLatch` is a versatile tool that can help improve the reliability and performance of your multithreaded applications.
