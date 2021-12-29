---
layout: post
title:  "Linux Minecraft Server mit Systemd und Backups"
author: "Nick Hildebrandt"
date: 2020-12-13
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
