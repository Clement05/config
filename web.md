#Web server configuration 

This is confing commands executing from [Grafikart.fr tutorial](https://www.grafikart.fr/formations/serveur-linux).

This have been done on a raspberry pi 2

## First Step: get update/upgrade

```
#!bash

sudo apt-get update
sudo apt-get upgrade

```

## Second Step: configure Apache 2

### Get Apache 2

```
#!bash

sudo apt-get install apache2

```
### Create a "www" repo in your user folder

```
#!bash

mkdir /home/pi/www

```

### Make a simlink from the created repo to the accessible folder on the internet

```
#!bash

sudo ln -s /home/pi/www /var/www/g-ingenierie.fr

```

### Change log file in a file that it is not accessible from the internet

```
#!bash

sudo mkdir /home/pi/log
sudo touch /home/pi/log/error.log

```

The command to listen log is 

```
#!bash

sudo tail - f /home/pi/log/error.log

```

### Then we need to active the rewrite module to allow .htaccess files doing their job

```
#!bash

sudo a2enmod rewrite
sudo service apache2 restart

```

### It is time now to configure the virtualhost

```
#!bash

sudo nano /etc/apache2/sites-available/001-myawesomesite.conf

```

The 001-myawesomesite.conf content is:

```
#!text

<VirtualHost *:80>
	ServerAdmin contact@myawesomesite.com
	ServerName myawesomesite.com
	ServerAlias *.myawesomesite.com
	
	DocumentRoot /var/www/myawesomesite.com>

	<Directory /var/www/myawesomesite.com
		Options -Indexes +FollowSymLinks
		AllowOverride All
	</Directory>

	ErrorLog /home/pi/log/error.log # note that we configure this in the previous step

</VirtualHost>

```

The we restart apache

```
#!bash

sudo service apache2 restart

```

### Activate the website

```
#!bash

sudo a2ensite 001-myawesomesite
sudo service apache2 reload

```

### Testing the configuration can be done via:

```
#!bash

/usr/sbin/apache2ctl configtest

```

### We can had .htacces in the website folder that redirect url

```
#!bash

sudo nano /home/pi/www/.htaccess

```

The .htaccess content is:

```
#!text

RewriteEngine On
RewriteCond %{HTTP_HOST} !^myawesomesite.com$ [NC]
RewriteRule ^(.*)$ http://myawesomesite.com/$1 [R=301,L]

```

### Enhance safety

Be sure that 000-default.conf Document root is /var/www/html

```
#!bash

sudo nano /etc/apache2/sites-available/000-default.conf

```

Verify DocumentRoot

```
#!text

DocumentRoot /var/www/html

```

Remove indexes and useless directory then reload

```
#!bash

sudo service apache2 reload

```

You can also dismiss 000-default with 

```
#!bash

sudo a2dissite 000-default
sudo service apache2 reload

```

### You need to add Apache 2 permissions to your user pi

Add your user pi in the Apache 2 group www-data

```
#!bash

sudo usermod -g www-data pi

```


## Step 3: Install PHP

### Install PHP as a apache mod

```
#!bash

sudo apt-get install libapache2-mod-php5 php5

```

### Configure debug (only when your are developping)

```
#!bash

sudo nano /etc/php5/apache2/php.ini

```

In php.ini file find display errors and set it to ON

```
#!text

display_errors On

```

Then restart Apache 2

```
#!bash

sudo service apache2 restart

```

### Install Curl to download package from url

```
#!bash

sudo apt-get install curl

```

### Install Composer

```
#!bash

cd
curl -sS https://getcomposer.org/installer | php

```

### Install Laravel

#### Get Laravel
```
#!bash

php composer.phar create-project --prefer-dist laravel/laravel demo

```

#### Give permissions to Laravel folders

```
#!bash

sudo chmod g+w -R demo/storage
sudo chown -R pi:www-data demo/storage

```

#### Move Laravel folder to our www folder

```
#!bash

sudo mv /home/pi/demo /home/pi/www/demo

```

## Step 4: Install MySQL

### Get MySQL

```
#!bash

sudo apt-get install mysql-server mysql-client
sudo apt-get install php5-mysql
sudo service apache2 restart

```

### Adminer to administrate MySQL (or PhpMyAdmin if you prefer)

Go to [https://www.adminer.org/#download](https://www.adminer.org/#download)

Copy it in /home/pi/wwww through FTP

## Step 5: Install PostFix to send mail

### Get PostFix

```
#!bash

sudo apt-get install postfix

```

Choose config : site interet

Configure main.cf

```
#!bash

sudo nano /etc/postfix/main.cf

```

```
#!text

mydestination = localhost.loacaldomain, localhost
inet_interfaces = loopback-only
relay_host = smtp.IAP.com # input your IAP smtp

```

### Restart and install mail utils for testing

```
#!bash

sudo service postfix restart
sudo apt-get install mailutils

```

## Step 6: Configure firewall

### Get iptables

```
#!bash

sudo apt-get install iptables

```

### Create a file firewall.sh

firewall.sh contents:

```
#!/bin/sh

# On vide les règles déjà existantes
iptables -t filter -F
iptables -t filter -X

# On refuse toutes les connexions
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
iptables -t filter -P OUTPUT DROP
echo "On interdit tout"

# On autorise les connexions déjà établie
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# On autorise le loop-back (localhost)
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -t filter -A OUTPUT -o lo -j ACCEPT

# On autorise le ping
iptables -t filter -A INPUT -p icmp -j ACCEPT
iptables -t filter -A OUTPUT -p icmp -j ACCEPT

# On autorise le SSH (à adapter suivant votre cas)
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT

# Si on a un serveur DNS
iptables -t filter -A OUTPUT -p tcp --dport 53 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 53 -j ACCEPT
iptables -t filter -A INPUT -p udp --dport 53 -j ACCEPT

# NTP (pour avoir un serveur à l'heure)
iptables -t filter -A OUTPUT -p udp --dport 123 -j ACCEPT

# HTTP
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 8443 -j ACCEPT

# FTP
iptables -t filter -A OUTPUT -p tcp --dport 21 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 20 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 20 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 21 -j ACCEPT
iptables -t filter -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# MAIL
## SMTP
iptables -t filter -A INPUT -p tcp --dport 25 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 25 -j ACCEPT
## POP3
iptables -t filter -A INPUT -p tcp --dport 110 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 110 -j ACCEPT
## IMAP
iptables -t filter -A INPUT -p tcp --dport 143 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 143 -j ACCEPT
echo "trafic IMAP sur le port 143 autorisé"
## POP3S
iptables -t filter -A INPUT -p tcp --dport 995 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 995 -j ACCEPT  

```

### Give execution to firewall.sh

```
#!bash

sudo sh /home/pi/firewall.sh

```

### Get fail2ban

```
#!bash

sudo apt-get install fail2ban

```

### Move fail2ban.conf into jail.local (because .conf is erase at launch)

```
#!bash

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

```

### Edit jail.local

```
#!bash

sudo nano /etc/fail2ban/jail.local

```

jail.local content is:

```
#!text

bantime = 3600
destmail = contact.g-ingenierie.fr
sender = pi@raspberrypi
mta = mail
action = %(action_mwl)s

```

### Restart fail2ban

```
#!bash

sudo service fail2ban restart

```

### fail2ban commands are:

```
#!bash

sudo fail2ban-client status
sudo fail2ban-client status ssh
sudo fail2ban-client set ssh unbanip 10.0.0.2

```
