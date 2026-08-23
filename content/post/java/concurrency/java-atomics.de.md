---
title: "Atomare Klassen in Java"
date: 2024-06-18T11:12:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

Für robuste Multithread-Anwendungen müssen gemeinsam genutzte Daten korrekt und effizient verwaltet werden. Java stellt dafür unter anderem `volatile`-Variablen und atomare Klassen aus dem Paket `java.util.concurrent.atomic` bereit. Beide Mechanismen helfen mehreren Threads beim Zugriff auf gemeinsamen Zustand, verfolgen jedoch unterschiedliche Ziele.

**Warum atomare Klassen verwenden?**

Atomare Klassen führen unteilbare Operationen auf einer einzelnen Variablen ohne explizite Synchronisierung aus. Inkrement, Dekrement, Addition oder compare-and-set (CAS) werden als ein logischer Schritt abgeschlossen: Kein anderer Thread kann einen Zwischenzustand sehen oder sich zwischen Lesen und Schreiben einschalten.

Das Paket `java.util.concurrent.atomic` enthält unter anderem:

- `AtomicInteger`;
- `AtomicLong`;
- `AtomicBoolean`;
- `AtomicReference`.

Diese Klassen bieten threadsichere Methoden zum Lesen, Ändern und bedingten Aktualisieren von Werten. In einfachen Fällen ersetzen sie `synchronized` und explizite Locks.

**Wichtige Eigenschaften atomarer Klassen**

- **Atomarität.** Eine Operation ist unteilbar; andere Threads beobachten nur den Zustand davor oder danach.
- **Nicht blockierende Ausführung.** Die meisten Methoden nutzen hardwarenahe Prozessorinstruktionen und parken keinen Thread beim Warten auf einen Monitor.
- **Lock-free-Ansatz.** Da normalerweise kein gegenseitiger Ausschluss verwendet wird, sinkt das Risiko langer Wartezeiten und von Deadlocks.
- **Sichtbarkeit.** Lese- und Schreibzugriffe erfüllen die Garantien des Java Memory Model, sodass andere Threads das Ergebnis sehen können.

Atomare Klassen sind deshalb nicht automatisch immer schneller als Locks. Bei starker Konkurrenz können wiederholte CAS-Versuche ebenfalls teuer werden. Die richtige Wahl hängt vom Algorithmus und von der tatsächlichen Last ab.

**Beispiel mit AtomicInteger**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicExample {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet();  // atomares Inkrement
    }

    public int getValue() {
        return counter.get();
    }

    public static void main(String[] args) {
        AtomicExample example = new AtomicExample();

        // 1000 Threads erzeugen, die den Zähler erhöhen
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Auf das Ende aller Threads warten
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        System.out.println("Final counter value: " + example.getValue());
    }
}
```

`incrementAndGet()` führt Lesen, Erhöhen und Schreiben als eine atomare Operation aus. Nach dem Ende aller 1000 Threads enthält der Zähler daher den erwarteten Wert. Bei einem gewöhnlichen `int` könnten gleichzeitige `++`-Operationen Updates verlieren, selbst wenn das Feld `volatile` wäre.

**Unterschiede zwischen volatile und atomaren Klassen**

1. **Sichtbarkeit und Atomarität.**
   - `volatile` sorgt dafür, dass Threads aktuelle Schreiboperationen sehen, schützt aber keine zusammengesetzten Aktionen.
   - Atomare Klassen bieten für ihre Methoden sowohl Sichtbarkeit als auch Atomarität.
2. **Komplexe Updates.**
   - `volatile` eignet sich für unabhängige Lesezugriffe und Zuweisungen.
   - Atomare Klassen stellen `incrementAndGet()`, `decrementAndGet()`, `addAndGet()` und `compareAndSet()` für sichere read-modify-write-Operationen bereit.
3. **Implementierung.**
   - `volatile` nutzt die Regeln des Java Memory Model zur Veröffentlichung und Ordnung von Zugriffen.
   - Atomare Klassen verwenden meist hardwaregestütztes CAS und wiederholen die Operation bei Bedarf.
4. **Einsatzbereiche.**
   - `volatile` passt zu Flags, Bereitschaftsanzeigen und einfachem Zustand.
   - Atomare Klassen eignen sich für Zähler, Referenzen und Werte, die mehrere Threads häufig aktualisieren.

**Zusammenfassung**

Wer den Unterschied zwischen `volatile` und atomaren Klassen versteht, kann den einfachsten korrekten Mechanismus auswählen. `volatile` löst das Sichtbarkeitsproblem; atomare Klassen ergänzen unteilbare zusammengesetzte Operationen. Betrifft eine Invariante nur eine Variable, bieten sie häufig eine klare nicht blockierende Lösung. Müssen mehrere Werte gemeinsam geändert, lange kritische Abschnitte ausgeführt oder Bedingungen abgewartet werden, sind Locks oder andere höherwertige Concurrency-Werkzeuge besser geeignet.
