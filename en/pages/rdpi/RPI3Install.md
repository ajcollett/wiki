# RADIUSDesk on Raspberry Pi 3
## Setup

* We will be using a Raspberry pi 3 for this. [This](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

* The Ethernet port will be setup to be the `www`, the point where we connect the Pi to the internet.

* The Wifi will be setup as a wifi network, RPI3 by default

* None of the default settings will be changed, but I will try point out the things that should. Like the raspberry pi root password.

* If you want indepth explanations on some of the steps, have a look at where I got the wiki parts from, in each section.

* I assume you are working from a clean install of [__Rasbian Lite__](https://www.raspberrypi.org/downloads/raspbian/)

## Install Nginx (LEMP stack)
__Note:__ [Reference, and great source of knowledge](http://www.radiusdesk.com/getting_started/install_ubuntu_nginx)

1. Install all the things
Nginx, php5-fpm, mysql-server, mysql-client, php5-mysql, php5-xcache, php5-cli, php5-gd, php5-curl, subversion

```
sudo apt-get install nginx mysql-server mysql-client php5-mysql php5-xcache php5-cli php5-gd php5-curl subversion
```

2. Start and stop Nginx to check that there are no issues, there shouldn't be.
```
sudo service nginx stop
sudo service nginx start
sudo service nginx status
```
Navigate to the pi's ip to see if you get the default greeting.
__Note:__ Nginx, in Debian, serves from ` `

## Configure Nginx to interpret .php files
1. Edit the config so we create a unix socket instead of a websocket
```
sudo nano -c37 /etc/php5/fpm/pool.d/www.conf
```
Change the `listen` part to the following
```
;listen = 127.0.0.1:9000
listen = /var/run/php5-fpm.sock
```
