---
title: "Использование Semaphore в Java"
date: 2024-06-20T08:16:17Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Что такое Semaphore в Java**

`Semaphore` — примитив синхронизации, который ограничивает число потоков, одновременно обращающихся к общему ресурсу. Он входит в пакет `java.util.concurrent` и особенно полезен для ресурсов с фиксированной ёмкостью: пула соединений с базой данных, сетевых сокетов, принтеров или файловых дескрипторов.

**Основные свойства Semaphore**

1. **Разрешения.** Семафор хранит набор permits. Перед работой поток получает разрешение, а после завершения возвращает его.
2. **Блокирующий и неблокирующий захват.** `acquire()` ждёт свободное разрешение, тогда как `tryAcquire()` позволяет немедленно продолжить по альтернативному пути.
3. **Справедливость.** Семафор может быть справедливым или несправедливым. Справедливый режим обычно выдаёт разрешения в порядке запросов, уменьшая риск starvation.
4. **Несколько разрешений.** Один поток может запросить и освободить более одного permit, если операция занимает несколько единиц ресурса.

**Базовый пример Semaphore**

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
         boolean acquired = false;
         try {
            System.out.println(Thread.currentThread().getName() + " is waiting for a permit.");
            semaphore.acquire();
            acquired = true;
            System.out.println(Thread.currentThread().getName() + " acquired a permit.");

            // Имитация работы с общим ресурсом
            Thread.sleep(2000);

            System.out.println(Thread.currentThread().getName() + " releasing a permit.");
         } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
         } finally {
            if (acquired) {
               semaphore.release();
            }
         }
      }
   }
}
```

Семафор создаётся с тремя разрешениями. Запускаются десять потоков, но только три из них могут одновременно пройти через `acquire()` и выполнить критическую секцию. Остальные ждут. После `release()` разрешение становится доступно следующему участнику.

Флаг `acquired` важен: если поток был прерван до успешного захвата, он не должен вызывать `release()`, иначе количество разрешений ошибочно увеличится.

**Расширенное использование Semaphore**

1. **Попытка без блокирования.** Поток может проверить доступность ресурса и немедленно выбрать другой сценарий:

   ```java
   if (semaphore.tryAcquire()) {
       try {
           // Критическая секция
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " could not acquire a permit.");
   }
   ```
2. **Захват с тайм-аутом.** Ограниченное ожидание помогает не зависнуть при перегрузке:

   ```java
   if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
       try {
           // Критическая секция
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " timed out waiting for a permit.");
   }
   ```
3. **Справедливый семафор.** Второй аргумент конструктора включает очередь в порядке ожидания:

   ```java
   Semaphore fairSemaphore = new Semaphore(MAX_PERMITS, true);
   ```

Справедливость полезна, если нельзя допустить длительное ожидание отдельных потоков, но дополнительное управление очередью может снизить пропускную способность.

**Практические сценарии**

1. **Ограничение параллелизма.** Контроль числа одновременных запросов к внешнему сервису или дорогой операции.
2. **Пулы соединений.** Каждый permit соответствует доступному соединению с базой данных.
3. **Управление ресурсами.** Ограничение доступа к небольшому числу принтеров, файлов или устройств.
4. **Защита от перегрузки.** Лишние задачи ждут или отклоняются, вместо того чтобы одновременно занять память и CPU.

**Итоги**

`Semaphore` предоставляет надёжный способ ограничивать конкурентный доступ к общим ресурсам. В отличие от обычной взаимной блокировки, он может пропускать заданное число потоков одновременно. Важно освобождать только действительно полученные разрешения и всегда делать это в `finally`. Правильное применение семафора помогает эффективно использовать ограниченные ресурсы и уменьшать конкуренцию в многопоточных приложениях.
