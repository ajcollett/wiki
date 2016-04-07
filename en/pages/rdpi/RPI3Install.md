# RADIUSDesk on Raspberry Pi 3
## Setup

* We will be using a Raspberry pi 3 for this. [This](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

* The Ethernet port will be setup to be the `www`, the point where we connect the Pi to the internet.

* The Wifi will be setup as a wifi network, RPI3 by default

* None of the default settings will be changed, but I will try point out the things that should. Like the raspberry pi root password.

* If you want indepth explanations on some of the steps, have a look at where I got the wiki parts from, in each section.

* I assume you are working from a clean install of [__Rasbian Lite__](https://www.raspberrypi.org/downloads/raspbian/)

## Install Nginx
__Note:__ [From here](http://www.radiusdesk.com/getting_started/install_ubuntu_nginx)

* Install all the things
Lang pack, Nginx, php5-fpm, mysql-server, mysql-client, php5-mysql, php5-xcache, php5-cli, php5-gd, php5-curl, subversion

```bash
sudo apt-get install language-pack-en-base nginx mysql-server mysql-client php5-mysql php5-xcache php5-cli php5-gd php5-curl subversion
```