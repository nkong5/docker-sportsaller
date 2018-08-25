# Setup Docker
> This process is to be done once

* Download Docker for MacOS from the Docker store. You might need a docker account if you don't already have one
	* Docker for MacOS Download: https://store.docker.com/editions/community/docker-ce-desktop-mac
	* This link could function without a docker account: https://download.docker.com/mac/stable/Docker.dmg
	* Install Docker as described on their website. If all goes well, you should see a whale icon with a container on it on the upper status bar of your MacOS.

# Setup Docker Sportsaller
> This process is to be done once

* Open the terminal and navigate to a directory where you want your docker and magento 2 files to be stored.
* In this directory clone the docker installation and configuration with  ``` git clone https://github.com/nkong5/docker-sportsaller.git ```. The repository files contain the initial docker configuration und database dump of the sport-saller shop.
* In the same directory checkout the magento 2 code of the sportsaller shop
* After that, docker-compose will be used to download and build the container and shop
* After the build of the container is done, then docker is ready to be accessed and further configured

These are the commands:

```
$ mkdir sportsaller-dev
$ cd sportsaller-dev
$ git clone https://github.com/nkong5/docker-sportsaller.git
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

# Starting the docker environment
> Bevor the container is started, it must be made sure that httpd, nginx, MySQL or similar services are not running on the operating system!

* The container is using the following ports:
	* localhost:80 -> HTTP
	* localhost:443 -> HTTPS
	* localhost:3306 -> MariaDB
	* localhost:9000 -> PHP-FPM
* These ports must be free. That can be checked as follows:

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

* The file  `docker-sportsaller/composer.env` should be adjusted. It is especially necessary here to fill in the Github Auth Token for `composer`, in order to later use `composer install`. Generate the token here if you don't already have one: `https://github.com/settings/tokens`
* The file `docker-sportsaller/global.env` can also be adjusted. It has PHP und MAGE  variables.
* When it has been made sure that the aforementioned ports are free and the environment files have been adjusted accordingly, the container can be started.
* In this case you have to be in the folder `docker-sportsaller` 
* There you can simply run `docker-compose up`. It starts the whole container as needed. After the command has been run there is a lot of logging at the terminal
* To end the container at the terminal simply use CTRL-C

```
$ cd sportsaller-dev/docker-sportsaller
$ docker-compose up
Creating network "docker-sportsaller_default" with the default driver
Creating volume "docker-sportsaller_dbdata" with default driver
...
CTRL-C zum beenden der Container
```

* The Logging has outputs for every different docker container with the following prefixes:
	* db_1 -> MariaDB
	* fpm_1 -> PHP FPM
	* cli_1 / cron_1 -> Magento Cron
	* web_1 -> nginx

# Log into the PHP container on a different terminal window as below.
* Open a new terminal window
* Give in the following commands:

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

* Now you are at a shell where PHP is running. Normal windows commands like ls, cd, etc can be used to have a closer look at the folder structure, files and do other actions
* The htdocs magento root folder is found at `/var/www/magento` - this is actually the folder `sportsaller-dev/sportsaller` (Magento 2 code that was checked out) that has been mounted into the docker container.
* Other commands like `composer`, `php`, etc. can be also used

# Calling the Shop from the Webbrowser 
`composer install` and then `magento setup:install` have to be run in the container, before the shop can be accessible through a browser request without errors. 

Before running both commands make sure that you copy all extensions from the production environment into you vendor folder:

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

For  webserver access, we still need to write the hostname in `/etc/hosts` (this is especially the case when using a MacOS or another unix system):

```
$ sudo vi /etc/hosts
127.0.0.1 sport-saller.local
```

After that, call `https://sport-saller.local/` to reach the Magento 2 shop.

# Development
* The `htdocs` files of the docker container can also be edited locally on your machine at  `sportsaller-dev/sportsaller`. All the edits are automatically available in the container
* The MariaDB in the docker container can be accessed with a database manager like MySQL Workbench using the following credentials from a Mac:
	* Host: localhost
	* Port: 3306
	* User: root
	* Passwort: magento2
* The Magento DB can be accessed from a Mac or another OS with the following credentials:
	* Host: localhost
	* Port: 3306
	* Database: magento2
	* User: magento2
	* Passwort: magento2
* `xdebug` has been configured and should function in a standard way using PHPStorm

# Reset Docker 
* Use the following instructions to reset docket `https://docs.docker.com/docker-for-mac/#reset` -> then follow _Remove all data_ 
* After that the chapter _Setup Docker Sportsaller_ should be executed again

