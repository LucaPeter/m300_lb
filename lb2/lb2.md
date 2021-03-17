<h1>LB2 - Technische Dokumentation</h1>
<h2>Inhaltsverzeichnis</h2>
<p>Einleitung<br>Grafische Übersicht<br>Konfiguration<br>Testen<br>Quellenverzeichnis</p>

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

`config.vm.box = "ubuntu/xenial64"`

<p>Virtualisierungsplattform definieren.</p>

`db.vm.provider "virtualbox" do |vb|`

<p>Anzahl RAM für die VM.</p>

`vb.memory = "384"`

<p>Hostname des Datenbank-Servers.</p>

`db.vm.hostname = "db01"`

<p>Die Netzwerkeinstellungen mit der IP-Adresse und dem Port konfigurieren. Der MySQL Port ist nur im Private Network sichtbar</p>

```
db.vm.network "private_network", ip: "192.168.10.200"`
db.vm.network "forwarded_port", guest:3306, host:3306, auto_correct: false
```
<p>Shell-Skript innerhalb des Gastbetriebssystems direkt nach dem Hochfahren ausführen.</p>

`db.vm.provision "shell", path: "db.sh"`

config.vm.provision "shell", inline: <<-SHELL 
    set -o xtrace
	sudo apt-get update
	sudo apt-get -y install debconf-utils 
	sudo apt-get -y install apache2 
	sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password Hallo123'
	sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password Hallo123'
	sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server 
	
	sudo mkdir /usr/share/adminer
	sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
	sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
	echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
	sudo a2enconf adminer.conf 
	sudo service apache2 restart 
  
  sudo ufw allow 80/tcp 
  sudo ufw allow 22/tcp 
  sudo ufw -f enable 

<h2>Testen</h2>
<h2>Quellenverzeichnis</h2>