# Setup Docker
> Dieser Vorgang ist einmal durchzuführen

* Docker for MacOS aus dem Docker Store herunterladen - das erfordert einen Login, d.h. ein Account musst erstellt werden falls nicht vorhanden.
	* Docker for MacOS Download: https://store.docker.com/editions/community/docker-ce-desktop-mac
	* Alternativerweise ist es eventuell möglich Docker ohne Account mit diesem Link runterzuladen: https://download.docker.com/mac/stable/Docker.dmg
	* Dann Docker einfach installieren, wie auf der Website beschrieben. Am Ende ist in der Menubar (auf dem Mac) einen kleinen Walfisch mit Containern auf dem Rücken zu sehen.

# Setup Docker Sportsaller
> Dieser Vorgang ist einmal durchzuführen

* Nun den Terminal öffnen. Dort ein leeres Verzeichnis für das Projekt erstellen
* In diesem Verzeichnis die Docker-Installation mit ``` git clone https://github.com/nkong5/docker-sportsaller.git ``` clonen. Dieses enthält die Konfiguration für Docker und den initialen DB Dump des Sport-Saller-Shops.
* In das gleiche Verzeichnis den sportsaller Magento2-Code auschecken.
* Anschließend wird mit Docker Compose ersteinmal alles heruntergeladen und gebaut
* Nachdem der build für die Container durch ist kann sie dann gestartet werden.

Hier die Befehle:

```
$ mkdir sportsaller-dev
$ cd sportsaller-dev
$ git clone https://github.com/nkong5/docker-sportsaller.git
$ git clone https://github.com/milkycode/sportsaller.git
$ tree -L 2
.
|-- docker-sportsalleren-US
|   |-- composer.env
|   |-- db
|   |-- docker-compose.yml
|   |-- global.env
|   |-- mage2-nginx
|   |-- mage2-php-7.0-cli
|   |-- mage2-php-7.0-fpm
|   `-- web
`-- sportsaller
    |-- README.md
    |-- scripts
    `-- src

9 directories, 4 files
$ cd docker-sportsaller
$ docker-compose build
... das dauert eine ganze Weile, weil er PHP Module baut, etc
```

# Starten der Docker Umgebung
> Bevor der Container gestartet wird, musst  darauf geachtet werden, dass kein httpd, nginx, MySQL oder öhnliches auf dem Rechner läuft!

* Die Container stellen folgende Ports zur Verfügung:
	* localhost:80 -> HTTP
	* localhost:443 -> HTTPS
	* localhost:3306 -> MariaDB
	* localhost:9000 -> PHP-FPM
* Diese Ports müssen frei sein! Prüfen kann man so:

```
$ nc -v localhost 80
nc: connectx to localhost port 80 (tcp) failed: Connection refused
nc: connectx to localhost port 80 (tcp) failed: Connection refused

$ nc -v localhost 443
nc: connectx to localhost port 443 (tcp) failed: Connection refused
nc: connectx to localhost port 443 (tcp) failed: Connection refused

$ nc -v localhost 3306
nc: connectx to localhost port 3306 (tcp) failed: Connection refused
nc: connectx to localhost port 3306 (tcp) failed: Connection refused

$ nc -v localhost 9000
nc: connectx to localhost port 9000 (tcp) failed: Connection refused
nc: connectx to localhost port 9000 (tcp) failed: Connection refused
```

* Die Datei `sportsaller-dev/docker-sportsaller/composer.env` musst angepasst werden. Hier musst die Auth Token für `composer` hinterlegen werden, damit später `composer install` durchgeführt werden kann. Der Token kann hier erstellt werden `https://github.com/settings/tokens`
* Die Datei `sportsaller-dev/docker-sportsaller/global.env` kann auch angepasst werden. Hier sind Variablen für PHP und MAGE drin.
* Nachdem sichergestellt worden ist, dass die Ports frei sind und die Dateien angepasst wurden, kann der Container gestartet werden
* Dazu musst man sich im Verzeichnis `docker-sportsaller` befinden
* Dort dann einfach `docker-compose up` ausführen. Das startet die ganzen benötigten Container. Das Logging kommt auf auf dem Terminal raus
* Zum Beenden einfach CTRL-C drücken und warten bis die Container beendet sind

```
$ cd sportsaller-dev/docker-sportsaller
$ docker-compose up
Creating network "docker-sportsaller_default" with the default driver
Creating volume "docker-sportsaller_dbdata" with default driver
...
CTRL-C zum beenden der Container
```

* Im Logging sind die Outputs der einzelnen Container zu sehen, mit dem entsprechenden Prefix:
	* db_1 -> MariaDB
	* fpm_1 -> PHP FPM
	* cli_1 / cron_1 -> Magento Cron
	* web_1 -> nginx

# In den PHP Container 'einloggen'
* Ein neues Terminal aufmachen
* Folgende Zeilen eingeben:

```
$ cd sportsaller-dev/docker-sportsaller
$ docker-compose run cli
Starting docker-sportsaller_db_1 ... done
[ ok ] Starting enhanced syslogd: rsyslogd.
[ ok ] Starting Mail Transport Agent (MTA): sendmail.
root@cli:/# cd /var/www/magento/
root@cli:/var/www/magento# ls -1
CHANGELOG.md
COPYING.txt
Gruntfile.js.sample
LICENSE.txt
LICENSE_AFL.txt
app
auth.json.sample
autocomplete.php
bin
...

root@cli:/# exit
exit
$
```

* Jetzt ist man in einer Shell, in dem PHP läuft und Befehle wie ls, cd etc ausgeführt werden können
* Das htdocs befindet sich unter `/var/www/magento` - das ist in Wirklichkeit das Verzeichnis `sportsaller-dev/sportsaller` in dem der GIT Checkout liegt
* Befehle wie `composer`, `php`, etc. können auch verwendet werden

# Zugriff mit dem Webbrowser
Bevor die Seite auf dem Webbrowser aufgerufen werden kann, musst der Magento-Code fertig konfiguriert werden. Am besten das vendor-Verzeichnis mit allen Extensions aus dem Produktionssystem erstmal befüllen. Danach in den PHP-Container einloggen und `composer install` und anschließend einmal `magento setup:install` durchführen:

```
root@cli:/# cd /var/www/magento/
root@cli:/# composer install
...

root@cli:/# php bin/magento setup:install \
--db-host="db" \
--db-name="magento2" \
--db-user="magento2" \
--db-password="magento2" \
--admin-firstname="firstname" \
--admin-lastname="lastname" \
--admin-email="admin@domain.com" \
--admin-user="username" \
--admin-password="pass%1234"
```

Nun noch ein Eintrag in der `/etc/hosts` auf dem Mac mit dem Hostname für den Webserver:

```
$ sudo vi /etc/hosts
127.0.0.1 sport-saller.local
```

Danach sollte man mit `https://sport-saller.local/` auf den Shop kommen.

# Development
* Das `htdocs` kann  auf dem Mac lokal unter `sportsaller-dev/sportsaller` editiert werden - alles was hier geändert wird, kommt automatisch in die Container
* Die MariaDB im Container kann mit MySQL Workbench oder etwas ähnlichen mit folgenden Credentials von einem Mac aus angesprochen werden:
	* Host: localhost
	* Port: 3306
	* User: root
	* Passwort: magento2
* Die Magento DB hat von dem Mac aus folgende Credentials:
	* Host: localhost
	* Port: 3306
	* Database: magento2
	* User: magento2
	* Passwort: magento2
* `xdebug` ist auch installiert und konfiguriert und sollte wie üblich mit PHPStorm laufen.

# Docker zurücksetzen
* Anleitung unter `https://docs.docker.com/docker-for-mac/#reset` -> _Remove all data_ befolgen
* Danach muss der Abschnitt _Setup Docker Sportsaller_ wieder durchgeführt werden

