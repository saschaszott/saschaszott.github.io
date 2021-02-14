---
layout: post
title:  "Python 3 Webhosting mit Netcup"
---

### Vorbemerkungen

- das Webhosting-Paket von Netcup bietet leider keine Möglichkeit per `pip` Pakete nachzuinstallieren
- nach dem Login per SSH steht kein `python` zur Verfügung
- allerdings wird WSGI in Verbindung mit Phusion Passenger unterstützt
- es ist daher hilfreich, wenn man sich vorab unter https://www.phusionpassenger.com/docs/tutorials/what_is_passenger/ über das Konzept informiert
- die folgende Anleitung basiert teilweise auf dem Quickstart Tutorial unter https://www.phusionpassenger.com/docs/tutorials/quickstart/python/
- ferner war folgendes Github-Repo hilfreich: https://github.com/phusion/passenger-python-flask-demo/tree/end_result

Die Anleitung im Netcup-Wiki unter https://www.netcup-wiki.de/wiki/Python_Webprogrammierung scheint nicht mehr dem Stand der Technik zu entsprechen.
Die darin enthaltenen Anweisungen kann man daher ignorieren ;)

### Konfigurationsanleitung für eine einfache Flask-App

#### Vorbereitungen im WCP (Webhosting Control Panel)

1. Neue Subdomain anlegen
   - Name der Subdomain: `flask-app.saschaszott.de`
   - Dokumentenstamm (Document Root): '/flask-app/static`
   - Domain mit Let's Encrypt schützen: Checkbox aktivieren
   - bis zu 5 Minuten warten bis die Subdomain im WCP angezeigt wird
2. Hosting-Einstellungen im WCP aufrufen
   - Dauerhafte, für SEO geeignete 301-Weiterleitung von HTTP zu HTTPS: Checkbox aktivieren
   - Zertifikat: Eintrag `Lets Encrypt flask-app.saschaszott.de (flask-app.saschaszott.de)` aus Dropdown auswählen
   - PHP-Unterstützung: Checkbox deaktivieren
   - CGI-Unterstützung: Checkbox deaktivieren
   - FastCGI-Unterstützung: Checkbox deaktivieren
   - Benutzerdefinierte Fehlerdokumente: Checkbox deaktivieren
3. File Manager (Unterverzeichnis `flask-app/static`)
   - `index.html` löschen
   - `favicon.ico` löschen
   - Verzeichnis `cgi-bin` löschen
4. Eintrag "Python" im WCP aufrufen
   - App Root: `/flask-app`
   - alle anderen Einstellungen können belassen werden
   - auf Button _Einschalten_ drücken
   - es erscheint die Warnmeldung _Warnung: Startup Datei existiert nicht_

#### Python-Projekt auf lokaler Entwicklungsmaschine erstellen

Es wird im Folgenden davon ausgegangen, dass auf der lokalen Entwicklungmaschine Python 3 (hier: 3.6) installiert ist.

```
$ mkdir flask-app
$ cd flask-app
$ echo 'from app import MyApp as application' > passenger_wsgi.py
$ mkdir templates
```

Im Verzeichnis `flask-app` eine Datei `app.py` anlegen mit folgendem Inhalt:

```py
from flask import Flask, render_template
MyApp = Flask(__name__)

@MyApp.route("/")
def index():
    return render_template('index.html')

if __name__ == "__main__":
    MyApp.run()
```

Im Verzeichnis `flask-app/templates` eine Datei `index.html` anlegen mit folgenden Inhalt:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello</title>
  <style>
    html, body {
      font-family: sans-serif;
      background: #f0f0f0;
      margin: 4em;
    }

    .main {
      background: white;
      border: solid 1px #c0c0c0;
      border-radius: 8px;
      padding: 2em;
    }
  </style>
</head>
<body>
  <section class="main">
    <h1>Hello world!</h1>
  </section>
</body>
</html>
```

#### Deployment des Python-Projekts

Per SCP oder FTP den Inhalt des lokal gespeicherten Verzeichnis `flask-app`
auf den Webspace unter `/flask-app` kopieren.

```sh
$ scp -r flask-app/ hostingxxx@188.XXX.XXX.XXX:/
```

Im Webhosting sollte das Verzeichnis `/flask-app` nun folgende Struktur aufweisen:

```
.
├── app.py
├── passenger_wsgi.py
├── static
└── templates
    └── index.html
```

#### Phusion Passenger Server neustarten

In das WCP wechseln und dort für die oben angelegte Subdomain auf den Eintrag
_Python_ klicken.

⚠ Achtung Stolperfalle: die WCP-Session läuft nach ca. 15 Minuten Inaktivität stillschweigend ab.
Leider merkt man das nicht immer unmittelbar, wenn man sich im WCP bewegt. Ich habe mich schon
mehrfach gewundert, warum meine Python-Einstellungen nicht übernommen wurden. Irgendwann habe ich
dann bemerkt, dass ich zwar die Python-Einstellungen aufrufen konnte, aber meine Session schon
abgelaufen war, was letztendlich dazu führte, dass die Einstellungen nicht erfolgreich gespeichert
wurden. Daher mein Tipp: immer mal auf CCP (Customer Control Panel) in einem zweiten Browser-Tab
klicken und schauen, ob man dort tatsächlich noch angemeldet ist.

In den Python-Einstellungen sollte nun die oben noch angezeigte Warnmeldung (`passenger_wsgi.py`
konnte nicht gefunden werden) verschwunden sein.

Nun auf den Button `Anwendung Neuladen (Applikation Neuladen)` klicken.

⚠ Achtung Stolperfalle: wenn man im Formular Einstellungen ändern will, so muss man
erst auf `Ausschalten` und dann erneut auf `Einschalten` klicken, damit die Änderungen
tatsächlich übernommen werden. Damit rechnet man nicht. Ich habe es auch erst unter
https://www.netcup-wiki.de/wiki/Plesk_Onyx_Panel_Webhosting#Python zufällig entdeckt.

#### Erster Aufruf der Subdomain

Vorwarnung: wir erwarten einen Fehler, weil `Flask` nicht zur Verfügung steht

Im Browser die URL https://flask-app.saschaszott.de aufrufen.

Es sollte eine Fehlerseite mit folgender Aussgabe zurückgegeben werden:

```
We're sorry, but something went wrong.
The issue has been logged for investigation. Please try again later.
```

Will man mehr über den Fehler wissen, so kann man in den Python-Einstellungen (s.o.)
auch den Entwicklungsmodus temporär aktivieren. Man erhält dann beim Aufruf der o.g.
URL folgende Fehlerseite:


