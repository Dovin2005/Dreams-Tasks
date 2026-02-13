📘 Truelysell Laravel — Docker Deployment Guide

This document explains how to deploy the Truelysell Laravel project using Docker with:

PHP 8.4 + Apache

MySQL 8 (Container)

Custom Docker network

SQL dump import

No docker-compose (manual setup)

🏗 Architecture
Docker Network: truelysell-network

Containers:
1️⃣ truelysell-mysql   → mysql:8
2️⃣ truelysell-app     → php:8.4-apache (custom build)

Laravel connects to MySQL using:

DB_HOST=truelysell-mysql
📂 Project Structure
truelysell-laravel-v1/
│
├── Dockerfile
├── .env
├── truelysell-laravel-new.sql
├── public/
├── storage/
├── bootstrap/
├── composer.json
├── package.json
🐳 STEP 1 — Clean Previous Containers
docker stop truelysell-app truelysell-mysql 2>/dev/null
docker rm truelysell-app truelysell-mysql 2>/dev/null
🌐 STEP 2 — Create Docker Network
docker network create truelysell-network

Verify:

docker network ls
🛢 STEP 3 — Run MySQL Container

⚠️ Important: Do NOT expose port 3306 unless required.

docker run -d \
  --name truelysell-mysql \
  --network truelysell-network \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=truelysell \
  -e MYSQL_USER=truelyselluser \
  -e MYSQL_PASSWORD=StrongPassword123 \
  mysql:8

Wait until MySQL is ready:

docker logs truelysell-mysql

Look for:

ready for connections
⚙️ STEP 4 — Configure .env

Create .env from example:

cp .env.example .env

Update the following:

APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=truelysell-mysql
DB_PORT=3306
DB_DATABASE=truelysell
DB_USERNAME=truelyselluser
DB_PASSWORD=StrongPassword123
🐘 STEP 5 — Dockerfile

Ensure your Dockerfile contains:

FROM php:8.4-apache

RUN apt-get update && apt-get install -y \
    git unzip curl libzip-dev \
    libonig-dev libxml2-dev \
    && docker-php-ext-install pdo_mysql mbstring xml zip bcmath

RUN a2enmod rewrite

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

COPY . .

RUN composer install --no-interaction --prefer-dist --optimize-autoloader

RUN chown -R www-data:www-data storage bootstrap/cache \
    && chmod -R 775 storage bootstrap/cache

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
    /etc/apache2/sites-available/*.conf \
    /etc/apache2/apache2.conf \
    /etc/apache2/conf-available/*.conf

EXPOSE 80

🔨 STEP 6 — Build Laravel Image
docker build --no-cache -t truelysell-app .

🚀 STEP 7 — Run Laravel Container
docker run -d \
  --name truelysell-app \
  --network truelysell-network \
  -p 80:80 \
  truelysell-app
  
🔐 STEP 8 — Generate APP_KEY
docker exec -it truelysell-app php artisan key:generate
docker exec -it truelysell-app php artisan config:clear
docker restart truelysell-app
🗑 STEP 9 — Reset Database (Optional)

To remove all tables:

docker exec -it truelysell-mysql mysql -u root -p

Inside MySQL:

DROP DATABASE truelysell;
CREATE DATABASE truelysell;
EXIT;
📥 STEP 10 — Import SQL Dump

From host:

cat truelysell-laravel-new.sql | docker exec -i truelysell-mysql \
mysql -u truelyselluser -pStrongPassword123 truelysell

Wait for command to complete.

🔍 STEP 11 — Verify Tables
docker exec -it truelysell-mysql mysql -u truelyselluser -p

Then:

USE truelysell;
SHOW TABLES;
🧹 STEP 12 — Clear Laravel Cache
docker exec -it truelysell-app php artisan config:clear
docker exec -it truelysell-app php artisan cache:clear
docker restart truelysell-app
🌍 STEP 13 — Access Application

Open:

http://localhost

Application should now load successfully.

🧠 Important Notes
Container Communication

Containers communicate using Docker network.

DB_HOST must match MySQL container name.

Port mapping for MySQL is NOT required for internal communication.

Passwords Are Case-Sensitive

MySQL credentials must exactly match .env.

🛠 Useful Commands
Check running containers
docker ps
View container logs
docker logs truelysell-app
docker logs truelysell-mysql
Stop containers
docker stop truelysell-app truelysell-mysql
