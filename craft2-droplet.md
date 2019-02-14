#Digitalocean Manual Setup for Craft2

Create Sudo User
`apt update && apt upgrade`

`adduser dev`
*Create Strong password, add to lastpass ‘Server Sudo Users’ note*

`usermod -aG sudo dev`
`su - dev`
`sudo ls -la /root` (to test sudo privileges)

##Initial Server build

`sudo apt update && sudo apt upgrade -y`

`sudo apt install apache2 mysql-server php-mysql php libapache2-mod-php php-cli php-gd php-common php-curl git nodejs npm ruby-full git-core imagemagick php-imagick php-mbstring php-gettext`

*PINK PASSWORD MYSQL SCREEN: generate secure password for root. WRITE IT DOWN!*

Note: php7.0-mbstring and nodejs-legacy are failing to install with this method. Testing new method with DigitalOcean LAMP one-click install.

`sudo a2enmod rewrite expires cgi ssl headers`

`sudo apt-get -y install gcc make autoconf libc-dev pkg-config`
`sudo apt-get -y install php7.2-dev`
`sudo apt-get -y install libmcrypt-dev`
`sudo pecl install mcrypt-1.0.1`

`touch /etc/php/7.2/mods-available/mcrypt.ini`

`sudo phpenmod mcrypt`

`apt install nodejs npm`

`sudo npm install gulp bower -g`
`sudo gem install sass less`

`sudo apt update && sudo apt upgrade`
`sudo service apache2 restart`

`curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

##mysql update for craft:

`sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

*add the following to the bottom:*

**sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION**

`sudo service mysql restart`

`sudo vim /etc/php/7.2/apache2/php.ini`

*Some basic vim commands: https://coderwall.com/p/adv71w/basic-vim-commands-for-getting-started
press "i" to enter insert mode
search with "/"
:wq to write and quit*


###Change:
upload_max_filesize = 256M
post_max_size = 256M
max_execution_time = 600
max_input_time = 600
max_input_vars = 10000
memory_limit = 256M

`sudo mkdir /var/www/logs`

##craft apache config:

`sudo vim /etc/apache2/sites-enabled/`{{SITE_SPECIFIC_FILENAME}}`.conf`

###Add Text:
<VirtualHost \*:80>
    ServerName {{SITE_DOMAIN}}
    ServerAlias <otherdomain>
    ServerAdmin brian@seejepp.com
    DocumentRoot /var/www/{{DOMAIN}}/public

   LogLevel warn

   ErrorLog /var/www/logs/error.log
    CustomLog /var/www/logs/access.log combined

   <Directory />
        Options FollowSymLinks
        AllowOverride All
    </Directory>
    <Directory /var/www/{{DOMAIN}}/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
        Header set Access-Control-Allow-Origin "\*"
    </Directory>
</VirtualHost>

##SWAP:
*From root folder:*

`sudo fallocate -l 4G /swapfile`
`sudo chmod 600 /swapfile`
`sudo mkswap /swapfile`
`sudo swapon /swapfile`

`sudo nano /etc/fstab`

*add the following to bottom:*
**/swapfile   none    swap    sw    0   0**

`sudo nano /etc/sysctl.conf`

*add the following to bottom:*
**vm.swappiness=10**

`sudo service apache2 restart`

##Sendmail Service:

`sudo apt-get install mailutils`

###Choose:
**<Internet Site>**
*Write in the domain*

`sudo nano /etc/postfix/main.cf`

*Search for: inet_interfaces (usually at bottom), change:*
**inet_interfaces = loopback-only**

`sudo service postfix restart`

*Test the mail utilities*
`sudo echo "This is the body" | mail -s "This is the subject" brian@bahlr.com`

##Create Bitbucket Repository, Finish Setup

*Fork Craftcms-template bitbucket repo into new site-specific repository: https://bitbucket.org/brian_seejepp/craftcms-template/src*

*Add server key to new repository:*
`sudo ssh-keygen -t rsa -C "server@domain.com"`
`sudo cat ~/.ssh/id_rsa.pub`

*Copy the key, add it to the bitbucket keys list (Repo > Settings > Access Keys)*
*Git clone the repo into the droplet.

Point the Domain to droplet IP from client’s DNS.
Edit or Add A Record, host @, points to Droplet IP*

###FTP the missing files:
craft/app
public/uploads

###Create Directories:
public/cache
public/imager

###Add server to sequel pro:
Basic Settings:

**MySQL Host: 127.0.0.1
Username: root
Password: {{ mysql password }}
SSH Host: {{ Domain }}
SSH User: root
SSH Key: Add from computer**


###Add Craft Permissions:

*From Root (remember to change ‘project_name’ to the project directory name):*
`sudo chown -R www-data:www-data /var/www`
`sudo chmod -R 770 /var/www`
`sudo chmod -R 775 /var/www/html`
`sudo chmod -R 775 /var/www/project_name/craft`
`sudo chmod 774 /var/www/project_name/craft/app`
`sudo chmod 774 /var/www/project_name/craft/config`
`sudo chmod 774 /var/www/project_name/craft/storage`
`sudo chmod 774 /var/www/project_name/public/uploads`
`sudo chmod 774 /var/www/project_name/public/cache`
`sudo chmod 774 /var/www/project_name/public/imager`

###Craft 3 Permissions (from root):
`sudo chown -R www-data:www-data /var/www`
`sudo chmod -R 770 /var/www`
`sudo chmod -R 775 /var/www/html`

*From project folder:*
`sudo chmod -R 775 .env composer.json composer.lock config/license.key storage/ vendor/ web/cpresources/`
