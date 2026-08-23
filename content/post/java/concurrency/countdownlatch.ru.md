---
title: "CountDownLatch в Java с примерами"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch в Java с примерами**

При разработке конкурентных Java-приложений часто необходимо синхронизировать несколько потоков. Класс `CountDownLatch` помогает организовать такую координацию: один или несколько потоков могут ждать, пока другие потоки завершат заданный набор операций.

**Что такое CountDownLatch?**

Представьте подготовку к забегу. Все участники должны стартовать одновременно, поэтому они ждут окончания обратного отсчёта. Когда счётчик достигает нуля, забег начинается. `CountDownLatch` работает похожим образом: при создании он получает число событий или задач, которые должны произойти, прежде чем ожидающие потоки смогут продолжить работу.

Latch является одноразовым. После достижения нуля его счётчик нельзя увеличить или вернуть к исходному значению. Если нужен многократно используемый барьер для одной группы потоков, лучше рассмотреть `CyclicBarrier`.

**Как работает CountDownLatch?**

1. **Инициализация.** Объект создаётся с начальным count — количеством вызовов `countDown()`, необходимых для открытия latch.
2. **Уменьшение счётчика.** Каждый `countDown()` уменьшает значение на единицу. Дополнительные вызовы после нуля ничего не меняют.
3. **Ожидание.** `await()` блокирует вызывающий поток, пока счётчик не станет равен нулю.
4. **Гарантия видимости.** Действия до `countDown()` становятся видимыми потоку, который успешно вернулся из `await()`.

**Базовый пример**

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
            latch.await();  // Главный поток ждёт нулевого счётчика
            System.out.println("All tasks are completed. Main thread resumes.");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Имитация работы
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();  // Уменьшаем счётчик в любом случае
            }
        }
    }
}
```

В этом примере `CountDownLatch` получает начальное значение 3. Запускаются три задачи, каждая имитирует работу в течение секунды и затем вызывает `countDown()`. Главный поток останавливается на `latch.await()` и возобновляется только после завершения всех задач.

Вызов находится в `finally`, поэтому ошибка отдельной задачи не оставит главный поток в вечном ожидании. Такое решение означает «задача закончила попытку», а не обязательно «задача завершилась успешно»; реальный код должен отдельно сохранять ошибки или результаты.

**Пример с тайм-аутом**

Иногда ждать бесконечно нельзя. Перегруженный `await()` возвращает `boolean` и позволяет ограничить ожидание.

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
            boolean completed = latch.await(5, TimeUnit.SECONDS);
            if (completed) {
                System.out.println("All tasks are completed. Main thread resumes.");
            } else {
                System.out.println("Timeout occurred before all tasks completed.");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();
            }
        }
    }
}
```

Главный поток ждёт не более пяти секунд. Если за это время все три задачи уменьшили счётчик, `await()` возвращает `true`. В противном случае он возвращает `false`, и приложение может отменить незавершённые операции, сообщить об ошибке или перейти к резервному сценарию. Сам тайм-аут не останавливает worker threads автоматически.

**Практические сценарии CountDownLatch**

1. **Одновременный запуск нескольких потоков.** Latch со значением 1 удерживает работников до общего стартового сигнала:

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
               Thread.sleep(1000); // Подготовительная работа
               startLatch.countDown();
               System.out.println("Go!");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class Worker implements Runnable {
           @Override
           public void run() {
               try {
                   startLatch.await();
                   System.out.println(Thread.currentThread().getName() + " is working.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               }
           }
       }
   }
   ```

   Все пять работников запускаются и ждут. Единственный `countDown()` открывает latch и позволяет им продолжить почти одновременно. Этот приём удобен в нагрузочных тестах или при запуске группы задач после завершения инициализации.

2. **Ожидание завершения нескольких потоков.** Latch со значением, равным числу задач, позволяет агрегировать их результаты:

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
               completionLatch.await();
               System.out.println("All subtasks completed. Aggregating results.");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class SubTask implements Runnable {
           @Override
           public void run() {
               try {
                   Thread.sleep(1000);
                   System.out.println(Thread.currentThread().getName() + " subtask completed.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               } finally {
                   completionLatch.countDown();
               }
           }
       }
   }
   ```

   Главный поток приступает к агрегации только после того, как все subtasks завершили попытку выполнения. В современном коде похожую задачу также решают `ExecutorService`, `Future` или `CompletableFuture`, но `CountDownLatch` остаётся простым способом выразить зависимость от фиксированного числа событий.

**Итоги**

`CountDownLatch` — надёжный одноразовый механизм координации потоков. Он подходит для общего старта, ожидания завершения набора задач и разделения большой работы на независимые части. Важно правильно выбрать начальное значение, гарантировать `countDown()` для каждого ожидаемого события и обрабатывать interruption. Если ожидание не должно быть бесконечным, используйте тайм-аут и явно определите поведение незавершённых задач.
