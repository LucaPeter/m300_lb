FROM ubuntu

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update
RUN apt-get -q -y install apache2

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2

RUN mkdir -p /var/lock/apache2 /var/run/apache2
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
RUN echo "Das ist meine Webseite" >> /var/www/index.html

EXPOSE 80
VOLUME /var/www/html