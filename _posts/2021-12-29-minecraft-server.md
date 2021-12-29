---
layout: post
title:  "Linux Minecraft Server mit Systemd und Backups"
author: "Nick Hildebrandt"
date: 2021-12-29
permalink: /minecraft-server/
---

## Noch ein Minecraft Server Tutorial?

Viele werden sich fragen, weshalb ich ein weiteres Tutorial zum Erstellen eines Minecraftservers auf Linuxbasis mache, wenn es diese auf YouTube doch schon wie sand am mehr gibt. Die Antwort ist relativ einfach, die meisten aller Videos oder Artikel, die ich bisher gesehen habe, waren in ihrer Umsetzung zu Erstellung eines solchen Servers eher mangelhaft da z.b. Zum Starten nur Cron oder eine zwielichtige web GUI genutzt wurde. Dies resultiere dann oft in der Ausführung von Java unter dem Rootbenutzer. Zudem wird das Thema Backup eines Minecraftservers immer sehr stiefmütterlich behandelt weshalb ich auch darauf einen genaueren Blick werfen werde.

## Hardware und Basis

Natürlich muss zunächst die Basis für einen Sollchenserver hergestellt werden. Diese sollte sich grundsätzlich für den 24/7 Betrieb eigen und mindestens 2-4 GB Arbeitsspeicher zur Verfügung stellen. So empfiehlt es sich auf eBay und Co. nach gebrauchten Pcs z.b. Thinkcenter oder Optiplex aus Büroumgebungen Ausschau zu halten. Hiebei ist jedoch zu beachten dass bei solch alten und meist auch nicht sehr leisen Systemen der Stromverbrauch gerade im Dauerbetrieb, unter Beachtung der deutschen Strompreise, eine nicht unerhebliche Rolle speilt. So könnte man meinen, dass ein Raspberry Pi hiefür doch perfekt geeignet sei. In meinen Tests musste ich jedoch Festellen dass alles unter einem Raspberry Pi 4 automatisch wegfällt, da ab Minecraft 1.16 nicht genügend Arbeitsspeicher zur Verfügung steht. So lässt sich nur der Raspberry Pi 4 Mit mindestens 4 oder gleich 8 GB Arbeitsspeicher empfehlen.
Solltet ihr wie ich bereits einen Proxmox, Kubernetes oder Vmware Server haben könnt ihr selbstverständlich auch diesen verwenden

## Das Betriebssystem

Oft wird empfohlen, einfach Windows oder irgendein überladendes Linuxderivat mit voller Desktopumgebung zu verwenden. Doch dies ist gerade auf älterer oder begrenzter Hardware wie dem Raspberry Pi hinderlich. So empfehle ich Debian oder Ubuntu in ihrer Server Version ohne grafische Oberfläche nur mit den Standartsystemwerkzeugen zu verwenden.

## Mein Setup

- Proxmox 6 LXC (unprivilegiert, nesting)
- Debian 11
- 4GB Ram
- 2CPU Cores
- Java 17
- Minecraft 1.18

## Erforderliche Pakete installieren

Minecraft 1.18 benötigt mindestens Java 1.14 weshalb wird, da es in den offiziellen Paketquellen von Debian 11 ist Java 17 installieren. Genauer verwenden wir hier der headless Version, da wir die grafischen Komponenten von Java nicht benötigen und sie Installation so minimal wie möglichen seine soll. Zudem Benötigen wir das GNU Tool Screen um die die Minecraft Server Konsole zugreifen zu können.

```shell
sudo apt install openjdk-17-jre-headless screen -y
```

## Minecraft Benutzer und Gruppe anlegen

Damit der Server mit den Niedrigst möglichen Berechtigungen ausgeführt wird, erstellen wir einen Benutzer Minecraft, welcher kein Passwort hat und zur ebenfalls erstellten Gruppe Minecraft gehört.

```shell
sudo groupadd minecraft
```

```shell
sudo useradd -m -d /var/minecraft -g minecraft minecraft
```

## Minecraft Ordner erstellen und Dateien Bereitstellen

Das Heimatverzeichnis des gerade erstellen Benutzers ist "var/minecraft" dieses werden wir zunächst erstellen und anschließend alle notwendigen Dateien in diesem Verzeichnis lagern. Schlussendlich stellen wir dann noch sicher, dass der Benutzer Minecraft der Eigentümer von diesem Verzeichnis ist und lesen sowie schreiben kann.

```shell
sudo mkdir /var/minecraft
```
### Minecraft Server Version

- [Vanilla](https://www.minecraft.net/de-de/download/server)
- [Paper - Empfohlen für Plugins](https://papermc.io/)

```shell
sudo wget DOWNLOD-LINK
```

```shell
sudo nano /var/minecraft/eula.txt
```

```shell
# eula.txt
eula=true
```

```shell
sudo chown -R minecraft:minecraft /var/minecraft
```
