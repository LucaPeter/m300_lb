# LB3 - Container Umgebung mit Reverse Proxy und Apache-Webserver

## Inhaltsverzeichnis
* 1 [Einleitung](#einleitung) 
* 2 [Service Beschreibung](#service-beschreibung)
* 3 [Service Anwendung](#service-anwendung)
* 4 [Grafische Übersicht](#grafische-übersicht)
* 5 [Code Beschreibung](#code-beschreibung)
* 6 [Testen](#testen)
* 7 [Quellenverzeichnis](#quellenverzeichnis)

## Einleitung
Das ist meine technische Dokumentation zu meinem Container Projekt. Hier sind alle wichtigen Informationen dokumentiert. Die Umgebung besitzt einen Reverse Proxy und dahinter drei Apache-Webserver. Dabei werden die Apache-Webserver vom Reverse Proxy geschützt. Ich habe zwei Apache Container und einen eigenen Container, welcher auch auf Apache basiert, erstellt.

### Apache-Webserver
Beim Apache-Webserver handelt es sich um den weltweit am meisten verwendeten Webserver. Dieser ist ein quelloffenes und freies Produkt der Apache Software Foundation. Der Apache-Webserver bietet besondere Eigenschaften. Er ist sehr variabel und kann durch den freien Quellcode individuell angepasst werden. Des Weiteren besitzt er eine Schnittstelle für viele Programmiersprachen.

### Reverse Proxy
Der Reverse Proxy leitet Anfragen aus dem Internet an den internen Webserver weiter. Dieser wird allerdings nur dann kontaktiert, wenn der Reverse-Proxy die Anfrage nicht aus seinem eigenen Cache beantworten kann. Dadurch ist der Webserver vor direkten Angriffen von aussen sicher.

## Service Beschreibung

## Service Anwendung
Damit man diese Umgebung zum Laufen bringt, müssen ein paar Vorbereitungen gemacht werden:

1. Ein Verzeichnis in der Docker VM erstellen.
2. Die Dateien Dockerfile, docker-compose.yml und nginx.conf in das Verzeichnis einfügen.
3. In der Datei "nginx.conf" muss bei der Linie 7 die IP des Host's, auf dem die Docker Umgebung läuft, eingetragen werden.
4. Im Verzeichnis drei Ordner mit den Namen http1, http2 und http3 erstellen.
5. In diesen Ordnern die index.html Datei einfügen.

Anschliessend kann der Container mit docker build -t web03 . gebaut werden. Nachdem der Cointainer gebaut wurde, startet man die Umgebung mit "docker-compose up -d".

## Grafische Übersicht
Die Umgebung besitzt ein internes Netzwerk mit dem Subnetz 172.28.0.0/16. Der Reverse Proxy und die drei Apache-Webserver haben eine eigene IP-Adresse. Jedoch ist nur der Reverse Proxy von extern über den NAT-Port 80 erreichbar. NAT (Network Address Translation) wird hier verwendet, um die öffentliche IP-Adresse in die private und umgekehrt umzuwandeln.

![Netzwerkplan](./Bilder/Netzwerkplan.png)

## Code Beschreibung
In diesem Kapitel werde ich den Code von jeder Datei beschreiben.

### Dockerfile
Einen Layer aus dem Docker-Image ubuntu:18.04 erstellen.

`FROM ubuntu:18.04`

Update des Systems und Installation des Apache-Webservers.

`RUN apt-get update && apt-get install -y apache2`

Apache-Umgebungsvariablen setzen.
```
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_RUN_DIR /var/www/htm
```

Mit dem Befehl EXPOSE wird der Port angegeben, an dem der Container auf Verbindungen wartet.

`EXPOSE 80`

Die Software des Images mit allen Argumenten ausführen.

`CMD ["/usr/sbin/apache2", "-D", "FOREGROUND"]`


### docker-compose.yml
Es wir die Version 3 für das Docker-Compose Format verwendet. Unter "Services" sind die Konfigurationen vom Reverse Proxy und von den Webservern definiert. Als erstes werden die Startparameter vom Reverse Proxy (nginx) festgelegt. Als Image wird die neuste Version von nginx benutzt. Anschliessend wird der Container Name, sowie das Volume definiert. Das Volume zeigt auf das nginx.conf File. Ausserdem werden die Ports und das Netzwerk angegeben. Beim Netzwerk ist die IP-Adresse des Reverse Proxys definiert.
```
version: '3'
services:
  nginx:
    image: nginx:latest
    container_name: production_nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 80:80
    restart: always
    networks:
        testing_net:
            ipv4_address: 172.28.1.2
```

```
  Web_App_1:
    image: httpd
    container_name: Web_App_1
    volumes:
      - ./http1/index.html:/usr/local/apache2/htdocs/index.html
    expose:
      - "80"
    restart: always
    depends_on:
      - nginx
    networks:
        testing_net:
            ipv4_address: 172.28.1.3

  Web_App_2:
    image: httpd
    container_name: Web_App_2
    volumes:
      - ./http2/index.html:/usr/local/apache2/htdocs/index.html
    expose:
      - "80"
    restart: always
    depends_on:
      - Web_App_1
    networks:
        testing_net:
            ipv4_address: 172.28.1.4
```

```
  Web_App_3:
    image: web03
    container_name: Web_App_3
    volumes:
      - ./http3/index.html:/var/www/html/index.html
    expose:
      - "80"
    restart: always
    networks:
        testing_net:
            ipv4_address: 172.28.1.5
```

Hier wird das interne Netzwerk mit dem Subnetz "172.28.0/16" erstellt.
```
networks:
    testing_net:
        ipam:
            driver: default
            config:
                - subnet: 172.28.0.0/16
```

### nginx.conf


## Testen
Ich habe sechs Testfälle durchgeführt. Dabei wurde jeder Service auf die Funktion getestet. Das Ergebnis wird mit einem Bild gezeigt.

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 1 | Docker | Der Befehl "docker build -t web03 ." wird eingegeben. Danach wird der Container mit dem Image "web03" gebaut. | Nach dem Eingeben des Befehls wurde der Container mit dem Image gebaut.  | OK |

![Test1](./Bilder/Test1.png)

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 2 | Docker | Der Befehl "docker-compose up -d" wird eingegeben. Anschliessend wird der Container erstellt und gestartet. | Nach dem Eingeben des Befehls wurde der Container erstellt und gestartet. | OK |

![Test2](./Bilder/Test2.png)

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 3 | Proxy Server | Wenn im Browser "http://localhost:8080" eingegeben wird, gibt Nginx den Fehler 404 zurück. | Auf der Webseite wird der Fehler 404 zurückgegeben.  | OK |

![Test3](./Bilder/Test3.png)

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 4 | Webserver 1 (Web_App_1)| Wenn im Browser "http://localhost:8080/web1" eingegeben wird, erscheint eine rote Webseite mit dem Text "Webseite 1" und "Roter Hintergrund". | Die Webseite ist rot und wird mit dem Text "Webseite 1" und "Roter Hintergrund" angezeigt.  | OK |

![Test4](./Bilder/Test4.png)

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 5 | Webserver 2 (Web_App_2)| Wenn im Browser "http://localhost:8080/web2" eingegeben wird, erscheint eine blaue Webseite mit dem Text "Webseite 2". | Die Webseite ist blau und wird mit dem Text "Webseite 2" und "Blauer Hintergrund" angezeigt. | OK |

![Test5](./Bilder/Test5.png)

| Nr.            | Service           | Erwartung         | Ergebnis | OK? |
|:---------------|:------------------|:----------------- |:---------|:----|
| 6 | Webserver 3 (Web_App_3)| Wenn im Browser "http://localhost:8080/web3" eingegeben wird, erscheint eine gelbe Webseite mit dem Text "Webseite 3". | Die Webseite ist gelb und wird mit dem Text "Webseite 3" und "Gelber Hintergrund" angezeigt. | OK |

![Test6](./Bilder/Test6.png)


## Quellenverzeichnis
* Apache-Webserver: https://de.wikipedia.org/wiki/Apache_HTTP_Server
* Reverse Proxy: https://www.wintotal.de/reverse-proxy/
* httpd Image: https://hub.docker.com/_/httpd
* nginx Image: https://hub.docker.com/_/nginx