---
title: "Cấu hình LAMP cho Manjaro"
date: 2025-01-15T11:11:36+07:00
slug: /cau-hinh-lamp-cho-manjaro/
description: Hướng dẫn cấu hình LAMP cho Manjaro Linux.
image: images/Install-LAMP-On-Manjaro.png
caption: Cấu hình LAMP cho Manjaro Linux.
categories:
  - linux
  - laravel
  - web development
tags:
  - linux
  - lamp
  - manjaro
  - laravel
draft: false
---

This tutorial walk you through installing and configuring Apache, MySQL, PHP (LAMP).
LAMP is the the acronym of Linux, Apache, MySQL/MariaDB, PHP/Perl/Python.

## Important

Never add packages without a full system sync - to sync your system execute
```bash
sudo pacman -Syu
```
> The commands in this guide requires root permissions and must be prepended with sudo.

## Install Apache
Install Apache web server using command
```bash
pacman -S apache
```
Edit the */etc/httpd/conf/httpd.conf* file,
```bash
vim /etc/httpd/conf/httpd.conf
```
Search for and comment the following line if it is not already
```
[...]
# LoadModule unique_id_module modules/mod_unique_id.so
[...]
```
Search for ServerAdmin and replace with a valid email (best practice - not necessary in test environments)
```
[...]
ServerAdmin you@example.com
[...]
```
Next edit the ServerName variable to something meaningful - at the bare minimum use your server’s IP address
```
[...]
ServerName ip.x.y.z:80
[...]
```
Save and close the file then enable and start the web service
```
systemctl enable --now httpd
```
Verify the status of the service
```
# systemctl status httpd
● httpd.service - Apache Web Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Fri 2022-11-11 13:03:33 CET; 4s ago
[...]
Nov 11 13:03:33 test systemd[1]: Started Apache Web Server.
```

### Test web service
Test the webservice by creating a sample page in the default web root in /srv/http
```
vim /srv/http/index.html
```
Add text - no need for our test to be strictly html compliant
```
<h2>It works!</h2>
```
Now, open your web browser and navigate to
```
http://ip.x.y.z
```
You should be greeted with the It works message.

### Install MariaDB
MariaDB is the default implementation of MySQL in Manjaro. To install MariaDB execute
```
pacman -S mariadb
```
Initialize the MariaDB data directory prior to starting the service, by using the installer script (do not change --datadir)
```
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```
When the script has completed enable and start the service
```
systemctl enable --now mariadb
```
You can verify the MariaDB service status (shortened)
```
# systemctl status mariadb
● mariadb.service - MariaDB 10.9.3 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: active (running) since Fri 2022-11-11 13:09:51 CET; 10s ago
[...]
Nov 11 13:09:51 test systemd[1]: Started MariaDB 10.9.3 database server.
```
Secure your MariaDB service
It is recommended to secure your database installation using the provided script. Read the prompts carefully - the root password is not your system password but for MariaDB.
```
mariadb-secure-installation
```
### Install PHP
Manjaro uses the - at any time - latest php version. To install PHP and the apache PHP module
```
pacman -S php php-apache
```
Proceed to configure Apache PHP module by editing the file /etc/httpd/conf/httpd.conf
```
vim /etc/httpd/conf/httpd.conf
```
Search and locate the following and edit to read as below
```
[...]
#LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
[...]
```
Scroll to the bottom of the file and add for current PHP
```
LoadModule php_module modules/libphp.so
AddHandler php-script .php
Include conf/extra/php_module.conf
```
Check your config
```
apachectl configtest
```
Save the file and restart the httpd service.
```
apachectl restart
```
### Test PHP
Create a file info.php file in the web service root folder
```
vim /srv/http/info.php
```
With content
```
<?php phpinfo(); ?>
```
Save the file and open your web browser and navigate to http://ip.x.y.z/info.php which should then provide you with the current php configuration, enabled modules etc.

### Install phpMyAdmin
phpMyAdmin is a graphical MySQL/MariaDB administration tool that can be used to create, edit and delete databases. To install phpMyAdmin
```
pacman -S phpmyadmin
```
Create/Edit the file */etc/php/conf.d/phpmariadb.ini*
```
vim /etc/php/conf.d/phpmariadb.ini
extension=bz2
extension=iconv
extension=mysqli
extension=pdo_mysql
```
Save and close the file. Verify your ini-file is loaded
```
php --ini
```
Create Apache configuration
Then create a new Apache configuration to be able to load phpMyAdmin
```
vim /etc/httpd/conf/extra/phpmyadmin.conf
```
```
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
<Directory "/usr/share/webapps/phpMyAdmin">
    DirectoryIndex index.php
    AllowOverride All
    Options FollowSymlinks
    Require all granted
</Directory>
```
Edit the Apache configuration
```
vim /etc/httpd/conf/httpd.conf
```
Include phpMyAdmin configuration at the very end of the file
```
[...]
Include conf/extra/phpmyadmin.conf
```
Save and close the httpd.conf - then test your config
```
apachectl configtest
```
Restart apache
```
apachectl restart
phpMyadmin config
```
Option 1
Edit the phpMyAdmin */etc/webapps/phpmyadmin/config.inc.php* and add a value for blowfish_secret
```
vim /etc/webapps/phpmyadmin/config.inc.php
```
Generate a random number in hex format
```
openssl rand -hex 16
```
Add the value inside the empty quotation marks
```
$cfg['blowfish_secret'] = 'your-generated-value';
```
Add temp folder
```
$cfg['TempDir'] = '/tmp';
```
Save the file

Option 2
Use terminal tools to add the mentioned configuration values.
```
sed -i -e "/blowfish/s/''/'$(openssl rand -hex 16)'/gi" /etc/webapps/phpmyadmin/config.inc.php
echo "\$cfg['TempDir'] = '/tmp';" >> /etc/webapps/phpmyadmin/config.inc.php
```
### Test phpMyAdmin
Open your browser and navigate to

http://ip.x.y.z/phpmyadmin

Source: https://forum.manjaro.org/t/howto-install-apache-mariadb-mysql-php-lamp/13000
