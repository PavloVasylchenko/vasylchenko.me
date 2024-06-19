---
title: "Використання CyclicBarrier у Java"
date: 2024-06-19T13:25:32Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Використання CyclicBarrier у Java**

Клас `CyclicBarrier` у Java є засобом синхронізації, що дозволяє групі потоків чекати один на одного до досягнення загальної бар'єрної точки. Це корисно в сценаріях, коли потрібно, щоб декілька потоків почали обробку одночасно після досягнення певного стану.
На відміну від `CountDownLatch`, `CyclicBarrier` можна використовувати повторно після того, як всі очікуючі потоки будуть звільнені, що робить його підходящим для циклічних або ітеративних операцій.

**Основні особливості CyclicBarrier**

- **Повторне використання**: Після звільнення всіх очікуючих потоків, бар'єр можна використовувати повторно.
- **Необов'язковий Runnable**: Можна вказати необов'язкову задачу `Runnable`, яка буде виконана після досягнення бар'єру всіма потоками.

**Базове використання CyclicBarrier**

Ось простий приклад, що демонструє використання `CyclicBarrier`:

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("Усі учасники досягли бар'єру, починаємо гру");
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
                System.out.println(Thread.currentThread().getName() + " чекає на бар'єрі.");
                barrier.await();  // Очікування на бар'єрі
                System.out.println(Thread.currentThread().getName() + " перейшов через бар'єр.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}

```

У цьому прикладі:

- **Ініціалізація бар'єру**: `CyclicBarrier` ініціалізується з кількістю потоків (`NUMBER_OF_THREADS`), які повинні чекати на бар'єр, та необов'язковою задачею Runnable, яка виконується після досягнення бар'єру всіма потоками.
- **Реалізація завдання**: Клас Task реалізує `Runnable` і викликає `barrier.await()`. Кожен потік чекає на бар'єрі, доки всі потоки не досягнуть його.
- **Виконання потоків**: Створюються три потоки, і кожен з них чекає на бар'єрі. Після того, як всі потоки досягли бар'єру, виконується дія бар'єра, і потоки продовжують виконання.

**Розширене використання CyclicBarrier**

1. **Обробка зламаного бар'єру**: Якщо один з очікуючих потоків переривається або перевищує час очікування, 
`CyclicBarrier` вважається зламаним. 
Ви можете перевірити, чи зламаний бар'єр, використовуючи `barrier.isBroken()` та обробити це відповідно.
2. **Час очікування**: Можна вказати час очікування для методу `await()`, щоб уникнути нескінченного очікування.
    ```java
    barrier.await(5, TimeUnit.SECONDS);
    ```
3. **Скидання бар'єру**: Ви можете скинути бар'єр до початкового стану, використовуючи `barrier.reset()`, 
що може бути корисним, якщо ви хочете використовувати бар'єр на іншій фазі або ітерації.

**Приклад з часом очікування та скиданням**

Ось приклад, що включає обробку часу очікування та скидання бар'єру:

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
            System.out.println("Усі учасники досягли бар'єру, починаємо гру");
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
                System.out.println(Thread.currentThread().getName() + " чекає на бар'єрі.");
                barrier.await(5, TimeUnit.SECONDS);  // Очікування на бар'єрі з тайм-аутом
                System.out.println(Thread.currentThread().getName() + " перейшов через бар'єр.");
            } catch (InterruptedException | BrokenBarrierException | TimeoutException e) {
                e.printStackTrace();
                if (barrier.isBroken()) {
                    System.out.println("Бар'єр зламаний, скидаємо...");
                    barrier.reset();  // Скидання бар'єру, якщо він зламаний
                }
            }
        }
    }
}
```

У цьому розширеному прикладі:

- **Обробка часу очікування**: Кожен потік чекає на бар'єрі з часом очікування 5 секунд. Якщо час очікування перевищено, кидається `TimeoutException`. 
- **Перевірка зламаного бар'єру**: Якщо бар'єр зламаний, потік виводить повідомлення та скидає бар'єр за допомогою `barrier.reset()`.

**Висновок**
Клас `CyclicBarrier` у Java забезпечує потужний механізм для синхронізації фіксованої кількості потоків на загальній 
бар'єрній точці. 
Його повторне використання та необов'язкова дія бар'єра роблять його гнучким інструментом для координації потоків 
у циклічних або ітеративних завданнях. 
Розуміння та використання `CyclicBarrier` дозволяє розробникам створювати 
надійні багатопотокові додатки, які вимагають синхронізації між кількома потоками.
