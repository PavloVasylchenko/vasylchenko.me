---
title: "Понимание ReentrantLock в Java"
date: 2024-06-18T14:11:55Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

В Java важно правильно управлять конкурентным доступом к общим ресурсам: это сохраняет целостность данных и предотвращает race condition. Пакет `java.util.concurrent.locks` предоставляет несколько механизмов блокировки, один из самых часто используемых — `ReentrantLock`.

**Что такое ReentrantLock?**

`ReentrantLock` — примитив синхронизации с более гибким API, чем у обычного блока `synchronized`. Он предоставляет явные операции захвата и освобождения блокировки. Слово reentrant означает, что поток, уже владеющий lock, может захватить его повторно и не заблокирует сам себя. Внутренний счётчик владения увеличивается при каждом `lock()` и уменьшается при каждом `unlock()`.

**Основные возможности ReentrantLock**

1. **Реентерабельность.** Один поток может несколько раз захватить ту же блокировку. Для полного освобождения число вызовов `unlock()` должно соответствовать числу успешных захватов.
2. **Справедливость.** Lock может быть справедливым или несправедливым. Справедливый вариант обычно передаёт доступ потоку, ожидающему дольше всех; несправедливый допускает barging, когда новый поток опережает очередь.
3. **Condition.** Метод `newCondition()` создаёт условия, с помощью которых потоки могут ждать определённого состояния и получать сигнал о его изменении.
4. **Прерываемое ожидание.** `lockInterruptibly()` позволяет прервать поток, пока он ожидает блокировку.
5. **Попытка без ожидания.** `tryLock()` возвращает результат немедленно или может ждать в пределах заданного тайм-аута.

**Базовое использование ReentrantLock**

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;

    public void increment() {
        lock.lock();
        try {
            counter++;
        } finally {
            lock.unlock();
        }
    }

    public int getCounter() {
        lock.lock();
        try {
            return counter;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockExample example = new ReentrantLockExample();

        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("Final counter value: " + example.getCounter());
    }
}
```

Здесь `ReentrantLock` защищает методы `increment()` и `getCounter()` от одновременного доступа. Вызов `lock()` захватывает блокировку, а `unlock()` обязательно находится в `finally`. Благодаря этому lock будет освобождён даже при исключении внутри критической секции. Размещать потенциально аварийный код между `lock()` и `try` не следует.

**Расширенные возможности**

1. **Справедливая блокировка.** Передайте `true` конструктору:

   ```java
   Lock fairLock = new ReentrantLock(true);
   ```

   Это уменьшает вероятность голодания долго ожидающих потоков, хотя может снизить общую пропускную способность.
2. **Try Lock.** Попытка без блокирования позволяет выбрать альтернативный путь:

   ```java
   if (lock.tryLock()) {
       try {
           // Операции под защитой lock
       } finally {
           lock.unlock();
       }
   } else {
       // Блокировку получить не удалось
   }
   ```
3. **Условия.** Они подходят для более сложной координации:

   ```java
   Lock lock = new ReentrantLock();
   Condition condition = lock.newCondition();

   public void awaitCondition() throws InterruptedException {
       lock.lock();
       try {
           condition.await();
       } finally {
           lock.unlock();
       }
   }

   public void signalCondition() {
       lock.lock();
       try {
           condition.signal();
       } finally {
           lock.unlock();
       }
   }
   ```

`await()` атомарно освобождает lock и переводит поток в ожидание. После сигнала поток должен снова захватить блокировку, прежде чем продолжить. На практике условие следует проверять в цикле, чтобы защититься от ложных пробуждений и изменений состояния другими потоками.

**Когда ReentrantLock предпочтительнее synchronized**

1. Нужен справедливый порядок для уменьшения starvation.
2. Ожидающий блокировку поток должен реагировать на interruption.
3. Требуется неблокирующая или ограниченная по времени попытка захвата.
4. Нужны несколько condition-очередей для сложного протокола ожидания и уведомления.

**Итоги**

`ReentrantLock` — мощный и гибкий механизм синхронизации. Он расширяет возможности `synchronized`, предоставляя справедливость, прерываемый захват, `tryLock()` и condition variables. За эту гибкость приходится платить явным управлением жизненным циклом: каждый успешный захват должен освобождаться в `finally`. При правильном использовании `ReentrantLock` помогает создавать надёжный и поддерживаемый многопоточный код.
