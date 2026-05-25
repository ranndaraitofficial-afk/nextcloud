# Nextcloud Self-Hosting Setup on Ubuntu Server

This guide explains how to install and configure a self-hosted Nextcloud server on Ubuntu using Apache, MariaDB, and PHP.

---

## Requirements

### Minimum (Lab/Test)
- 2 CPU cores
- 4 GB RAM
- 40 GB storage

### Recommended
- 4 CPU cores
- 8+ GB RAM
- SSD storage

---

## 1. Update System

sudo apt update && sudo apt upgrade -y

---

## 2. Install Apache

sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

Test in browser:
http://SERVER_IP

---

## 3. Install MariaDB

sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb

Secure it:
sudo mysql_secure_installation

Recommended answers:
- Set root password: YES
- Remove anonymous users: YES
- Disallow remote root login: YES
- Remove test database: YES
- Reload privileges: YES

---

## 4. Create Database

sudo mysql

CREATE DATABASE nextcloud;

CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'StrongPassword123!';

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';

FLUSH PRIVILEGES;
EXIT;

---

## 5. Install PHP and Dependencies

sudo apt install php libapache2-mod-php php-gd php-json php-mysql php-curl php-mbstring php-intl php-imagick php-xml php-zip php-bz2 php-cli php-common php-apcu php-redis php-gmp unzip wget -y

Check version:
php -v

---

## 6. Download Nextcloud

cd /tmp
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
sudo mv nextcloud /var/www/

---

## 7. Set Permissions

sudo chown -R www-data:www-data /var/www/nextcloud
sudo chmod -R 755 /var/www/nextcloud

---

## 8. Configure Apache

sudo nano /etc/apache2/sites-available/nextcloud.conf

Paste:

<VirtualHost *:80>
    ServerName your-domain.com

    DocumentRoot /var/www/nextcloud

    <Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>

Enable config:

sudo a2enmod rewrite headers env dir mime ssl
sudo a2ensite nextcloud.conf
sudo a2dissite 000-default.conf
sudo systemctl restart apache2

---

## 9. Firewall Setup

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status

---

## 10. Install SSL (Let’s Encrypt)

sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache

---

## 11. Finish Setup in Browser

Open:

https://your-domain.com

Enter:
- Admin username
- Admin password

Database:
- DB User: nextclouduser
- DB Password: StrongPassword123!
- DB Name: nextcloud
- DB Host: localhost

---

## 12. Cron Job (Important)

sudo crontab -u www-data -e

Add:

*/5 * * * * php -f /var/www/nextcloud/cron.php

---

## 13. PHP Optimization

sudo nano /etc/php/*/apache2/php.ini

Change:

memory_limit = 512M
upload_max_filesize = 2G
post_max_size = 2G
max_execution_time = 360

Restart Apache:
sudo systemctl restart apache2

---

## Optional Improvements

- Redis caching
- External storage (NAS / HDD)
- Reverse proxy setup
- Automated backups
- LDAP/Active Directory integration
- Docker deployment

---

## Architecture Example

Internet
   |
pfSense
   |
Core Switch
   |
Ubuntu Nextcloud Server

---

## Useful Commands

sudo systemctl status apache2
sudo systemctl status mariadb
sudo systemctl restart apache2
sudo tail -f /var/log/apache2/error.log

---

## Official Resources

https://nextcloud.com  
https://docs.nextcloud.com  
https://ubuntu.com/server/docs  
