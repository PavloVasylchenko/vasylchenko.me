---
title: "Semaphore in Java"
date: 2024-06-20T08:16:17Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Understanding Semaphore in Java**

A `Semaphore` in Java is a synchronization aid that regulates the number of threads that can access a shared resource simultaneously. 
It’s part of the `java.util.concurrent` package and is commonly used to control access to resources that have a limited number of permits, 
such as database connections or network sockets.

**Key Features of Semaphore**

1. **Permits**: A semaphore maintains a set of permits, and threads acquire permits to proceed and release them when they are done.
2. **Blocking** and Non-blocking Acquisition: Threads can either wait until a permit is available or try to acquire a permit without blocking.
3. **Fairness**: Semaphores can be fair or non-fair. Fair semaphores grant permits in the order they were requested.

**Basic Usage of Semaphore**

Here’s a simple example to illustrate the usage of Semaphore:

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
   private static final int MAX_PERMITS = 3;
   private static final Semaphore semaphore = new Semaphore(MAX_PERMITS);

   public static void main(String[] args) {
      for (int i = 0; i < 10; i++) {
         new Thread(new Task()).start();
      }
   }

   static class Task implements Runnable {
      @Override
      public void run() {
         try {
            System.out.println(Thread.currentThread().getName() + " is waiting for a permit.");
            semaphore.acquire();  // Acquire a permit
            System.out.println(Thread.currentThread().getName() + " acquired a permit.");

            // Simulate some work with the shared resource
            Thread.sleep(2000);

            System.out.println(Thread.currentThread().getName() + " releasing a permit.");
         } catch (InterruptedException e) {
            e.printStackTrace();
         } finally {
            semaphore.release();  // Release the permit
         }
      }
   }
}
```

In this example:

1. **Initialization**: A semaphore with 3 permits is created.
    ```java
    if (semaphore.tryAcquire()) {
        try {
            // Critical section
        } finally {
            semaphore.release();
        }
    } else {
        System.out.println(Thread.currentThread().getName() + " could not acquire a permit.");
    }
    ```
2. **Task Implementation**: The Task class implements Runnable and tries to acquire a permit before proceeding. If a permit is available, the thread acquires it and simulates some work. After completing the work, the thread releases the permit.
    ```java
    if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
        try {
            // Critical section
        } finally {
            semaphore.release();
        }
    } else {
        System.out.println(Thread.currentThread().getName() + " timed out waiting for a permit.");
    }
    ```
3. **Thread Execution**: Ten threads are started, but only three can hold a permit at any time, thus only three threads can execute the critical section simultaneously.
    ```java
    Semaphore fairSemaphore = new Semaphore(MAX_PERMITS, true);
    ```

**Advanced Usage of Semaphore**

1. Try to Acquire Permit Without Blocking:
2. Acquire Permit with Timeout:
3. Fair Semaphores:

**Use Cases for Semaphore**

1. **Rate Limiting**: Control the number of concurrent access to a resource.
2. **Connection Pooling**: Manage access to a pool of database connections.
3. **Resource Management**: Limit access to a limited number of resources, like printers or file handles.

**Summary**

Semaphores in Java provide a robust mechanism for controlling access to shared resources, 
especially when you need to limit the number of concurrent accesses. 
By understanding how to use semaphores effectively, you can ensure that your multithreaded 
applications manage resources efficiently and avoid contention issues.