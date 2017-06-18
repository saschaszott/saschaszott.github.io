---
layout: post
title:  "SolrCloud"
---

## Wichtige Begriffe

Mit der SolrCloud-Funktion kann man seit Solr 4 ein Cluster von Solr-Servern aufbauen. Ein Cluster ist ein verteiltes System, das aus mehreren Solr-Servern besteht, die auf unterschiedlichen Hosts (physisch oder virtualisiert) laufen. Für die ersten Versuche kann man die Solr-Server auch auf dem lokalen Rechner (localhost) ausführen. Dafür müssen die Solr-Server aber auf unterschiedlichen Ports gestartet werden.

Unter einem Solr-Server ist hierbei ein Java-Prozess zu verstehen, in dem die Solr-Webapp läuft (in einem eingebetten Jetty Servlet Container).

Im Zusammenhang mit SolrCloud gibt es einige wichtige Begriffe, die man kennen sollte:

Ist ein Suchindex zu groß oder ist die *Query Load* zu groß, so dass der Betrieb auf einer einzelnen Maschine nicht mehr darstellbar ist, so kann der Suchindex auf mehrere Maschinen verteilt werden (horizontale Skalierung). Die Verteilung kann auch genutzt werden, um ein redundantes System zu erzielen, das z.B. den Ausfall eines Solr-Servers ohne Effekt für den Endbenutzer aushält.

Ein Suchindex ist ein gewöhnlicher Lucene-Index, der im Dateisystem als eine Menge von Dateien gespeichert ist.

Ein Suchindex kann in mehrere Teile geschnitten werden. Dieses Konzept nennt man **Sharding**. Die einzelnen Teile sind wiederum eigenständige Lucene-Indizes und werden **Shards** genannt. Die Menge aller Shards wird als **Collection** bezeichnet. Eine Collection ist folglich ein logischer Lucene-Index, der die Dokumente aller Shards umfasst. Man kann jedem Solr-Server des SolrCloud-Clusters einen anderen Shard zuweisen. Dazu darf die Anzahl Shards nicht größer als die Anzahl der zur Verfügung stehenden Solr-Server sein.

Wird ein neues Dokument indexiert, so wird zuerst bestimmt, in welchem Shard das Dokument gespeichert werden muss. Diesen Vorgang nennt man **Document Routing**. Dazu wird auf Basis der Dokument-ID der zugehörige Wert der MurmurHash3-Funktion (32 Bit) berechnet. Beispielweise bekommt die Dokument-ID 1 den hexadezimalen Hashwert *0x9416AC93* zugewiesen. Jeder Shard bekommt bei der Erzeugung einen ID-Bereich zugewiesen (z.B. im Falle von 2 Shards: *0-7fffffff* und *80000000-ffffffff*). Das Dokument wird nun dem Shard zugeordnet, in dessen ID-Bereich der zuvor berechnete Hashwert des Dokuments fällt. In dem Beispiel wird das Dokument mit der ID 1 dem zweiten Shard zugewiesen.

Will man sicherstellen, dass bestimmte Dokumente dem gleichen Shard zugewiesen werden, so kann durch das Setzen eines Präfix vor die Dokument-ID die Hashberechnung beeinflussen. Hierbei ist zu bemerken, dass man nicht explizit festlegt, welchem Shard die Dokumente zuegwiesen werden sollen, sondern lediglich, dass sie dem gleichen Shard zugewiesen werden sollen. Vor die Dokument-ID wird dazu ein gemeinsames Präfix geschrieben, das mittels Ausrufezeichen von der ursprünglichen Dokument-ID getrennt wird

Beispiel: Die Dokumente mit ID 1 und 2 sollen im gleichen Shard landen. Standardmäßig wäre das bei 2 Shards nicht der Fall, da der hexadezimale Murmur3-Hashwert von 2 (0x129E217) nicht in den gleichen ID-Bereich fällt, wie der Hashwert von Dokument 1 (0x9416AC93).
Durch das Voranstellen eines gemeinsamen Präfix (z.B. *s1!1* bzw. *s1!2*), wird nun aber sichergestellt, dass die beiden Dokumente tatsächlich im gleichen Shard landen.

Problematisch an diesem Vorgehen ist, dass jeder Solr-Server exklusiv einen Teil der Collection, also des Gesamtindex, hält. Fällt ein Shard aus, dann können die in dem Shard abgelegten Dokumente nicht mehr gefunden werden. Die Lösung dieses Problems besteht in der Einführung von Shard-Kopien. Für jeden Shard wird eine Menge von exakten Kopien erzeugt. Diese Kopien bezeichnet man als **Replicas**. Die Shard-Replicas werden auf anderen Solr-Servern repliziert. Die Anzahl der Kopien ist ein Konfigurationsparamter des SolrCloud-Clusters.

Ein Shard umfasst nun einen *Master* und eine vorgegebene Anzahl von Kopien, die Replicas. Der Master wird auch als **Shard Leader** bezeichnet. Alle Dokumente, die bei der Indexierung dem Shard zugewiesen werden, werden zum Shard Leader geleitet und dort indexiert. Anschließend kümmert sich der Leader um die Replikation zu den einzelnen Replicas des Shards.

Eine Suchanfrage gegen das SolrCloud-Cluster wird nun beantwortet, indem ein Broadcast an alle Shard Leader gesendet wird und schließlich die einzelnen Teilergebnisse zu einem Gesamtranking zusammengeführt werden. Der Benutzer erhält eine Ergebnisliste und merkt nicht, dass der Gesamtindex gar nicht physisch auf einem Solr-Server existiert, sondern verteilt auf mehrere Solr-Server ist.

Fällt nun ein Solr-Server aus, so kann es passieren, dass auf dem ausgefallenen Solr-Server ein Shard Leader existierte. Für jeden Shard muss im SolrCloud-Cluster ein Leader existieren. Daher muss in diesem Fall ein neuer Leader bestimmt werden. Dazu wird aus den Shard Replicas ein Solr-Server ausgewählt (**Shard Leader Election**). Dieser wird neuer Shard Leader.

Wird der ausgefallene Solr-Server wieder hochgefahren, so meldet er sich im SolrCloud-Cluster an und muss nun reintegriert werden. In Abhängigkeit von der Dauer des Ausfalls kann der Dokumentenbestand auf dem Solr-Server veraltet sein. Daher muss eine Wiederherstellungsoperation (**Node Recovery**) gestartet werden, die zum Ziel hat den Dokumentenbestand des Solr-Servers wieder mit dem Zustand im Cluster zu synchronisieren. Dazu hat jedes Index-Dokument ein Feld *_version_*, das von Solr verwaltet und zur effizienten Synchronisation verwendet wird. War der Solr-Server vor der Downtime Shard Leader, so wird er nun Shard Replica. Eine automatische Ernennenung des Solr-Servers zum Shard Leader erfolgt nicht.

## Beispiele

Ein SolrCloud-Cluster besteht aus 2 Solr-Servern und 2 Shards. Jeder Shard hat zwei Kopien (einen Shard Leader und ein Shard Replica).

Es ergebit sich folgende Aufteilung:

Shard 1: Leader_1, Replica_1

Shard 2: Leader_2, Replica_2

Die vier Teilen können nun wie folgt auf die beiden Solr-Server verteilt werden:

Server 1: Leader_1, Replica_2

Server 2: Leader_2, Replica_1

Fällt ein Solr-Server aus, so gibt es für den Benutzer keine Auswirkungen, da jeder Solr-Server den gesamten Index (bestehend aus zwei Shards) enthält. Im Falle des Ausfalls hält der verbleibende Solr-Server zwei Shard Leader.

## Starten von SolrCloud

Ein lokales Beispiel-Setup kann erzeugt werden mittels

````
bin/solr -e cloud
````

Anschließend werden alle erfolrderlichen Einstellungen interaktiv abgefragt.

## Herunterfahren eines Clusterknotens

Einen Knoten (Solr Node) herunterfahren mittels

````
bin/solr stop -p 8984
````

Die Referenzierung des Knotens erfolgt über seinen Port (hier 8984).

## Neustarten eines Clusterknotens

Ein Knoten kann wieder hochgefahren werden mittels

````
bin/solr start -cloud -p 8984 -s "example/cloud/node2/solr" -z localhost:9983
````

Als Wert des Parameters **z** wird die Adresse des Zookeeper-Servers angegeben.

## Gesamtes SolrCloud Cluster herunterfahren

Alle Knoten des SolrCloud Clusters können heruntergefahren werden mittels

````
bin/solr stop -all
````
