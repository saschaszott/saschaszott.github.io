---
layout: post
title:  "How to fix an invalid config file in a stopped Docker container (on Mac OS X)"
date:   2019-01-27 12:00:00 -0600
---

In einem MySQL Docker-Container (für die Entwicklung) habe ich eine temporäre Änderung an der 
MySQL-Konfigurationsdatei `etc/mysql/mysql.conf.d/mysqld.cnf` vorgenommen, um alle eintreffenden 
SQL-Statements in einer Protokolldatei zu erfassen (mir ist bekannt, dass man eigentlich
keine Konfigurationsänderungen in bestehenden Containern vornehmen sollte, sondern besser das
`Dockerfile` anpasst, einen neues Image erzeugt und schließlich einen neuen Container aus
dem Image erzeugt und startet).

Für schnelle Testzwecke auf der Entwicklungsmaschine ist das Vorgehen aber praktisch.
In der Datei `etc/mysql/mysql.conf.d/mysqld.cnf` habe ich folgende Zeilen am Ende hinzugefügt:

```
general_log_file=/var/log/mysql/mysql.log
general_log=1
```

Dabei ist mir nun ein Schreibfehler im Namen des Konfigurationsschlüssels `general_log_file` 
unterlaufen, so dass die Konfigurationsdatei nach dem Speichern nicht mehr valide war.

Der Neustart des MySQL Docker-Containers musste folglich erfolglos sein. Und nun? Wie kann
ich eine zentrale Konfigurationsdatei in einem Docker-Container editieren, der sich nicht
mehr hochfahren lässt, weil der Service (hier MySQL) aufgrund der invaliden Konfigurationsdatei
nicht mehr gestartet werden kann.

Eine weitere Besonderheit in meinem Setup: die Docker Container werden mittels **Docker Desktop for Mac**
unter Mac OS X betrieben.

Zuerst brauchen wir die ID des Problem-Containers (hier hat dieser den Namen `sick-container`). Dazu ruft man
```
$ docker inspect sick-container
```
auf. Auf der Kommandozeile wird ein umfangreiches JSON-Objekt ausgegeben. Am Anfang wird unter `Id` die
(ziemlich lange) Container-ID ausgegeben. Unter `Args` wird der Befehl aufgelistet, der beim Start des
Containers ausgeführt wird (in meinem Fall steht dort `mysqld` für den Start des MySQL Datenbank-Daemons).

Im Folgenden nehme ich an, dass der `overlay2` Storage Driver von Docker für den Container `sick-container`
verwendet wird (`overlay2` ist aktuell der Standard).

Unter `UpperDir` wird nun der Pfad zu einem Verzeichnis angegeben, in dem alle Dateien (und Verzeichnisse)
gespeichert sind, die im Container verändert oder neu angelegt wurden (im Vergleich zum Image, aus dem der
Container erzeugt wurde). Docker verwendet für die Container ein *layered filesystem*, das nach dem
*Copy-on-Write*-Prinzip funktioniert. Demnach werden Dateien erst dann in das `UpperDir` kopiert, wenn sie
verändert oder im Container neu angelegt wurden. Im Detail kann man das unter

https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay-driver-works

nachlesen (dort befindet sich auch ein Schaubild).

Um den Wert des Eintrags schneller anzuzeigen, kann man den o.g. `inspect`-Befehl mit `grep` kombinieren
(oder `jq` bemühen, um im JSON zu selektieren):
```
$ docker inspect sick-container | grep UpperDir
```
In meinem Fall erhalte ich folgende Ausgabe (ID wurde hier für bessere Lesbarkeit gekürzt):
```
"UpperDir": "/var/lib/docker/overlay2/b31b192345f1…/diff",
```

Leider gibt es unter Mac OS X aber kein Verzeichnis `/var/lib/docker`. Dieser Umstand ergibt sich
daraus, dass Docker nicht nativ auf Mac OS X läuft, da kein Linux-Kernel für die Ausführung zur
Verfügung steht. Genau dieses Problem adressiert das Werkzeug **Docker Desktop for Mac**. Es stellt
einen Linux-Kernel in einer virtuellen Maschine bereit, in der dann schließlich der Docker Container
ausgeführt werden kann. Hierbei wird der leichtgewichtige Hypervisor **Hyperkit** für Mac OS X
verwendet (im Gegensatz zu VirtualBox). Dieser Hypervisor ist Bestandteil von **Docker Desktop for Mac**.

Wir brauchen also zuerst einen Zugang auf die virtuelle Maschine, in der der Docker Daemon ausgeführt wird.
In dieser VM können wir dann in das Verzeichnis `/var/lib/docker` wechseln.

Dazu verwenden wir das nützliche Tool `screen`:
```
$ screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

Man wird dann freundlich von einem großen Wal begrüßt und bekommt auch gleich eine Root-Shell:
```

Welcome to LinuxKit

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""___/ ===
          {                       /  ===-
           ______ O           __/
                          __/
              ___________/

linuxkit-025000000001 login: root (automatic login)
```

Nun kann man in das o.g. Verzeichnis wechseln:
```
linuxkit-025000000001:~# cd /var/lib/docker/overlay2/b31b192345f1…/diff
```

Dort findet man nun die Datei `mysqld.cnf` im Untervezeichnis `etc/mysql/mysql.conf.d`.
Die kaputte Konfigurationsdatei kann nun mittels `vi` editiert werden. Nach dem 
Speichern der Änderung kann man sich von der Session abtrennen (*detach*).

Nun startet man den Docker Container neu. Der MySQL Service sollte wieder fehlerfrei
gestartet werden.
