---
title: "Розуміння Exchanger у Java"
date: 2024-06-21T07:16:12Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Розуміння Exchanger у Java**

Клас `Exchanger` у Java є точкою синхронізації, в якій потоки можуть попарно обмінюватися елементами. Кожен потік передає якийсь об’єкт у метод exchange, з’єднується з потоком-партнером і отримує об’єкт свого партнера на виході.

Це корисно в сценаріях, коли два потоки повинні обмінюватися даними один з одним, наприклад, у схемі виробник-споживач, де виробники генерують дані, а споживачі їх обробляють.

**Основні особливості Exchanger**

1. **Обмін об’єктами**: Дозволяє двом потокам обмінюватися даними.
2. **Блокуюча операція**: Метод exchange є блокуючою операцією, тобто він чекатиме, доки інший потік буде готовий до обміну.
3. **Тайм-аути**: Ви можете вказати тайм-аут, щоб уникнути нескінченного блокування.

**Базове використання Exchanger**

Ось простий приклад, що демонструє використання `Exchanger`:

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
                String data = "Дані від виробника";
                System.out.println("Виробник: " + data);
                String response = exchanger.exchange(data);  // Обмін даними зі споживачем
                System.out.println("Виробник отримав: " + response);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Дані від споживача";
                System.out.println("Споживач: " + data);
                String response = exchanger.exchange(data);  // Обмін даними з виробником
                System.out.println("Споживач отримав: " + response);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

У цьому прикладі:

1. **Ініціалізація Exchanger**: Створюється об’єкт Exchanger для обміну даними типу String.
2. **Реалізація виробника та споживача**: Класи Producer і Consumer реалізують Runnable і обмінюються даними за допомогою методу exchanger.exchange.
3. **Запуск потоків**: Запускаються два потоки: один для виробника і один для споживача. Виробник і споживач обмінюються даними та виводять отримані дані.

**Розширене використання Exchanger з тайм-аутом**

Ви також можете використовувати метод exchange з тайм-аутом, щоб уникнути нескінченного блокування, якщо інший потік не досягне точки обміну протягом зазначеного часу.

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
                String data = "Дані від виробника";
                System.out.println("Виробник: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);  // Обмін даними з тайм-аутом
                System.out.println("Виробник отримав: " + response);
            } catch (InterruptedException | TimeoutException e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Дані від споживача";
                System.out.println("Споживач: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);  // Обмін даними з тайм-аутом
                System.out.println("Споживач отримав: " + response);
            } catch (InterruptedException | TimeoutException e) {
                e.printStackTrace();
            }
        }
    }
}
```

У цьому розширеному прикладі:

1. **Обробка тайм-ауту**: Метод exchange використовується з тайм-аутом 5 секунд. Якщо інший потік не досягне точки обміну протягом цього часу, кидається `TimeoutException`.
2. **Обробка винятків**: Обробляються як `InterruptedException`, так і `TimeoutException`, щоб впоратися з ситуацією тайм-ауту.

**Висновок**

Клас `Exchanger` у Java забезпечує потужний механізм для двох потоків обмінюватися даними в точці синхронізації. Він особливо корисний у сценаріях, коли парні потоки повинні обмінюватися інформацією. Розуміючи та використовуючи Exchanger, ви можете полегшити пряме спілкування між потоками у структурованому та синхронізованому режимі, покращуючи координацію та продуктивність ваших багатопотокових додатків.
