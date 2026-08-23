---
title: "Использование CyclicBarrier в Java"
date: 2024-06-19T13:15:18Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Как работает CyclicBarrier в Java**

Класс `CyclicBarrier` — это средство синхронизации, позволяющее группе потоков ждать друг друга в общей точке барьера. Он полезен, когда несколько потоков независимо выполняют одну фазу работы, а перейти к следующей могут только после того, как все участники достигнут заданного состояния.

В отличие от `CountDownLatch`, барьер можно использовать повторно после освобождения ожидающих потоков. Поэтому `CyclicBarrier` хорошо подходит для циклических и итеративных алгоритмов, где одни и те же участники синхронизируются после каждой фазы.

**Основные возможности CyclicBarrier**

- **Повторное использование.** Когда все потоки достигли барьера и были освобождены, начинается новый цикл ожидания.
- **Необязательное действие.** В конструкторе можно передать `Runnable`, который выполнит последний прибывший поток до того, как остальные продолжат работу.
- **Фиксированное число участников.** Количество parties задаётся при создании и определяет, сколько вызовов `await()` требуется для открытия барьера.

**Базовый пример CyclicBarrier**

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
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

В этом примере:

- **Инициализация.** Барьер получает число потоков `NUMBER_OF_THREADS` и необязательное действие, которое запускается после прибытия последнего участника.
- **Задача.** Каждый `Task` вызывает `barrier.await()` и останавливается в этой точке, пока туда не придут остальные.
- **Выполнение.** Запускаются три потока. После третьего вызова `await()` выполняется barrier action, а затем все участники продолжают работу.

**Расширенные возможности CyclicBarrier**

1. **Сломанный барьер.** Если один из ожидающих потоков прерван или превышает тайм-аут, барьер переходит в broken state. Остальные участники получают `BrokenBarrierException`. Состояние проверяется через `barrier.isBroken()`.
2. **Тайм-аут.** Перегруженный `await()` позволяет не ждать бесконечно:

   ```java
   barrier.await(5, TimeUnit.SECONDS);
   ```
3. **Сброс.** `barrier.reset()` возвращает объект в исходное состояние. Ожидающие в этот момент потоки получат исключение, поэтому reset необходимо согласовывать на уровне протокола приложения.

**Пример с тайм-аутом и сбросом**

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
                barrier.await(5, TimeUnit.SECONDS);
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException | TimeoutException e) {
                e.printStackTrace();
                if (barrier.isBroken()) {
                    System.out.println("Barrier is broken, resetting...");
                    barrier.reset();
                }
            }
        }
    }
}
```

Каждый поток ждёт не более пяти секунд. Если все участники успели, действие барьера выполняется и начинается следующая фаза. Если ожидание превышено, возникает `TimeoutException`, а барьер становится сломанным. Пример проверяет состояние и сбрасывает объект, чтобы его можно было использовать снова.

В реальной системе важно избежать одновременных несогласованных вызовов `reset()` несколькими потоками. Обычно решение о восстановлении принимает управляющий компонент, который также знает, можно ли безопасно повторить фазу.

**Итоги**

`CyclicBarrier` синхронизирует фиксированное число потоков в общей точке. Возможность повторного использования и barrier action делают его удобным для параллельных вычислений, состоящих из нескольких раундов. Тайм-ауты и обработка broken state помогают не зависнуть при сбое одного участника. Правильно спроектированный протокол с `CyclicBarrier` позволяет надёжно координировать циклическую многопоточную работу.
