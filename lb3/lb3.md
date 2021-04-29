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

## Grafische Übersicht
Die Umgebung besitzt ein internes Netzwerk mit dem Subnetz 192.168.0.0/16. Der Reverse Proxy und die drei Apache-Webserver haben eine eigene IP-Adresse. Jedoch ist nur der Reverse Proxy von extern über den NAT-Port 8080 erreichbar. NAT (Network Address Translation) wird hier verwendet, um die öffentliche IP-Adresse in die private und umgekehrt umzuwandeln.

![Netzwerkplan](./Bilder/Netzwerkplan.png)

## Code Beschreibung

## Testen

## Quellenverzeichnis
* Apache-Webserver: https://de.wikipedia.org/wiki/Apache_HTTP_Server
* Reverse Proxy: https://www.wintotal.de/reverse-proxy/