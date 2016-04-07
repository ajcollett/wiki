# RADIUSDesk on Raspberry Pi 3
## Setup

* We will be using a Raspberry pi 3 for this. [This](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

* The Ethernet port will be setup to be the `www`, the point where we connect the Pi to the internet.

* The Wifi will be setup as a wifi network, RPI3 by default

* None of the default settings will be changed, but I will try point out the things that should. Like the raspberry pi root password.

* If you want indepth explanations on some of the steps, have a look at where I got the wiki parts from, in each section.

* I assume you are working from a clean install of [__Rasbian Lite__](https://www.raspberrypi.org/downloads/raspbian/)

* I often include `sudo service <service> status` in the wiki, this should output `active (running)`, and some other things.

* Make sure you expand the file system! This is going to take lots of space.

## Install Nginx (LEMP stack)
__Note:__ [Reference, and great source of knowledge](http://www.radiusdesk.com/getting_started/install_ubuntu_nginx)

1. Install all the things
Nginx, php5-fpm, mysql-server, mysql-client, php5-mysql, php5-xcache, php5-cli, php5-gd, php5-curl, subversion, vim

__Warning! Be sure to supply a root password for the MySQL database when asked for it if you are security conscious else simply hit the ESC key.__

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
```
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
```
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

## Performance tune Nginx

### Modify expiry date for certain files
* Edit the /etc/nginx/sites-available/default file:
```
sudo vim /etc/nginx/sites-available/default
```
* Add the following inside the server section:
```
location ~* ^.+\.(jpg|jpeg|gif|png|ico|js|css)$ {
    rewrite ^/cake2/rd_cake/webroot/(.*)$ /cake2/rd_cake/webroot/$1 break;
    rewrite ^/cake2/rd_cake/(.*)$ /cake2/rd_cake/webroot/$1 break;
    access_log off;
    expires max;
    add_header Cache-Control public;
}
```
* Reload Nginx:
```
sudo service nginx reload
sudo service nginx status
```

### Compress the text before sending it to client
* Edit the main config file of Nginx.
```
sudo vim /etc/nginx/nginx.conf
```
* Change the compression section to contain the following:
```
  ##
  # Gzip Settings
  ##

  gzip on;
  #gzip_disable "msie6";

  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  # gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js;
  #gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
  gzip_buffers 16 8k;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
```
* Restart Nginx
```
sudo service nginx restart
sudo service nginx status
```

## Install RADIUSDesk

__Warning!__ Check out the __Install RADIUSdesk__ section on [the original wiki](http://www.radiusdesk.com/getting_started/install_ubuntu_nginx)

### Install CakePHP
1. Download 2.8.X version. I found this to work. You may need to adjust this.
```
wget https://github.com/cakephp/cakephp/archive/2.8.3.tar.gz
```
2. Copy and extract it inside the directory that Nginx is serving its content from.
```
sudo cp 2.8.3.tar.gz /var/www/html
cd /var/www/html
sudo tar -xzvf 2.8.3.tar.gz
sudo ln -s ./cakephp-2.8.3 ./cake2
```
3. Reload php5-fpm
```
sudo service php5-fpm reload
sudo service php5-fpm status
```

### Install RADIUSdesk CakePHP Application

We will be using `subversion` we installed earlier to checkout RADIUSdesk

1. Check out the rd_cake branch from trunk to /var/www/html
```
cd /var/www/html/cake2
sudo svn checkout svn://dvdwalt@svn.code.sf.net/p/radiusdesk/code/trunk/rd_cake ./rd_cake
```
2. Change the following directories to be writable by www-data:
```
sudo chown -R www-data. /var/www/html/cake2/rd_cake/tmp
sudo chown -R www-data. /var/www/html/cake2/rd_cake/Locale
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/img/flags
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/img/nas
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/img/realms
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/img/dynamic_details
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/img/dynamic_photos
sudo chown -R www-data. /var/www/html/cake2/rd_cake/webroot/files/imagecache
```

__warning!__ The below is not working yet. I'm working on it!

3. Now we need to edit a few files for the log file viewer to control radiusd
```
sudo vim /usr/share/nginx/html/cake2/rd_cake/Setup/Scripts/radmin_wrapper.pl
```
  * We are using radiusd, not freeradius. Change so this section looks like this:
  ```
        #___ Start ____
        if($arg1 eq 'start'){
            #system("/etc/init.d/radiusd start");
            system("service radiusd start");
            #system("/etc/init.d/freeradius start");
        }
        
        #___ Stop ____
        if($arg1 eq 'stop'){
            #system("/etc/init.d/radiusd stop");
            system("service radiusd stop");
            #system("/etc/init.d/freeradius stop");
        }
  ```

### The database
1. Create the following blank databases:
```
mysql -u root
create database rd;
GRANT ALL PRIVILEGES ON rd.* to 'rd'@'127.0.0.1' IDENTIFIED BY 'rd';
GRANT ALL PRIVILEGES ON rd.* to 'rd'@'localhost' IDENTIFIED BY 'rd';
exit;
```
2. Populate the database (trunk):
```
mysql -u root rd < /var/www/html/cake2/rd_cake/Setup/Db/rd.sql
```
3. Make things perform better for RPI
```
mysql -u root
USE rd;
DELETE FROM phrase_values WHERE language_id=16 OR language_id=15 OR language_id=13 OR language_id=5 OR language_id=14;
exit;
```

### Configure Nginx
1. Edit `/etc/nginx/sites-enabled/default` to enable rewrite for our URLS
```
sudo vim /etc/nginx/sites-enabled/default
```
  * Add the following section inside the server section:
  ```
  location /cake2/rd_cake {
       rewrite ^/cake2/rd_cake/(.*)$ /cake2/rd_cake/webroot/$1 break;
       try_files $uri $uri/ /cake2/rd_cake/webroot/index.php?q=$uri&$args;
  }
  ```
2. Reload Nginx
```
sudo service nginx reload
sudo service nginx status
```
3. Testing things out

  Go here:
  > http://ip_of_pi/cake2/rd_cake/phrase_values/get_language_strings.json?\_dc=1355816922405&language=

  You should see something like this:
  ```
   {"data":{"phrases":{"spclCountry":"United Kingdom","spclLanguage":"English","sUsername":"......","success":true}
  ```


## Viewer component
1. Check out the latest code of the viewer component under the /var/www/nginx/html/ directory:
```
cd /var/www/html/
sudo svn checkout svn://dvdwalt@svn.code.sf.net/p/radiusdesk/code/trunk/rd ./rd
```
2. Get ExtJS toolkit This is a large file!
```
cd /var/www/html/
sudo svn checkout svn://svn.code.sf.net/p/radiusdesk/code/extjs ./
sudo mv  ext-6-sencha_cmd.tar.gz ./rd
cd /var/www/html/rd
sudo tar -xzvf ext-6-sencha_cmd.tar.gz
```
3. Now try to log in on the following URL with username root and password admin:
> http://ip_of_pi/rd/build/production/Rd/index.html

## Cron scripts
* __RADIUSdesk__ requires a few scripts to run periodically in order to maintain a healthy and working system.
```
sudo cp /var/www/html/cake2/rd_cake/Setup/Cron/rd /etc/cron.d/
```
* If you want to change the default intervals at which the scripts get executed, just edit the /etc/cron.d/rd file.

## Installing FreeRADIUS 3
This section requires some compiling. This isn't too much of an issue on the RPI 3 though.

1. First, download FR 3, the one in the repo's is only 2.x. Then extract.
```
cd ~/
wget ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-3.0.11.tar.bz2
bunzip2 freeradius-server-3.0.11.tar.bz2
tar -xvf freeradius-server-3.0.11.tar
```
2. Install for libs we need for compile
```
sudo apt-get install libtalloc-dev libssl-dev libmysqlclient-dev libperl-dev libperl5.20
```
2. Now we will configure, compile and install freeradius 3. We need to change some default install directories so that things play nicely with the default config.
```
cd freeradius-server-3.0.11/
./configure --prefix='/usr' --localstatedir='/var' --with-logdir='/var/log/freeradius' --sysconfdir='/etc' --with-raddbdir='/etc/freeradius' --libdir='/usr/lib/freeradius'
make
make install
```

## Configuring FreeRADIUS version 3.x
1. Do the following to configure FreeRADIUS 3.x to work with RADIUSdesk
* Stop the service if it might be running
```
sudo service radiusd stop
```
* Backup the original config files
```
sudo mv /etc/freeradius /etc/freeradius.orig
```
* Copy the RADIUSdesk specific ones and extract
```
sudo cp /var/www/html/cake2/rd_cake/Setup/Radius/freeradius-3-radiusdesk.tar.gz /etc/
cd /etc
sudo tar -xzvf freeradius-3-radiusdesk.tar.gz
```
* Now, we need to create the systemd startup file
```
vim /etc/systemd/system/radiusd.service
```
* Add the below and save
  ```
  [Unit]
    Description=FreeRADIUS multi-protocol policy server
    After=syslog.target network.target mysql.service
    Documentation=man:radiusd(8) man:radiusd.conf(5) http://wiki.freeradius.org/ http://networkradius.com/doc/

  [Service]
    Type=forking
    PIDFile=/run/radiusd/radiusd.pid
    ExecStartPre=/usr/sbin/radiusd -Cxm -lstdout
    ExecStart=/usr/sbin/radiusd
    Restart=on-failure
    RestartSec=5

  [Install]
    WantedBy=multi-user.target

  ```
* Configure the site wide shared secret. This will be the value used by ALL Dynamic Clients.
```
sudo vim /etc/freeradius/sites-enabled/dynamic-clients
```
* Look for this part in the file and change FreeRADIUS-Client-Secret to the value you choose to use.
  ```
  #  Echo the IP address of the client.
  FreeRADIUS-Client-IP-Address = "%{Packet-Src-IP-Address}"

  # require_message_authenticator
  FreeRADIUS-Client-Require-MA = no

  # secret
  FreeRADIUS-Client-Secret = "testing123"

  # shortname
  FreeRADIUS-Client-Shortname = "%{Packet-Src-IP-Address}"
  ```
* Change the name of the program from `freeradius` to `radiusd`:
```
#  name of the running server.  See also the "-n" command-line option.
name = radiusd
```
* Create a user and group `freerad`
```
sudo adduser --system freerad
sudo addgroup freerad
sudo addgroup freerad freerad
```
* *chown* the directories that FreeRADIUS will user
```
sudo chown -R freerad:freerad /var/log/freeradius/
sudo chown -R freerad:freerad /etc/freeradius/
sudo chown -R freerad:freerad /var/run/radiusd
```
* After you completed these commands you can test if FreeRADIUS starts up fine with systemd
```
sudo service freeradius start
sudo service freeradius status
```
* If all is well, let FR3 start at boot
```
sudo systemctl enable radiusd.service
```


## Add script to sudoers file
* To create the ability for the web server to exercise some control over FreeRADIUS, we will have a custom script which is added to the sudoers file.
```
sudo visudo
```
* Add this at the bottom
```
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL www-data ALL = NOPASSWD:/var/www/html/cake2/rd_cake/Setup/Scripts/radmin_wrapper.pl
```
* Now to confirm it's happened
```
sudo cat /etc/sudoers
```
* This will allow the root user in RADIUSdesk to start and stop FreeRADIUS and also to do on-the-fly activation of debug traces.


## Install NodeJS

1. Download NodeJS and extract. You should be able to get the latest.
```
cd ~/
wget https://nodejs.org/dist/v4.4.2/node-v4.4.2-linux-armv7l.tar.xz
sudo tar -xvf node-v4.4.2-linux-armv7l.tar.xz
mv node-v4.4.2-linux-armv7l node
```
2. Install dependencies
```
sudo apt-get install python-software-properties python g++ make software-properties-common
```
3. Install node modules globally
```
cd ~/node/bin
sudo ./npm -g install tail
sudo ./npm -g install socket.io@0.9.x
sudo ./npm -g install connect
sudo ./npm -g install mysql
sudo ./npm -g install forever
```
4. We need to install a start-up file, start the Node.js server up and confirm that it works.
```
sudo cp /var/www/html/cake2/rd_cake/Setup/Node.js/nodejs-socket-io /etc/init.d
sudo chmod 755 /etc/init.d/nodejs-socket-io
sudo update-rc.d nodejs-socket-io defaults
```
* Change the script
```
sudo vim /etc/init.d/nodejs-socket-io
```
* At this point
```
NAME="Node.js SocketIO"
NODE_BIN_DIR="/usr/bin"
NODE_PATH=/usr/lib/nodejs:/usr/lib/node_modules:/usr/share/javascript
#APPLICATION_DIRECTORY="/usr/share/nginx/www/cake2/rd_cake/Setup/Node.js"
APPLICATION_DIRECTORY="/usr/share/nginx/html/cake2/rd_cake/Setup/Node.js"
APPLICATION_START="Logfile.node.js"
PIDFILE="/var/run/nodejs-socket-io.pid"
LOGFILE="/var/log/nodejs-socket-io.log"
```
* Replace with
```
NAME="Node.js SocketIO"
NODE_BIN_DIR="/home/pi/node/bin"
NODE_PATH=/home/pi/node/lib/nodejs:/home/pi/node/lib/node_modules:/usr/share/javascript
APPLICATION_DIRECTORY="/var/www/html/cake2/rd_cake/Setup/Node.js"
APPLICATION_START="Logfile.node.js"
PIDFILE="/var/run/nodejs-socket-io.pid"
LOGFILE="/var/log/nodejs-socket-io.log"
```
* Test
```
sudo systemctl daemon-reload
sudo service nodejs-socket-io restart
sudo service nodejs-socket-io status
```
* Confirm it is running by checking the log file output:
```
sudo cat /var/log/nodejs-socket-io.log
```
* Results in....
```
info: socket.io started
Up and running on port 8000
```

## Install the dynamic login pages
__Note:__ [Reference](http://www.radiusdesk.com/getting_started/install_ubuntu_dynamic_login_pages)

The Dynamic Login Page include a desktop and a mobile login page.
1. Installation
  ```
  cd /var/www/html

  sudo svn checkout svn://dvdwalt@svn.code.sf.net/p/radiusdesk/code/trunk/rd_login ./rd_login
  ```
2. Configuration
You should checkout [this page for config.](http://www.radiusdesk.com/getting_started/install_ubuntu_dynamic_login_pages)

## Install Coova Chilli
__Note:__ [Reference](http://www.radiusdesk.com/getting_started/install_ubuntu_coovachilli)

1. Get the binaries and extract
```
cd ~/
wget https://coova.github.io/coova-chilli/coova-chilli-1.3.0.tar.gz
sudo tar -xvf coova-chilli-1.3.0.tar.gz
```
2. Confure, build and install
```
sudo apt-get install debhelper
cd coova-chilli-1.3.0
export CFLAGS="-Wno-error"
./configure --prefix=/usr
dpkg-buildpackage -us -uc
cd ../
sudo dpkg -i coova-chilli_1.3.0_armhf.deb
```

### Configure
* Edit the default file
```
sudo vim /etc/default/chilli
```
* Change it to this:
```
START_CHILLI=1
CONFFILE="/etc/chilli.conf"
HS_USER="chilli"
```
* Edit the config file
```
sudo vim /etc/chilli/defaults
```
* Changing the `HS_LANIF=eth1` to `HS_LANIF=wlan0`
* Save and start
```
sudo service chilli start
sudo service chilli status
```
* Make sure there is a tun interface present when you look at the feedback of `ip a`.
```
ip a
7: tun0: <POINTOPOINT,UP,LOWER_UP> mtu 1500 qdisc mq state UNKNOWN group default qlen 100
    link/none
    inet 10.1.0.1/24 scope global tun0
       valid_lft forever preferred_lft forever
```
* Create the `/etc/chilli/config` config
```
sudo vim /etc/chilli/config
```
* Add the below, as a starting point for how you finally want things
```
HS_WANIF=eth0              # The internet access port
HS_LANIF=wlan0             # Subscriber Interface for client devices
HS_NETWORK=10.1.0.0        # HotSpot Network (must include HS_UAMLISTEN)
HS_NETMASK=255.255.0.0     # HotSpot Network Netmask
HS_UAMLISTEN=10.1.0.1      # HotSpot IP Address (on subscriber network)
HS_UAMPORT=3990            # HotSpot UAM Port (on subscriber network)
HS_UAMUIPORT=4990          # HotSpot UAM "UI" Port (on subscriber network, for embedded portal)
HS_NASID=localhost
HS_RADIUS=localhost
HS_RADIUS2=localhost
HS_RADSECRET=testing123    # Set to be your RADIUS shared secret
HS_UAMSECRET=greatsecret     # Set to be your UAM secret
HS_UAMALIASNAME=chilli
HS_SSID="Struisbaai"
HS_NASIP=127.0.0.1    # To explicitly set NAS-IP-Address
HS_UAMSERVER=$HS_UAMLISTEN
HS_UAMFORMAT=http://\$HS_UAMLISTEN/cake2/rd_cake/dynamic_details/chilli_browser_detect/
HS_MACAUTH=on              # To turn on MAC Authentication
HS_TCP_PORTS="80 23 8000"
HS_MODE=hotspot
HS_TYPE=chillispot
HS_WWWDIR=/etc/chilli/www
HS_WWWBIN=/etc/chilli/wwwsh
HS_PROVIDER=Coova
HS_PROVIDER_LINK=http://www.coova.org/
HS_LOC_NAME="My HotSpot"           # WISPr Location Name and used in portal
HS_COAPORT=3799
```
* Comment the following line out of `/etc/chilli/defaults`
```
#   Same principal goes for HS_UAMHOMEPAGE.
#HS_UAMHOMEPAGE=http://\$HS_UAMLISTEN:\$HS_UAMPORT/www/coova.html
```
*  Also comment the DNS server settings out in /etc/chilli/default to force CoovaChilli to use the DNS servers of the system that it is running on.
```
# OpenDNS Servers
#HS_DNS1=208.67.222.222
#HS_DNS2=208.67.220.220
```
* Use the following `/etc/chilli/ipup.sh` file as a guideline
```
UAM server specified as 10.1.0.1
iptables -I INPUT -i tun0 -p tcp -m tcp --dport 80 --dst 10.1.0.1 -j ACCEPT
iptables -I INPUT -i tun0 -p tcp -m tcp --dport 443 --dst 10.1.0.1 -j ACCEPT
iptables -I INPUT -i tun0 -p tcp -m tcp --dport 22 --dst 10.1.0.1 -j ACCEPT
iptables -I INPUT -i tun0 -p tcp -m tcp --dport 8000 --dst 10.1.0.1 -j ACCEPT
```
* Use the following `/etc/chilli/ipdown.sh` file as a guideline
```
UAM server specified as 10.1.0.1
iptables -D INPUT -i tun0 -instp tcp -m tcp --dport 80 --dst 10.1.0.1 -j ACCEPT
iptables -D INPUT -i tun0 -p tcp -m tcp --dport 443 --dst 10.1.0.1 -j ACCEPT
iptables -D INPUT -i tun0 -p tcp -m tcp --dport 22 --dst 10.1.0.1 -j ACCEPT
iptables -D INPUT -i tun0 -p tcp -m tcp --dport 8000 --dst 10.1.0.1 -j ACCEPT
```

## Add NAT support
__Warning!__ Failing to do this step will leave you with a broken system.

* Edit the `/etc/init.d/chilli` file and add the following in this section:
        ```
        test ${HS_ADMINTERVAL:-0} -gt 0 && {
            (crontab -l 2>&- | grep -v $0
                echo "*/$HS_ADMINTERVAL * * * * $0 radconfig"
                ) | crontab - 2>&-
        }
        
        #NAT mod
        iptables -F POSTROUTING -t nat
        iptables -I POSTROUTING -t nat -o $HS_WANIF -j MASQUERADE
        #END NAT mod
        
        ifconfig $HS_LANIF 0.0.0.0
        ```
* Test things out
```
sudo systemctl daemon-reload
sudo service chilli restart
sudo service chilli status
```
* Make things permenent
```
sudo update-rc.d chilli start 99 2 3 4 5 . stop 20 0 1 6 .
```
* Restart the pi
```
sudo reboot
```
* Check that all the things are running, some errors may be present though in chilli, because the hotspot is not up yet.
```
sudo service chilli status
sudo service radiusd status
sudo service nodejs-socket-io status
```

## Final stretch, Setup the wifi HotSpot
Now we will need to setup the wireless hotspot.

* Edit the interfaces file
```
sudo vim /etc/network/interfaces
```
* Comment out the `wlan0` and `wlan1` config
```
#allow-hotplug wlan0
#iface wlan0 inet manual
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
#
#allow-hotplug wlan1
#iface wlan1 inet manual
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```
* Then add this at the bottom
```
allow-hotplug wlan0
iface wlan0 inet manual
```
* Install `hostapd`
```
sudo apt-get install hostapd
```
* Edit `/etc/default/hostapd`
```
#DAEMON_CONF=""
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
* Now create `/etc/hostapd/hostapd.conf`, and edit if you'd like
```
interface=wlan0
ssid=RPI3
hw_mode=g
channel=6
macaddr_acl=0
#auth_algs=1
ignore_broadcast_ssid=1
#wpa=2
#wpa_passphrase=ping
#wpa_key_mgmt=WPA-PSK
#wpa_pairwise=TKIP
#rsn_pairwise=CCMP
```
* Now reboot and try connect to your new hotspot!

__Note!__ Please do message me if you find anything a-miss, or have suggestions!

## Cleanup
We want the Pi image to be small! So clean up. __If__ you don't intend to develope on here again.
* Remove the packages downloaded and installed.
```
sudo rm /var/www/html/2.8.3.tar.gz
sudo rm /var/www/html/rd/ext-6-sencha_cmd.tar.gz
sudo rm /home/pi/freeradius-server-3.0.11.tar
sudo rm /etc/freeradius-3-radiusdesk.tar.gz
```
* Remove the things we used to build with that we no longer need. 
```
sudo apt-get autoremove debhelper libtalloc-dev libssl-dev libmysqlclient-dev libperl-dev g++ vim 

```
