---
title: "Конкурентні колекції в Java"
date: 2024-06-25T13:47:58Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Конкурентні колекції в Java**

Java надає кілька конкурентних колекцій, які є частиною пакету `java.util.concurrent`. Ці колекції призначені для ефективного управління багатопоточними середовищами, забезпечуючи потокобезпечні операції та покращуючи продуктивність у порівнянні з традиційними методами синхронізації. У цій статті ми розглянемо ці колекції, їхні випадки використання та надамо більш складні приклади коду, щоб допомогти вам зрозуміти, як ефективно їх використовувати.

**Вступ до конкурентних колекцій**

Конкурентні колекції призначені для підтримки високої конкурентності, зберігаючи при цьому потокобезпеку. Основні конкурентні колекції, які надає Java:

- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`
- `ConcurrentLinkedQueue`
- `ConcurrentLinkedDeque`
- `LinkedBlockingQueue`
- `PriorityBlockingQueue`

Давайте розглянемо кожну з цих колекцій з більш складними прикладами.

**ConcurrentHashMap**

`ConcurrentHashMap` - це потокобезпечна реалізація HashMap, яка дозволяє одночасні операції читання та запису.

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        // Імітуємо кілька потоків, що оновлюють карту
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
            e.printStackTrace();
        }

        // Одночасна операція читання
        Runnable readerTask = () -> {
            map.forEach((key, value) -> System.out.println(Thread.currentThread().getName() + ": " + key + ": " + value));
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```
**CopyOnWriteArrayList**

`CopyOnWriteArrayList` - це потокобезпечний варіант ArrayList, де всі змінюючі операції реалізовані шляхом створення нової копії основного масиву.

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteArrayListExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

        // Імітуємо кілька потоків, що оновлюють список
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
            e.printStackTrace();
        }

        // Одночасна операція читання
        Runnable readerTask = () -> {
            for (String item : list) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` - це потокобезпечний варіант HashSet, який реалізований за допомогою `CopyOnWriteArrayList`.

```java
import java.util.concurrent.CopyOnWriteArraySet;

public class CopyOnWriteArraySetExample {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();

        // Імітуємо кілька потоків, що оновлюють набір
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
            e.printStackTrace();
        }

        // Одночасна операція читання
        Runnable readerTask = () -> {
            for (String item : set) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };
        
        Thread reader1 = new Thread(readerTask);
        Thread reader2 = new Thread(readerTask);
        
        reader1.start();
        reader2.start();
    }
}
```

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` - це необмежена потокобезпечна черга, заснована на зв’язаних вузлах. Це реалізація алгоритму, що не використовує блокування.

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueExample {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

        // Імітуємо кілька потоків, що додають у чергу
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
            e.printStackTrace();
        }

        // Імітуємо кілька потоків, що споживають з черги
        Runnable consumerTask = () -> {
            while (!queue.isEmpty()) {
                String item = queue.poll();
                if (item != null) {
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            }
        };
        
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        consumer1.start();
        consumer2.start();
    }
}
```

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` - це потокобезпечний варіант LinkedList, який підтримує одночасний доступ до обох кінців.

```java
import java.util.concurrent.ConcurrentLinkedDeque;

public class ConcurrentLinkedDequeExample {
    public static void main(String[] args) {
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();

        // Імітуємо кілька потоків, що додають у деку
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
            e.printStackTrace();
        }

        // Імітуємо кілька потоків, що споживають з деки
        Runnable consumerTask = () -> {
            while (!deque.isEmpty()) {
                String item = deque.pollFirst();
                if (item != null) {
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            }
        };
        
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        consumer1.start();
        consumer2.start();
    }
}
```

**LinkedBlockingQueue**

`LinkedBlockingQueue` - це необов’язково обмежена блокуюча черга, заснована на зв’язаних вузлах. Вона підтримує операції, які чекають, поки черга стане не порожньою при отриманні елемента, і чекають, поки з’явиться місце в черзі при зберіганні елемента.

```java
import java.util.concurrent.LinkedBlockingQueue;

public class LinkedBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

        // Імітуємо кілька потоків, що виробляють і споживають
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
                while (true) {
                    String item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };
        
        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);
        
        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
    }
}
```

**PriorityBlockingQueue**

`PriorityBlockingQueue` - це необмежена блокуюча черга, яка використовує ті ж правила впорядкування, що і PriorityQueue, і надає блокуючі операції вилучення.

```java
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();

        // Імітуємо кілька потоків, що виробляють і споживають
        Runnable producerTask = () -> {
            try {
                for (int i = 0; i < 1000; i++) {
                    queue.put((int) (Math.random() * 1000));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (true) {
                    Integer item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        Thread consumer1 = new Thread(consumerTask);
        Thread consumer2 = new Thread(consumerTask);

        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
    }
}
```

**Порівняльна таблиця**

Нижче наведена порівняльна таблиця різних розглянутих конкурентних колекцій:

| Collection            | Thread-Safe | Blocking Operations | Use Case                                        |
|-----------------------|-------------|---------------------|-------------------------------------------------|
| ConcurrentHashMap     | Так         | Ні                  | Висока конкурентність для пар ключ-значення     |
| CopyOnWriteArrayList  | Так         | Ні                  | Часті читання, рідкісні записи                  |
| CopyOnWriteArraySet   | Так         | Ні                  | Потокобезпечний набір із частими читаннями      |
| ConcurrentLinkedQueue | Так         | Ні                  | Висока конкурентність для операцій FIFO         |
| ConcurrentLinkedDeque | Так         | Ні                  | Висока конкурентність для двобічних операцій    |
| LinkedBlockingQueue   | Так         | Так                 | Блокуюча черга FIFO для схеми виробник-споживач |
| PriorityBlockingQueue | Так         | Так                 | Блокуюча черга з пріоритетним впорядкуванням    |

**Висновок**

Конкурентні колекції в Java забезпечують надійний та ефективний спосіб управління багатопоточними операціями на колекціях. Використовуючи ці колекції, ви можете покращити продуктивність та надійність своїх додатків. Розуміння відповідних випадків використання та як реалізовувати ці колекції допоможе вам приймати кращі проектні рішення у ваших багатопоточних додатках.

Кожна колекція має свої сильні сторони і призначена для конкретних сценаріїв. Наприклад, `ConcurrentHashMap` відмінно підходить для сценаріїв з високою конкуренцією з частими операціями читання та запису, тоді як `CopyOnWriteArrayList` краще підходить для ситуацій з частими читаннями, але рідкісними записами. Подібним чином, блокуючі черги, такі як `LinkedBlockingQueue` та `PriorityBlockingQueue`, є необхідними для реалізації схем виробник-споживач, де потоки повинні чекати наявності доступних ресурсів.

Обираючи правильну конкурентну колекцію, ви можете уникнути поширених проблем багатопоточності, таких як умови перегонів, взаємоблокування та надмірна конкуренція, що призводить до більш масштабованого та підтримуваного коду.






