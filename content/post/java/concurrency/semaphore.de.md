---
title: "Semaphore in Java verwenden"
date: 2024-06-20T08:16:17Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Was ist ein Semaphore in Java?**

`Semaphore` ist ein Synchronisationswerkzeug, das die Anzahl der Threads begrenzt, die gleichzeitig auf eine gemeinsam genutzte Ressource zugreifen dürfen. Es gehört zu `java.util.concurrent` und eignet sich besonders für Ressourcen mit fester Kapazität, etwa Verbindungspools, Netzwerk-Sockets, Drucker oder Dateideskriptoren.

**Wichtige Eigenschaften von Semaphore**

1. **Permits.** Das Semaphor verwaltet eine Menge von Genehmigungen. Ein Thread erwirbt vor der Arbeit ein Permit und gibt es danach zurück.
2. **Blockierender und nicht blockierender Erwerb.** `acquire()` wartet auf eine freie Genehmigung; `tryAcquire()` ermöglicht sofort einen alternativen Ablauf.
3. **Fairness.** Ein Semaphor kann fair oder unfair sein. Im fairen Modus werden Permits gewöhnlich in der Reihenfolge der Anfragen vergeben, wodurch Starvation unwahrscheinlicher wird.
4. **Mehrere Permits.** Ein Thread kann mehrere Genehmigungen erwerben und freigeben, wenn eine Operation mehrere Einheiten der Ressource benötigt.

**Grundlegendes Beispiel für Semaphore**

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
   private static final int MAX_PERMITS = 3;
   private static final Semaphore semaphore = new Semaphore(MAX_PERMITS);

   public static void main(String[] args) {
      for (int i = 0; i < 10; i++) {
         new Thread(new Task()).start();
      }
   }

   static class Task implements Runnable {
      @Override
      public void run() {
         boolean acquired = false;
         try {
            System.out.println(Thread.currentThread().getName() + " is waiting for a permit.");
            semaphore.acquire();
            acquired = true;
            System.out.println(Thread.currentThread().getName() + " acquired a permit.");

            // Arbeit mit der gemeinsam genutzten Ressource simulieren
            Thread.sleep(2000);

            System.out.println(Thread.currentThread().getName() + " releasing a permit.");
         } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
         } finally {
            if (acquired) {
               semaphore.release();
            }
         }
      }
   }
}
```

Das Semaphor besitzt drei Permits. Zehn Threads werden gestartet, doch nur drei können `acquire()` gleichzeitig passieren und den kritischen Abschnitt ausführen. Die übrigen warten. Nach `release()` kann ein weiterer Teilnehmer die freigegebene Genehmigung erhalten.

Die Variable `acquired` ist wichtig: Wurde der Thread vor dem erfolgreichen Erwerb unterbrochen, darf er `release()` nicht aufrufen, da sonst die Gesamtzahl der Permits fälschlich steigen würde.

**Erweiterte Verwendung von Semaphore**

1. **Versuch ohne Blockieren.** Der Thread prüft die Ressource und wählt sofort einen anderen Weg, wenn sie nicht verfügbar ist:

   ```java
   if (semaphore.tryAcquire()) {
       try {
           // Kritischer Abschnitt
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " could not acquire a permit.");
   }
   ```
2. **Erwerb mit Timeout.** Eine begrenzte Wartezeit verhindert unbegrenztes Blockieren bei Überlastung:

   ```java
   if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
       try {
           // Kritischer Abschnitt
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " timed out waiting for a permit.");
   }
   ```
3. **Faires Semaphor.** Der zweite Konstruktorparameter aktiviert eine geordnete Warteschlange:

   ```java
   Semaphore fairSemaphore = new Semaphore(MAX_PERMITS, true);
   ```

Fairness ist nützlich, wenn kein Thread zu lange warten darf. Die Verwaltung der Reihenfolge kann jedoch den Gesamtdurchsatz senken.

**Einsatzbereiche**

1. **Begrenzung der Parallelität.** Kontrolle gleichzeitiger Anfragen an einen externen Dienst oder eine teure Operation.
2. **Verbindungspools.** Jedes Permit steht für eine verfügbare Datenbankverbindung.
3. **Ressourcenverwaltung.** Beschränkung des Zugriffs auf wenige Drucker, Dateien oder Geräte.
4. **Überlastschutz.** Zusätzliche Aufgaben warten oder werden abgewiesen, statt gleichzeitig Speicher und CPU zu belegen.

**Zusammenfassung**

`Semaphore` begrenzt den konkurrierenden Zugriff auf gemeinsam genutzte Ressourcen zuverlässig. Anders als ein Mutex kann es eine festgelegte Anzahl von Threads gleichzeitig passieren lassen. Nur tatsächlich erworbene Permits dürfen freigegeben werden, und dies sollte stets in `finally` geschehen. Richtig eingesetzt hilft ein Semaphor, knappe Ressourcen effizient zu nutzen und Contention zu reduzieren.
