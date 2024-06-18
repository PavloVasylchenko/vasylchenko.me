---
title: "Розуміння Reentrant Lock в Java"
date: 2024-06-18T14:12:03Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

У Java управління одночасним доступом до спільних ресурсів є надзвичайно важливим для забезпечення цілісності даних та запобігання проблемам, таким як умови гонки. Пакет java.util.concurrent.locks забезпечує різні механізми блокування для досягнення цієї мети, і одним з найпоширеніших є ReentrantLock.

**Що таке ReentrantLock?**

ReentrantLock - це примітив синхронізації, який пропонує більше гнучкості та функцій порівняно з традиційними блоками synchronized. Він є частиною пакету java.util.concurrent.locks і надає більш розширений API для операцій блокування та розблокування. Як випливає з назви, повторюваний блок (reentrant lock) дозволяє потоку, який утримує блокування, повторно заходити в блокування кілька разів без виникнення deadlock.

**Основні характеристики ReentrantLock**

1. **Повторюваність (Reentrancy)**: Замок може бути захоплений одним і тим же потоком кілька разів. Це означає, що якщо потік вже утримує замок і намагається захопити його знову, він успішно захопить його без блокування.
2. **Справедливість (Fairness)**: `ReentrantLock` може бути створений як справедливий або несправедливий. Справедливий замок гарантує, що потік, який найдовше чекає, отримує доступ до замка першим, тоді як несправедливий замок може дозволити потокам “підрізати чергу”.
3. **Умовні змінні (Condition Variables)**: `ReentrantLock` надає умовні змінні через метод `newCondition()`, дозволяючи потокам чекати на виконання певних умов.
4. **Переривання при захопленні замка (Interruptible Lock Acquisition)**: Потоки можуть бути перервані під час очікування на захоплення замка.
5. **Спроба захоплення замка (Try Lock)**: Метод `tryLock()` дозволяє потоку спробувати захопити замок без блокування.

**Основне використання ReentrantLock**

Ось простий приклад, щоб продемонструвати використання `ReentrantLock`:

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;

    public void increment() {
        lock.lock();  // Захоплюємо замок
        try {
            counter++;
        } finally {
            lock.unlock();  // Завжди розблоковуємо замок у блоці finally
        }
    }

    public int getCounter() {
        lock.lock();  // Захоплюємо замок
        try {
            return counter;
        } finally {
            lock.unlock();  // Завжди розблоковуємо замок у блоці finally
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockExample example = new ReentrantLockExample();

        // Створюємо 1000 потоків, які інкрементують лічильник
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Чекаємо завершення всіх потоків
        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("Фінальне значення лічильника: " + example.getCounter());
    }
}
```

У цьому прикладі:

- `ReentrantLock` використовується для забезпечення потокобезпеки методів `increment` та `getCounter`.
- Метод `lock.lock()` захоплює замок, а `lock.unlock()` розблоковує його. 
Виклик unlock розташований у блоці `finally`, щоб гарантувати розблокування навіть у разі виникнення виключення.

**Розширені можливості**

1. **Справедливий замок**: Щоб створити справедливий ReentrantLock, передайте true до конструктора:
    ```java 
    Lock fairLock = new ReentrantLock(true);
    ```
    Це гарантує, що потік, який найдовше чекає, отримає замок першим.
2. **Спроба захоплення замка**: Щоб уникнути блокування, використовуйте tryLock():
    ```java
    if (lock.tryLock()) {
        try {
            // Виконання операцій під захистом замка
        } finally {
            lock.unlock();
        }
    } else {
        // Обробка випадку, коли замок не був захоплений
    }
    ```
3. **Умовні змінні**: Використовуйте умовні змінні для складнішої координації потоків:
    ```java
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    
    public void awaitCondition() throws InterruptedException {
        lock.lock();
        try {
            condition.await();  // Очікування до сигналу
        } finally {
            lock.unlock();
        }
    }
    
    public void signalCondition() {
        lock.lock();
        try {
            condition.signal();  // Сигнал одному потоку, який чекає
        } finally {
            lock.unlock();
        }
    }
    ```

**Висновок**

`ReentrantLock` забезпечує потужний та гнучкий механізм для синхронізації потоків у Java. 
Він розширює можливості традиційних блоків `synchronized`, пропонуючи такі функції, як повторюваність, справедливість, переривання при захопленні замка, несинхронні спроби та умовні змінні. 
Розуміння та ефективне використання ReentrantLock може призвести до більш надійних та підтримуваних багатопоточних додатків.