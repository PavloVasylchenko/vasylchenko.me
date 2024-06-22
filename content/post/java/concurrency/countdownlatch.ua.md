---
title: "Розуміння та використання CountDownLatch у Java"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch у Java з прикладами використання**

У світі багатопоточності в Java синхронізація декількох потоків є загальною проблемою. 
Клас `CountDownLatch` у Java є потужним засобом синхронізації, який допомагає ефективно впоратися з цією проблемою. 
Він дозволяє одному або кільком потокам чекати, доки набір операцій, які виконуються в інших потоках, не завершиться.

**Що таке CountDownLatch?**

Уявіть, що ви організовуєте перегони. Ви хочете, щоб усі бігуни почали бігти одночасно. Для цього ви можете використовувати зворотний відлік. Коли зворотний відлік досягає нуля, гонка починається. CountDownLatch працює подібним чином. Він ініціалізується з рахунком, який представляє кількість операцій або потоків, які повинні завершитися перед тим, як продовжити.

**Як працює CountDownLatch?**

1. **Ініціалізація**: CountDownLatch ініціалізується з рахунком. Цей рахунок представляє кількість разів, які метод `countDown()` повинен бути викликаний перед тим, як засувка відкриється.
2. **Зменшення рахунку**: Кожен виклик методу `countDown()` зменшує рахунок. Коли рахунок досягає нуля, всі очікуючі потоки звільняються.
3. **Очікування**: Метод `await()` блокується, доки рахунок не досягне нуля.

**Простий приклад**

Розглянемо простий приклад, щоб побачити, як працює CountDownLatch:

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
            latch.await();  // Головний потік чекає, поки рахунок засувки не досягне нуля
            System.out.println("Усі завдання завершені. Головний потік продовжує роботу.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Імітація роботи за допомогою сну
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " завершено.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                latch.countDown();  // Зменшення рахунку засувки
            }
        }
    }
}
```

У цьому прикладі:

- CountDownLatch ініціалізується з рахунком 3.
- Запускаються три потоки, кожен з яких представляє завдання.
- Кожне завдання імітує роботу, сплячи одну секунду, а потім викликає countDown(), щоб зменшити рахунок засувки.
- Головний потік чекає на latch.await(), доки всі завдання не завершаться, тобто доки рахунок не досягне нуля.

**Розширений приклад з тайм-аутом**

Іноді вам не потрібно чекати нескінченно. Ви можете використовувати метод await з тайм-аутом, щоб уникнути цього.

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
            boolean completed = latch.await(5, TimeUnit.SECONDS);  // Головний потік чекає з тайм-аутом
            if (completed) {
                System.out.println("Усі завдання завершені. Головний потік продовжує роботу.");
            } else {
                System.out.println("Час очікування вичерпано до завершення всіх завдань.");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Імітація роботи за допомогою сну
                Thread.sleep(2000);  // Довший сон, щоб потенційно викликати тайм-аут
                System.out.println(Thread.currentThread().getName() + " завершено.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                latch.countDown();  // Зменшення рахунку засувки
            }
        }
    }
}
```
У цьому прикладі:

- Метод await використовується з тайм-аутом 5 секунд.
- Кожне завдання імітує довшу роботу, сплячи 2 секунди.
- Якщо всі завдання завершуються в межах тайм-ауту, головний потік продовжує роботу нормально. Інакше він виводить повідомлення про тайм-аут.

**Реальні випадки використання CountDownLatch**

1. **Одночасний запуск декількох потоків**: Іноді потрібно забезпечити, щоб набір потоків почав обробку одночасно. 
Використовуючи CountDownLatch, ініціалізований значенням 1, ви можете змусити всі потоки чекати, поки засувка не буде зменшена один раз, забезпечуючи синхронізований старт.
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
                System.out.println("Готові... Увага...");
                Thread.sleep(1000); // Імітація підготовчої роботи
                startLatch.countDown();  // Сигнал всім потокам почати
                System.out.println("Марш!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        static class Worker implements Runnable {
            @Override
            public void run() {
                try {
                    startLatch.await();  // Чекати сигналу для початку роботи
                    System.out.println(Thread.currentThread().getName() + " працює.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    ```
2. **Очікування завершення декількох потоків**: Ви можете використовувати CountDownLatch, щоб чекати завершення набору потоків перед продовженням роботи. Це корисно для агрегування результатів від декількох потоків.
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
                completionLatch.await();  // Головний потік чекає завершення всіх завдань
                System.out.println("Усі підзадачі завершені. Агрегація результатів.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        static class SubTask implements Runnable {
            @Override
            public void run() {
                try {
                    // Імітація роботи за допомогою сну
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName() + " підзадача завершена.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    completionLatch.countDown();  // Зменшення рахунку засувки
                }
            }
        }
    }
    ```
   
**Висновок**

Клас `CountDownLatch` у Java забезпечує надійний механізм для синхронізації декількох потоків, забезпечуючи завершення завдань у координованому режимі. 
Незалежно від того, чи управляєте ви запуском декількох потоків, очікуєте на їх завершення або розділяєте завдання на під
