---
title: "Concurrent collections in Java with examples"
date: 2024-06-25T13:46:57Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Concurrent collections in Java with examples**

Java provides several concurrent collections that are part of the `java.util.concurrent` package. 
These collections are designed to handle multithreaded environments efficiently, providing thread-safe operations and 
improving performance over traditional synchronization techniques. 
In this article, we will explore these collections, 
their use cases, and provide more complex code examples to help you understand 
how to use them effectively.

**Introduction to Concurrent Collections**

Concurrent collections are designed to support high concurrency while maintaining thread safety. The main concurrent collections provided by Java are:

- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`
- `ConcurrentLinkedQueue`
- `ConcurrentLinkedDeque`
- `LinkedBlockingQueue`
- `PriorityBlockingQueue`

Let’s dive into each of these collections with more complex examples.

**ConcurrentHashMap**

`ConcurrentHashMap` is a thread-safe implementation of HashMap that allows concurrent read and write operations.

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // Simulate multiple threads updating the map
        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                map.put(Thread.currentThread().getName() + i, i);
            }
        };
        
        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        
        writer1.start();
        writer2.start();
        
        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Concurrent read operation
        Runnable readerTask = () -> {
            map.forEach((key, value) -> System.out.println(Thread.currentThread().getName() + ": " + key + ": " + value));
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```
**CopyOnWriteArrayList**

`CopyOnWriteArrayList` is a thread-safe variant of ArrayList where all mutative operations are implemented by making a fresh copy of the underlying array.

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteArrayListExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        
        // Simulate multiple threads updating the list
        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                list.add(Thread.currentThread().getName() + i);
            }
        };
        
        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        
        writer1.start();
        writer2.start();
        
        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Concurrent read operation
        Runnable readerTask = () -> {
            for (String item : list) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` is a thread-safe variant of `HashSet` that is implemented using a `CopyOnWriteArrayList`.

```java
import java.util.concurrent.CopyOnWriteArraySet;

public class CopyOnWriteArraySetExample {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
        
        // Simulate multiple threads updating the set
        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                set.add(Thread.currentThread().getName() + i);
            }
        };
        
        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        
        writer1.start();
        writer2.start();
        
        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Concurrent read operation
        Runnable readerTask = () -> {
            for (String item : set) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` is an unbounded thread-safe queue based on linked nodes. 
It is an implementation of a wait-free, lock-free algorithm.

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueExample {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
        
        // Simulate multiple threads adding to the queue
        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                queue.add(Thread.currentThread().getName() + i);
            }
        };
        
        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        
        producer1.start();
        producer2.start();
        
        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Simulate multiple threads consuming from the queue
        Runnable consumerTask = () -> {
            while (!queue.isEmpty()) {
                String item = queue.poll();
                if (item != null) {
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            }
        };
        
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        consumer1.start();
        consumer2.start();
    }
}
```

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` is a thread-safe variant of LinkedList supporting concurrent access to both ends.

```java
import java.util.concurrent.ConcurrentLinkedDeque;

public class ConcurrentLinkedDequeExample {
    public static void main(String[] args) {
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();
        
        // Simulate multiple threads adding to the deque
        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                deque.addLast(Thread.currentThread().getName() + i);
            }
        };
        
        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        
        producer1.start();
        producer2.start();
        
        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Simulate multiple threads consuming from the deque
        Runnable consumerTask = () -> {
            while (!deque.isEmpty()) {
                String item = deque.pollFirst();
                if (item != null) {
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            }
        };
        
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        consumer1.start();
        consumer2.start();
    }
}
```

**LinkedBlockingQueue**

`LinkedBlockingQueue` is an optionally bounded blocking queue based on linked nodes. 
It supports operations that wait for the queue to become non-empty when retrieving an element, 
and wait for space to become available in the queue when storing an element.

```java
import java.util.concurrent.LinkedBlockingQueue;

public class LinkedBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
        
        // Simulate multiple threads producing and consuming
        Runnable producerTask = () -> {
            try {
                for (int i = 0; i < 1000; i++) {
                    queue.put(Thread.currentThread().getName() + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };
        
        Runnable consumerTask = () -> {
            try {
                while (true) {
                    String item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };
        
        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
    }
}
```

**PriorityBlockingQueue**

`PriorityBlockingQueue` is an unbounded blocking queue that uses the same ordering rules as `PriorityQueue` and supplies blocking retrieval operations.

```java
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();
        
        // Simulate multiple threads producing and consuming
        Runnable producerTask = () -> {
            try {
                for (int i = 0; i < 1000; i++) {
                    queue.put((int) (Math.random() * 1000));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (true) {
                    Integer item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);

        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
    }
}
```

**Comparison Table**

Here is a comparison table of the different concurrent collections discussed:

| Collection            | Thread-Safe | Blocking Operations | Use Case                                     |
|-----------------------|-------------|---------------------|----------------------------------------------|
| ConcurrentHashMap     | Yes         | No                  | High concurrency for key-value pairs         |
| CopyOnWriteArrayList  | Yes         | No                  | Frequent reads, infrequent writes            |
| CopyOnWriteArraySet   | Yes         | No                  | Thread-safe set with frequent reads          |
| ConcurrentLinkedQueue | Yes         | No                  | High concurrency for FIFO operations         |
| ConcurrentLinkedDeque | Yes         | No                  | High concurrency for double-ended operations |
| LinkedBlockingQueue   | Yes         | Yes                 | Blocking FIFO queue for producer-consumer    |
| PriorityBlockingQueue | Yes         | Yes                 | Blocking queue with priority ordering        |

**Summary**

Concurrent collections in Java provide a robust and efficient way to handle multithreaded operations on collections. By using these collections, you can improve the performance and reliability of your applications. Understanding the appropriate use cases and how to implement these collections will help you make better design decisions in your concurrent applications.

Each collection has its own strengths and is designed for specific scenarios. For instance, `ConcurrentHashMap` is excellent for high-concurrency scenarios with frequent reads and writes, while `CopyOnWriteArrayList` is better suited for situations with frequent reads but infrequent writes. Similarly, blocking queues like `LinkedBlockingQueue` and `PriorityBlockingQueue` are essential for implementing producer-consumer patterns where threads need to wait for available resources.

By choosing the right concurrent collection, you can avoid common concurrency issues such as race conditions, deadlocks, and excessive contention, leading to more scalable and maintainable code.

This completes our exploration of concurrent collections in Java. We hope this article has provided you with a comprehensive understanding of how to use these collections effectively in your multithreaded applications.











