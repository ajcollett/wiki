# Manager Server Installation on basic Ubuntu Server

This guide will help you setup the basic install for [Manager accounting software](http://manager.io) on an Ubuntu 16.04 instance, secure the server and setup SSL.

## Short Version

If you are familiar with manager and linux like operating systems, this may be a useful summary of the following guide. I aim to:

  2. Install a Nginx instance,
  1. Setup a firewall with minimal open ports (`22, 443, 80`),
  3. Install and setup Manager on the default port (`8080`),
  4. Install and configure LetsEncrypt,
  5. Setup Nginx to:
    * Reverse proxy our Manager instance from internal port `8080` to external port `443` with `https`,
    * Redirect `http (80)` traffic to `https (443)`.  

## Server Setup
 1. We will need:

  * A server somewhere. We will assume an Ubuntu 16.04 installation on a Digital Ocean instance, however these steps should work in many places.
  * Sudo and ssh access to the server.
  * Some command line experience.
  * Nginx to reverse proxy the site through https.
  * UFW (uncomplicated Fire Wall) for security.

 2. Steps:

  * Get your server installed, with openSSH for connection
    In this case, we will deploy a Digital Ocean Droplet, and once it is up, carry on from there.
  * Log-in to your server
```
    ssh <user>@<server>
```
  * Install Nginx
```
    sudo apt install nginx
```
  * Setup the config for your new server config,
  `sudo nano /etc/nginx/sites-available/manager.conf`
```
    TODO
```
  * Setup the firewall
  You should read the [Ubuntu Documentation on how to use UFW](https://help.ubuntu.com/lts/serverguide/firewall.html).
  Open these ports for the Server.
```
  sudo ufw allow 22
  sudo ufw allow 443
  sudo ufw allow 80
```
  You may want to add more ports, but these will be the ones we use foro manager.

## Manager Installation
W.I.P
## LetsEncrypt Setup
W.I.P
