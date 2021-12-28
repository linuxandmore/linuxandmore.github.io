---
layout: post
title:  "PXE Multioboot Server"
author: "Nick Hildebrandt"
date: 2020-12-13
permalink: /pxe-server/
---

## Was ist PXE?

PXE ist ein Verfahren von Intel, um Computer einen netzwerkbasierten Startvorgang zu ermöglichen.
Damit ist gemeint, dass ein Computer nicht das Betriebssystem von der Festplatte lädt, sondern aus dem Netzwerk.
PXE fasst verschiedene Verfahren zusammen, um einen Computer übers Netzwerk zu Booten. Früher brauchte man noch
für jede Netzwerkkarte einen eigenen Treiber, damit das gelang. PXE bringt die notwendigen Funktionen mit. 
Es ist eine Erweiterung für eine normierte Schnittstelle für Netzwerkzugriffe. PXE ist im BIOS der Netzwerkkarte 
integriert. So ist das Booten übers Netzwerk bei allen Netzwerkkarten identisch.

## Was ist TFTP?

Das Trivial File Transfer Protocol, kurz TFTP, ist ein sehr einfaches Client-Server-Protokoll, 
das den Transfer von Dateien in Computernetzwerken regelt. Eine erste Spezifikation wurde im Juni 1981 
in RFC 783 veröffentlicht. Der aktuelle Standard ist im 1992 erschienenen RFC 1350 nachzulesen.
Das TFTP-Protokoll setzt standardmäßig auf dem ebenfalls minimalistischen Transportprotokoll
UDP (User Datagram Protocol) auf, das eine Datenübertragung ohne feste Verbindung zwischen den
Kommunikationspartnern vorsieht. Die Implementierung von TFTP auf Basis anderer Protokolle ist grundsätzlich
aber ebenfalls möglich.

## Benötigte Pakete installieren

Für einen PXE Server benötigt man einen Netboot fähigen Bootloader und den Dateidienst TFTP. In diesem Fall werden wir Syslinux als Bootloader und das offizielle TFTP Paket aus dem Debian Repository. Um später dann noch Boot Images zu Entpacken Benötigen wir unzip.

```shell
sudo apt install syslinux-common syslinux-efi tftp-hpa unzip -y
```

## PXE und Syslinux Dateien

### Erstellen des TFTP-Root

Der TFTP-Root ist ein Ordner auf dem Dateisystem der frei über TFTP verfügbar ist und von dem alle benötigten Dateien für das Booten geladen werden.
In diesem Beispiel wird er auf dem Wurzelverzeichnis erstellt, um Berechtigungs- und Zugarmproblemen vorzubeugen.

```shell
sudo mkdir /tftpboot
```

### BIOS (Legacy) oder UEFI?

Es gibt zwei Möglichkeiten für das BIOS deines Rechners einem Bootmanager (Syslinux) zu laden, BIOS (Legacy) und das neuere UEFI. Grundsätzlich empfiehlt sich in der heutigen Zeit immer UEFI, jedoch denke ich, dass aufgrund von Kompatibilität und Portabilität BIOS (Legacy) für einen PXE Server die bessere Wahl ist (Nicht zuletzt auch aufgrund meiner eigenen Erfahrung).

**Was ist BIOS (Legacy) Boot?**

Beim "Basic Input/Output System" (BIOS) handelt es sich um die Firmware eines PCs. Sie bildet die zentrale
moderner Nachfolger: Mit seiner grafischen Benutzeroberfläche lässt sich UEFI-BIOS deutlich komfortabler
bedienen als das BIOS und bietet auch deutlich mehr Funktionen. Das BIOS sorgt dafür, dass nach dem Einschalten
des PCs seine Komponenten wie beispielsweise die Grafikkarte, die Festplatte und die Netzwerkschnittstelle korrekt
erkannt und initialisiert werden, bevor Windows startet - das OS wäre ansonsten nicht lauffähig.

**Was ist UEFI Boot?**

UEFI​, das "Unified Extensible Firmware Interface", macht grundsätzlich genau das Gleiche wie das BIOS, 
bringt aber deutlich mehr Funktionen mit, unterstützt die neueste Hardware und sieht obendrein viel moderner aus.
Ob Sie es bei Ihrem Rechner mit einem BIOS oder einem UEFI-BIOS zu tun haben, können Sie deshalb leicht erkennen.
Ein UEFI-BIOS ist normalerweise grafisch aufwendig gestaltet, lässt sich per Maus steuern und ähnelt hinsichtlich
seines Benutzeroberflächen-Designs einer Windows-Anwendung.

### Initialisierung des Bootloaders

**Initialisierung mit UEFI**

```shell
sudo cd /tftpboot
wget https://github.com/MacAndMoreYT/PXE/raw/master/UEFI/UEFI.zip
unzip *.zip
rm -r UEFI.zip
```

**Initialisierung BIOS (Legacy)**

```shell
sudo cd /tftpboot
wget https://github.com/MacAndMoreYT/PXE/raw/master/BIOS/BIOS.zip
unzip *.zip
rm -r BIOS.zip
```

##  Konfiguration des TFTP Servers

```shell
sudo nano /etc/default/tftpd-hpa
```
```shell
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure"
```
```shell
sudo systemctl restart tftpd-hpa.service
```
## Eintrag im Router (DHCP Server) Vornehmen

Damit die Clients via DHCP den PXE Server "Sehen" können, müssen wir am Router via TFTP  Auf die Bootdateien und Syslinux verweisen!
Weitere Informationen findest du im Video.

```shell
UEFI = PXE via TFTP = syslinux.efi
BIOS = PXE via TFTP = pxelinux.0
```

## Betriebssystem Images Downloaden

Nun musst du nur noch für den Bootloader die entsprechenden Betriebssystem-Images bereitstellen.
Da dies bei jedem Betriebssystem anders ist, solltest du individuell nach Dokumentationen suchen. In jenem Fall findest du unten eine Liste kompatibler Betriebssysteme. Ferner ist es wichtig, zu jedem hinzugefügten Element den entsprechenden Eintrag in der "pxelinux.cfg/default" vorzunehmen.

### Beispiel Debian

[Debian NetBoot Image Download](https://www.debian.org/distrib/netinst#netboot)

**Eintrag in der "pxelinux.cfg/default"**

```shell
#pxelinux.cfg/default

LABEL Debian 10 x64
MENU LABEL Debian 10 Installer (Automatic)
KERNEL debian-10/linux
append initrd=debian-10/initrd.gz 
```

### Weitere Images

- [Debian NetBoot Image Download](https://www.debian.org/distrib/netinst#netboot)
- [Ubuntu NetBoot Image Download](https://ubuntu.com/download/alternative-downloads)
- [Freedos, PartedMagic (GitHub)](https://github.com/MacAndMoreYT/PXE)
