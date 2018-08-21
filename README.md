# Setup Docker
> Dieser Vorgang ist einmal durchzuführen

* Docker for MacOS aus dem Docker Store herunterladen - das erfordert einen Login, d.h. du musst dir einen Account erstellen.
	* Docker for MacOS Download: https://store.docker.com/editions/community/docker-ce-desktop-mac
	* Du kannst auch versuchen, ob dieser Link hier ohne Login funktioniert: https://download.docker.com/mac/stable/Docker.dmg
	* Dann Docker einfach installieren, wie auf der Website beschrieben. Am Ende musst du in deiner Menubar einen kleinen Walfisch mit Containern auf dem Rücken sehen.

# Setup Docker Sportsaller
> Dieser Vorgang ist einmal durchzuführen

* Nun öffnest du ein Terminal. Dort machst du irgendwo in deinem Homeverzeichnis ein neues, leeres Verzeichnis
* In dieses Verzeichnis entpackst du das Zip File, das ich dir geschickt habe. Dieses enthält die Konfiguration für Docker und den initialen DB Dump, den du mir auch schon geschickt hattest
* In das gleiche Verzeichnis legst du den sportsaller git checkout
* Anschließend wird mit Docker Compose ersteinmal alles heruntergeladen und gebaut
* Nachdem der build für die Container durch ist kannst du sie dann starten.

Hier die Befehle:

```
$ mkdir sportsaller-dev
$ cd sportsaller-dev
$ unzip docker-sportsaller.zip (aus meiner Mail)
$ git clone git@github.com:milkycode/sportsaller.git
$ tree -L 2
.
|-- docker-sportsaller
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
> Bevor du den Container startest, musst du darauf achten, dass kein httpd, nginx, MySQL oder öhnliches auf deinem Rechner läuft!

* Die Container stellen dir folgende Ports zur Verfügung:
	* localhost:80 -> HTTP
	* localhost:443 -> HTTPS
	* localhost:3306 -> MariaDB
	* localhost:9000 -> PHP-FPM
* Diese Ports müssen frei sein! Prüfen kannst du das so:

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

* Die Datei `sportsaller-dev/docker-sportsaller/composer.env` musst du anpassen. Hier musst du die Auth Token für `composer` hinterlegen, damit du später `composer install` machen kannst
* Die Datei `sportsaller-dev/docker-sportsaller/global.env` kannst du anpassen. Hier sind Variablen für PHP und MAGE drin.
* Nachdem du sichergestellt hast dass die Ports frei sind und die Dateien angepasst wurden, kannst du den Container starten
* Dazu musst du im Verzeichnis `docker-sportsaller` sein
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

* Im Logging siehst du den Output der einzelnen Container, mit dem entsprechenden Prefix:
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

* Jetzt bist du in einer Shell, die in dem PHP läuft und kannst mit ls, etc rumgucken
* Das htdocs findest du unter `/var/www/magento` - das ist in Wirklichkeit dein Verzeichnis `sportsaller-dev/sportsaller` in dem dein GIT Checkout liegt
* Du kannst Befehle wie `composer`, `php`, etc. verwenden

# Zugriff mit dem Webbrowser
Damit der Magento Kram bei mir funktioniert hat musste ich mich zunächst in den PHP-Container einloggen und `composer install` und anschließend einmal `magento setup:install` machen:

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

Wie das funktioniert weisst du sicherlich besser als ich. Das mit dem `composer install` hat bei mir halt nicht wirklich funktioniert, weil ich nicht alle Module herunterladen konnte - also nicht die kommerziellen...

Nun noch ein Eintrag in der `/etc/hosts` auf deinem Mac mit dem Hostname für den Webserver:

```
$ sudo vi /etc/hosts
127.0.0.1 hosting2018.sport-saller.de
```

Danach solltest du mit `https://hosting2018.sport-saller.de/` auf den Shop kommen.

# Development
* Das `htdocs` kannst du auf deinem Mac lokal unter `sportsaller-dev/sportsaller` editieren - alles was du hier machst kommt automatisch in die Container
* Die MariaDB im Container kannst du mit MySQL Workbench oder etwas ähnlichen mit folgenden Credentials von deinem Mac aus ansprechen:
	* Host: localhost
	* Port: 3306
	* User: root
	* Passwort: magento2
* Die Magento DB hat von deinem Mac aus folgende Credentials
	* Host: localhost
	* Port: 3306
	* Database: magento2
	* User: magento2
	* Passwort: magento2
* `xdebug` hab ich noch nicht gemacht

# Docker zurücksetzen
* Anleitung unter `https://docs.docker.com/docker-for-mac/#reset` -> _Remove all data_ befolgen
* Danach muss der Abschnitt _Setup Docker Sportsaller_ wieder durchgeführt werden

