---
title: "Understanding Reentrant Lock in Java"
date: 2024-06-18T14:11:55Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

In Java, managing concurrent access to shared resources is crucial to ensure data integrity and prevent issues like 
race conditions. The `java.util.concurrent.locks` package provides various locking mechanisms to achieve this, 
and one of the most commonly used is the `ReentrantLock`.

**What is ReentrantLock?**

`ReentrantLock` is a synchronization primitive that offers more flexibility and features compared to the 
traditional synchronized blocks. It is part of the `java.util.concurrent.locks` package and 
provides a more extensive API for locking and unlocking operations. As the name suggests, 
a reentrant lock allows the thread holding the lock to re-enter the lock multiple times without causing a deadlock.

**Key Features of ReentrantLock**

1. **Reentrancy**: The lock can be acquired multiple times by the same thread. 
This means if a thread currently holds the lock and tries to acquire it again, it will succeed without blocking.
2. **Fairness**: `ReentrantLock` can be created as fair or non-fair. 
A fair lock ensures that the longest-waiting thread gets access to the lock first, 
while a non-fair lock may permit barging, where threads can cut in line.
3. **Condition Variables**: `ReentrantLock` provides condition variables through 
the `newCondition()` method, allowing threads to wait for specific conditions to be met.
4. **Interruptible Lock Acquisition**: Threads can be interrupted while waiting to acquire the lock.
5. **Try Lock**: The `tryLock()` method allows a thread to attempt to acquire the lock without blocking.

**Basic Usage of ReentrantLock**

Here’s a simple example to illustrate the use of `ReentrantLock`:

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;

    public void increment() {
        lock.lock();  // Acquire the lock
        try {
            counter++;
        } finally {
            lock.unlock();  // Always release the lock in a finally block
        }
    }

    public int getCounter() {
        lock.lock();  // Acquire the lock
        try {
            return counter;
        } finally {
            lock.unlock();  // Always release the lock in a finally block
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockExample example = new ReentrantLockExample();

        // Create 1000 threads that increment the counter
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Wait for all threads to finish
        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("Final counter value: " + example.getCounter());
    }
}
```

In this example:

- The `ReentrantLock` is used to ensure that the increment and `getCounter` methods are thread-safe.
- The `lock.lock()` method acquires the lock, and `lock.unlock()` releases it. 
The unlock call is placed in a finally block to ensure the lock is released even if an exception occurs.

**Advanced Features**

1. **Fair Lock**: To create a fair ReentrantLock, pass true to the constructor:  
    ```java 
    Lock fairLock = new ReentrantLock(true);
    ```
    This ensures that the longest-waiting thread will get the lock first.
2. **Try Lock**: To avoid blocking, use tryLock():
    ```java
    if (lock.tryLock()) {
        try {
            // Perform lock-protected operations
        } finally {
            lock.unlock();
        }
    } else {
        // Handle the case where the lock was not acquired
    }
    ```
3. **Condition Variables**: Use condition variables for more complex thread coordination:
    ```java
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    
    public void awaitCondition() throws InterruptedException {
        lock.lock();
        try {
            condition.await();  // Wait until signaled
        } finally {
            lock.unlock();
        }
    }
    
    public void signalCondition() {
        lock.lock();
        try {
            condition.signal();  // Signal one waiting thread
        } finally {
            lock.unlock();
        }
    }
    ```

**When to Use ReentrantLock Over Synchronized**

1. **Fairness**: If you need a fair locking mechanism to prevent thread starvation.
2. **Interruptibility**: When you need the ability to interrupt threads waiting for the lock.
3. **Non-blocking Attempts**: If you need to attempt to acquire the lock without blocking.
4. **Condition Variables**: When complex waiting and notification patterns are required.

**Summary**

`ReentrantLock` provides a powerful and flexible mechanism for thread synchronization in Java. 
It extends the capabilities of traditional synchronized blocks by offering features 
like reentrancy, fairness, interruptible lock acquisition, non-blocking attempts, and condition variables. 
Understanding and using `ReentrantLock` effectively can lead to more robust and maintainable concurrent applications.
