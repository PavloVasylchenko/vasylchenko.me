---
title: "CyclicBarrier in Java verwenden"
date: 2024-06-19T13:15:18Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CyclicBarrier in Java verstehen**

Die Klasse `CyclicBarrier` ist ein Synchronisationswerkzeug, mit dem eine Gruppe von Threads an einem gemeinsamen Barrierepunkt aufeinander wartet. Sie eignet sich, wenn mehrere Threads eine Arbeitsphase unabhängig ausführen, aber erst dann zur nächsten übergehen dürfen, wenn alle denselben Zustand erreicht haben.

Anders als `CountDownLatch` kann die Barriere nach der Freigabe der wartenden Threads erneut verwendet werden. Deshalb passt `CyclicBarrier` gut zu zyklischen oder iterativen Algorithmen, bei denen sich dieselben Teilnehmer nach jeder Phase synchronisieren.

**Wichtige Eigenschaften von CyclicBarrier**

- **Wiederverwendbar.** Sind alle Threads angekommen und freigegeben, beginnt automatisch ein neuer Barrierenzyklus.
- **Optionale Aktion.** Dem Konstruktor kann ein `Runnable` übergeben werden, den der zuletzt eintreffende Thread ausführt, bevor die anderen fortfahren.
- **Feste Teilnehmerzahl.** Die Zahl parties wird beim Erstellen festgelegt und bestimmt, wie viele `await()`-Aufrufe die Barriere öffnen.

**Grundlegendes Beispiel für CyclicBarrier**

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

In diesem Beispiel:

- **Initialisierung.** Die Barriere erhält `NUMBER_OF_THREADS` und eine optionale Aktion, die beim Eintreffen des letzten Teilnehmers ausgeführt wird.
- **Task.** Jeder `Task` ruft `barrier.await()` auf und hält an, bis die übrigen Teilnehmer eintreffen.
- **Ausführung.** Drei Threads werden gestartet. Nach dem dritten `await()` läuft die Barrierenaktion; anschließend setzen alle Threads ihre Arbeit fort.

**Erweiterte Verwendung von CyclicBarrier**

1. **Gebrochene Barriere.** Wird ein wartender Thread unterbrochen oder überschreitet sein Zeitlimit, wechselt die Barriere in den broken state. Die übrigen Teilnehmer erhalten eine `BrokenBarrierException`. Der Zustand ist über `barrier.isBroken()` abrufbar.
2. **Zeitlimit.** Eine Überladung von `await()` verhindert unbegrenztes Warten:

   ```java
   barrier.await(5, TimeUnit.SECONDS);
   ```
3. **Zurücksetzen.** `barrier.reset()` stellt den Ausgangszustand wieder her. Bereits wartende Threads erhalten dabei eine Ausnahme, daher muss das Zurücksetzen Teil eines koordinierten Protokolls sein.

**Beispiel mit Timeout und Reset**

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class CyclicBarrierAdvancedExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await(5, TimeUnit.SECONDS);
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException | TimeoutException e) {
                e.printStackTrace();
                if (barrier.isBroken()) {
                    System.out.println("Barrier is broken, resetting...");
                    barrier.reset();
                }
            }
        }
    }
}
```

Jeder Thread wartet höchstens fünf Sekunden. Treffen alle rechtzeitig ein, wird die Aktion ausgeführt und die nächste Phase beginnt. Läuft die Zeit ab, entsteht eine `TimeoutException` und die Barriere gilt als gebrochen. Das Beispiel prüft diesen Zustand und setzt das Objekt für eine spätere Verwendung zurück.

In einer echten Anwendung sollten mehrere Threads `reset()` nicht unkoordiniert aufrufen. Üblicherweise entscheidet eine übergeordnete Komponente, ob die Phase sicher wiederholt werden kann, und steuert die Wiederherstellung.

**Zusammenfassung**

`CyclicBarrier` synchronisiert eine feste Anzahl von Threads an einem gemeinsamen Punkt. Wiederverwendbarkeit und optionale Barrierenaktion machen sie für parallele Berechnungen in mehreren Runden geeignet. Timeouts und die Behandlung des broken state verhindern, dass ein ausgefallener Teilnehmer das System dauerhaft blockiert. Mit einem klaren Protokoll lassen sich zyklische Aufgaben robust koordinieren.
