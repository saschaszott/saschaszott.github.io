---
layout: post
title:  "Hinweise zu Docker for Mac"
date:   2019-08-06 12:00:00 -0600
---

Folgt man der Docker-Prämisse "One Process Per Container", so sollte es in einem Docker-Container nur wenige (im besten Fall eine)
relevante Logdatei für den ausgeführten Prozess geben. Beispiel: In einem Container, in dem ein Apache Webserver läuft, sind die 
beiden Logdateien `access.log` und `error.log` relevant. Ggf. hat man noch eine spezifische Logdatei für die im Apache eingebundene
Application (meist als `VirtualHost`).

Folgt man der zweiten Docker-Prämisse "Containers Should Be Stateless", so sollten keine Logdateien in den Container geschrieben werden.
Eine Ausnahme könnten Test-Container sein, die man lediglich zur Ausführung von Tests im Rahmen von Continuous Integration startet
und nach der Asführung der Tests wieder entsorgt.

Um die zweite Prämisse zu erfüllen, könnte man Logdateien (Verzeichnisse, in denen sich die Logdateien befinden) über Volumes in
den Docker Host mounten. Eine zweite Möglichkeit besteht darin, die Logausabe nicht in Dateien zu schreiben, sondern nach `stdout` bzw.
`stderr`. Solche Logausgaben werden dann im Docker Host in entsprechende JSON-Logdateien geschrieben (bei der Verwendung des 
Standard-Logging-Drivers von Docker, momentan ist das `json-file`). Der Container "bläht" sich dadurch nicht mehr auf.

Dieses Vorgehen kann man sich z.B. im Dockerfile des offiziellen Apache HTTPd Images ansehen: https://github.com/docker-library/httpd/blob/master/2.4/Dockerfile

Dort wird die Apache Konfiguration entsprechend angepasst:

```
sed -ri \
	-e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g' \
	-e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' \
	-e 's!^(\s*TransferLog)\s+\S+!\1 /proc/self/fd/1!g' \
	"$HTTPD_PREFIX/conf/httpd.conf" \
	"$HTTPD_PREFIX/conf/extra/httpd-ssl.conf" \
```

`/proc/self/fd/1` bezeichnet hierbei `stdout` und `/proc/self/fd/2` bezeichnet `stderr`.

## Wo finde ich nun aber die Logausgabe, wenn der Prozess seine Logausgaben in `stdout` bzw. `stderr` schreibt?

Nutzt man Docker unter Linux ist die Frage recht einfach zu beantworten.

Jeder Docker-Container wird durch ein Verzeichnis unterhalb von `/var/lib/docker/containers` abgebildet. Der Verzeichnisname
entspricht der ID des Docker-Containers (Langform). In dem Verzeichnis befindet sich eine Datei `<CONT_ID>-json.log`, in
der die Ausgabe in `stdout` bzw. `stderr` gesammelt wird. Da es sich um eine JSON-Datei handelt, kann die Datei auch mit Tools
wie `jq` bequem ausgewertet und gefiltert werden.

## Besonderheit: Docker for Mac

Docker for Mac nutzt einen anderen Ansatz. Es existiert kein Verzeichnis `/var/lib/docker` auf dem MacOS-Host. Grund dafür: 
bei Docker for Mac gibt es (im Gegensatz zu Linux, wo Docker "nativ" läuft) eine zusätzliche Hypervisor-Schicht (Hyperkit).
Der Zugriff auf die oben angesprochene Logdatei erfordert daher einen Umweg.

In einem Terminal führt man folgenden Befehl aus:

```
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

Es kann vorkommen, dass statt des Verzeichnisnamens `0` eine andere ID verwendet wird. Zur Not im Dateisystem des MacOS-Hosts prüfen.

Es melde sich dann der Hyperkit Hypervisor mit einer Willkommensmeldung:
```
Welcome to LinuxKit

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""__/ ===
          {                       /  ===-
           _____ O           __/
                        __/
              _________/

docker-desktop login: root (automatic login)

Welcome to LinuxKit!

NOTE: This system is namespaced.
The namespace you are currently in may not be the root.
System services are namespaced; to access, use `ctr -n services.linuxkit ...`
login[2569]: root login on 'ttyS0'
docker-desktop:~# 
```

Nun kann man in das o.g. Container-Verzeichnis unterhalb von `/var/lib/docker` wechseln und die Logdatei öffnen.

Alternativ kann man mit dem Befehl `docker logs <CONT_ID>` den Inhalt der Logdatei anzeigen. Es wird mit der Option `-f`
auch ein "Follow Mode" angeboten, der neu hinzugefügte Zeilen ausgibt.

Wird die Logdatei allerdings zu groß, so ist zum Löschen der beschriebene Umweg erforderlich. `docker logs` hilft hier leider
nicht weiter. Als Alternative kann man den Docker Daemon anweisen eine Log Rotation durchzuführen.

## Log Rotation aktivieren

Die vom Docker Daemon geschriebenen Logdateien wachsen unbeschränkt, sofern keine enstprechende Einstellung vorgenommen wurde.
Durch das Aktivieren von Log Rotation kann einerseits die Größe einer Logdatei beschränkt werden. Außerdem kann die Anzahl an
Logdateien beschränkt werden. Dadurch kann die Gesamtgröße der Logdateien begrenzt werden.

Die Log Rotation kann über die Konfigurationsdatei `/etc/docker/daemon.json` gesteuert werden. **Achtung** Eine Veränderung
der Einstellung greift nur für neu angelegte Container. Bestehende Container respektieren diese Änderung nicht.

Die Einstellung in `daemon.json` ist global. Sie kann beim Erstellen neuer Container selektiv überschrieben werden.

Wird der Standard Logging Driver von Docker verwendet, so müsste folgende Änderung in `daemon.json` vorgenommen werden,
wenn maximal 5 Log-Dateien mit jeweils maximal 10 MB angelegt werden dürfen. Ist das Limit überschritten, so wird die
älteste Lognachricht überschrieben (Rotation).

```json
{
   "log-driver": "json-file",
   "log-opts": {
      "max-size": "10m",
      "max-file": "5"
   }
}
```

**Achtung**: Die Werte müssen zwingend in *Double Quotes* geschrieben werden!

Anschließend muss der Docker Daemon neu gestartet werden. Wie schon gesagt: die Einstellung greift nur für neu angelegte Container!

## Log Rotation unter Docker for Mac

Die Datei `daemon.json` kann auf dem MacOS-Host nicht gefunden werden. Auch hier muss wieder mittels `screen` in die virtuelle
Machine, die von HyperKit ausgeführt wird, gewechselt werden.

Die Datei befindet sich dann unter `./run/config/docker/daemon.json`.

Sie kann auf der Kommandozeile auch ohne Editor bearbeitet werden:

```
# Inhalt ausgeben
cat /run/config/docker/daemon.json

# Datei neu schreiben
cat - > /run/config/docker/daemon.json

# zum Beenden und Speichern: STRG+C drücken
```
