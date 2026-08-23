---
title: "Конкурентные коллекции в Java с примерами"
date: 2024-06-25T13:46:57Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Конкурентные коллекции в Java с примерами**

В пакете `java.util.concurrent` есть несколько коллекций, специально созданных для многопоточной среды. Они предоставляют потокобезопасные операции и обычно масштабируются лучше, чем обычная коллекция, полностью закрытая одним `synchronized`-блоком. Рассмотрим основные реализации, типичные сценарии и примеры их использования.

**Обзор конкурентных коллекций**

Java предоставляет следующие популярные варианты:

- `ConcurrentHashMap`;
- `CopyOnWriteArrayList`;
- `CopyOnWriteArraySet`;
- `ConcurrentLinkedQueue`;
- `ConcurrentLinkedDeque`;
- `LinkedBlockingQueue`;
- `PriorityBlockingQueue`.

У каждой структуры своя модель конкурентности. Потокобезопасность отдельного метода не делает атомарной произвольную последовательность вызовов, поэтому для составных действий нужно использовать специальные методы коллекции или дополнительную синхронизацию.

**ConcurrentHashMap**

`ConcurrentHashMap` — потокобезопасная реализация map, допускающая конкурентные чтения и обновления. Она не блокирует всю таблицу для каждой операции и предоставляет атомарные методы `putIfAbsent`, `compute`, `merge` и `replace`.

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                map.put(Thread.currentThread().getName() + i, i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> map.forEach((key, value) ->
                System.out.println(Thread.currentThread().getName() + ": " + key + ": " + value));

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

Два writer threads одновременно добавляют элементы, после чего readers обходят карту. Итерация является weakly consistent: она не бросает `ConcurrentModificationException` и может видеть часть изменений, происходящих параллельно.

**CopyOnWriteArrayList**

`CopyOnWriteArrayList` — потокобезопасный вариант `ArrayList`. Каждая изменяющая операция создаёт новую копию внутреннего массива. Чтение и итерация очень дешёвы и не требуют блокировки, но частые записи создают значительные расходы памяти и копирования.

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteArrayListExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                list.add(Thread.currentThread().getName() + i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> {
            for (String item : list) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

Итератор работает со snapshot массива на момент создания и не видит последующие изменения. Коллекция хорошо подходит для списков подписчиков или конфигурации, которые читаются часто, а изменяются редко.

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` — потокобезопасное множество на основе `CopyOnWriteArrayList`. Оно сохраняет уникальность элементов и наследует ту же модель: быстрые стабильные чтения, snapshot-итераторы и дорогие изменения.

```java
import java.util.concurrent.CopyOnWriteArraySet;

public class CopyOnWriteArraySetExample {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                set.add(Thread.currentThread().getName() + i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> {
            for (String item : set) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

Эта структура удобна для небольших множеств listeners или разрешений, где обходов намного больше, чем добавлений и удалений.

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` — неограниченная потокобезопасная FIFO-очередь на связанных узлах. Она использует неблокирующий алгоритм и хорошо масштабируется, когда многие потоки одновременно добавляют и извлекают элементы.

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueExample {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                queue.add(Thread.currentThread().getName() + i);
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        producer1.start();
        producer2.start();

        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable consumerTask = () -> {
            String item;
            while ((item = queue.poll()) != null) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

`poll()` атомарно извлекает head или возвращает `null`. Проверять `isEmpty()` перед `poll()` не нужно: состояние может измениться между двумя вызовами. Очередь не поддерживает блокирующее ожидание элемента и не ограничивает producer, поэтому backpressure необходимо реализовать отдельно.

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` — неблокирующая потокобезопасная двусторонняя очередь. Потоки могут конкурентно добавлять и извлекать элементы с обоих концов.

```java
import java.util.concurrent.ConcurrentLinkedDeque;

public class ConcurrentLinkedDequeExample {
    public static void main(String[] args) {
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                deque.addLast(Thread.currentThread().getName() + i);
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        producer1.start();
        producer2.start();

        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable consumerTask = () -> {
            String item;
            while ((item = deque.pollFirst()) != null) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

Двусторонний доступ полезен для work-stealing, истории задач и алгоритмов, которым нужны операции как FIFO, так и LIFO.

**LinkedBlockingQueue**

`LinkedBlockingQueue` — опционально ограниченная блокирующая FIFO-очередь на связанных узлах. `take()` ждёт появления элемента, а `put()` в bounded queue ждёт свободного места. Это естественная основа для producer-consumer и простой backpressure.

```java
import java.util.concurrent.LinkedBlockingQueue;

public class LinkedBlockingQueueExample {
    public static void main(String[] args) {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

        Runnable producerTask = () -> {
            try {
                for (int i = 0; i < 1000; i++) {
                    queue.put(Thread.currentThread().getName() + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    String item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        new Thread(producerTask).start();
        new Thread(producerTask).start();
        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

Ограничение 10 заставляет быстрых producers ждать, когда consumers не успевают. В реальном приложении нужен протокол завершения: interruption, специальный poison pill или управление через executor.

**PriorityBlockingQueue**

`PriorityBlockingQueue` — неограниченная блокирующая очередь, использующая порядок `PriorityQueue`. `take()` ждёт элемент, но producers никогда не блокируются из-за размера. Элементы должны реализовывать `Comparable` или очередь должна получить `Comparator`.

```java
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityBlockingQueueExample {
    public static void main(String[] args) {
        PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                queue.put((int) (Math.random() * 1000));
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    Integer item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        new Thread(producerTask).start();
        new Thread(producerTask).start();
        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

Очередь гарантирует приоритет для доступных элементов при извлечении, но общий порядок наблюдаемых результатов зависит от одновременного добавления и потребления. Поскольку capacity не ограничена, приложение должно отдельно контролировать рост памяти.

**Сравнение**

| Коллекция | Потокобезопасна | Блокирующие операции | Подходящий сценарий |
|---|---|---|---|
| `ConcurrentHashMap` | Да | Нет | Частые конкурентные операции с ключами и значениями |
| `CopyOnWriteArrayList` | Да | Нет | Очень частое чтение, редкие изменения списка |
| `CopyOnWriteArraySet` | Да | Нет | Небольшое множество с частыми обходами |
| `ConcurrentLinkedQueue` | Да | Нет | Масштабируемые неблокирующие FIFO-операции |
| `ConcurrentLinkedDeque` | Да | Нет | Конкурентный доступ к обоим концам |
| `LinkedBlockingQueue` | Да | Да | Ограниченный producer-consumer FIFO |
| `PriorityBlockingQueue` | Да | Да, при чтении | Потребление элементов по приоритету |

**Итоги**

Конкурентные коллекции позволяют работать с данными из нескольких потоков без ручной блокировки каждой операции. Правильный выбор зависит не только от интерфейса, но и от профиля нагрузки. `ConcurrentHashMap` хорошо масштабирует частые чтения и обновления ключей. Copy-on-write-структуры подходят для почти неизменяемых наборов с большим числом читателей. Linked queues обеспечивают неблокирующий обмен, а blocking queues координируют producers и consumers и могут задавать backpressure.

Необходимо учитывать семантику итераторов, ограниченность capacity, стоимость копирования и способ завершения потребителей. Даже потокобезопасная коллекция не защищает бизнес-инвариант, охватывающий несколько вызовов. Выбрав структуру под конкретный сценарий и используя её атомарные методы, можно уменьшить race conditions, deadlocks и лишнюю конкуренцию, получив более масштабируемый и поддерживаемый код.
