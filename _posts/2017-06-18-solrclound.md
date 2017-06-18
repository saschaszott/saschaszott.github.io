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

Ein Suchindex kann in mehrere Teile geschnitten werden. Dieses Konzept nennt man **Sharding**. Die einzelnen Teile sind wiederum eigenständige Lucene-Indizes und werden **Shards** genannt. Man kann jedem Solr-Server des SolrCloud-Clusters einen anderen Shard zuweisen. Dazu darf die Anzahl Shards nicht größer als die Anzahl der zur Verfügung stehenden Solr-Server sein.

Wird ein neues Dokument indexiert, so wird zuerst bestimmt, in welchem Shard das Dokument gespeichert werden muss. Diesen Vorgang nennt man **Document Routing**. Es ist auch möglich, den Shard explizit vorzugeben, indem man den Shardnamen getrennt durch ein Ausrufezeichen als Präfix vor die Dokument-ID fügt.

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
