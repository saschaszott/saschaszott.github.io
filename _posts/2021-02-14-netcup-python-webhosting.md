---
layout: post
title:  "Python 3 Webhosting mit Netcup"
---

# Konfigurationsanleitung für eine einfache Flask-App

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
