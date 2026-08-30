---
title: "Von der Idee zu Google Play: Wie ich im Urlaub mit KI ein Spiel entwickelt habe"
date: "2026-08-30T00:00:00Z"
draft: false
description: "Wie die Erinnerung an ein einfaches Browser-Strategiespiel zu einem Experiment mit mehreren KI-Modellen, einem funktionierenden Prototyp und meinem eigenen Android-Spiel wurde."
slug: "from-flash-game-to-android-game"
categories:
  - "Game Development"
  - "Artificial Intelligence"
tags:
  - "Synaptic Front"
  - "AI-assisted development"
  - "Indie Game"
series: "Synaptic Front with AI"
series_order: 1
---

Als ich in der Oberstufe war, bekamen wir zu Hause einen Internetanschluss, der für damalige Verhältnisse ziemlich schnell war. Mit ihm hielten Browserspiele Einzug in meinen Alltag: Man musste nichts installieren, sie luden schnell und eigneten sich perfekt für kurze Pausen. Während der Computer ein Archiv entpackte oder ein Programm installierte, konnte ich einen neuen Tab öffnen, eine kurze Runde spielen und wenige Minuten später zu meiner eigentlichen Arbeit zurückkehren. Unter all den Flash-Spielen blieb mir ein Strategiespiel besonders im Gedächtnis, auch wenn ich seinen Namen irgendwann vergaß.

Das Spielfeld bestand aus Systemen, die durch Linien verbunden waren und zwischen denen sich Flotten bewegten. Die Oberfläche zeigte sofort, wem jeder Knoten gehörte, wohin man Kräfte schicken konnte und aus welcher Richtung der Gegner näher kam. Jede Entscheidung verschob das Gleichgewicht: Ein Angriff stärkte eine Front, schwächte aber das Ausgangssystem, und ein eroberter Knoten konnte aus einem sicheren Gebiet eine neue Grenze machen. Die Regeln waren einfach, ihre Folgen wurden innerhalb weniger Sekunden sichtbar – genau deshalb blieben selbst kurze Partien spannend.

Später verschwand Flash, Computer und Gewohnheiten änderten sich, und der Name des Spiels blieb weder in meinen Lesezeichen noch in meinem Gedächtnis erhalten. Mehrmals versuchte ich, es anhand der Beschreibung zu finden, stieß aber nur auf ähnliche Projekte. Die Idee selbst war dagegen noch sehr präsent: ein Netz verbundener Knoten, Bewegung auf festgelegten Routen und Spannung, die aus wenigen verständlichen Regeln entsteht. Jahre später wurde diese Erinnerung zum Ausgangspunkt für Synaptic Front.

#### Von der Erinnerung zum eigenen Spiel

Der Gedanke, ein ähnliches Spiel zu entwickeln, kam mir mehr als einmal, blieb aber lange auf der Liste jener Ideen, für die man nie Zeit findet. Für einen ersten Prototyp hätte ich eine Technologie wählen und mich mit Spielschleife, Kartendarstellung, Animation und Steuerung beschäftigen müssen. Selbst wenn sich die Mechanik am Ende als langweilig erwiesen hätte, wäre zuerst Zeit in unbekannte Werkzeuge geflossen. Das machte das Experiment zu teuer: Mehrere Wochen Arbeit konnten mit einer Erkenntnis enden, die ich eigentlich nach ein oder zwei Abenden haben wollte.

Inzwischen habe ich mehr als 15 Jahre Erfahrung in der Java-Entwicklung. Ich habe an Unternehmensanwendungen und stark ausgelasteten Serversystemen gearbeitet; komplexe Architekturen und große Projekte schrecken mich daher nicht ab. Ein Spiel hatte ich in all diesen Jahren jedoch nie entwickelt. Auch Android, Kotlin und Compose gehörten nicht zu meinem Arbeitsalltag. Programmieren konnte ich – und trotzdem hätte ich für diese konkrete Idee in einem unbekannten Gebiet anfangen müssen.

> **„Die KI hat dieses Spiel nicht erfunden – sie hat mir geholfen, endlich mit der Entwicklung anzufangen.“**

In letzter Zeit arbeite ich allerdings viel mit KI-Werkzeugen und kann inzwischen gut einschätzen, wo sie tatsächlich Zeit sparen. Ich beschloss, diese Erfahrung mit der alten Idee zu verbinden und endlich herauszufinden, ob daraus ein Spiel werden konnte. Möglichst schnell wollte ich eine Version erreichen, die ich starten und selbst beurteilen konnte, ohne zunächst mehrere Wochen nur einen neuen Stack kennenzulernen. An dem Experiment waren Claude, GPT, Kimi und das gerade erschienene Fable beteiligt.

Ein Spiel eignete sich dafür besser als eine abstrakte Testaufgabe. Ich erinnerte mich an das gewünschte Gefühl, verstand die grundlegenden Regeln und merkte sofort, ob eine neue Version meiner Vorstellung näher kam. Entscheidend war nicht, wie viel Code ein KI-Modell schrieb, sondern ob das Ergebnis Spaß machte.

#### Eine Spezifikation als gemeinsamer Bezugspunkt

Deshalb begann die Arbeit mit den Regeln und nicht mit Codegenerierung. Ich formulierte einige Ausgangsideen und bat Fable, daraus eine Markdown-Spezifikation zu erstellen. So entstand `SPEC.md`, das erste Dokument mit dem Gesamtkonzept, dem Aufbau der Karte, den Regeln für Systembesitz, Kräfteproduktion, Flottenbewegung und Eroberung. Anschließend prüften weitere Modelle die Spezifikation nacheinander. Sie präzisierten Formulierungen, fanden Mehrdeutigkeiten und schlugen Ergänzungen vor; ich behielt nützliche Änderungen und entfernte alles, was das Spiel von der ursprünglichen Idee wegführte.

Als das Konzept wuchs, reichte eine einzelne `SPEC.md` nicht mehr aus. Hinzu kamen `VISUAL.md` für Gestaltungs- und Animationsentscheidungen, `PLOT.md` für die Handlung einzelner Level und `LORE.md` für den Kanon der Spielwelt. Darauf folgte `CONTENT_PLAN.md`, das Geschichte und Kampagnenideen in einen konkreteren technischen Levelplan verwandelte. Später ergänzten `ECONOMY.md` zur Weiterentwicklung der Wirtschaft und ihrer Mechaniken sowie `OPERATORS.md` mit den vom Spieler gesteuerten Operator-Helden und ihren Fähigkeiten die Sammlung.

Anfangs trennte diese Struktur die verschiedenen Projektbereiche gut, doch die Dokumente wuchsen schnell und wurden selbst unhandlich. Ich zerlegte sie in kleinere thematische Dateien und fügte zur Navigation eine `INDEX.md` hinzu. Nun ließ sich eine bestimmte Regel, ein Handlungselement oder eine Fähigkeit finden, ohne jedes Mal das gesamte Material zu laden. Das half mir ebenso wie den Modellen: Zu einer Aufgabe konnte ich nur den dafür relevanten Teil der Dokumentation mitgeben.

> **„Wenn der Code von einem Modell zum nächsten wanderte, bewahrte die Spezifikation das Projekt vor dem Chaos.“**

Dieses Dokumentensystem war mehr als eine ausführliche Beschreibung des künftigen Spiels. Es trennte bereits getroffene Entscheidungen von den Annahmen eines bestimmten Modells. Beschränkt man sich auf eine Aufforderung wie „Erstelle ein Strategiespiel mit Punkten und Linien“, füllt das Modell alle Lücken selbst: Gegnerverhalten, Bewegungsgeschwindigkeit, Ergebnisberechnung und viele weitere Details. Zwei Implementierungen unterscheiden sich dann nicht nur in der Codequalität, sondern sind praktisch zwei verschiedene Spiele.

Die Dokumentation verringerte die Zahl zufälliger Entscheidungen und half, jedes Modell zur vereinbarten Projektversion zurückzuführen. Besonders wertvoll wurde das, als die Arbeit zwischen verschiedenen Modellen wechselte. Ich musste den Kontext nicht jedes Mal neu erzählen: Das nächste Modell erhielt den passenden Teil der Spezifikation, die aktuelle Implementierung und eine konkrete Aufgabe. Unveränderliche Anforderungen waren die Dokumente dennoch nicht. Nach Testpartien passte ich Regeln an und aktualisierte anschließend ihre Beschreibung, damit Code und Absicht weiter übereinstimmten.

Als die Spezifikation detailliert genug war, ließ ich die Modelle erste Spielvarianten als einzelne HTML-Dateien erstellen. Dieses Format wählte ich wegen der schnellen Überprüfung, nicht wegen einer späteren Architektur. Die Datei ließ sich direkt im Browser öffnen – ohne Projekt, Build, Abhängigkeiten oder eigene Infrastruktur. Spiellogik, Oberfläche und Animation fanden gemeinsam darin Platz, sodass das Ergebnis jeder Iteration fast sofort sichtbar wurde.

Das Experiment war nie als strenger Benchmark gedacht. Die Modelle erhielten weder völlig identische Prompts noch feste Limits oder ein formales Bewertungssystem. Die ersten Varianten entstanden unabhängig; später wanderte die Arbeit wegen Abonnementgrenzen und unterschiedlicher Aufgaben immer häufiger von einem Werkzeug zum nächsten. Ein Modell legte die Grundlage, das nächste behob ein Problem, ein drittes schlug die Weiterentwicklung einer Mechanik vor. Am Ende verglich ich weniger die Modelle selbst als verschiedene Wege, ihre aufeinanderfolgende Arbeit an einem Projekt zu organisieren.

#### Der erste Prototyp und kurze Entwicklungszyklen

Die Kernmechanik blieb kompakt. Auf der Karte lagen durch Routen verbundene Systeme, die nach und nach Einheiten produzierten. Spieler und Computergegner schickten Flotten zwischen benachbarten Knoten, eroberten neue Positionen und versuchten, bereits besetzte zu halten. Für einen erfolgreichen Angriff genügte es nicht, ein schwaches Ziel zu wählen: Man musste bedenken, wie sehr die ausgesandte Flotte die eigene Verteidigung schwächte und ob der Gegner die geöffnete Richtung ausnutzen konnte.

Die erste überzeugende Version entstand in dem Moment, als die gewählte Flotte tatsächlich entlang der Linie zum Zielsystem flog. Die Animation sah besser aus, als ich von einem frühen HTML-Prototyp erwartet hatte, doch etwas anderes war wichtiger: Endlich ließ sich die Mechanik in Aktion beurteilen. Bis dahin gab es nur eine Erinnerung, eine Spezifikation und einzelne Regeln. Nun konnte ich einen Zug machen, seine Folgen sehen und prüfen, ob auf der Karte jene Spannung entstand, wegen der ich das Experiment begonnen hatte.

Der Prototyp bestätigte, dass sich die Fortsetzung lohnte, und es begannen kurze Entwicklungszyklen. Ich präzisierte eine Regel in der Spezifikation, gab dem Modell eine klar begrenzte Aufgabe, startete die neue Version und spielte einige Partien. Verhielt sie sich anders als erwartet, wurde die Abweichung zur nächsten Aufgabe. Dieser Ablauf war wertvoller als reine Generierungsgeschwindigkeit, weil jede Änderung schnell ein überprüfbares Ergebnis lieferte.

In diesen Iterationen konnte ich Produktionsrate, Flottengeschwindigkeit und das Verhältnis von Angriff zu Verteidigung verändern und unmittelbar beobachten, wie sich die neuen Werte auswirkten. Nach und nach wurde klar, wann Schwierigkeit zur Suche nach einer Lösung anregte und wann sie nur unfair wirkte. Die alte Erinnerung gab die Richtung vor, doch die konkreten Regeln von Synaptic Front entstanden erst beim Testen.

Als nächstes Hilfswerkzeug kam ein einfacher Level-Editor hinzu. Die erste Karte ließ sich zwar von Hand in den Daten beschreiben, für eine echte Prüfung brauchte ich jedoch verschiedene Anordnungen von Knoten und Routen. Mit dem Editor konnte ich solche Varianten schneller erstellen und testen, ob das Spiel auch jenseits eines gelungenen Layouts interessant blieb. Er war nicht für Nutzer gedacht und brauchte keinen Produktfeinschliff, beschleunigte aber den Hauptzyklus deutlich. Gerade für solche internen Werkzeuge war KI-Generierung besonders nützlich: Ein kleiner Implementierungsaufwand zahlte sich schon in den nächsten Iterationen aus.

#### Von HTML zu Android

Nach mehreren funktionierenden Versionen war klar, dass die Idee den ersten Test bestanden hatte. Gleichzeitig entstand eine schwierigere Aufgabe: den gefundenen Prozess über eine eigenständige HTML-Datei hinauszuführen. Android lag mir näher als iOS, deshalb wollte ich die nächste Version mit Kotlin und Compose entwickeln. Dieser Schritt trennte das schnelle Experiment von der Entwicklung einer echten Anwendung.

> **„HTML bewies, dass die Idee funktioniert; Android sollte daraus ein Produkt machen.“**

Bis dahin hatte ich bereits mehrere Modelle am Projekt ausprobiert. Fable war das erste: Es war gerade erschienen, und ich wollte sehen, wie es mit einer realen Aufgabe umging. Das damals in Codex verfügbare GPT-5.5 blieb nach meinem Eindruck dahinter zurück, weshalb ich Codex selbst kaum nutzte. In den frühen Phasen der Android-Entwicklung blieb Claude mein wichtigstes Werkzeug.

Im Browserprototyp konnten Spiellogik, Darstellung und Zustand nebeneinander liegen, und eine misslungene Version ließ sich leicht vollständig ersetzen. Im Android-Projekt kamen Anwendungslebenszyklus, Datenspeicherung, Zustandsverwaltung, Tests und die Prüfung auf einem echten Gerät hinzu. Ein funktionierender Bildschirm blieb ein wichtiges Ergebnis, war aber kein Beleg mehr für ein fertiges Produkt. Jede Änderung musste im Kontext der bestehenden Struktur und ihrer Auswirkungen auf den Rest der Anwendung beurteilt werden.

Mit der Veröffentlichung von Sol änderte sich die Lage. Das neue Modell arbeitete deutlich sicherer als 5.5 und auf einem mit Fable vergleichbaren Niveau. Deshalb kündigte ich Claude und setzte die Entwicklung in Codex fort. Zu diesem Zeitpunkt hatte ich drei Limit-Resets angesammelt, und Plus reichte aus, um ohne Eile weiterzuarbeiten. Den Großteil der Android-App entwickelte ich in diesem Rhythmus: nächste Aufgabe stellen, Ergebnis prüfen und weitermachen, sobald das Limit wieder frei war.

Wie Claude konnte Codex selbst den Android-Emulator starten, die Anwendung öffnen, Screenshots aufnehmen und eine Änderung überprüfen. Wurde ein Bildschirm falsch dargestellt oder funktionierte ein Ablauf nicht, konnte das Modell das Problem im Emulator sehen, zum Code zurückkehren, es beheben und erneut testen. Für die Android-Entwicklung war das viel nützlicher als bloße Dateigenerierung: Ein großer Teil des Zyklus von der Änderung bis zur visuellen Kontrolle lief ohne manuelles Wechseln zwischen Werkzeugen ab.

Etwa in der Mitte der Entwicklung erschien Kimi K3, dem ich nun ebenfalls Aufgaben übergab. Der spätere Teil der Android-App entstand somit gemeinsam: Manche Teile schrieb Codex mit Sol, andere Kimi K3. Die Dienste erreichten ihre Limits zu unterschiedlichen Zeiten; statt zu warten, wechselte ich zum nächsten Modell. Zugleich war das ein weiterer Prozesstest: Wie gut konnte ein neues Werkzeug das Projekt seines Vorgängers fortsetzen?

Jedes Modell las und klärte zunächst die Dokumentation und änderte erst dann den Code. Wurde bei der Arbeit eine neue Einschränkung entdeckt oder eine andere Entscheidung getroffen, floss sie zurück in die Dokumente und wurde Teil des gemeinsamen Kontexts. Die Übergabe funktionierte damit in beide Richtungen: Das nächste Modell erhielt das gesammelte Wissen und hinterließ seinem Nachfolger eine genauere Beschreibung. Allmählich verlor die Wahl eines einzigen „besten“ Modells ihren Sinn. Viel wichtiger wurden Dokumentationsqualität, Aufgabengröße und die Möglichkeit, eine Änderung selbstständig zu überprüfen.

#### Wie sich die Rolle der KI mit dem Projekt verändert

Beim frühen Prototyp waren Fehler billig. Funktionierte eine Version schlecht, konnte ich sie nach einer Spielsitzung verwerfen. Deshalb ließen sich in dieser Phase große Teile der Implementierung an Modelle übergeben und Varianten schnell ausprobieren. Im Android-Projekt begann dasselbe Vertrauen die Arbeit zu bremsen. Die Codebasis wuchs, Komponenten wurden voneinander abhängig, und die Behebung eines Problems konnte eine Annahme verletzen, auf der ein anderer Teil der Anwendung beruhte.

> **„Je größer das Projekt wurde, desto weniger Arbeit konnte ich der KI vorbehaltlos überlassen.“**

Mit der Entwicklung des Projekts verschob sich die Rolle der KI von der Erzeugung ganzer Implementierungen hin zur technischen Unterstützung. Es wurde sinnvoller, mehrere Lösungswege zu besprechen, Hypothesen zu prüfen, die Quelle einer Abweichung zu suchen und kleine, kontrollierte Änderungen vorzubereiten. Nach jedem wesentlichen Schritt waren weiterhin Tests und manuelle Prüfungen nötig, und die endgültige Entscheidung blieb bei dem Menschen, der das Produkt verstand und für das Ergebnis verantwortlich war.

Auch die Synchronisierung von Dokumentation und Code wurde entscheidend. Korrigierte ich ein Verhalten von Hand, genügte es nicht, die Änderung nur im Repository zu speichern. Die neue Regel oder Architekturentscheidung musste ebenso in die Dokumente einfließen, die die Modelle erhielten. Andernfalls arbeiteten Mensch und KI schon nach wenigen Iterationen mit verschiedenen Projektständen: Das Modell reproduzierte eine alte Einschränkung oder versuchte, eine verworfene Entscheidung wiederherzustellen, und Zeit ging für vermeidbare Widersprüche verloren.

Diese Erfahrung zeigte nicht, dass KI Spieleentwicklung einfach macht oder das Erlernen eines neuen Stacks überflüssig werden lässt. Ihr Wert lag woanders: Die Werkzeuge brachten mich zur ersten testbaren Version, bevor mich die Einstiegskosten dazu gebracht hätten, die Idee aufzugeben.

Nach dem erfolgreichen HTML-Prototyp änderte sich die Aufgabe. An die Stelle von Zweifeln an der Mechanik traten gewöhnliche Fragen der Produktentwicklung: Wie pflegt man den Code, wo zieht man Zustandsgrenzen, wie überprüft man Änderungen und wie bewahrt man das Wissen über getroffene Entscheidungen? So wurde aus einem Experiment mit mehreren Modellen und eigenständigen HTML-Dateien nach und nach ein Android-Projekt – und aus der Erinnerung an ein namenloses Flash-Spiel wurde Synaptic Front.

> **Synaptic Front ausprobieren**
>
> Das Spiel ist bereits bei [Google Play](https://play.google.com/store/apps/details?id=me.vasylchenko.synapticfront) verfügbar. Dort kannst du sehen, was aus der Idee dieser Geschichte geworden ist, und selbst spielen.
>
> *Fortsetzung folgt.*
