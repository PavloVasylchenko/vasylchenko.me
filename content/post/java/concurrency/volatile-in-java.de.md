---
title: "Das Schlüsselwort volatile in Java verstehen"
date: 2024-06-17T10:00:00Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

Das Java-Schlüsselwort `volatile` kennzeichnet eine Variable, deren Wert von mehreren Threads geändert werden kann. Wird eine Variable als `volatile` deklariert, müssen Threads ihren aktuellen Wert beobachten, anstatt weiterhin mit einer veralteten lokalen Kopie zu arbeiten. Diese Garantie ist in nebenläufigen Programmen entscheidend, weil ein Thread sonst eine Änderung eines anderen Threads möglicherweise nicht bemerkt.

**Warum volatile verwenden?**

Zur Leistungsoptimierung können Threads und Prozessoren Werte in Registern oder Caches halten. Dadurch kann ein Thread mit einem alten Wert weiterarbeiten, obwohl ein anderer bereits einen neuen geschrieben hat. `volatile` definiert Sichtbarkeitsregeln: Eine Schreiboperation veröffentlicht die Änderung, und ein späterer Lesezugriff erhält den aktualisierten Wert.

**Wichtige Eigenschaften von volatile**

1. **Sichtbarkeit.** Änderungen an einer `volatile`-Variablen werden für andere Threads sichtbar. Die Schreiboperation darf nicht nur in der lokalen Sicht des schreibenden Threads verbleiben.
2. **Atomarität.** `volatile` macht zusammengesetzte Operationen nicht atomar. `counter++` besteht beispielsweise aus Lesen, Erhöhen und Schreiben; zwei Threads können sich dabei überschneiden und ein Update verlieren.
3. **Reihenfolge.** Das Java Memory Model beschränkt die Umordnung von Lese- und Schreibzugriffen rund um `volatile`. Eine Schreiboperation steht in einer happens-before-Beziehung zu einem späteren Lesezugriff auf dieselbe Variable in einem anderen Thread.

**Wann ist volatile sinnvoll?**

1. Wenn mehrere Threads auf eine einzelne, einfache Variable zugreifen.
2. Wenn ein Leser den zuletzt von einem anderen Thread veröffentlichten Wert sehen muss.
3. Wenn der Zustand durch eine einzelne Zuweisung geändert wird und nicht an read-modify-write-, check-then-act-Operationen oder komplexen Invarianten beteiligt ist.

Typische Beispiele sind ein Stoppsignal, ein Bereitschaftsstatus oder eine Referenz auf eine unveränderliche Konfiguration.

**Beispiel für volatile**

```java
public class VolatileExample {
    private volatile boolean flag = false;

    public void writer() {
        flag = true;  // Schreiben in eine volatile-Variable
    }

    public void reader() {
        if (flag) {  // Lesen der volatile-Variablen
            System.out.println("Flag is true!");
        }
    }

    public static void main(String[] args) {
        VolatileExample example = new VolatileExample();

        Thread writerThread = new Thread(() -> {
            try {
                Thread.sleep(1000); // simulierte Arbeit
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            example.writer();
            System.out.println("Flag set to true by writer thread");
        });

        Thread readerThread = new Thread(() -> {
            while (!example.flag) {
                // warten, bis das Flag true ist
            }
            example.reader();
        });

        writerThread.start();
        readerThread.start();
    }
}
```

In diesem Beispiel ist `flag` als `volatile` deklariert. Der schreibende Thread wartet eine Sekunde und setzt den Wert anschließend auf `true`. Der lesende Thread prüft das Flag in einer Schleife und erkennt dank der Sichtbarkeitsgarantie die Änderung. Danach verlässt er die Schleife und ruft `reader()` auf.

Ohne `volatile` verpflichtet das Java Memory Model den Leser nicht dazu, den gemeinsam genutzten Wert erneut abzurufen. Theoretisch könnte er daher unbegrenzt `false` sehen. Die aktive Warteschleife wäre auch in einer echten Anwendung keine gute Lösung; hier veranschaulicht sie lediglich das Sichtbarkeitsproblem.

**Einschränkungen von volatile**

1. **Keine Atomarität für zusammengesetzte Aktionen.** Inkremente, ein Vergleich mit anschließender Schreiboperation oder die koordinierte Aktualisierung mehrerer Felder benötigen andere Mechanismen.
2. **Kein Schutz für komplexen Zustand.** Müssen mehrere Werte konsistent bleiben, sind `synchronized`, `ReentrantLock` oder eine threadsichere Datenstruktur geeigneter.
3. **Kein Ersatz für Thread-Koordination.** Zum Warten auf Ereignisse sollten `CountDownLatch`, Lock-Conditions, Queues oder andere höherwertige Abstraktionen eingesetzt werden.

**Zusammenfassung**

`volatile` ist ein leichtgewichtiges Werkzeug, um einfachen Zustand zwischen Threads zu veröffentlichen. Es garantiert Sichtbarkeit und eine bestimmte Speicherreihenfolge, macht zusammengesetzte Aktionen jedoch nicht atomar. Für Flags und unabhängige Zuweisungen reicht es oft aus; Zähler und komplexe Zustandsübergänge erfordern atomare Klassen, Locks oder synchronisierte Bereiche. Die richtige Wahl sorgt für korrekten und zugleich verständlichen nebenläufigen Code.
