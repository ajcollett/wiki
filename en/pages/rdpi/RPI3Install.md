# RADIUSDesk on Raspberry Pi 3
## Setup

* We will be using a Raspberry pi 3 for this. [This](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

* The Ethernet port will be setup to be the `www`, the point where we connect the Pi to the internet.

* The Wifi will be setup as a wifi network, RPI3 by default

* None of the default settings will be changed, but I will try point out the things that should. Like the raspberry pi root password.

* If you want indepth explanations on some of the steps, have a look at where I got the wiki parts from, in each section.

* I assume you are working from a clean install of [__Rasbian Lite__](https://www.raspberrypi.org/downloads/raspbian/)

* I often include `sudo service <service> status` in the wiki, this should output `active (running)`, and some other things.

## Install Nginx (LEMP stack)
__Note:__ [Reference, and great source of knowledge](http://www.radiusdesk.com/getting_started/install_ubuntu_nginx)

1. Install all the things
Nginx, php5-fpm, mysql-server, mysql-client, php5-mysql, php5-xcache, php5-cli, php5-gd, php5-curl, subversion, vim
```
sudo apt-get install nginx mysql-server mysql-client php5-mysql php5-xcache php5-cli php5-gd php5-curl subversion vim php5-fpm
```
2. Start and stop Nginx to check that there are no issues, there shouldn't be.
```
sudo service nginx stop
sudo service nginx start
sudo service nginx status
```
Navigate to the pi's ip to see if you get the default greeting.

__Note:__ Nginx, in Debian, serves from `/var/www/html`

## Configure Nginx to interpret .php files
1. Edit the config so we create a unix socket instead of a websocket
```
sudo vim -c37 /etc/php5/fpm/pool.d/www.conf
```
* Make sure the `listen` part is as follows:
```
listen = /var/run/php5-fpm.sock
```
* Restart `fpm`
```
sudo service php5-fpm restart
sudo service php5-fpm status
```

## Modify Nginx
1. Copy the default confi file to a new one
```bash
cd /etc/nginx/sites-available/
sudo cp default default.orig
```
2. Edit the new config file
```
sudo vim default
```
* Add *index.php* to this line:
```
# Add index.php to the list if you are using PHP
index index.php index.html index.htm index.nginx-debian.html;
```
* Activate PHP precessing by uncommenting this this section. Note that we use the UNIX socket:
```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php5-cgi alone:
#       fastcgi_pass 127.0.0.1:9000;
        # With php5-fpm:
        fastcgi_pass unix:/var/run/php5-fpm.sock;
}
```
* Enable the hiding of *.htaccess* files, just in case
```
location ~ /\.ht {
        deny all;
}
```
* Reload `nginx`
```bash
sudo service nginx reload
sudo service nginx status
```
* Create a test *.php* script
```
sudo vim /var/www/html/test.php
```
* Add the below
```
<?php
    phpinfo();
?>
```
* Visit __http://*your pi ip*/test.php__ to test. You should see a bunch of php data in your browser.
