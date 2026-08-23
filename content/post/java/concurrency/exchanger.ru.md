---
title: "Понимание Exchanger в Java"
date: 2024-06-21T07:16:11Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Что такое Exchanger в Java**

Класс `Exchanger` создаёт точку синхронизации, в которой два потока образуют пару и обмениваются объектами. Каждый поток передаёт свой объект методу `exchange()`, ждёт партнёра, а после встречи получает объект, предоставленный вторым потоком.

Такой механизм полезен, когда двум потокам нужно напрямую передавать данные друг другу. Например, producer может заполнять буфер, consumer — обрабатывать предыдущий, а затем они меняются буферами без общей очереди. `Exchanger` отвечает одновременно за передачу значения и синхронизацию момента обмена.

**Основные возможности Exchanger**

1. **Обмен объектами.** Два потока передают и получают значения одного параметризованного типа.
2. **Блокирующая операция.** `exchange()` ждёт, пока в той же точке не появится партнёр.
3. **Тайм-аут.** Перегруженный метод позволяет ограничить время ожидания и избежать вечной блокировки.
4. **Попарная работа.** Если приходит больше двух потоков, `Exchanger` объединяет их в пары; заранее определить конкретного партнёра нельзя.

**Базовый пример Exchanger**

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
                String response = exchanger.exchange(data);
                System.out.println("Producer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data);
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

В примере:

1. **Инициализация.** Создаётся `Exchanger<String>`, поэтому участники обмениваются строками.
2. **Producer и Consumer.** Оба класса реализуют `Runnable`, подготавливают собственные данные и вызывают один экземпляр `exchanger.exchange()`.
3. **Выполнение.** Первый пришедший поток блокируется. Когда второй достигает точки обмена, значения атомарно меняются местами, и оба потока продолжают работу.

Если один из участников будет прерван во время ожидания, `exchange()` выбросит `InterruptedException`. Код восстанавливает interrupt status, чтобы вызывающий уровень мог корректно обработать отмену.

**Exchanger с тайм-аутом**

Если партнёр может не дойти до точки синхронизации, бессрочное ожидание нежелательно. Версия метода с тайм-аутом позволяет завершить операцию предсказуемо.

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
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);
                System.out.println("Producer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (TimeoutException e) {
                System.out.println("Producer timed out");
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (TimeoutException e) {
                System.out.println("Consumer timed out");
            }
        }
    }
}
```

Оба потока ждут не более пяти секунд. Если партнёр не появился, возникает `TimeoutException`, и приложение может повторить попытку, освободить ресурс или завершить задачу. Обрабатывать timeout и interruption лучше отдельно: тайм-аут является ожидаемым результатом протокола, а interruption обычно означает запрос на отмену.

**Когда использовать Exchanger**

- для попарного обмена заполненным и пустым буфером;
- для сравнения или объединения промежуточных результатов двух работников;
- для конвейера, где две стадии должны встречаться после каждого раунда;
- в тестах конкурентного кода, когда требуется точная точка синхронизации двух потоков.

Для произвольного числа producers и consumers обычно удобнее `BlockingQueue`. `Exchanger` лучше всего работает именно там, где протокол естественно состоит из пар.

**Итоги**

`Exchanger` объединяет передачу данных и синхронизацию двух потоков в одной операции. Он делает попарное взаимодействие явным и не требует отдельного общего контейнера. Тайм-аут защищает от вечного ожидания, а корректная обработка interruption позволяет остановить задачу. В подходящих сценариях этот класс упрощает координацию и делает многопоточный алгоритм понятнее.
