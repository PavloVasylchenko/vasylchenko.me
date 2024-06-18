---
title: "Atomics in Java"
date: 2024-06-18T11:12:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

In Java, ensuring correct and efficient concurrency is crucial for developing robust multithreaded applications.
Two important concepts in this context are the use of `volatile` variables and atomic classes from
the `java.util.concurrent.atomic` package.
While both serve to manage shared data among multiple threads, they have different purposes and mechanisms.

**Why Use Atomics in Java?**

Atomic classes in Java provide a way to perform atomic operations on single variables without using synchronization.
These classes ensure that operations like increment, decrement, addition, and compare-and-swap (CAS) are performed atomically, meaning they complete as a single, indivisible step.

The `java.util.concurrent.atomic` package includes several classes, such as:

- AtomicInteger
- AtomicLong
- AtomicBoolean
- AtomicReference

These classes offer methods that perform atomic operations, ensuring thread safety without the overhead of synchronization.

**Key Characteristics of Atomic Classes**

- **Atomicity**: Atomic operations are indivisible. When one thread performs an atomic operation,
  no other thread can observe the operation in an intermediate state.
- **Non-blocking**: Most atomic operations are implemented using low-level CPU instructions
  that do not block threads, thus offering higher performance under contention compared to traditional locking mechanisms.
- **Lock-free**: Atomic classes generally provide lock-free thread-safe operations,
  reducing the risk of contention and deadlock.

**Example of Using Atomic Classes**

Here’s a simple example using `AtomicInteger`:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicExample {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet();  // atomic increment operation
    }

    public int getValue() {
        return counter.get();
    }

    public static void main(String[] args) {
        AtomicExample example = new AtomicExample();

        // Create 1000 threads that increment the counter
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Wait for all threads to finish
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        System.out.println("Final counter value: " + example.getValue());
    }
}
```

In this example, the `AtomicInteger` ensures that the `increment` method is thread-safe without using synchronization.

**Difference Between volatile and Atomic Classes**

1. Visibility vs. Atomicity:
    - `volatile`: Ensures visibility of changes to variables across threads. When a thread writes to a volatile variable, the new value is immediately visible to other threads. However, it does not guarantee atomicity. For example, incrementing a volatile variable is not atomic.
    - `Atomic Classes`: Provide both visibility and atomicity. They ensure that read-modify-write operations (like incrementing) are atomic, meaning they complete in a single step without interference from other threads.
      2 Complex Operations:
    - `volatile`: Suitable for scenarios where simple read and write operations are sufficient. It does not handle compound actions (like incrementing or comparing and setting a value) atomically.
    - `Atomic Classes`: Ideal for more complex operations where atomicity is required. They provide methods like incrementAndGet(), decrementAndGet(), addAndGet(), and compareAndSet() that ensure these operations are performed atomically.
      3 Implementation:
    - `volatile`: Relies on the Java Memory Model to ensure visibility. It does not use any special hardware instructions.
    - `Atomic Classes`: Often use low-level CPU instructions (like compare-and-swap) to ensure atomicity, making them more efficient under certain conditions.
      4 Usage:
    - `volatile`: Use when you need to ensure visibility of changes and do not require atomicity for compound actions. Suitable for flags and simple state variables.
    - `Atomic Classes`: Use when you need atomicity for operations. Suitable for counters, references, and variables that undergo frequent updates by multiple threads.

**Summary**

Understanding the differences between volatile and atomic classes is essential for developing effective
concurrent applications in Java. While volatile ensures visibility, atomic classes provide both visibility
and atomicity, making them more suitable for scenarios that involve compound actions and require thread-safe
operations without the overhead of synchronization. By leveraging the right tool for the right scenario,
you can improve the correctness and performance of your multithreaded Java applications.