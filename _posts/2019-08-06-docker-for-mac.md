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
`stderr`. Solche Logausgaben werden dann im Docker Host in entsprechende JSON-Logdateien geschrieben (bei der Verwendung des Standard-
Logging-Drivers von Docker, momentan ist das `json-file`). Der Container "bläht" sich dadurch nicht mehr auf.

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

Hierbei bezeichnet `/proc/self/fd/1` `stdout` und `/proc/self/fd/2` `stderr`.

## Wo finde ich nun aber die Logausgabe?

Nutzt man Docker unter Linux ist die Frage recht einfach zu beantworten.

Jeder Docker Container wird durch ein Verzeichnis unterhalb von `/var/lib/docker/containers` abgebildet. Der Verzeichnisname
entspricht damit der ID des Docker-Containers (Langform). In dem Verzeichnis befindet sich eine Datei `<CONT_ID>-json.log`, in
der die Ausgabe in `stdout` bzw. `stderr` gesammelt wird. Da es sich um eine JSON-Datei handelt, kann die Datei auch mit Tools
wie `jq` bequem ausgewertet und gefiltert werden.

## Besonderheit: Docker for Mac

Docker for Mac nutzt einen anderen Ansatz. Es existiert kein Verzeichnis `/var/lib/docker` auf dem MacOS-Host. Grund dafür: 
bei Docker for Mac gibt es (im Gegensatz zu Linux, wo Docker "nativ" läuft) eine zusätzliche Hypervisor-Schicht (Hyperkit).
Der Zugriff auf die oben angesprochene Logdatei erfordert daher einen Umweg:

```
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

Es kann vorkommen, dass statt der Verzeichnis `0` eine andere ID verwendet wird. Zur Not im Dateisystem des Hosts prüfen.

Es melde sich dann der Hyperkit Hypervisor:
```

```

Nun kann man in das o.g. Container-Verzeichnis unterhalb von `/var/lib/docker` wechseln.






