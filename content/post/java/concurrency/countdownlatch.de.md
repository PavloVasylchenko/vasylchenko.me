---
title: "CountDownLatch in Java mit Beispielen"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch in Java mit Beispielen**

Bei nebenläufigen Java-Anwendungen müssen häufig mehrere Threads synchronisiert werden. Die Klasse `CountDownLatch` erleichtert diese Koordination: Ein oder mehrere Threads können warten, bis andere Threads eine festgelegte Menge von Operationen abgeschlossen haben.

**Was ist CountDownLatch?**

Stellen Sie sich den Start eines Rennens vor. Alle Läufer sollen gleichzeitig beginnen und warten deshalb auf das Ende eines Countdowns. Sobald der Zähler null erreicht, startet das Rennen. `CountDownLatch` funktioniert ähnlich: Das Objekt wird mit der Anzahl von Ereignissen oder Aufgaben erstellt, die eintreten müssen, bevor wartende Threads fortfahren dürfen.

Ein Latch ist nur einmal verwendbar. Hat sein Zähler null erreicht, kann er weder erhöht noch auf den Ausgangswert zurückgesetzt werden. Wird für dieselbe Thread-Gruppe eine wiederverwendbare Barriere benötigt, ist `CyclicBarrier` meist die passendere Wahl.

**Wie funktioniert CountDownLatch?**

1. **Initialisierung.** Das Objekt erhält einen Anfangswert count, also die Zahl der notwendigen `countDown()`-Aufrufe.
2. **Herunterzählen.** Jeder Aufruf von `countDown()` reduziert den Wert um eins. Zusätzliche Aufrufe nach null haben keine Wirkung.
3. **Warten.** `await()` blockiert den aufrufenden Thread, bis der Zähler null erreicht.
4. **Sichtbarkeit.** Aktionen vor `countDown()` werden für den Thread sichtbar, der erfolgreich aus `await()` zurückkehrt.

**Grundlegendes Beispiel**

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            latch.await();  // Haupt-Thread wartet auf den Zähler null
            System.out.println("All tasks are completed. Main thread resumes.");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Arbeit simulieren
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();  // Zähler in jedem Fall verringern
            }
        }
    }
}
```

Das `CountDownLatch` wird mit 3 initialisiert. Drei Tasks starten, simulieren jeweils eine Sekunde Arbeit und rufen anschließend `countDown()` auf. Der Haupt-Thread hält bei `latch.await()` an und setzt seine Ausführung erst nach dem Ende aller Aufgaben fort.

Der Aufruf steht in `finally`, damit der Fehler eines Tasks den Haupt-Thread nicht dauerhaft warten lässt. Dieses Design bedeutet „der Ausführungsversuch ist beendet“, nicht unbedingt „der Task war erfolgreich“. Fehler oder Ergebnisse müssen in einer echten Anwendung getrennt gespeichert werden.

**Erweitertes Beispiel mit Timeout**

Nicht immer darf unbegrenzt gewartet werden. Die zeitlich begrenzte Überladung von `await()` liefert einen `boolean` und erlaubt eine maximale Wartezeit.

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchTimeoutExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            boolean completed = latch.await(5, TimeUnit.SECONDS);
            if (completed) {
                System.out.println("All tasks are completed. Main thread resumes.");
            } else {
                System.out.println("Timeout occurred before all tasks completed.");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();
            }
        }
    }
}
```

Der Haupt-Thread wartet höchstens fünf Sekunden. Verringern alle drei Tasks den Zähler rechtzeitig, gibt `await()` den Wert `true` zurück. Andernfalls ist das Ergebnis `false`, und die Anwendung kann offene Operationen abbrechen, einen Fehler melden oder auf einen Ersatzablauf wechseln. Der Timeout beendet die Worker-Threads nicht automatisch.

**Praktische Einsatzbereiche von CountDownLatch**

1. **Mehrere Threads gleichzeitig starten.** Ein Latch mit dem Wert 1 hält alle Worker bis zu einem gemeinsamen Startsignal zurück:

   ```java
   import java.util.concurrent.CountDownLatch;

   public class ConcurrentStartExample {
       private static final int NUMBER_OF_THREADS = 5;
       private static final CountDownLatch startLatch = new CountDownLatch(1);

       public static void main(String[] args) {
           for (int i = 0; i < NUMBER_OF_THREADS; i++) {
               new Thread(new Worker()).start();
           }

           try {
               System.out.println("Ready... Set...");
               Thread.sleep(1000); // Vorbereitung
               startLatch.countDown();
               System.out.println("Go!");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class Worker implements Runnable {
           @Override
           public void run() {
               try {
                   startLatch.await();
                   System.out.println(Thread.currentThread().getName() + " is working.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               }
           }
       }
   }
   ```

   Alle fünf Worker starten und warten. Ein einziger `countDown()`-Aufruf öffnet das Latch und lässt sie nahezu gleichzeitig fortfahren. Dieses Muster ist für Lasttests oder den Start mehrerer Aufgaben nach einer Initialisierungsphase nützlich.

2. **Auf das Ende mehrerer Threads warten.** Ein Latch mit der Zahl der Aufgaben ermöglicht die spätere Aggregation ihrer Ergebnisse:

   ```java
   import java.util.concurrent.CountDownLatch;

   public class WaitForCompletionExample {
       private static final int NUMBER_OF_TASKS = 3;
       private static final CountDownLatch completionLatch = new CountDownLatch(NUMBER_OF_TASKS);

       public static void main(String[] args) {
           for (int i = 0; i < NUMBER_OF_TASKS; i++) {
               new Thread(new SubTask()).start();
           }

           try {
               completionLatch.await();
               System.out.println("All subtasks completed. Aggregating results.");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class SubTask implements Runnable {
           @Override
           public void run() {
               try {
                   Thread.sleep(1000);
                   System.out.println(Thread.currentThread().getName() + " subtask completed.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               } finally {
                   completionLatch.countDown();
               }
           }
       }
   }
   ```

   Der Haupt-Thread beginnt erst mit der Aggregation, nachdem alle Teilaufgaben ihren Ausführungsversuch beendet haben. In modernem Code lösen auch `ExecutorService`, `Future` oder `CompletableFuture` ähnliche Probleme, doch `CountDownLatch` bleibt eine einfache Möglichkeit, eine Abhängigkeit von einer festen Ereigniszahl auszudrücken.

**Zusammenfassung**

`CountDownLatch` ist ein robuster, einmal verwendbarer Mechanismus zur Thread-Koordination. Er eignet sich für einen gemeinsamen Start, das Warten auf eine Task-Gruppe und die Aufteilung großer Arbeit in unabhängige Teile. Der Anfangswert muss stimmen, jedes erwartete Ereignis benötigt ein garantiertes `countDown()`, und Interrupts müssen behandelt werden. Soll die Wartezeit begrenzt sein, verwenden Sie einen Timeout und definieren Sie ausdrücklich, was mit noch laufenden Aufgaben geschieht.
