---
layout: post
title:  "Pritunl - VPN im LXC Container"
author: "Nick Hildebrandt"
date: 2021-01-17
permalink: /vpn-server/
---

## Was ist eine VPN?

Eine der besten Möglichkeiten zum Sichern von Daten in und aus Ihrem Netzwerk könnte die Verwendung eines VPN sein.
Es verschlüsselt den gesamten Online-Verkehr zwischen einem VPN-Server und einem Smartphone oder Laptop, um diese zu
sichern. Dazu wird die Geräteidentität maskiert und eine sichere Verbindung hergestellt.
Daher wird es für Hacker schwierig, auf vertraulichen Daten zuzugreifen

## Was ist Pritunl?

Pritunl bietet ein effizientes VPN mit komplexen Gateway-Links und Site-to-Site-Links und ermöglicht 
Remotebenutzern den Zugriff auf lokale Netzwerke über OpenVPN und Wireguard. Die Konfiguration und 
Überwachung geschieht hierbei über eine leicht zu bedienende Weboberfläche.

## Was ist OpenVPN?

OpenVPN ist eine freie Software zum Aufbau eines Virtuellen Privaten Netzwerkes über eine verschlüsselte
TLS-Verbindung. Zur Verschlüsselung kann OpenSSL oder mbed TLS benutzt werden. OpenVPN verwendet wahlweise
UDP oder TCP zum Transport

## Vorbereitungen bei Proxmox LXC Containern

Da OpenVPN zum Erstellen von Subnetzen auf die Linux Kernel Module TUN/TAP Zugreifen muss und diese bei LXC Container nicht freigegeben sind, müssen sie als Geräte simuliert werden. Dies bringt keine Nachteile mit sich, da nur zwei Ordner in "/dev" beim Systemstart erstellt werden müssen. In diesem Beispiel wird dafür Cron und ein kleines Shell Skript genutzt.

**Cron Eintrag**

```shell
sudo crontab -e
```
```shell
@reboot sh /root/tun-tap.sh
```

**Das Skript**

```shell
sudo nano  /root/tun-tap.sh
```
```shell
mkdir -p /dev/net
mknod -m 666 /dev/net/tun c 10 200
```

**Neustarten um die Änderungen Anzuwenden**

```shell
sudo reboot
```

## Installation von Pritunl (Skript)

Zur Vereinfachung der Installation von Pritunl habe ich ein simples Shell Skript geschrieben welches dies vereinfacht. Einfach herunterladen und Ausführen!

```shell
sudo wget https://macandmore.ts13.de/download/vpn-server/install.sh && bash install.sh
```

## Erhalten der Zugangsdaten

Zum Einrichten der Pritunl Instanz wird neben dem Passwort zur Anmeldung ein Setup-Key benötigt. Dieser lässt sich genauso wie das Passwort über folgende Befehle auf der Kommandozeile ausgeben.

```shell
sudo pritunl setup-key
pritunl default-password
```

## Configuration des Server

Nun müssen die unten aufgeführten Punkte in der web Oberfläche abgeändert werden. Da dies bei jeder Installation abweichen ist, verweise ich hier auf mein Video.

- Nutzname und Passwort ändern
- Public Address auf DDNS / Domain setzten
- Organisation erstellen
- Nutzer erstellen
- Server erstellen und der Organisation zuweisen

## Aufbauen einer VPN-Verbindung

Nachdem der erstellte Server gestartet wurde, muss man zunächst die OpenVPN Datei des erstellten Benutzers herunterladen. Dies geschieht im Falle von Pritunl ebenfalls über die Weboberfläche. Als Download bekommt man nun eine ZIP-Datei ZIP, welche entpackt eine .ovpn und .crt Datei enthält. Die .ovpn Datei fungiert hierbei als Konfigurationsdatei welche nun beispielsweise in die OpenVPN Connect App am Smartphone importiert werden kann. Für den PC gibt es für Windows und Mac die Offizielle Connect App und für Linux entsprechende Module für den Netzwerkmanager.
