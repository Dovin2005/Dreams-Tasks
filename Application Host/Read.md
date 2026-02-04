📘 Truelysell Laravel — Local Deployment Guide (Ubuntu + Apache)

This document explains how this project was deployed locally on Ubuntu using Apache, PHP 8.4, MySQL, Node.js 20, and a SQL dump import.

🧱 1. System Requirements

This project requires:

Ubuntu 22.04+

Apache 2

PHP 8.4

MySQL 8

Composer

Node.js 20+

npm

Laravel 12 and its dependencies require PHP ≥ 8.4, which is why upgrading PHP was mandatory.

🔧 2. Install Apache

Apache is used as the web server.

sudo apt update
sudo apt install apache2 -y


Check service:

systemctl status apache2


Test in browser:

http://localhost

🐘 3. Install PHP 8.4

Ubuntu default PHP was older, so we installed PHP 8.4 from Ondřej Surý PPA.

sudo add-apt-repository ppa:ondrej/php
sudo apt update


Install PHP + extensions Laravel needs:

sudo apt install php8.4 php8.4-cli php8.4-mysql php8.4-xml php8.4-mbstring \
php8.4-curl php8.4-bcmath php8.4-zip libapache2-mod-php8.4 -y


Make PHP 8.4 default:

sudo update-alternatives --set php /usr/bin/php8.4


Disable old PHP modules in Apache:

sudo a2dismod php8.1 php8.3
sudo a2enmod php8.4
sudo systemctl restart apache2

📦 4. Install Composer

Composer manages PHP dependencies.

curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

🟢 5. Install Node.js 20

Frontend assets are built using Vite, which requires Node 18+.

Remove old Node:

sudo apt remove nodejs npm libnode-dev -y
sudo apt autoremove -y


Install Node 20:

curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y


Verify:

node -v
npm -v

📂 6. Move Project to Apache Folder

Apache serves apps from:

/var/www/html


Move project:

sudo mv truelysell-laravel-v1 /var/www/html/
cd /var/www/html/truelysell-laravel-v1

🔐 7. File Permissions

Laravel needs write access to storage and bootstrap/cache.

sudo chown -R dovin:dovin .
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

⚙️ 8. Configure .env

Create environment file:

cp .env.example .env


Edit:

nano .env


Set:

APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_DATABASE=truelysell
DB_USERNAME=truelyselluser
DB_PASSWORD=StrongPassword123


Generate key:

php artisan key:generate

🛢️ 9. Database Reset & SQL Import

Instead of running migrations, an SQL dump was imported.

Login MySQL:

sudo mysql


Reset DB:

DROP DATABASE IF EXISTS truelysell;
CREATE DATABASE truelysell CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;


Import SQL dump:

mysql -u truelyselluser -p truelysell < truelysell-laravel-new.sql


To log errors:

mysql -u truelyselluser -p truelysell < truelysell-laravel-new.sql 2> import-errors.log


Verify:

sudo mysql
USE truelysell;
SHOW TABLES;

🌐 10. Apache Virtual Host Setup

Create config:

sudo nano /etc/apache2/sites-available/truelysell.conf


Add:

<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /var/www/html/truelysell-laravel-v1/public

    <Directory /var/www/html/truelysell-laravel-v1/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/truelysell_error.log
    CustomLog ${APACHE_LOG_DIR}/truelysell_access.log combined
</VirtualHost>


Enable site:

sudo a2dissite 000-default.conf
sudo a2ensite truelysell.conf
sudo a2enmod rewrite
sudo systemctl reload apache2

🧹 11. Clear Laravel Cache
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
php artisan storage:link

sudo systemctl restart apache2

🚀 12. Open Application
Visit:

http://localhost
