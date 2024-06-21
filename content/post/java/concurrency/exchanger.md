---
title: "Understanding Exchanger in Java"
date: 2024-06-21T07:16:11Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Understanding Exchanger in Java**

The `Exchanger` class in Java is a synchronization point at which threads can pair and exchange elements within pairs. Each thread presents some object on entry to the exchange method, matches with a partner thread, and receives its partner’s object on return.

This is useful in scenarios where two threads need to exchange data with each other, such as in a producer-consumer pattern where producers generate data and consumers process it.

**Key Features of Exchanger**

1. **Exchange Objects**: It allows two threads to exchange data.
2. **Blocking Operation**: The exchange method is a blocking operation, meaning it will wait until another thread is ready to exchange.
3. **Timeouts**: You can specify a timeout to avoid indefinite blocking.

**Basic Usage of Exchanger**

Here’s a simple example to illustrate the usage of `Exchanger`:

```java
import java.util.concurrent.Exchanger;

public class ExchangerExample {
    private static final Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new Producer()).start();
        new Thread(new Consumer()).start();
    }

    static class Producer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Producer";
                System.out.println("Producer: " + data);
                String response = exchanger.exchange(data);  // Exchange data with consumer
                System.out.println("Producer received: " + response);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data);  // Exchange data with producer
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

In this example:

1. **Exchanger Initialization**: An Exchanger object is created for exchanging String data.
2. **Producer and Consumer Implementation**: The Producer and Consumer classes implement Runnable and exchange data using the exchanger.exchange method.
3. **Thread Execution**: Two threads are started, one for the producer and one for the consumer. The producer and consumer exchange data and print the received data.


**Advanced Usage of Exchanger with Timeout**

You can also use the exchange method with a timeout to avoid indefinite blocking if the other thread does not reach the exchange point within the specified time.

```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class ExchangerTimeoutExample {
    private static final Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new Producer()).start();
        new Thread(new Consumer()).start();
    }

    static class Producer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Producer";
                System.out.println("Producer: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);  // Exchange data with timeout
                System.out.println("Producer received: " + response);
            } catch (InterruptedException | TimeoutException e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);  // Exchange data with timeout
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException | TimeoutException e) {
                e.printStackTrace();
            }
        }
    }
}
```

In this advanced example:

1. **Timeout Handling**: The exchange method is used with a timeout of 5 seconds. If the other thread does not reach the exchange point within this time, a `TimeoutException` is thrown.
2. **Exception Handling**: Both `InterruptedException` and `TimeoutException` are handled to manage the timeout scenario.

**Summary**

The `Exchanger` class in Java provides a powerful mechanism for two threads to exchange data at a synchronization point. It is particularly useful in scenarios where paired threads need to swap information. By understanding and leveraging Exchanger, you can facilitate direct communication between threads in a structured and synchronized manner, enhancing the coordination and performance of your multithreaded applications.


