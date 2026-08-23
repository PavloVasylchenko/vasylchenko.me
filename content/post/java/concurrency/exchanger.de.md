---
title: "Exchanger in Java verstehen"
date: 2024-06-21T07:16:11Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Was ist Exchanger in Java?**

Die Klasse `Exchanger` erzeugt einen Synchronisationspunkt, an dem zwei Threads ein Paar bilden und Objekte austauschen. Jeder Thread übergibt seinen Wert an `exchange()`, wartet auf den Partner und erhält beim Zusammentreffen das Objekt des jeweils anderen Threads.

Dieser Mechanismus eignet sich, wenn zwei Threads Daten direkt untereinander weitergeben müssen. Ein Producer kann beispielsweise einen Puffer füllen, während ein Consumer den vorherigen verarbeitet; danach tauschen beide die Puffer ohne gemeinsame Queue. `Exchanger` übernimmt zugleich die Wertübertragung und die zeitliche Synchronisierung.

**Wichtige Eigenschaften von Exchanger**

1. **Austausch von Objekten.** Zwei Threads übergeben und empfangen Werte desselben generischen Typs.
2. **Blockierende Operation.** `exchange()` wartet, bis ein anderer Thread denselben Punkt erreicht.
3. **Timeout.** Eine Methodenüberladung begrenzt die Wartezeit und verhindert dauerhaftes Blockieren.
4. **Paarweise Zuordnung.** Treffen mehr als zwei Threads ein, ordnet `Exchanger` sie paarweise zu; ein bestimmter Partner kann nicht vorab ausgewählt werden.

**Grundlegende Verwendung von Exchanger**

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

Im Beispiel:

1. **Initialisierung.** Es wird ein `Exchanger<String>` erstellt, sodass die Teilnehmer Strings austauschen.
2. **Producer und Consumer.** Beide Klassen implementieren `Runnable`, bereiten eigene Daten vor und rufen dieselbe Instanz über `exchanger.exchange()` auf.
3. **Ausführung.** Der zuerst eintreffende Thread blockiert. Erreicht der zweite den Austauschpunkt, werden die Werte atomar vertauscht und beide Threads fahren fort.

Wird ein Teilnehmer während des Wartens unterbrochen, wirft `exchange()` eine `InterruptedException`. Der Code stellt den Interrupt-Status wieder her, damit die aufrufende Ebene die Abbruchanforderung korrekt verarbeiten kann.

**Exchanger mit Timeout**

Kann der Partner den Synchronisationspunkt möglicherweise nicht erreichen, ist unbegrenztes Warten unerwünscht. Die zeitlich begrenzte Variante beendet die Operation vorhersehbar.

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

Beide Threads warten höchstens fünf Sekunden. Erscheint der Partner nicht, entsteht eine `TimeoutException`; die Anwendung kann dann erneut versuchen, Ressourcen freigeben oder die Aufgabe beenden. Timeout und Interrupt sollten getrennt behandelt werden: Ein Timeout kann ein reguläres Protokollergebnis sein, während ein Interrupt meist einen Abbruchwunsch signalisiert.

**Wann Exchanger einsetzen?**

- zum Austausch eines vollen gegen einen leeren Puffer;
- zum Vergleichen oder Zusammenführen von Zwischenergebnissen zweier Worker;
- in einer Pipeline, deren zwei Stufen sich nach jeder Runde treffen müssen;
- in Concurrency-Tests mit einem exakt definierten Synchronisationspunkt für zwei Threads.

Bei beliebig vielen Producern und Consumern ist `BlockingQueue` meist besser geeignet. `Exchanger` spielt seine Stärken aus, wenn das Protokoll natürlicherweise aus Paaren besteht.

**Zusammenfassung**

`Exchanger` verbindet Datenübertragung und Synchronisierung zweier Threads in einer Operation. Die paarweise Kommunikation wird explizit, ohne einen separaten gemeinsamen Container zu benötigen. Ein Timeout schützt vor endlosem Warten, und korrekte Interrupt-Behandlung ermöglicht sauberes Abbrechen. In passenden Szenarien vereinfacht die Klasse die Koordination und macht den nebenläufigen Algorithmus verständlicher.
