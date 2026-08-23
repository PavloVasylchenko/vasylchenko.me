---
title: "Les collections concurrentes en Java avec des exemples"
date: 2024-06-25T13:46:57Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Les collections concurrentes en Java avec des exemples**

Le package `java.util.concurrent` contient plusieurs collections conçues pour les environnements multithread. Elles fournissent des opérations thread-safe et passent généralement mieux à l’échelle qu’une collection classique protégée par un unique bloc `synchronized`. Examinons les principales implémentations, leurs cas d’usage et des exemples pratiques.

**Présentation des collections concurrentes**

Java propose notamment :

- `ConcurrentHashMap` ;
- `CopyOnWriteArrayList` ;
- `CopyOnWriteArraySet` ;
- `ConcurrentLinkedQueue` ;
- `ConcurrentLinkedDeque` ;
- `LinkedBlockingQueue` ;
- `PriorityBlockingQueue`.

Chaque structure adopte un modèle de concurrence différent. Le fait qu’une méthode soit thread-safe ne rend pas atomique une suite arbitraire d’appels ; les actions composées doivent employer les méthodes atomiques de la collection ou une synchronisation supplémentaire.

**ConcurrentHashMap**

`ConcurrentHashMap` est une implémentation thread-safe de map qui accepte lectures et mises à jour concurrentes. Elle ne verrouille pas toute la table à chaque opération et propose des méthodes atomiques telles que `putIfAbsent`, `compute`, `merge` et `replace`.

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

Deux writers ajoutent simultanément des éléments, puis des readers parcourent la map. L’itération est weakly consistent : elle ne lève pas `ConcurrentModificationException` et peut voir une partie des modifications effectuées en parallèle.

**CopyOnWriteArrayList**

`CopyOnWriteArrayList` est une variante thread-safe de `ArrayList`. Chaque modification crée une nouvelle copie du tableau interne. Les lectures et itérations sont rapides et sans verrou, mais les écritures fréquentes coûtent cher en mémoire et en copies.

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

L’itérateur travaille sur un instantané du tableau pris lors de sa création et ne voit pas les changements ultérieurs. Cette collection convient aux listes d’abonnés ou aux configurations consultées souvent mais rarement modifiées.

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` est un ensemble thread-safe implémenté avec `CopyOnWriteArrayList`. Il garantit l’unicité et conserve le même modèle : lectures stables et rapides, itérateurs snapshot et modifications coûteuses.

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

Cette structure est adaptée aux petits ensembles de listeners ou de permissions, lorsque les parcours sont beaucoup plus fréquents que les ajouts et suppressions.

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` est une file FIFO non bornée et thread-safe fondée sur des nœuds chaînés. Son algorithme non bloquant passe bien à l’échelle lorsque de nombreux threads ajoutent et retirent des éléments simultanément.

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

`poll()` retire atomiquement la tête ou renvoie `null`. Il est inutile de tester `isEmpty()` avant, car l’état peut changer entre les deux appels. La file n’attend pas de manière bloquante et ne limite pas les producteurs ; le backpressure doit être géré séparément.

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` est une file double non bloquante et thread-safe. Plusieurs threads peuvent ajouter et retirer des éléments aux deux extrémités.

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

L’accès aux deux extrémités est utile pour le work stealing, l’historique des tâches et les algorithmes qui combinent des opérations FIFO et LIFO.

**LinkedBlockingQueue**

`LinkedBlockingQueue` est une file FIFO bloquante, éventuellement bornée, basée sur des nœuds chaînés. `take()` attend un élément, tandis que `put()` attend une place libre dans une file limitée. Elle constitue une base naturelle pour producer-consumer et un backpressure simple.

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

La capacité de 10 oblige les producers rapides à attendre lorsque les consumers prennent du retard. Une application réelle doit prévoir un protocole d’arrêt : interruption, poison pill spécial ou gestion par un executor.

**PriorityBlockingQueue**

`PriorityBlockingQueue` est une file bloquante non bornée qui suit les règles d’ordre de `PriorityQueue`. `take()` attend un élément, mais les producers ne sont jamais bloqués par la capacité. Les éléments doivent implémenter `Comparable`, ou la file doit recevoir un `Comparator`.

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

La file respecte la priorité des éléments disponibles au moment du retrait, mais l’ordre global observé dépend des insertions et consommations concurrentes. Sa capacité n’étant pas limitée, l’application doit surveiller séparément l’utilisation de la mémoire.

**Tableau comparatif**

| Collection | Thread-safe | Opérations bloquantes | Cas d’usage |
|---|---|---|---|
| `ConcurrentHashMap` | Oui | Non | Accès fréquents à des paires clé-valeur |
| `CopyOnWriteArrayList` | Oui | Non | Lectures fréquentes, écritures rares |
| `CopyOnWriteArraySet` | Oui | Non | Petit ensemble parcouru très souvent |
| `ConcurrentLinkedQueue` | Oui | Non | Opérations FIFO non bloquantes et évolutives |
| `ConcurrentLinkedDeque` | Oui | Non | Accès concurrent aux deux extrémités |
| `LinkedBlockingQueue` | Oui | Oui | FIFO bornée pour producer-consumer |
| `PriorityBlockingQueue` | Oui | Oui, en lecture | Consommation selon la priorité |

**Résumé**

Les collections concurrentes permettent à plusieurs threads de manipuler des données sans verrouiller manuellement chaque opération. Le choix dépend de l’interface et du profil de charge. `ConcurrentHashMap` convient aux lectures et mises à jour fréquentes. Les structures copy-on-write servent aux ensembles presque immuables avec de nombreux lecteurs. Les linked queues permettent un échange non bloquant, tandis que les blocking queues coordonnent producteurs et consommateurs et peuvent fournir du backpressure.

Il faut tenir compte de la sémantique des itérateurs, de la capacité, du coût des copies et de la façon d’arrêter les consommateurs. Une collection thread-safe ne protège pas non plus un invariant métier couvrant plusieurs appels. En choisissant la bonne structure et ses méthodes atomiques, on réduit les race conditions, les deadlocks et la contention inutile, pour obtenir un code plus évolutif et maintenable.
