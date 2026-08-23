---
title: "ReentrantLock in Java verstehen"
date: 2024-06-18T14:11:55Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

In Java muss der gleichzeitige Zugriff auf gemeinsam genutzte Ressourcen korrekt gesteuert werden, damit Daten konsistent bleiben und keine Race Conditions entstehen. Das Paket `java.util.concurrent.locks` bietet dafür verschiedene Sperrmechanismen; einer der am häufigsten eingesetzten ist `ReentrantLock`.

**Was ist ReentrantLock?**

`ReentrantLock` ist ein Synchronisationsmechanismus mit einer flexibleren API als ein gewöhnlicher `synchronized`-Block. Er stellt explizite Operationen zum Sperren und Freigeben bereit. „Reentrant“ bedeutet, dass der besitzende Thread denselben Lock erneut erwerben kann, ohne sich selbst zu blockieren. Ein interner Zähler steigt bei jedem `lock()` und sinkt bei jedem `unlock()`.

**Wichtige Funktionen von ReentrantLock**

1. **Reentrancy.** Derselbe Thread darf den Lock mehrfach erwerben. Zur vollständigen Freigabe sind ebenso viele `unlock()`-Aufrufe erforderlich.
2. **Fairness.** Der Lock kann fair oder unfair sein. Ein fairer Lock bevorzugt normalerweise den am längsten wartenden Thread; ein unfairer Lock erlaubt es neuen Threads, sich vorzudrängen.
3. **Condition.** `newCondition()` erzeugt Bedingungen, über die Threads auf einen bestimmten Zustand warten und bei Änderungen signalisiert werden können.
4. **Unterbrechbares Warten.** Mit `lockInterruptibly()` kann ein wartender Thread unterbrochen werden.
5. **Versuch ohne Warten.** `tryLock()` liefert sofort ein Ergebnis oder wartet höchstens für den angegebenen Zeitraum.

**Grundlegende Verwendung von ReentrantLock**

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

Im Beispiel schützt `ReentrantLock` die Methoden `increment()` und `getCounter()` vor gleichzeitigem Zugriff. `lock()` erwirbt die Sperre, während `unlock()` stets in `finally` steht. Dadurch wird der Lock auch dann freigegeben, wenn im kritischen Abschnitt eine Ausnahme auftritt. Zwischen `lock()` und dem Beginn des `try` sollte kein potenziell fehlschlagender Code liegen.

**Erweiterte Funktionen**

1. **Fairer Lock.** Übergeben Sie dem Konstruktor `true`:

   ```java
   Lock fairLock = new ReentrantLock(true);
   ```

   Das verringert das Risiko, dass lange wartende Threads verhungern, kann jedoch den Gesamtdurchsatz reduzieren.
2. **Try Lock.** Ein nicht blockierender Versuch ermöglicht eine alternative Strategie:

   ```java
   if (lock.tryLock()) {
       try {
           // Durch den Lock geschützte Operationen
       } finally {
           lock.unlock();
       }
   } else {
       // Der Lock konnte nicht erworben werden
   }
   ```
3. **Conditions.** Sie ermöglichen eine komplexere Koordination:

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

`await()` gibt den Lock atomar frei und versetzt den Thread in den Wartezustand. Nach einem Signal muss der Thread die Sperre erneut erwerben, bevor er fortfährt. In echtem Code wird die Bedingung in einer Schleife geprüft, um spurious wakeups und zwischenzeitliche Zustandsänderungen durch andere Threads abzufangen.

**Wann ReentrantLock statt synchronized verwenden?**

1. Wenn eine faire Reihenfolge Starvation verhindern soll.
2. Wenn wartende Threads unterbrechbar sein müssen.
3. Wenn ein nicht blockierender oder zeitlich begrenzter Erwerbsversuch benötigt wird.
4. Wenn mehrere `Condition`-Warteschlangen für ein komplexes Warte- und Benachrichtigungsprotokoll erforderlich sind.

**Zusammenfassung**

`ReentrantLock` ist ein leistungsfähiger und flexibler Synchronisationsmechanismus. Er ergänzt `synchronized` um Fairness, unterbrechbaren Erwerb, `tryLock()` und Bedingungsvariablen. Diese Flexibilität verlangt explizites Lebenszyklusmanagement: Jeder erfolgreiche Erwerb muss in `finally` freigegeben werden. Richtig eingesetzt unterstützt `ReentrantLock` robusten und gut wartbaren nebenläufigen Code.
