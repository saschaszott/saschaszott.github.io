---
layout: post
title:  "ZooInspector"
date:   2017-06-17 16:16:01 -0600
---

ZooInspector ist ein GUI, um die im Zookeeper-Server abgelegten Konfigurationsdateien anzuzeigen. 

ZooInspector erlaubt es auch Änderungen an den abgelegten Konfigurationsdateien vorzunehmen.

Um ZooInspector (hier in der aktuellen Version 3.4.9) zu starten muss in das Verzeichnis `contrib/ZooInspector` gewechselt werden 
und dort der Befehl

````
java -cp ../../lib/*:lib/*:zookeeper-3.4.9-ZooInspector.jar:../../zookeeper-3.4.9.jar org.apache.zookeeper.inspector.ZooInspector
````

ausgeführt werden.
