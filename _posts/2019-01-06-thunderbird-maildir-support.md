---
layout: post
title:  "Unterstützung des Maildir Mailbox Formats in Thunderbird"
date:   2019-01-16 12:00:00 -0600
---

# Unterstützung des Maildir Mailbox Formats in Thunderbird

Ich habe mich schon länger gewundert, warum mein inkrementelles Backup regelmäßig mehrere GB umfasst,
obwohl ich seit dem letzten inkrementellen Backup nur an wenigen (kleinen) Dateien Änderungen 
vorgenommen habe bzw. nur wenige (kleine) Dateien neu angelegt habe. Es kann durchaus im Abstand von
wenigen Stunden passieren, dass ein inkrementelles Backup auf mehrere GB wächst.

Dieser Beobachtung habe ich lange Zeit keine große Aufmerksamkeit gewidmet. Nun hatte ich aber die
Zeit der Ursache auf den Grund zu gehen. Und nach einer Weile wurde ich auch fündig: der Mail-Client
Thunderbird (momentan setze ich Version 60 ein) speichert alle Mails eines Postfachs (im TB-Sprech
"profile" genannt) in **einer** MBOX-Datei. Bei mir ist diese Datei schon einige GB groß, da sich
alle Mails inkl. Anhänge in dieser Datei befinden.

Jede neue Mail (gesendet oder empfangen) führt nun zu einer (wenn auch kleinen) Änderung an der
MBOX-Datei. Auch die Komprimieren-Funktion von TB führt zur Umorganisation der MBOX-Datei. Aus 
Sicht des inkrementellen Backup-Mechanismus führt jede Änderung (so klein sie auch sein mag) zur
Notwendigkeit die gesamte MBOX-Datei ins Backup aufzunehmen. Das führt zur Aufblähung des Backups,
die nur durch das Zusammenfügen mehrerer Inkremente zu einem Inkrement gelöst werden kann (Merge).
Diese Operation ist aufwendig.

Die Lösung des Problems ist eigentlich recht einfach: statt alle Mails eines Postfachs in eine
Datei zu packen, wird eine Datei pro Mail spendiert. Neu eintreffende Mails führen zu neuen Dateien.
Diese werden beim nächsten inkrementellen Backupdurchlauf gesichert. Gelöschte Mails führen zum
Löschen von Dateien. Aus Sicht des Backups muss nur das Entfernen in den Metadaten gespeichert werden.
Die Speicherung / Organisation von E-Mails eines Postfachs in dieser Art und Weise wird als Maildir-Format
bezeichnet.

Viele Mail-Clients unterstützen sowohl das MBOX- als auch das Maildir-Format. Wikipedia hat dazu
eine hilfreiche 
[Vergleichstabelle](https://en.wikipedia.org/wiki/Comparison_of_email_clients#Database,_folders_and_customization)
(siehe Spalte "Message File Format").

Leider unterstützt Thunderbird das Maildir-Format noch nicht vollumfänglich. Die Entwickler haben das
Feature als *experimental* ausgewiesen. Das Feature kann über einen Umweg aktiviert werden. TB erlaubt
dann auch die Konvertierung eines bestehenden MBOX-Profils in ein Maildir-Profils.

Unter

https://www.thunderbird.net/en-US/thunderbird/60.0beta/releasenotes/

wird der entsprechende Umweg beschrieben (`mail.store_conversion_enabled` heißt der Konfigurationsschlüssel).

Leider gibt es noch einige offenen Punkte bis die Maildir-Unterstützung in TB als stabil angesehen werden
kann. Ich würde momentan noch nicht auf Maildir setzen, da mir das Risiko des Mailverlusts zu groß erscheint.
Es gibt aber auch Benutzer, die das Feature schon einsetzen und bislang keine Probleme festgestellt haben.

Hier ohne Anspruch auf Vollständigkeit einige Links:

* [Thunderbird: From mbox to maildir](https://www.wilderssecurity.com/threads/thunderbird-from-mbox-to-maildir.389599/)
* [Status of MailDir](http://forums.mozillazine.org/viewtopic.php?f=28&t=3034422)

Mozilla hat zu der Thematik eine Wikiseite spendiert. Hier sind auch die entsprechenden Bugzilla Issues verlinkt:

https://wiki.mozilla.org/Thunderbird/Maildir

Das Bugzilla Ticket [845952](https://bugzilla.mozilla.org/show_bug.cgi?id=845952) ist ein Meta-Ticket, in dem 
alle relevanten Tickets zur Problematik verlinkt sind. 

Mein Wunsch für 2019: hoffentlich stecken die TB-Entwickler etwas Zeit in die weitere Arbeit am Maildir-Support
in Thunderbird und die baldige Fertigstellung eines stabilen, produktionsreifen Zustands. Mein Backup-Medium würde 
sich darüber zumindest sehr freuen.

## Nachtrag 1

Die Maildir-Implementierung in TB entspricht nicht vollständig der [Maildir-Spezifikation](http://cr.yp.to/proto/maildir.html).
Im Gegensatz zur Spec werden bei TB nur die beiden Verzeichnisse `tmp` und `cur` verwendet. Das Verzeichnis `new` wird
dagegen nicht genutzt.

Ein MBOX-vs-Maildir Benchmark aus dem Jahr 2003 kann unter http://www.courier-mta.org/mbox-vs-maildir/ abgerufen werden.

## Nachtrag 2

Um die MBOX-Datei zu verkleinern, kann das **Archive**-Feature von TB verwendet werden. Jede Mail kann in TB in
ein Archiv verschoben werden. Die Mail ist dann weiterhin durchsuchbar und sie wird weiterhin im Mail Client
angezeigt. Für Einzelheiten zur Funktion sei auf den [KB-Eintrag](https://support.mozilla.org/en-US/kb/archived-messages)
verwiesen.

Jedes Archiv wird in einer eigenen MBOX-Datei verwaltet. Wenn man z.B. eine monatsweise Organisation des Archivs
vornimmt, wird nach spätestens 31 Tagen eine neue MBOX-Datei angelegt. Diese Datei kann pro Monat nur eine moderate
Größe annehmen. Nach Monatsablauf wird das Archiv i.d.R. nur noch selten modifiziert, so dass die zugehörige MBOX-Datei
relativ stabil ist und nur noch selten ins inkrementelle Backup wandert.

Dieser Ansatz erfordert aber manuelle Arbeit, weil die Mails durch den Benutzer in das Archiv verschoben werden müssen.
Es ist also aus meiner Sicht nur ein Workaround um das oben beschriebene Problem zu lösen.


