---
layout: post
title:  "SSH Server Absichern und Keys richtig Benutzen"
author: "Nick Hildebrandt"
date: 2021-03-29
permalink: /ssh-server-keys/
---

* TOC
{:toc}

## Was ist SSH?

SSH, auch bekannt als Secure Shell oder Secure Socket Shell, ist ein Netzwerkprotokoll, das Benutzern, insbesondere Systemadministratoren, eine sichere Möglichkeit bietet, über ein ungesichertes Netzwerk auf einen Computer zuzugreifen. Neben der Bereitstellung von sicheren Netzwerkdiensten bezieht sich SSH auf die Suite von Dienstprogrammen, die das SSH-Protokoll implementieren. 

## Unser Ziel

Unser Ziel wird es sein den viel benutzen SSH Dienst so sicher wie möglich zu konfigurieren und automatische sperren sowie DDOS bzw. Brutforce Schutzkonzepte zu implementieren. Natürlich ist der hier gezeigte weg nur ein Part der sicheren Konfiguration eines Linux Servers und sollte von euch entsprechend den Umständen und Anforderungen angepasst werden. Erweitert mit anderen Schutzkonzepten ist dies aber ein guter Ausgangspunkt zum sicheren Server.

## Benötigte Pakete Installieren

Um einen SSH Server sicher zu betreiben, benötigen wir neben dem Server selbst noch Fail2Ban auf welches ich später noch genauer eingehen werde sowie sudo zu Kontrolle von Berechtigungen. Alle Pakete befinden sich in den offiziellen Repositorys der meisten Distributionen und können wie unten beschrieben installiert werden.

```shell
sudo apt openssh-server fail2ban sudo
```

## Neuen Benutzer mit sudo Anlegen

Wir werden nun einen neuen Benutzer anlegen und diese n mit entsprechenden sudo rechten ausstatten so, dass der Root Nutzer nicht zum Login verwenden werden muss.

> **Achtung:** Das Erstellen eines neuen Benutzers mit sudo rechten verschafft keinen sicherheitsrelevanten Vorteil. Es dient lediglich zur besseren Verwaltung von SSH Keys sowie der Berechtigung von Diensten. 

```shell
sudo adduser benutzername
usermod -aG sudo benutzername
su benutzername
```

## SSH Key Generieren und auf Server Kopieren

In diesem Schritt generieren wir auf unserem Rechner einen Schlüssel, mit dem der Log-in über SSH authentifiziert wird. So wird man sich nur mit diesem Schlüssel und ohne die Gefahr eines unsicheren Passworts Anmelden können. Danach wird der erstellte Schlüssel mit dem in SSH integrierten Befehl auf den Server kopiert und automatisch hinzugefügt.
Solltet Ihr Windows benutzten, müsstet Ihr auf eurem Computer den SSH Agenten installieren, um schlüssel erstellen und kopieren zu können.

```shell
ssh-keygen -t rsa -b 4096
```
```shell
ssh-copy-id -i ~/.ssh/id_rsa.pup user@host
```

**Wichtig zu Beachten:**

- Jeder Client benötigt einen eigenen Key!
- Gebe niemals die id_rsa weiter!
- Generiere Keys in sicherer umgebebung und erneuere sie regelmäßig!

## SSH Conf Anpassen

Nun passen wir die Konfigurationsdatei des SSH Servers an. Sie wird hinsichtlich der unten aufgeführten punkte abgeändert. Das vorgebende Beispiel kann dabei eins zu ein kopiert bzw. angepasst werden.
Danach wird der Dienst neu gestartet, um die Änderungen zu übernehmen.

- Standard Port 22 behalten (nur im Zweifel ändern)
- Root Login deaktivieren
- Anzahlen der falschen versuche auf 3
- Anzahl der gleichzeitigen Verbindungen auf 2
- Zugriff auf den erstellten Benutzer beschränken
- System Authentifizierung (PAM) Deaktivieren
- X11 Weiterleitung deaktivieren

```shell
sudo nano /etc/ssh/sshd_config
```
```shell
# $OpenBSD: sshd_config,v 1.89 2013/02/06 00:20:42 dtucker Exp $
# This is the sshd server system-wide configuration file. See
# sshd_config(5) for more information.
# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin
# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented. Uncommented options override the
# default value.
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
# The default requires explicit activation of protocol 1
#Protocol 2
# HostKey for protocol version 1
#HostKey /etc/ssh/ssh_host_key
# HostKeys for protocol version 2
#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_dsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
# Lifetime and size of ephemeral version 1 server key
#KeyRegenerationInterval 1h
#ServerKeyBits 1024
# Logging
# obsoletes QuietMode and FascistLogging
#SyslogFacility AUTH
#LogLevel INFO
# Authentication:
#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
MaxAuthTries 6
MaxSessions 2
AllowUsers macandmore
RSAAuthentication yes
PubkeyAuthentication yes
# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile %h/.ssh/authorized_keys
#AuthorizedPrincipalsFile none
#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody
# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#RhostsRSAAuthentication no
# similar for protocol version 2
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# RhostsRSAAuthentication and HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no
# Change to no to disable s/key passwords
ChallengeResponseAuthentication no
# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no
# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication. Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM no
#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
PrintMotd no # pam does that
#PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
UsePrivilegeSeparation sandbox # Default for new installations.
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS yes
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none
# no default banner path
#Banner none
# override default of no subsystems
Subsystem sftp /usr/lib/ssh/sftp-server
# Example of overriding settings on a per-user basis
#Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
```
```shell
sudo systemctrl restart sshd.service
```

## Fail2Ban für mehr Sicherheit

### Was ist Fail2Ban

Fail2Ban ist eine verbreitete Linux-Sicherheitssoftware, die durch die Überwachung und Analyse von Log-Dateien des Betriebssystems und von Anwendungen unerlaubte Zugriffe verhindert. Mit Fail2Ban können Angriffe von Hackern und insbesondere Brute-Force-Angriffe mit wenig Aufwand und effektiv unterbunden werden.

### Anpassen der Richtlinien

Standardmäßig sind in alle wichtigen regeln (Jails) in Fail2Ban schon aktiviert, darunter auch SSH. Um aber nun ein Höchstmaß an Sicherheit zu erreichen, sollte man die Einstellungen zu den Bann Aktionen anpassen, sodass sie früher auslösen. Danach wird der Dienst neu gestartet, um die Änderungen zu übernehmen.

```shell
sudo nano /etc/fail2ban/jail.conf
```
```shell
bantime = 3600m
findtime = 360m
maxretry = 3
```
```shell
sudo systemctl restart fail2ban.service
```

### IP Unbannen

Falls man versehentlich sich selbst aussperrt, ist es sicherlich gut zu wissen das Fail2Ban die Möglichkeit bietet gesperrte IP-Adressen zu entbannen. Dies wird mit dem unten aufgeführten Befehl gemacht und setzt alle mit dieser IP bezogenen Sperrungen zurück.

```shell
sudo fail2ban-client set jailname unbanip ip
```

## Root Deaktivieren

Um nun noch nutzen aus den erstellten User mit sudo rechten zu ziehen, können wir Root gänzlich deaktivieren. Dies führt dazu, dass sich Root nicht anmelden kann da das Password deaktiviert wurde.

```shell
sudo passwd -d root
```

## Nützliche SSH Befehle

| Befehl                     | Funktion                        |
|:--------------------------:|:-------------------------------:|
| Installierte Keys anzeigen | ssh-add -l                      |
| Key Löschen                | ssh-add -d id_rsa               |
| Key hinzufügen             | chmod 400 id_rsa ssh-add id_rsa |
