---
layout: post
title:  "Python 3 Webhosting mit Netcup"
---

### Vorbemerkungen

- das Webhosting-Paket von Netcup bietet leider keine Möglichkeit Python Packages per `pip` (oder `conda`) nachzuinstallieren
- nach dem Login per SSH steht kein `python` (weder 2 noch 3) zur Verfügung
- allerdings wird im Webhosting WSGI in Verbindung mit **Phusion Passenger** unterstützt
- es ist daher hilfreich, wenn man sich vorab unter https://www.phusionpassenger.com/docs/tutorials/what_is_passenger/ über das Konzept von Phusion Passanger informiert
- die folgende Anleitung basiert teilweise auf dem Quickstart Tutorial unter https://www.phusionpassenger.com/docs/tutorials/quickstart/python/
- ferner war folgendes Github-Repo hilfreich: https://github.com/phusion/passenger-python-flask-demo/tree/end_result
- die Anleitung wurde mit Python 3 erstellt und gestest -- vermutlich kann die Anleitung mit leichten Modifikationen aber auch verwendet werden, um ein Python 2 Projekt zu hosten

⚠ Achtung Stolperfalle: Die Anleitung zum Deployment von Python-Anwenungen im Netcup-Wiki unter https://www.netcup-wiki.de/wiki/Python_Webprogrammierung scheint nicht mehr dem Stand der Technik zu entsprechen. Die darin enthaltenen Anweisungen kann man daher ignorieren ;)

### Konfigurationsanleitung für eine einfache Flask-App

#### Vorbereitungen im WCP (Webhosting Control Panel)

1. Neue Subdomain anlegen
   - Name der Subdomain: `flask-app.saschaszott.de`
   - Dokumentenstamm (Document Root): `/flask-app/static`
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
klicken und schauen, ob man dort tatsächlich noch angemeldet ist. Außerdem hat es bei mir mit dem
Safari nicht zuverlässig funktioniert die Änderungen zu speichern. Mit Chrome habe ich diese
Probleme nicht beobachtet, daher würde ich für diese Konfigurationstätigkeiten diesen Browser empfehlen.

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
im WCP auch den Entwicklungsmodus temporär aktivieren. Man erhält dann beim Aufruf der o.g.
URL folgende Fehlerseite:

![Passenger Error Details](/resources/passenger-error-details.png)

Die Fehlermeldung resultiert aus dem Fehlen des Flask Packages:

```
ImportError: No module named 'flask'
```

Wie oben beschrieben, ist es im Webhosting-Tarif leider nicht möglich fehlende Packages
per `pip` zu installieren. Über einen Umweg (Anlegen eines _virtual environment_ `venv`)
kommen wir aber dennoch zum Ziel und können das fehlende Flask Package installieren.

#### Erzeugen eines Virtual Environment in der lokalen Entwicklungsumgebung

Wir wechseln wieder in das Verzeichnis `flask-app`, das wir oben angelegt haben.

Zum Anlegen eines _Virtual Environment_ mit Python 3 sind folgende Schritte erforderlich:

```sh
$ python3 -m venv venv
```

Der Befehl führt zum Anlegen eines Verzeichnis mit dem Namen `venv`, das folgenden Inhalt
aufweist:

```
bin
include
lib
pyvenv.cfg
```

Um die virtuelle Umgebung zu starten und das Python Package `flask` zu installieren, sind
folgende Schritte erforderlich:

```sh
$ . venv/bin/activate
(venv) $ pip install flask
```

Die Installation von Flask führt zur Installation weiterer Abhängigkeiten:

```
Collecting flask
  Using cached https://files.pythonhosted.org/packages/f2/28/2a03252dfb9ebf377f40fba6a7841b47083260bf8bd8e737b0c6952df83f/Flask-1.1.2-py2.py3-none-any.whl
Collecting click>=5.1 (from flask)
  Using cached https://files.pythonhosted.org/packages/d2/3d/fa76db83bf75c4f8d338c2fd15c8d33fdd7ad23a9b5e57eb6c5de26b430e/click-7.1.2-py2.py3-none-any.whl
Collecting Werkzeug>=0.15 (from flask)
  Using cached https://files.pythonhosted.org/packages/cc/94/5f7079a0e00bd6863ef8f1da638721e9da21e5bacee597595b318f71d62e/Werkzeug-1.0.1-py2.py3-none-any.whl
Collecting Jinja2>=2.10.1 (from flask)
  Using cached https://files.pythonhosted.org/packages/7e/c2/1eece8c95ddbc9b1aeb64f5783a9e07a286de42191b7204d67b7496ddf35/Jinja2-2.11.3-py2.py3-none-any.whl
Collecting itsdangerous>=0.24 (from flask)
  Using cached https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10.1->flask)
  Using cached https://files.pythonhosted.org/packages/45/17/5b6a3a0afa0cb9827781ee43d8842a3540ac9d49855cad936099c7b9416b/MarkupSafe-1.1.1-cp36-cp36m-macosx_10_9_x86_64.whl
Installing collected packages: click, Werkzeug, MarkupSafe, Jinja2, itsdangerous, flask
Successfully installed Jinja2-2.11.3 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 flask-1.1.2 itsdangerous-1.1.0
```

Anschließend befinden sich die gerade installierten Python Packages unter `venv/lib/python3.6/site-packages`.

#### Kopieren des Virtual Environment auf den Webspace

Die virtuelle Umgebung kann nun auf den Webspace kopiert werden, z.B. mittels SCP.

```
$ cd venv/lib/python3.6/site-packages
$ scp -r flask hostingXXX@188.XXX.XXX.XXX:/flask-app
$ scp -r click hostingXXX@188.XXX.XXX.XXX:/flask-app
$ scp -r itsdangerous hostingXXX@188.XXX.XXX.XXX:/flask-app
$ scp -r werkzeug hostingXXX@188.XXX.XXX.XXX:/flask-app
$ scp -r jinja2/ hostingXXX@188.XXX.XXX.XXX:/flask-app
$ scp -r markupsafe/ hostingXXX@188.XXX.XXX.XXX:/flask-app
```

#### Abschluss

Nun kann man erneut die URL https://flask-app.saschaszott.de aufrufen.

Die obige Fehlermeldung sollte durch das Kopieren der fehlenden Python Packages nun
verschwunden sein.

Es sollte im Webbrowser folgende Ausgabe erscheinen:

![Flask Result](/resources/flask-result.png)

#### Offene Punkte

Hier führe ich noch einige Fragen auf, die ich bis dato nicht geklärt habe:

- Kann man das Kopieren der Python Packages von der lokalen Entwicklungsumgebung auf den Webspace
  einfach automatisieren (ohne die Kaskade der SCP-Befehle)?
- Warum führt das Erzeugen der `venv` zu weiteren Paketen im Verzeichnis `venv/lib/python3.6/site-packages`,
  z.B. `setuptools`, `pkg_resources`, `easy_install.py`?
