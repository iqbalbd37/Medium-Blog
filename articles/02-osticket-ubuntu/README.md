# How to Install osTicket on Ubuntu 22.04

![osTicket](https://img.shields.io/badge/osTicket-FF6B35?style=for-the-badge&logo=ticket&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)

## Overview

A complete step-by-step guide to installing osTicket on Ubuntu 22.04, transforming your customer support workflow with a professional, enterprise-grade ticketing system.

**Read on Medium:** [Click Here](https://medium.com/@iqbalbd37/how-to-install-osticket-on-ubuntu-22-04-complete-step-by-step-guide-dbf0d056852f)

---

## What is osTicket?

osTicket is a powerful, open-source ticketing system that helps businesses manage, organize, and track customer support requests efficiently. Unlike expensive commercial solutions, osTicket offers enterprise-grade features completely free of charge.

### Key Features

- **Custom Fields & Forms**: Collect exactly the information you need from customers
- **Smart Routing**: Automatically direct tickets to the right department or agent
- **SLA Management**: Set and track service level agreements
- **Agent Collision Prevention**: Prevent multiple agents from working on the same ticket
- **Advanced Search**: Find tickets quickly with powerful filtering options
- **Email Integration**: Convert emails into tickets automatically
- **Task Management**: Create internal tasks for your team

---

## Prerequisites

- Ubuntu 22.04 server with root access
- A domain name pointed to your server (optional but recommended)
- Basic familiarity with Linux commands

---

## Table of Contents

1. [Step 1: System Update and Preparation](#step-1-system-update-and-preparation)
2. [Step 2: Install and Secure MariaDB Database](#step-2-install-and-secure-mariadb-database)
3. [Step 3: Install Apache Web Server](#step-3-install-apache-web-server)
4. [Step 4: Install PHP 8.2 and Required Extensions](#step-4-install-php-82-and-required-extensions)
5. [Step 5: Download osTicket](#step-5-download-osticket)
6. [Step 6: Configure osTicket](#step-6-configure-osticket)
7. [Step 7: Configure Apache Virtual Host](#step-7-configure-apache-virtual-host)
8. [Step 8: Web-Based Installation](#step-8-web-based-installation)
9. [Step 9: Post-Installation Security](#step-9-post-installation-security)
10. [Step 10: Access Your Helpdesk](#step-10-access-your-helpdesk)

---

## Step 1: System Update and Preparation

```bash
sudo apt update && sudo apt upgrade -y
```

If prompted for reboot:

```bash
sudo reboot
```

Set hostname:

```bash
sudo hostnamectl set-hostname support.yourdomain.com
```

Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add:

```
192.168.1.100 support.yourdomain.com
```

---

## Step 2: Install and Secure MariaDB Database

Install MariaDB:

```bash
sudo apt install mariadb-server -y
```

Secure installation:

```bash
sudo mysql_secure_installation
```

Create the osTicket database:

```bash
sudo mysql -u root -p
```

Run these SQL commands:

```sql
CREATE DATABASE osticket_db;
GRANT ALL PRIVILEGES ON osticket_db.* TO 'osticket_user'@'localhost' IDENTIFIED BY 'your_strong_password';
FLUSH PRIVILEGES;
EXIT;
```

---

## Step 3: Install Apache Web Server

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

---

## Step 4: Install PHP 8.2 and Required Extensions

Add the PHP repository:

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update -y
```

Install PHP 8.2 and extensions:

```bash
sudo apt install -y php8.2 php8.2-cli php8.2-common \
  php8.2-mysql php8.2-gd php8.2-imap php8.2-intl \
  php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip \
  php8.2-apcu libapache2-mod-php8.2 php8.2-bcmath
```

Verify:

```bash
php -v
```

---

## Step 5: Download osTicket

Install tools:

```bash
sudo apt install curl wget unzip jq -y
```

Download latest osTicket:

```bash
wget "$(curl -s https://api.github.com/repos/osTicket/osTicket/releases/latest \
  | jq -r '.assets[0].browser_download_url')"
```

Extract:

```bash
unzip osTicket-v*.zip -d osTicket
sudo mv osTicket /var/www/
```

---

## Step 6: Configure osTicket

Create configuration file:

```bash
cd /var/www/osTicket/upload/include
sudo cp ost-sampleconfig.php ost-config.php
```

Set permissions:

```bash
sudo chown -R www-data:www-data /var/www/osTicket
sudo find /var/www/osTicket -type d -exec chmod 755 {} \;
sudo find /var/www/osTicket -type f -exec chmod 644 {} \;
sudo chmod 664 /var/www/osTicket/upload/include/ost-config.php
```

---

## Step 7: Configure Apache Virtual Host

Disable default site:

```bash
sudo a2dissite 000-default.conf
```

Create virtual host:

```bash
sudo nano /etc/apache2/sites-available/osticket.conf
```

Add:

```apache
<VirtualHost *:80>
    ServerAdmin admin@yourdomain.com
    DocumentRoot /var/www/osTicket/upload
    ServerName support.yourdomain.com

    <Directory /var/www/osTicket/upload>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/osticket_error.log
    CustomLog ${APACHE_LOG_DIR}/osticket_access.log combined
</VirtualHost>
```

Enable site:

```bash
sudo a2ensite osticket.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## Step 8: Web-Based Installation

Open your browser and navigate to:

```
http://support.yourdomain.com
```

Fill in:

- **System Settings**: Your helpdesk name and default email
- **Admin User**: Create your administrator account
- **Database Settings**: Enter the database details we created earlier

Click "Install Now".

---

## Step 9: Post-Installation Security

1. Remove write permissions:

```bash
sudo chmod 644 /var/www/osTicket/upload/include/ost-config.php
```

2. Remove setup directory:

```bash
sudo rm -rf /var/www/osTicket/upload/setup/
```

3. Set proper permissions:

```bash
sudo chown -R www-data:www-data /var/www/osTicket
sudo find /var/www/osTicket -type d -exec chmod 755 {} \;
sudo find /var/www/osTicket -type f -exec chmod 644 {} \;
```

---

## Step 10: Access Your Helpdesk

- **Staff Login**: `http://support.yourdomain.com/scp/login.php`
- **Customer Portal**: `http://support.yourdomain.com`

---

## Tags

`#osTicket` `#Ubuntu` `#Helpdesk` `#DevOps` `#Linux` `#PHP` `#MariaDB` `#Tutorial`

---

[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@iqbalbd37)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/iqbalbd37)