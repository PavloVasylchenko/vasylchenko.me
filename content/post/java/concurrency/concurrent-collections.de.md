---
title: "Nebenläufige Collections in Java mit Beispielen"
date: 2024-06-25T13:46:57Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Nebenläufige Collections in Java mit Beispielen**

Das Paket `java.util.concurrent` enthält mehrere Collections, die speziell für Multithread-Umgebungen entwickelt wurden. Sie stellen threadsichere Operationen bereit und skalieren meist besser als eine gewöhnliche Collection, die vollständig durch einen einzigen `synchronized`-Block geschützt wird. Im Folgenden betrachten wir die wichtigsten Implementierungen, ihre Einsatzbereiche und praktische Beispiele.

**Überblick über nebenläufige Collections**

Java stellt unter anderem folgende Strukturen bereit:

- `ConcurrentHashMap`;
- `CopyOnWriteArrayList`;
- `CopyOnWriteArraySet`;
- `ConcurrentLinkedQueue`;
- `ConcurrentLinkedDeque`;
- `LinkedBlockingQueue`;
- `PriorityBlockingQueue`.

Jede Struktur verwendet ein eigenes Nebenläufigkeitsmodell. Die Threadsicherheit einer einzelnen Methode macht eine beliebige Folge von Aufrufen nicht atomar; zusammengesetzte Aktionen benötigen atomare Collection-Methoden oder zusätzliche Synchronisierung.

**ConcurrentHashMap**

`ConcurrentHashMap` ist eine threadsichere Map-Implementierung für gleichzeitige Lese- und Schreiboperationen. Sie sperrt nicht bei jeder Operation die gesamte Tabelle und bietet atomare Methoden wie `putIfAbsent`, `compute`, `merge` und `replace`.

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

Zwei Writer fügen gleichzeitig Einträge hinzu; anschließend durchlaufen Reader die Map. Die Iteration ist weakly consistent: Sie wirft keine `ConcurrentModificationException` und kann einen Teil parallel ausgeführter Änderungen beobachten.

**CopyOnWriteArrayList**

`CopyOnWriteArrayList` ist eine threadsichere Variante von `ArrayList`. Jede Änderung erzeugt eine neue Kopie des internen Arrays. Lesen und Iterieren sind schnell und benötigen keine Sperre, häufige Schreiboperationen verursachen jedoch hohe Kosten für Speicher und Kopien.

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

Der Iterator arbeitet mit einem Snapshot des Arrays zum Erstellungszeitpunkt und sieht spätere Änderungen nicht. Die Collection eignet sich für Listener-Listen oder Konfigurationen, die oft gelesen und selten verändert werden.

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` ist eine threadsichere Menge auf Basis von `CopyOnWriteArrayList`. Sie hält Elemente eindeutig und übernimmt dasselbe Modell: schnelle stabile Lesezugriffe, Snapshot-Iteratoren und teure Änderungen.

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

Die Struktur passt zu kleinen Mengen von Listenern oder Berechtigungen, die wesentlich häufiger durchlaufen als geändert werden.

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` ist eine unbegrenzte threadsichere FIFO-Queue auf Basis verketteter Knoten. Ihr nicht blockierender Algorithmus skaliert gut, wenn viele Threads gleichzeitig Elemente hinzufügen und entfernen.

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

`poll()` entfernt den Kopf atomar oder liefert `null`. Eine vorherige Prüfung mit `isEmpty()` ist unnötig, weil sich der Zustand zwischen beiden Aufrufen ändern könnte. Die Queue wartet nicht blockierend auf Elemente und begrenzt Producer nicht; Backpressure muss separat umgesetzt werden.

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` ist eine nicht blockierende threadsichere Double-Ended Queue. Threads können gleichzeitig an beiden Enden Elemente einfügen und entnehmen.

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

Der Zugriff auf beide Enden ist für Work Stealing, Aufgabenverläufe und Algorithmen nützlich, die FIFO- und LIFO-Operationen kombinieren.

**LinkedBlockingQueue**

`LinkedBlockingQueue` ist eine optional begrenzte blockierende FIFO-Queue auf verketteten Knoten. `take()` wartet auf ein Element; `put()` wartet in einer begrenzten Queue auf freien Platz. Damit ist sie eine natürliche Grundlage für Producer-Consumer und einfaches Backpressure.

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

Die Kapazität 10 zwingt schnelle Producer zum Warten, wenn die Consumer nicht nachkommen. Eine echte Anwendung benötigt ein Beendigungsprotokoll, etwa Interrupts, ein spezielles Poison Pill oder die Steuerung durch einen Executor.

**PriorityBlockingQueue**

`PriorityBlockingQueue` ist eine unbegrenzte blockierende Queue mit den Ordnungsregeln von `PriorityQueue`. `take()` wartet auf ein Element, Producer blockieren jedoch nie wegen der Kapazität. Elemente müssen `Comparable` implementieren oder die Queue benötigt einen `Comparator`.

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

Beim Entnehmen gilt die Priorität der aktuell verfügbaren Elemente, die insgesamt beobachtete Reihenfolge hängt jedoch von parallelen Einfügungen und Entnahmen ab. Da die Kapazität unbegrenzt ist, muss die Anwendung das Speicherwachstum selbst kontrollieren.

**Vergleichstabelle**

| Collection | Threadsicher | Blockierende Operationen | Einsatzbereich |
|---|---|---|---|
| `ConcurrentHashMap` | Ja | Nein | Häufige Zugriffe auf Schlüssel-Wert-Paare |
| `CopyOnWriteArrayList` | Ja | Nein | Viele Lese-, wenige Schreiboperationen |
| `CopyOnWriteArraySet` | Ja | Nein | Kleine, häufig durchlaufene Menge |
| `ConcurrentLinkedQueue` | Ja | Nein | Skalierbare nicht blockierende FIFO-Operationen |
| `ConcurrentLinkedDeque` | Ja | Nein | Gleichzeitiger Zugriff auf beide Enden |
| `LinkedBlockingQueue` | Ja | Ja | Begrenztes Producer-Consumer-FIFO |
| `PriorityBlockingQueue` | Ja | Ja, beim Lesen | Verarbeitung nach Priorität |

**Zusammenfassung**

Nebenläufige Collections ermöglichen mehreren Threads den Datenzugriff ohne manuelle Sperre jeder Operation. Die Wahl hängt von Schnittstelle und Lastprofil ab. `ConcurrentHashMap` skaliert bei häufigem Lesen und Aktualisieren. Copy-on-write-Strukturen passen zu fast unveränderlichen Mengen mit vielen Lesern. Linked Queues bieten nicht blockierenden Austausch, während Blocking Queues Producer und Consumer koordinieren und Backpressure ermöglichen.

Zu beachten sind Iteratorssemantik, Kapazität, Kopierkosten und das Beenden der Consumer. Auch eine threadsichere Collection schützt keine fachliche Invariante über mehrere Methodenaufrufe hinweg. Mit der passenden Struktur und ihren atomaren Methoden lassen sich Race Conditions, Deadlocks und unnötige Contention reduzieren, sodass der Code skalierbarer und wartbarer wird.
