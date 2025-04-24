# Laravel-Server-Setup-and-Configuration

# Laravel & React Server Setup and Configuration Tutorial With PHP8.3, Nginx, MariaDB & Node.js

This tutorial will guide you through setting up a server with Nginx, PHP 8.3, MariaDB, Node.js (using NVM), and deploying a Git repository. We will also configure SSL and install necessary packages.

## Prerequisites

- A server with Ubuntu installed.
- Root access to the server.
- Basic knowledge of SSH and command line operations.

## Step-by-Step Instructions

### 1. Initial Server Configuration

1. **Login as root**:
    ```bash
    ssh root@152.42.168.27
    ```

2. **Create Users**:
    ```bash
    adduser webrana
    passwd webrana
    ```

3. **Add users to the sudo group**:
    ```bash
    usermod -aG sudo webrana
    ```

4. **Setup UFW (Uncomplicated Firewall)**:
    ```bash
    sudo ufw status
    sudo ufw allow openssh
    sudo ufw allow www
    sudo ufw allow https
    sudo ufw enable
    ```

### 2. Setup Nginx

1. **Install Nginx**:
    ```bash
    sudo apt update
    sudo apt install nginx
    ```

2. **Create Nginx configuration**:

    [Nginx Conf](WebranaConf.md)

    ```bash
    sudo ln -s /etc/nginx/sites-available/webrana.conf /etc/nginx/sites-enabled/
    sudo mkdir /var/www/webrana
    ```

    ```bash
    sudo ln -s /etc/nginx/sites-available/nusadaya.conf /etc/nginx/sites-enabled/
    sudo mkdir /var/www/nusadaya
    ```

### 3. Install SSL with Certbot

1. **Install Certbot**:
    ```bash
    sudo apt install certbot python3-certbot-nginx
    ```

2. **Obtain SSL Certificate**:
    ```bash
    sudo certbot --nginx
    ```

### 4. Install PHP 8.3

1. **Add PHP repository**:
    ```bash
    sudo add-apt-repository ppa:ondrej/php -y
    sudo apt update
    ```

2. **Install PHP 8.3 and required extensions**:
    ```bash
    sudo apt install php8.3 php8.3-cli php8.3-common php8.3-curl php8.3-pgsql php8.3-fpm php8.3-gd php8.3-imap php8.3-intl php8.3-mbstring php8.3-mysql php8.3-opcache php8.3-readline php8.3-soap php8.3-xml php8.3-zip php8.3-fileinfo
    ```

### 5. Install MariaDB

1. **Add MariaDB repository**:
    ```bash
    sudo apt-get install apt-transport-https curl
    sudo mkdir -p /etc/apt/keyrings
    sudo curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'
    ```

2. **Create MariaDB source list**:
    ```bash
    cd /etc/apt/sources.list.d
    sudo vi mariadb.sources
    ```
    Add the following content:
    ```
    # MariaDB 11.4 repository list - created 2024-06-17 03:59 UTC
    # https://mariadb.org/download/
    X-Repolib-Name: MariaDB
    Types: deb
    # deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
    URIs: https://download.nus.edu.sg/mirror/mariadb/repo/11.4/ubuntu
    Suites: jammy
    Components: main main/debug
    Signed-By: /etc/apt/keyrings/mariadb-keyring.pgp
    ```

3. **Install MariaDB**:
    ```bash
    sudo apt-get update
    sudo apt-get install mariadb-server
    ```

4. **Secure MariaDB installation**:
    ```bash
    sudo mysql_secure_installation
    ```

5. **Configure MariaDB**:
    ```bash
    sudo mysql -u root -p
    ```
    Enter the following SQL commands:
    ```sql
    GRANT ALL ON *.* TO root@localhost IDENTIFIED BY '13799454';
    FLUSH PRIVILEGES;

    CREATE DATABASE webrana;

    GRANT ALL ON webrana.* TO webrana@localhost IDENTIFIED BY '13799454';
    ```

    ```sql
    CREATE DATABASE nusadaya;

    GRANT ALL ON nusadaya.* TO nusadaya@localhost IDENTIFIED BY '13799454';
    ```

### 6. Add Git Repository

1. **Install Git**:
    ```bash
    sudo apt install git
    ```

2. **Install Composer**:
    ```bash
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    ```

3. **Setup repository structure**:
    ```bash
    sudo chown webrana:www-data /var/www/webrana
    sudo chown juju:www-data /var/www/jujujoki

    cd /var
    sudo mkdir repo
    sudo mkdir repo/{name}.git
    cd repo/{name}.git
    sudo git init --bare
    cd hooks
    sudo vi post-receive
    ```

     ```bash
    sudo chown nusadaya:www-data /var/www/nusadaya
    ```
    Add the following content:
    ```sh
    #!/bin/sh
    git --work-tree=/var/www/{foldername} --git-dir=/var/repo/{name}.git checkout -f main
    ```

4. **Set permissions**:
    ```bash
    sudo chmod +x post-receive
    sudo chown -R {user}:{user} /var/repo/{name}.git
    ```

### 7. Deploy from Git

1. **Configure SSH agent** (in PhpStorm or terminal):
    ```bash
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
    ```

2. **Add Git remote**:
    ```bash
    git remote add production webrana@webrana.my.id:/var/repo/site.git
    git remote add productionjujujoki jujujoki@68.183.180.75:/var/repo/jujujoki.git
    ```

3. **Push to production**:
    ```bash
    git push production
    ```

### 8. Configure Laravel Application

1. **Set directory permissions**:
    ```bash
    sudo chown -R :www-data /var/www/webrana
    sudo chmod -R gu+w /var/www/webrana/storage/
    sudo chmod -R guo+w /var/www/webrana/storage/
    sudo chmod -R gu+w /var/www/webrana/bootstrap/cache/
    sudo chmod -R guo+w /var/www/webrana/bootstrap/cache/
    ```

    ```bash
    sudo chown -R :www-data /var/www/nusadaya
    sudo chmod -R gu+w /var/www/nusadaya/storage/
    sudo chmod -R guo+w /var/www/nusadaya/storage/
    sudo chmod -R gu+w /var/www/nusadaya/bootstrap/cache/
    sudo chmod -R guo+w /var/www/nusadaya/bootstrap/cache/
    ```

2. **Install dependencies and configure Laravel**:
    ```bash
    cd /var/www/webrana
    composer install --no-dev
    php artisan key:generate
    php artisan config:cache
    php artisan config:clear
    php artisan view:clear
    ```

### 9. Install Node.js using NVM
[Link](https://nodejs.org/en/download/package-manager)

1. **Install NVM (Node Version Manager)**:
    ```bash
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    ```

2. **Activate NVM**:
    ```bash
    export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
    ```

3. **Install Node.js**:
    ```bash
    nvm install node
    ```

4. **Verify installation**:
    ```bash
    node -v
    npm -v
    ```

Your server is now configured with Nginx, PHP 8.3, MariaDB, Node.js, SSL, and ready to deploy applications using Git.
