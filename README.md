# Server Provisioning and Setup Guide (Ubuntu 22.04)

This guide outlines the steps to provision an Ubuntu 22.04 server with Nginx, Node.js, Yarn, Git, PM2, and Certbot. Additionally, it includes configuring a reverse proxy using Nginx and setting up SSL with auto-renewal.

## Table of Contents
- [Install Nginx](#install-nginx)
- [Install NVM and Node.js](#install-nvm-and-nodejs)
- [Install Yarn](#install-yarn)
- [Install Git](#install-git)
- [Install PM2](#install-pm2)
- [Install Certbot](#install-certbot)
- [Auto Renewal of SSL Certificate](#auto-renewal-of-ssl-certificate)
- [Configure Nginx Reverse Proxy](#configure-nginx-reverse-proxy)
- [Obtain SSL Certificate](#obtain-ssl-certificate)

## Install Nginx
1. Login via SSH:
    ```sh
    sudo su
    ```
2. Update and upgrade the system:
    ```sh
    sudo apt update && sudo apt upgrade
    ```
3. Install Nginx:
    ```sh
    sudo apt install nginx
    ```
4. Verify Nginx installation:
    ```sh
    nginx -v
    ```
5. Configure firewall to allow Nginx and OpenSSH:
    ```sh
    sudo ufw allow 'Nginx Full'
    sudo ufw allow OpenSSH
    sudo ufw enable
    sudo ufw status
    ```
6. Start and enable Nginx service:
    ```sh
    sudo systemctl start nginx
    sudo systemctl status nginx
    sudo systemctl enable nginx
    ```

## Install NVM and Node.js
1. Install NVM (per March 2023):
    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    ```
    [Reference](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-22-04)
2. Load NVM:
    ```sh
    source ~/.bashrc
    ```
3. Install Node.js:
    ```sh
    nvm install 16.13.1
    nvm install --lts
    ```
4. Verify Node.js installation:
    ```sh
    node -v
    ```

## Install Yarn
1. Install Yarn globally using npm:
    ```sh
    npm install -g yarn
    ```
    [Reference](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-the-yarn-package-manager-for-node-js)
2. Verify Yarn installation:
    ```sh
    yarn -v
    ```

## Install Git
1. Check if Git is already installed:
    ```sh
    git --version
    ```
2. Install Git with default packages:
    ```sh
    sudo apt install git
    ```
    [Reference](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-22-04)
3. Verify Git installation:
    ```sh
    git --version
    ```

## Install PM2
1. Install PM2 globally using npm:
    ```sh
    npm install -g pm2
    ```
    [Reference](https://www.tutsmake.com/how-to-install-pm2-in-ubuntu-22-04/)
2. Verify PM2 installation:
    ```sh
    pm2 -v
    ```

## Install Certbot
1. Install Certbot and its Nginx plugin:
    ```sh
    sudo apt install certbot python3-certbot-nginx
    ```
## Obtain SSL Certificate
1. Run Certbot to obtain an SSL certificate:
    ```sh
    sudo certbot --nginx -d domainname.com
    ```
2. Restart Nginx:
    ```sh
    sudo systemctl restart nginx
    ```
3. Restart the application with PM2:
    ```sh
    pm2 restart app_name
    ```

## Auto Renewal of SSL Certificate
1. Open crontab to schedule auto-renewal:
    ```sh
    sudo crontab -e
    ```
2. Insert the following line to renew the certificate on the 1st of every month:
    ```cron
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    0 0 1 * * certbot renew >> /logs/certbot-cron.log 2>&1
    ```
    [Reference](https://crontab.guru/#0_0_1_*_*)
3. Save and exit the crontab editor.
4. Restart Nginx:
    ```sh
    sudo systemctl restart nginx
    ```


## Configure Nginx Reverse Proxy
1. Navigate to the Nginx sites-available directory:
    ```sh
    cd /etc/nginx/sites-available
    ```
2. Create and edit a new server block file:
    ```sh
    vim app_name
    ```
3. Insert the following server block configuration:
    ```nginx
    server {
        listen 80;
        server_name domainname.com;

        location / {
            proxy_pass http://127.0.0.1:PORT;
        }
    }
    ```
4. Save and exit the file.
5. Create a symbolic link to enable the site:
    ```sh
    sudo ln -s /etc/nginx/sites-available/app_name /etc/nginx/sites-enabled/app_name
    ```
6. Test Nginx configuration:
    ```sh
    sudo nginx -t
    ```
7. Backup and remove the default site configuration:
    ```sh
    cp default default.bak
    rm default
    ```
8. Restart Nginx:
    ```sh
    sudo systemctl restart nginx
    ```

