# LB2 - Datenbank-Server mit einer Firewall

## Inhaltsverzeichnis
* 1 [Einleitung](#einleitung) 
* 2 [Netzwerkplan](#netzwerkplan)
* 3 [Konfiguration](#konfiguration)
* 4 [Testen](#testen)
* 5 [Sicherheit](#sicherheit)
* 6 [Quellenverzeichnis](#quellenverzeichnis)

## Einleitung
In diesem Dokument sind alle wichtigen technischen Informationen zu meiner Umgebung dokumentiert. Die Umgebung besitzt einen Datenbank-Server und eine Firewall. Mit dem Notebook soll es möglich sein über das Tool [Adminer](https://www.adminer.org/) auf die MySQL-Datenbank zuzugreifen. Der Zugriff findet mit einem Benutzer und Passwort statt. Die Firewall in der Umgebung sollte für die Sicherheit sorgen. Dabei sollen nur die beiden Ports 22 und 80 zugelassen werden, da nur diese für den Zugriff benötigt werden.

## Netzwerkplan
Der Datenbank-Server besitzt die IP-Adresse "192.168.10.200". Er stellt die Services HTTP, SSH, Adminer und MySQL zur Verfügung. Zudem hat die Datenbank VM zwei Netzwerkadapter (Host-Only und NAT). Diese sind mit der Firewall verbunden, welche den Port 22 und 80 erlaubt. Somit ist die Datenbank über SSH und HTTP erreichbar.

![Netzwerkplan](./bilder/netzwerkplan.png)    

## Konfiguration
Eine benannte virtuelle Umgebung definieren.

`config.vm.define "db-server" do |db|`

Den Typ des Betriebssystems angeben, welches in Virtualbox verwendet wird.

`config.vm.box = "ubuntu/bionic64"`

Virtualisierungsplattform definieren.

`db.vm.provider "virtualbox" do |vb|`

Anzahl RAM für die VM.

`vb.memory = "1024"`

Hostname des Datenbank-Servers.

`db.vm.hostname = "db01"`

Die Netzwerkeinstellungen mit der IP-Adresse und dem Port konfigurieren. Der Port 8080 wird verwendet, um auf das User Interface via Webbrowser zu gelangen.

```
db.vm.network "private_network", ip: "192.168.10.200"`
db.vm.network "forwarded_port", guest:80, host:8080, auto_correct: false
```
Shell-Skript innerhalb des Gastbetriebssystems direkt nach dem Hochfahren ausführen. Das Skript wird im File "db.sh" gespeichert.

`db.vm.provision "shell", path: "db.sh"`

Die Shell starten, um die nötigen Befehle auszuführen.

```
config.vm.provision "shell", inline: <<-SHELL 
set -o xtrace
```

Alle Paketlisten neu einlesen und aktualiseren. Zudem ein zentrales System zur Verwaltung von Debian Paketen (debconf-utils) und einen Webserver (apache2) installieren.

```
sudo apt-get update
sudo apt-get -y install debconf-utils 
sudo apt-get -y install apache2 
```

Benutzer und Passwort für den Datenbank-Server konfigurieren.<br>
**Benutzer:** root<br>
**Passwort:** Hallo123

```
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password Hallo123'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password Hallo123'
```

PHP und seine Apachemodule, sowie MySQL installieren. Mit PHP wird die Datenbank auf dem Interface angezeigt

`sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server` 

Als nächstes installiert man Adminer. Adminer ist ein Tool zum Verwalten von Inhalten in MySQL-Datenbanken. Es ist eine Alternative zu phpMyAdmin. Dabei wird ein Verzeichnis erstellt und alle nötigen Dateien installiert.

```	
sudo mkdir /usr/share/adminer
sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
sudo a2enconf adminer.conf  
```

Webserver neustarten.

`sudo service apache2 restart`

Firewall Regeln konfigurieren. Die Firewall lässt den Port 22 für SSH und Port 80 für den Adminer frei.

```  
sudo ufw allow 80/tcp 
sudo ufw allow 22/tcp 
sudo ufw -f enable 
```

Nachdem man "vagrant up" ausgeführt hat, ist der Service über http://localhost:8080/adminer.php im Webbrowser erreichbar. Dabei muss man sich mit dem definierten Benutzernamen und Passwort anmelden.

![UserInterface](./bilder/user_interface.png)

## Testen
| Nr.            | Erwartung           | Ergebnis         | OK? |
|:---------------|:--------------------|:---------------- |:----|
| 1 | Die Datenbank ist via http://localhost:8080/adminer.php erreichbar. | Beim Aufrufen des Links erscheint das Anmeldefenster von Adminer. | OK |
| 2 | Die Anmeldung ist mit dem angegeben Benutzer und Passwort erfolgreich. | Mit dem angegeben Benutzer und Passwort kann man sich anmelden. | OK |
| 3 | Mit SSH kann auf die VM zugegriffen werden. | In Git Bash "vagrant ssh" eingegeben und die Verbindung zur VM wurde hergestellt. | OK |

## Sicherheit
* Datenbank Server bzw. MySQL ist mit einem Passwort geschützt.
* Firewall eingeschaltet. Port 22 für SSH und Port 80 für den Adminer freigeschaltet.

## Quellenverzeichnis
* Datenbank Konfigurationen: https://github.com/mc-b/M300/tree/master/vagrant/db
* Firewall Konfigurationen: https://github.com/mc-b/M300/tree/master/vagrant/fwrp