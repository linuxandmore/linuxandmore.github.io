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
sudo useradd -s /bin/bash -m -d /var/minecraft -g minecraft minecraft
```

## Minecraft Ordner erstellen und Dateien Bereitstellen

Das Heimatverzeichnis des gerade erstellen Benutzers ist "var/minecraft" diesem werden wir alle notwendigen Dateien und Konfigurationen lagen.

### Minecraft Server Version

- [Vanilla](https://www.minecraft.net/de-de/download/server)
- [Paper - Empfohlen für Plugins](https://papermc.io/)
- [Forge - Empfohlen für Mods](https://files.minecraftforge.net/net/minecraftforge/forge/)

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

### Ein Server Icon Verwenden

Möchte man das in der Auswahlliste der Server im Spiel ein Logo erschein, so erstellt man in "/var/minecraft" ein 64x64 pixel großen PNG bild mit dem Namen "server-icon.png". Daraufhin wird diesen beim Stat des Minecraftservers automatisch geladen.

### Minecraft Benutzer Lese und Schreibrechte geben

Dieser bereits oben erwähnte Schritt ist nach jeder Datei Erstellung bzw. Änderung im Ordner "/var/minecraft" notwendig.

```shell
sudo chown -R minecraft:minecraft /var/minecraft
```

## Systemd Service erstellen

Damit unser Minecraftserver automatisch gestartet und im Falle eines Absturzes neugestaltet wird, verwenden wir den System- und Sitzungs-Manager Systemd welcher bei fast allen modernen Linuxdistributionen als Standard Init-System verwendet wird.

```shell
sudo nano /etc/systemd/system/minecraft.service
```

```shell
[Unit]
Description=Minecraft Server

Wants=network.target
After=network.target

[Service]
User=minecraft
Group=minecraft
Nice=5
SuccessExitStatus=0 1

ProtectHome=true
ProtectSystem=full
PrivateDevices=true
NoNewPrivileges=true
PrivateTmp=true
InaccessibleDirectories=/root /sys /srv -/opt /media -/lost+found
ReadWriteDirectories=/var/minecraft
WorkingDirectory=/var/minecraft
ExecStart=/usr/bin/screen -DmS mc /usr/bin/java -Xms512M -Xmx1024M -jar server.jar nogui
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "say Minecraftserver geht in 10 Sekunden offline"\015'
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "save-all"\015'
ExecStop=/bin/sleep 10
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "stop"\015'
[Install]
WantedBy=multi-user.target
```

### Was man anpassen sollte

Der Inhalt der Servicedatei kann in den meisten Fällen so bleiben, wie er ist interessant sind lediglich folgende Punkte:

- "-Xms512M" (Min. Arbeitsspeicher)
- "-Xmx1024M" (Max. Arbeitsspeicher)
- "server.jar" (Name der heruntergeladenen Serverdatei)
- "say Minecraftserver geht in 10 Sekunden offline" (Ausgabe im Chat wenn der Server offline geht)
- "/bin/sleep 10" Wartezeit bis der Server offline geht

## Dienst Aktivieren und Server Starten

Nun muss der Dienst noch aktiviert werden, damit er automatisch mit dem System startet. Zusätzlich werden wir den Dienst einmal manuell starten sowie dessen Status ausgeben, um die Funktionalität des Servers sicherzustellen.


```shell
sudo systemctl enable minecraft.service
```

```shell
sudo systemctl start minecraft.service
```

```shell
sudo systemctl status minecraft.service
```

### Server Admin (OP) erstellen

Oft möchte man im Minecraft spiel selbst die Möglichkeit haben, Befehle über die Textzeile auszuführen. Dafür muss der jeweilige spiele die sogenannten "OP"Rechte besitzen. Diese können ihm nach der Anmeldung unter dem Minecraftbenutzer über die Screenkommandozeile zugewiesen werden.

```shell
sudo su minecraft
```

```shell
screen -p 0 -S mc -X eval 'stuff "op SPIELERNAME"\015'
```

## Die Minecraft "server.properties"

Die Minecraft "server.properties" Stellen die Zentrale Konfigurationsdatei des Minecraftservers dar. Sie werden automatisch beim ersten Start des Minecraftservers erstellt und können anschließen bearbeitet werden. Nach jeder Änderung muss der Minecraftserver mit dem unten aufgeführten Befehl neugestarten werden damit die Änderungen in kraft treten.

```shell
sudo nano /var/minecraft/server.properties
```
- [Bedeutung der einzelnen Optionen](https://minecraft.fandom.com/wiki/Server.properties)

```shell
sudo systemctl restart minecraft.service
```

## Die Firewall konfiguriren

Bei manchen Linuxdistributionen kommt als Firewall UFW bzw. Firewalld vorinstalliert mit. In diesem Fall muss der Minecraft Port 25565 (TCP & UDP) zwingend freigegeben werden. Für alle andere Distribution wie z. B. auch Debian wo standardmäßig keine Firewall installiert ist, stellt sich dieser Schritt zunächst als optional dar. Ich empfehle aber aus Sicherheitsgründen gerade auf V-Servern das Tool UFW zu verwenden.

```shell
sudo apt install ufw -y
```

```shell
sudo ufw allow 25565/tcp
sudo ufw allow 25565/udp
```

```shell
sudo ufw enable
```

> **Achtung:** sollte ihr Via SSH auf den Server zugreifen, so müsst ihr auch SSH freigeben.

```shell
sudo ufw allow ssh
```
