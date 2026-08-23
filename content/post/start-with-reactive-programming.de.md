---
title: "Erste Schritte mit reaktiver Programmierung"
date: 2022-08-04T22:00:00Z
categories:
  - "Manuals"
draft: false
---

Heute möchte ich über reaktive Streams sprechen.
Dieser Ansatz hat in den vergangenen Jahren stark an Bedeutung gewonnen. Spring ermöglicht es, ihn anstelle des servletbasierten Modells einzusetzen.
Auch Client-Bibliotheken bieten immer häufiger reaktive APIs an, und Datenbanken lassen sich über R2DBC und reaktive Treiber anbinden.
Ich selbst verwende Project Reactor ebenfalls zunehmend in meinen Modulen.

Reaktive Streams werden oft mit Java Streams verglichen, da beide Technologien mit funktionaler Programmierung zusammenhängen und ähnlich aussehen.
Ein gewöhnlicher `Stream` kann jedoch nur einmal konsumiert werden. Deshalb geben Entwickler nur selten Werte wie `Stream<String>` aus Methoden zurück.
Meist wird die gesamte Verarbeitungskette in einer Funktion aufgebaut. Für kleine Datentransformationen ist das bequem, die Aufrufe bleiben letztlich aber blockierend.

Vor einigen Monaten absolvierte ich einen Udemy-Kurs zu Project Reactor, um mein Wissen zu ordnen und neue Ideen kennenzulernen.
Der Kurs war interessant und hilfreich, insbesondere für Entwickler, die gerade erst in die reaktive Programmierung einsteigen.

Anschließend erstellte ich eine reaktive Umsetzung von Conways „Game of Life“.
Das Beispiel eignet sich besonders gut: Die Quelle kann eine unbegrenzte Folge erzeugen, während der Verbraucher später festlegt, wie viele Elemente er benötigt.
Der Code ist im Repository [GameOfLife](https://github.com/PavloVasylchenko/GameOfLife) verfügbar.

Das Projekt besteht aus mehreren Teilen:

- `CellState`, ein Enum für den lebenden oder toten Zustand einer Zelle;
- `Game`, die Klasse mit der Kernlogik und dem Generator der Spielzyklen;
- `UI`, das die Spielfelder in der Konsole ausgibt.

Das Feld wird als zweidimensionales Array `CellState[][]` dargestellt.
Eine öffentliche Methode nimmt den Anfangszustand entgegen und liefert einen `Flux` mit den daraus berechneten Folgezuständen:

```java
Flux<CellState[][]> game(CellState[][] initialField)
```

Die Methode legt bewusst keine Anzahl von Iterationen fest; diese Entscheidung trifft der Client.
Der Datenstrom entsteht in zwei Schritten:

1. `Mono.just(initialField)` setzt das Anfangsfeld an den Beginn der Folge.
2. `.concatWith(generate)` hängt den Generator für weitere Generationen an.

```java
Flux<CellState[][]> generate = Flux.generate(() -> initialField, (state, sink) -> {
    CellState[][] iterate = iterate(state);
    sink.next(iterate);
    return iterate;
});
```

Der Generator berechnet aus dem vorherigen Feld ein neues, sendet es an den aktuellen `Flux` und verwendet es als Zustand für den nächsten Schritt.

Der Test zeigt die Nutzung dieser Logik:

```java
@Test
public void printerTest() {
    new Game().game(getGliderField())
            .take(5)
            .doOnNext(UI::printState)
            .subscribe();
}
```

Das Spiel startet mit einem „Glider“, begrenzt den Datenstrom auf fünf Elemente und gibt jedes Feld aus.
Der Generator läuft nur viermal, weil der Anfangszustand bereits das erste Element bildet.
Für weitere Generationen oder zusätzliche Verarbeitung können einfach zusätzliche Operatoren in den bestehenden `Flux` eingefügt werden.
