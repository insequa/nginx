# Guide to installing a LEMP stack on Ubuntu

**Last update**: 03/06/2018, tested on Ubuntu 18.04

## Overview

This quide will walk you through the steps to setup, install and configure a LEMP stack on Digital Ocean with:

- Nginx
- PHP7.2 (php-fpm)
- MariaDB
- Other Optional packages: git, munin, rabbitmq, supervisor, node.js, Let's Encrypt, postfix

## Contents
   * [Basic installation process of LEMP](#basic-installation-process-of-lemp)
      * [Overview](#overview)
      * [Basic Server Setup](#basic-server-setup)
         * [Create Droplet](#create-droplet)
         * [Automation](#automation)
         * [Manual installation](#manual-installation)
         * [Additional setup steps](#manual-installation)
            * [Set the correct timezone](#set-the-correct-timezone)
      * [Webserver installation](#webserver-installation)
         * [Install Nginx](#install-nginx)
         * [Setup Firewall](#setup-firewall)
         * [Install MariaDB](#install-mariadb)
         * [Install PHP7.2](#install-php72)
         * [Choose and install PHP7.2 modules](#choose-and-install-php72-modules)
         * [Check the installed PHP version](#check-the-installed-php-version)
         * [Configure Nginx](#configure-nginx)
      * [Add new website, configuring PHP &amp; Nginx &amp; MariaDB](#add-new-website-configuring-php--nginx--mariadb)
         * [Create the dir structure for new website](#1-create-the-dir-structure-for-new-website)
         * [User groups and roles](#2-user-groups-and-roles)
         * [Update permissions](#3-update-permissions)
         * [Create new PHP-FPM pool for new site](#4-create-new-php-fpm-pool-for-new-site)
         * [Configure the new pool](#5-configure-the-new-pool)
         * [Restart PHP fpm and check it's running](#6-restart-php-fpm-and-check-its-running)
         * [Create new "vhost" for Nginx](#7-create-new-vhost-for-nginx)
         * [Configure the vhost](#8-configure-the-vhost)
         * [Enable the new vhost](#9-enable-the-new-vhost)
         * [MariaDB (MySQL)](#10-mariadb-mysql)
      * [Others](#others)
         * [Git Aware Prompt](#git-aware-prompt)
         * [Git](#git)
         * [Adminer](#adminer)
         * [Postfix (sending emails from PHP)](#postfix-sending-emails-from-php)
         * [Munin](#munin)
         * [Rabbitmq](#rabbitmq)
         * [Supervisor](#supervisor)
         * [Node.js &amp; NPM](#nodejs--npm)
      * [Todo](#todo)
      * [Reference](#reference)
         * [Setting PHP-FPM](#setting-php-fpm)
      * [License](#license)

## Basic Server Setup

### Create Droplet

For this part of the process, follow the steps outlined in the link below to create a Digital Ocean Droplet:

https://www.digitalocean.com/community/tutorials/automating-initial-server-setup-with-ubuntu-18-04

You will also need to ensure the Private networking and IPv6 options were selected during the creation (if you wish).

### Automation

Automated installation script coming soon!!!

### Manual Installation

We will assume that you have already setup and secured your Ubuntu server instance using the process listed above and for that reason, these setup steps are omitted.

You will also need to open up the ports on the Firewall for 'Nginx HTTP' and 'Nginx HTTPS' as directed below.

### Additional setup steps

### Set the correct timezone

```sh
sudo dpkg-reconfigure tzdata
```

We setup our timezone as Europe > London.

## Webserver installation

For the purpose of this tutorial, we will install Nginx from a package manager 

### 1. Install Nginx
```sh
sudo apt update
sudo apt install nginx
```

### 2. Setup Firewall

For this example, we will use the UFW firewall.

```sh
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow 'OpenSSH'
```

Confirm the status of the firewall:
```sh
sudo ufw status
```

If this shows your status is active and the server is allowing traffic for the above then this was successful.

For more information and further setup steps, please refer to the following Digital Ocean article:
https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-ubuntu-18-04

### 3. Install MariaDB

We will be using the most stable version of MariaDB at the time of writing which is 10.1. You can check what the latest stable release is by visiting:

https://downloads.mariadb.org/mariadb/repositories/#mirror=ukfast&distro=Ubuntu&distro_release=bionic--ubuntu_bionic

Follow steps 1-3 based on your setup to find the latest release.

```sh
sudo apt update # Update the local list of packages
sudo apt upgrade # Upgrade local packages
sudo apt -y install mariadb-server mariadb-client # Install the latest MariaDB Server and Client using apt
sudo systemctl stop mariadb.service # This will stop MariaDB from running.
```

Other commands to start and enable MariaDB are:

```sh
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```
We need to secure the installation of MariaDB using the following command:

```sh
sudo mysql_secure_installation
```
This will prompt a series of questions of which you should answer:

  * Current root password: Enter
  * Set root password: Enter a new password
  * Confirm the new password: Repeat new password for the last step
  * Remove anonymous users: Y
  * Remove remote root login: Y
  * Remove the test DB and access: Y
  * Reload the privilege table: Y
  
Now check that MariaDB is running:

```sh
sudo mysql -u root -p
```

When prompted, enter the new password for the root user that you created in the last step. You should see a message confirming that the install was successful.

```mysql
exit # To exit MySQL
```

### 4. Install PHP7.2
```sh
sudo apt install php-fpm php-mysql # Install PHP-FPM - the module to process PHP with NginX
php -v # Verify the installation and version
```
If you see PHP 7.2.x then you have installed the right version

### 5. Choose and install PHP7.2 modules
```sh
sudo apt-get -y install php7.2-fpm php7.2-curl php7.2-gd php7.2-json php7.2-mysql php7.2-sqlite3 php7.2-pgsql php7.2-bz2 php7.2-mbstring php7.2-soap php7.2-xml php7.2-zip
```

```sh
sudo nano /etc/php/7.2/fpm/php.ini # Make PHP more secure
```

```php
cgi.fix_pathinfo=0 # Find this condition, un-comment and change to resemple this
```

Save & close the file and then restart PHP-FPM

```sh
sudo systemctl restart php7.2-fpm
111

### 6. Check the installed PHP version
```sh
php -v
```

### 7. Configure Nginx

#### Configure `/etc/nginx/nginx.conf`

Now we can setup NginX to use the PHP processor to present dynamic content

```sh
sudo nano /etc/nginx/sites-available/default


```sh
worker_processes auto;
events {
        use epoll;
        worker_connections 1024; # ~ RAM / 2
        multi_accept on;
}
```

#### Default vhost

```sh
cd /etc/nginx/sites-available
sudo rm default
sudo wget https://raw.githubusercontent.com/lucien144/lemp-stack/master/nginx/sites-available/default
cd /etc/nginx/conf.d
sudo wget https://raw.githubusercontent.com/lucien144/lemp-stack/master/nginx/conf.d/gzip.conf
```

#### Setup default settings for all virtual hosts

```sh
sudo mkdir -p /etc/nginx/conf.d/server/
cd /etc/nginx/conf.d/server/
sudo wget https://raw.githubusercontent.com/lucien144/lemp-stack/master/nginx/conf.d/server/1-common.conf
```

#### Reload Nginx

```sh
sudo nginx -t && sudo nginx -s reload
```

## Add new website, configuring PHP & Nginx & MariaDB

Steps 1. - 9. can be skipped by calling the `add-vhost.sh`. Just download `add-vhost.sh`, `chmod u+x ./add-vhost.sh` and call it `sudo ./add-vhost.sh`.
The file is deleted automatically.

```sh
cd ~
wget https://raw.githubusercontent.com/lucien144/lemp-stack/master/add-vhost.sh
chmod u+x add-vhost.sh
sudo ./add-vhost.sh
```

### 1. Create the dir structure for new website
```sh
sudo mkdir -p /var/www/vhosts/new-website.tld/{web,logs,ssl}
```

### 2. User groups and roles
```sh
sudo groupadd new-website
sudo useradd -g new-website -d /var/www/vhosts/new-website.tld new-website
sudo passwd new-website
```

You can switch users by using `sudo su - new-website`


### 3. Update permissions
```sh
sudo chown -R new-website:new-website /var/www/vhosts/new-website.tld
sudo chmod -R 0775 /var/www/vhosts/new-website.tld
```

### 4. Create new PHP-FPM pool for new site
```sh
sudo nano /etc/php/7.2/fpm/pool.d/new-website.tld.conf
```

#### 5. Configure the new pool
```sh
[new-website]
user = new-website
group = new-website
listen = /run/php/php7.2-fpm-new-website.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off
pm = dynamic
pm.max_children = 5 # The hard-limit total number of processes allowed
pm.start_servers = 2 # When nginx starts, have this many processes waiting for requests
pm.min_spare_servers = 1 # Number spare processes nginx will create
pm.max_spare_servers = 3 # Number spare processes attempted to create
pm.max_requests = 500
chdir = /
```

##### 5.1 Configuring `pm.max_children`
1. Find how much RAM FPM consumes: `ps -A -o pid,rss,command | grep php-fpm` -> second row in bytes
    1. Reference: [https://overloaded.io/finding-process-memory-usage-linux](https://overloaded.io/finding-process-memory-usage-linux)
2. Eg. ~43904 / 1024 -> ~43MB per one process
3. Calculation: If server has 2GB RAM, let's say PHP can consume 1GB (with some buffer, otherwise we can use 1.5GB): 1024MB / 43MB -> ~30MB -> pm.max_childern = 30

##### 5.2 Configuring `pm.start_servers`, `pm.min_spare_servers`, `pm.max_spare_servers`
1. `pm.start_servers` == number of CPUs
1. `pm.min_spare_servers` = `pm.start_servers` / 2
1. `pm.max_spare_servers` = `pm.start_servers` * 3

#### 6. Restart PHP fpm and check it's running
```sh
sudo service php7.2-fpm restart
ps aux | grep new-site
```

### 7. Create new "vhost" for Nginx
```sh
sudo nano /etc/nginx/sites-available/new-site.tld
```

#### 8. Configure the vhost
```sh
server {
    listen 80;

    root /var/www/vhosts/new-site.tld/web;
    index index.php index.html index.htm;

    server_name www.new-site.tld new-site.tld;

    include /etc/nginx/conf.d/server/1-common.conf;

    access_log /var/www/vhosts/new-site.tld/logs/access.log;
    error_log /var/www/vhosts/new-site.tld/logs/error.log warn;

    location ~ \.php$ {
        try_files $uri $uri/ /index.php?$args;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.2-fpm-new-site.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### 9. Enable the new vhost
```
cd /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/new-site.tld new-site.tld
sudo nginx -t && sudo nginx -s reload
```

### 10. MariaDB (MySQL)
```sh
sudo mysql -u root
> CREATE DATABASE newwebsite_tld;
> CREATE USER 'newwebsite_tld'@'localhost' IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON newwebsite_tld.* TO 'newwebsite_tld'@'localhost';
> FLUSH PRIVILEGES;
```

## Others

### Git Aware Prompt

![](https://cdn-pro.dprcdn.net/files/acc_44118/mIFojX)

If you want to have nice git-aware prompt with some handy aliases, use this:
```
sudo su virtualhostuser
cd ~
mkdir ~/.bash && cd ~/.bash && git clone git://github.com/jimeh/git-aware-prompt.git && cd ~ && wget https://gist.githubusercontent.com/lucien144/56fbb184b1ec01fae1adf2e7abb626b6/raw/525ab312789643d61a937904e236edf8755028ca/.bashrc
bash
```
More information about aliases and other [in this gist](https://gist.github.com/lucien144/56fbb184b1ec01fae1adf2e7abb626b6).

### Git
```
sudo apt-get install git
```

### Adminer

[Adminer](https://www.adminer.org) is a mostly MySQL database management tool. It's really tiny, simple & easy to use.

```
cd /etc/nginx/conf.d/server/
sudo wget https://raw.githubusercontent.com/lucien144/lemp-stack/master/nginx/conf.d/server/4-adminer.conf
sudo mkdir -p /var/www/html/adminer/
cd /var/www/html/adminer/
sudo wget https://www.adminer.org/latest.php -O index.php
sudo chmod a+x index.php
sudo htpasswd -c .htpasswd user
sudo nginx -t && sudo nginx -s reload
```

Adminer is now ready at http://{server.ip}/adminer/

_Also, don't forget to change the username 👆._

### Postfix (sending emails from PHP)

In case you cannot send emails from PHP and getting error (`tail /var/log/mail.log`) `Network is unreachable`, you need to switch Postfix from IPv6 to IPv6.

```
sudo apt-get install postfix
sudo nano /etc/postfix/main.cf
```

Now change the line `inet_protocols = all` to `inet_protocols = ipv4` and restart postfix by `sudo /etc/init.d/postfix restart`.

You can also check if you have opened port 25 by `netstat -nutlap | grep 25`

### Munin

#### 1. Install
`apt-get install munin-node  munin`

#### 2. Configure Munin
1. Uncomment `#host 127.0.0.1` in `/etc/munin/munin-node.conf`
1. Append following code to `/etc/munin/munin-node.conf`

```
[nginx*]
env.url http://localhost/nginx_status
```

#### 3. Configure nginx `/etc/nginx/sites-available/default`
```
sudo nano /etc/nginx/sites-available/default
# Change listen 80 default_server; to
listen 80

#Change listen [::]:80 default_server; to
listen [::]:80

# Add settings for stub status to server {}
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }

# Add setting to access stats online

    location /stats {
        allow YOUR.IP.ADDRESS;
        deny all;
        alias /var/cache/munin/www/;
    }
```

#### 4. Install [plugins](https://www.github.com/munin-monitoring/contrib/)

```
cd /usr/share/munin/plugins
sudo wget -O nginx_connection_request https://raw.github.com/munin-monitoring/contrib/master/plugins/nginx/nginx_connection_request
sudo wget -O nginx_status https://raw.github.com/munin-monitoring/contrib/master/plugins/nginx/nginx_status
sudo wget -O nginx_memory https://raw.github.com/munin-monitoring/contrib/master/plugins/nginx/nginx_memory

sudo chmod +x nginx_request
sudo chmod +x nginx_status
sudo chmod +x nginx_memory

sudo ln -s /usr/share/munin/plugins/nginx_request /etc/munin/plugins/nginx_request
sudo ln -s /usr/share/munin/plugins/nginx_status /etc/munin/plugins/nginx_status
sudo ln -s /usr/share/munin/plugins/nginx_memory /etc/munin/plugins/nginx_memory
```

#### Restart Munin
`sudo service munin-node restart`

### Rabbitmq

Install PHP extension
```
sudo apt-get install php-amqp
```

Install RabbitMQ

```
echo 'deb http://www.rabbitmq.com/debian/ testing main' | sudo tee /etc/apt/sources.list.d/rabbitmq.list
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install rabbitmq-server
sudo service rabbitmq-server status
sudo rabbitmq-plugins enable rabbitmq_management
sudo ufw allow 15672
sudo rabbitmqctl add_user admin *********
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
sudo rabbitmqctl delete_user guest
sudo service rabbitmq-server restart
```

#### Installing plugin
1. Download the `.ez` plugin to `/usr/lib/rabbitmq/lib/rabbitmq_server-{version}/plugins`
1. Enable the plugin by `sudo rabbitmq-plugins enable {plugin name}`

### Supervisor

`sudo apt-get install supervisor`

#### Enable the web interface
```sh
echo "
[inet_http_server]
port=9001
username=admin
password=*********" | sudo tee --append /etc/supervisor/supervisord.conf

sudo service supervisor reload
sudo ufw allow 9001
```

The interface should be available on http://{SERVER_IP}:9001/

### Node.js & NPM
```
sudo apt-get install nodejs
sudo apt-get install npm
```

If you are getting error `/usr/bin/env: ‘node’: No such file or directory` run
```
sudo ln -s /usr/bin/nodejs /usr/bin/node
```

### Composer
```
wget https://raw.githubusercontent.com/composer/getcomposer.org/1b137f8bf6db3e79a38a5bc45324414a6b1f9df2/web/installer -O - -q | php -- --quiet
sudo mv composer.phar /usr/local/bin/composer
```
Reference: https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md

## Todo
- [ ] better vhost permissions for reading for other users
- [ ] better description of nginx configuration
- [ ] script for creating new vhost
- [x] composer
- [ ] Let's encrypt (?)
- [ ] s3cmd
- [ ] automysqlbackup
- [ ] SSH/SFTP jail (?)
    - https://www.linode.com/docs/tools-reference/tools/limiting-access-with-sftp-jails-on-debian-and-ubuntu
    - `makejail`


## Reference
- TOC created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)
- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04
- https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration
- https://gist.github.com/jsifalda/3331643
- http://serverfault.com/questions/627903/is-the-php-option-cgi-fix-pathinfo-really-dangerous-with-nginx-php-fpm
- https://easyengine.io/tutorials/nginx/tweaking-fastcgi-buffers/
- https://gist.github.com/magnetikonline/11312172
- https://www.digitalocean.com/community/questions/warning-your-environment-specifies-an-invalid-locale-this-can-affect-your-user-experience-significantly-including-the-ability-to-manage-packages
- https://www.digitalocean.com/community/tutorials/how-to-host-multiple-websites-securely-with-nginx-and-php-fpm-on-ubuntu-14-04
- https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql
- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04
- https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server
- http://stackoverflow.com/questions/21491996/installing-bower-on-ubuntu-
- http://ithelpblog.com/itapplications/howto-fix-postfixsmtp-network-is-unreachable-error/
- https://www.digitalocean.com/community/tutorials/how-to-create-hot-backups-of-mysql-databases-with-percona-xtrabackup-on-ubuntu-14-04
- https://github.com/jnstq/munin-nginx-ubuntu
- https://letsecure.me/secure-web-deployment-with-lets-encrypt-and-nginx/

### Setting PHP-FPM
 - https://www.if-not-true-then-false.com/2011/nginx-and-php-fpm-configuration-and-optimizing-tips-and-tricks/
 - http://myshell.co.uk/blog/2012/07/adjusting-child-processes-for-php-fpm-nginx/
 - https://jeremymarc.github.io/2013/04/22/nginx-and-php-fpm-for-performance
 - http://myshell.co.uk/blog/2012/07/adjusting-child-processes-for-php-fpm-nginx/
 - https://serversforhackers.com/video/php-fpm-process-management
 - https://overloaded.io/finding-process-memory-usage-linux

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).
