---
layout: post
title:  "Syncing your fork in Github"
date:   2019-01-16 12:00:00 -0600
---

Um in Github z.B. einen Pull Request für ein fremdes Projekt (Git-Repo) zu erstellen,
muss in einem ersten Schritt ein Fork des Git-Repos erzeugt werden.

In diesem Fork kann man schließlich seine Änderungen integrieren. Am besten nutzt man
dafür einen für diesen Zweck neu angelegten Branch, so dass man unterschiedliche Änderungen 
voneinander separieren kann und in Form einzelner *Pull Requests* an das Ursprungsprojekt 
distributieren kann.

Irgendwann wird es nun vorkommen, dass der eigene Fork nicht mehr mit dem aktuellen
Zustand des Git-Repos übereinstimmt, aus dem er erzeugt wurde. In diesem Fall zeigt einem
Github die Meldung

```
This branch is 42 commits behind fooproject:master
```

an. Leider bietet die Github-Web-UI keine Funktion an, um einen Branch des Fork mit
einem Branch des originalen Git-Repo zu synchronisieren.

Daher muss man die Konsole bemühen und folgende Schritte durchführen, wobei der erste
Schritt (`git remote …`) nur einmalig durchgeführt werden muss.

Im Folgenden sei angenommen, dass wir den Branch `foo-branch` des Forks mit dem
Branch `bar-branch` des originalen Git-Repos synchronisieren wollen, i.d.R. wird
`bar-branch` der `master`-Branch sein.

```bash
# neues remote mit dem Namen upstream anlegen
$ git remote add upstream https://github.com/$ORIGINAL_OWNER/$ORIGNAL_REPO.git

# Prüfung, ob der vorherige Befehl erfolgreich war (bzw. obsolete ist)
# in der Ausgabe sollte nun das upstream repo erscheinen (neben dem origin repo)
$ git remote -v

# branches und enthaltene Commits aus upstream laden
$ git fetch upstream

# auf den Branch foo-branch von origin wechseln
$ git checkout foo-branch

# Merge durchführen, d.h. foo-branch des Forks mit dem 
# Branch bar-branch des Upstream-Repo synchronisieren
# hierbei werden lokale Änderungen NICHT überschrieben
$ git merge upstream/bar-branch

# Änderungen im foo-branch des Forks auf Github übertragen
$ git push
```

In der Github-Web-UI sollte nun nach Auswahl des `foo-branch` 
die Meldung *This branch is even with $ORIGINAL_OWNER:master* erscheinen, sofern
`bar-branch` durch `master` ersetzt wurde.

Damit entspricht der Stand im Branch `foo-branch` des Forks dem Stand
des Branch `bar-branch` im Original-Repository.
