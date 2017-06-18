---
layout: post
title:  "SolrCloud"
---

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
