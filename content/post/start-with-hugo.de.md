---
title: "Erste Schritte mit Hugo"
date: 2020-10-19T19:00:00Z
categories:
  - "Manuals"
draft: false
---

Einen Blog mit Hugo erstellen.
Statische Websites sind in den vergangenen Jahren sehr beliebt geworden.
Das liegt vermutlich daran, dass sie relativ einfach zu pflegen sind und sich auf vielen Plattformen veröffentlichen lassen.
Während man früher für eine statische Website direkt mit HTML arbeiten musste, reichen heute für die meisten Aufgaben Markdown-Kenntnisse aus.
Hugo-Websites eignen sich hervorragend für persönliche Seiten, deren Inhalte nicht ständig aktualisiert werden müssen.
Hugo ist eine Sammlung von Werkzeugen, die aus Markdown-Dateien und Templates eine statische Website erzeugt.
Sehr praktisch sind außerdem die verfügbaren Docker-Images: Damit lässt sich die Website erstellen und bauen, ohne Go, Hugo oder weitere Komponenten lokal zu installieren.

Fangen wir an.
Wenn Sie Linux verwenden und Docker installiert haben ([docker.com](https://www.docker.com)), sind nur wenige Schritte erforderlich.
Erstellen Sie mit folgendem Befehl eine leere Website im Verzeichnis `vasylchenko.me`:

```bash
docker run --rm -it -v $(pwd)/:/src klakegg/hugo:ext new site vasylchenko.me
```

Wechseln Sie anschließend mit `cd vasylchenko.me` in das neue Verzeichnis.
Initialisieren Sie danach ein Git-Repository für die Versionsverwaltung:

```bash
git init
```

Nun ist es Zeit, ein Theme auszuwählen.
Eine Liste beliebter Themes finden Sie auf [themes.gohugo.io](https://themes.gohugo.io).
Für diese Website habe ich zum Beispiel [Devise](https://themes.gohugo.io/devise/) gewählt.
Öffnen Sie die Seite des Themes und führen Sie danach diesen Befehl aus:

```bash
git submodule add https://github.com/austingebauer/devise themes/devise
```

Der Befehl fügt das Repository des Themes als Git-Submodul zum Ordner `themes` hinzu.

Jetzt gibt es das Website-Verzeichnis `vasylchenko.me` und das Theme-Verzeichnis `vasylchenko.me/themes/devise`.
Als Nächstes muss das gewählte Theme in der Hugo-Konfiguration eingetragen werden.
Sie können die Datei manuell bearbeiten oder folgenden Befehl verwenden:

```bash
echo 'theme = "devise"' >> config.toml
```

Zum Schluss wird die Website gebaut und überprüft:

```bash
docker run --rm -it -v $(pwd):/src klakegg/hugo:ext
```

Nach dem Build finden Sie die Ausgabe im Ordner `vasylchenko.me/public` und können dort `index.html` im Browser öffnen.
Wenn die Website wie erwartet aussieht, lässt sich der Inhalt dieses Verzeichnisses auf einen statischen Hosting-Dienst hochladen.
