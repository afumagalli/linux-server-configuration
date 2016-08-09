# Linux Server Configuration Project

## Description

Preparing a baseline installation of a Linux distribution (Ubuntu) on an AWS virtual machine to host web applications. Actions include installing updates, setting up secutiy, and installing and configuring web and database servers.

This server in particular is set up to host the Item Catalog app from earlier in the Full Stack Nanodegree.

## Info

IP address: 52.43.8.245  
SSH port: 2200  
Public address: http://ec2-52-43-8-245.us-west-2.compute.amazonaws.com  
__Note:__ As of my graduation from the Full Stack Nanodegree program, this server will not be available. However, the information in this README may still prove useful.

## Walkthrough

### Log into server as root

1. Run `ssh -i ~/.ssh/key.rsa root@ip`

### Create a user with sudo permissions

1. Create a new user called grader: `adduser grader`
2. Add grader to sudoers by creating a new file: `nano /etc/sudoers.d/grader` and add the line `grader ALL=(ALL:ALL) ALL`
3. AWS will report `sudo: unable to resolve host ip-XX-XX-XX-XXX` for every `sudo` command. To stop this, modify the first line of `/etc/hosts` to be `127.0.0.1 localhost ip-XX-XX-XX-XXX`

### Set up ssh keys for user

1. Make a .ssh directory for grader: `mkdir /home/grader/.ssh`
2. Copy authorized keys from root: `cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys`
3. Change permissions: `chmod 700 /home/grader/.ssh` and `chmod 644 /home/grader/.ssh/authorized_keys`
4. Change owner and group: `chown grader /home/grader/.ssh` and `chown grader /home/grader/.ssh/authorized_keys` (repeat with `chgrp grader`)

### Change SSH port and disable root login

1. Edit `/etc/ssh/sshd_config`: change `Port` to `2200`
2. Change `PermitRootLogin without-password` to `PermitRootLogin no`
3. Make sure to have `PasswordAuthentication no`
4. Run `service ssh restart` to take effect
5. Logout of root and login to grader with `ssh -i ~/.ssh/udacity_key.rsa grader@ip -p 2200`

### Update packages

1. Run `sudo apt-get update`
2. Run `sudo apt-get upgrade`

### Configure timezone to UTC

1. Run `sudo dpkg-reconfigure tzdata`
2. Select "None of the above" and then "UTC"

### Configure Uncomplicated Firewall (UFW)

1. Run `sudo ufw default deny incoming`
2. Run `sudo ufw default allow outgoing`
3. Run `sudo ufw allow 2200/tcp`
4. Run `sudo ufw allow www`
5. Run `sudo ufw allow ntp`
6. Run `sudo ufw enable`

### Install and configure Apache

1. Run `sudo apt-get install apache2`
2. Run `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Start Apache with `sudo service apache2 start`

### Install git and clone catalog app

1. Run `sudo apt-get install git`
2. `cd /var/www` and make the `catalog` directory with `sudo mkdir catalog`. Then `cd catalog`
3. Clone the git repository with `sudo git clone https://github.com/afumagalli/item-catalog.git catalog`
4. Make a `catalog.wsgi` file with the following contents:

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

### Set up app

1. Install `pip` and `libpq-dev` (needed by `psycopg2`): `sudo apt-get install python-pip libpq-dev`
2. Install Flask: `sudo pip install flask flask-sqlalchemy`
3. Install other dependencies: `sudo pip install oauth2client sqlalchemy psycopg2`
4. Add public address to authorized JavaScript origins and redirect URIs
5. Create a config file for virtual host with `sudo nano /etc/apache2/sites-available/catalog.conf`
6. Add the below code then run `sudo a2dissite 000-default.conf` to disable the default virtual host and `sudo a2ensite catalog` to enable the catalog. Run `sudo service apache reload` to take effect.

```
<VirtualHost *:80>
    ServerName ip
    ServerAlias public-address
    ServerAdmin admin@ip
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/pytho$
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Install and configure PostgreSQL

1. Install PostgreSQL: `sudo apt-get install postgresql postgresql-contrib`
2. On installation, PostgreSQL creates a user named `postgres`. Login using `sudo su - postgres` then connect to `psql`
3. Create a PostgreSQL role named catalog: `CREATE USER catalog WITH PASSWORD 'password';`
4. Create the catalog database: `CREATE DATABASE catalog WITH OWNER catalog`
5. Quit psql and logout of postgres: `\q` `exit`
6. Run the database setup script: `python /var/www/catalog/catalog/database_setup.py`

### Launch app

1. Restart Apache with: `sudo service apache2 restart`
2. Visit http://ec2-52-36-157-254.us-west-2.compute.amazonaws.com

### Resources

[Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)  
[How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)  
[How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

