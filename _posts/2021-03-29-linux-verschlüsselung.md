---
layout: post
title:  "Linux - Ordner Verschlüsseln"
author: "Nick Hildebrandt"
date: 2021-03-29
permalink: /linux-verschlüsselung/
---

## Was wir Vorhaben

Ich möchte euch in diesem Artikel zeigen, wie Ihr mittels GPG ganze order oder einzelne Dateien verschlüsseln und signieren könnte, 
sodass sie z.B beim Transfer von unbefugten Einblicken geschützt sind.

## Was ist GPG

GPG ist ein Verschlüsselungssystem nach dem Public Key Verfahren. Es ist leider nicht zum S/MIME Verfahren kompatibel.
Im Vergleich zu S/MIME gibt es auch keinerlei Zertifikatsstruktur und somit fehlt das "automatische" Vertrauenskonzept.
Das Vertrauenskonzept "web of trust", welches hinter GPG steht, ist leider nicht trivial und wird erst später erklärt.
Der Einsatz von GPG ist gerade in der Open Source Welt sehr verbreitet.
GPG wird hier zur Signatur des Quellcodes oder zur Authentifizierung von Paketquellen gegenüber dem Linux Betriebssystems genutzt.
Man kann sogar beliebige Dateien mit GPG verschlüsseln. Unter Linux ist das GPG Paket bereits integriert und kann auf Kommandozeilenebene direkt genutzt werden.

## Erforderliche Pakete installieren

GnuPG sollte eigentlich schon auf jedem Linux-Rechner vorhanden sein. Im Zweifelsfall ist unten der entsprechende Befehl aufgeführt.
Zudem installieren wird noch Tar, um Archive zu erstellen, welche dann verschlüsselt werden.

```shell
sudo sudo apt install gnupg2 tar -y
```
## Order bzw. Datien archivieren

GnuPG kann nur einzelne Dateien Verschlüssen, damit wir aber mehrere Dateien oder ganze Ordner verwenden können,
ist die Erstellung eines Archivs essenziell. In diesem Beispiel wir Tar verwendet,
Ihr, seid aber bei der Auswahl frei und könnt alle möglichen Archive und sogar Kompressionen verwenden.

```shell
tar cfvz archiv.tar.gz inhalt1 inhalt2
```

## Mit GPG verschlüsseln

Mit dem unten aufgeführten Befehl wird dann das Archiv verschlüsselt. Dies kann je nach Rechnerleistung und vor allem Dateigröße Zeit in Anspruch nehmen.

```shell
gpg -c dateiname
```

## Entschlüsseln und Archiv entpacken

Nun möchte man vielleicht auch, dass Verschlüsselte wieder entschlüsseln. Das geht an jedem Rechner, der ebenfalls GnuPG installiert hat.
Zu beachten ist nur, dass die verschlüsselte Datei sowie der Schlüssel gebraucht wird,
um wie unten beschrieben wieder das erstellt Archiv zu erhalten und dieses letztendlich entpacken zu können.

```shell
gpg -d dateiname.gpg > archiv.tar.gz
tar xfvz archiv.tar.gz
```
