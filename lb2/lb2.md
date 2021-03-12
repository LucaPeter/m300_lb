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
<p>Den Typ des Betriebssystems angeben, welches in Virtualbox verwendet wird.</p>

`config.vm.box = "ubuntu/xenial64"`

<p>Eine benannte virtuelle Umgebung definieren.</p>

`config.vm.define "db-server" do |db|`

<p>Netzwerk Interface mit der IP-Adresse zuweisen.</p>

`db.vm.network "public_network", :bridge => "eth0", ip: "192.168.1.100"`

```
db.vm.network "forwarded_port", guest: 3306, host: 3306
db.vm.network "forwarded_port", guest: 80, host: 8306
```

`db.vm.provision "shell", path: “bootstrap.sh"`

<h2>Testen</h2>
<h2>Quellenverzeichnis</h2>