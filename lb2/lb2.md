<h1>LB2 - Datenbank-Server mit einer Firewall</h1>
<h2>Inhaltsverzeichnis</h2>
<p>Einleitung<br>Grafische Übersicht<br>Konfiguration<br>Testen<br>Sicherheit<br>Quellenverzeichnis</p>

<h2>Einleitung</h2>
<p>In meiner LB2 werde ich einen Datenbank-Server mit MySQL erstellen. Zudem werde ich eine Firewall einsetzen </p>
<h2>Grafische Übersicht</h2>
<p>+--------------------+<br>        
! Datenbank Server   !<br>          
! Host: db01         !<br>         
! IP: 192.168.1.100  !<br>
! Port: 3306         !<br>          
! Nat: -             !<br>          
+--------------------+</p>         

<h2>Konfiguration</h2>
<p>Eine benannte virtuelle Umgebung definieren.</p>

`config.vm.define "db-server" do |db|`

<p>Den Typ des Betriebssystems angeben, welches in Virtualbox verwendet wird.</p>

`config.vm.box = "ubuntu/bionic64"`

<p>Virtualisierungsplattform definieren.</p>

`db.vm.provider "virtualbox" do |vb|`

<p>Anzahl RAM für die VM.</p>

`vb.memory = "1024"`

<p>Hostname des Datenbank-Servers.</p>

`db.vm.hostname = "db01"`

<p>Die Netzwerkeinstellungen mit der IP-Adresse und dem Port konfigurieren. Der Port 8080 wird verwendet, um auf das User Interface via Webbrowser zu gelangen.</p>

```
db.vm.network "private_network", ip: "192.168.10.200"`
db.vm.network "forwarded_port", guest:80, host:8080, auto_correct: false
```
<p>Shell-Skript innerhalb des Gastbetriebssystems direkt nach dem Hochfahren ausführen. Das Skript wird im File "db.sh" gespeichert.</p>

`db.vm.provision "shell", path: "db.sh"`

<p>Die Shell starten, um die nötigen Befehle auszuführen.</p>

```
config.vm.provision "shell", inline: <<-SHELL 
set -o xtrace
```

<p>Alle Paketlisten neu einlesen und aktualiseren. Zudem ein zentrales System zur Verwaltung von Debian Paketen (debconf-utils) und einen Webserver (apache2) installieren.</p>

```
sudo apt-get update
sudo apt-get -y install debconf-utils 
sudo apt-get -y install apache2 
```

<p>User und Passwort für den Datenbank-Server konfigurieren.<br>
Benutzer: root<br>
Passwort: Hallo123</p>

```
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password Hallo123'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password Hallo123'
```

<p>PHP und seine Apachemodule, sowie MySQL installieren. Mit PHP wird die Datenbank auf dem Interface angezeigt</p>

`sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server` 

<p>Als nächstes installiert man Adminer. Adminer ist ein Tool zum Verwalten von Inhalten in MySQL-Datenbanken. Es ist eine Alternative zu phpMyAdmin. Dabei wird ein Verzeichnis erstellt und alle nötigen Dateien installiert.</p>

```	
sudo mkdir /usr/share/adminer
sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
sudo a2enconf adminer.conf  
```

<p>Webserver neustarten.

`sudo service apache2 restart`

<p>Firewall Regeln.</p> 

```  
sudo ufw allow 80/tcp 
sudo ufw allow 22/tcp 
sudo ufw -f enable 
```
<h2>Testen</h2>

<h2>Sicherheit</h2>
<p>- Datenbank Server bzw. MySQL ist mit einem Passwort geschützt<br>
- Nur die Ports 80 und 22 werden von den Firewall zugelassen</p>

<h2>Quellenverzeichnis</h2>