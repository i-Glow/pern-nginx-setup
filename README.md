# Debian server setup

This setup uses the combination of [Nginx](https://nginx.org/en/), [Nodejs](https://nodejs.org/en), [React](https://react.dev/), [PostgreSQL](https://www.postgresql.org/), [Certbot](https://certbot.eff.org/)

Update the packages manager

```bash
sudo apt update
```

## Install Git

Install Git

```bash
sudo apt install git
```

## To install Node JS (Using the Node Version Manager NVM)

<br>

### Install Node js

first install curl

```bash
sudo apt install curl
```

add the NVM github repository

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

reload bash conf

```bash
source ~/.bashrc
```

list and look for the right nodejs version

```bash
nvm list-remote
```

install the chosen version

```bash
nvm install v20.10.0
```

### Run Nodejs Application Server

\
Install pm2

```bash
sudo apt install pm2
```

Start nodejs Application Server

```bash
pm2 start /link_to_project/app.js
```

## Install a DBMS

### PostgreSQL

\
Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib
```

Open PSQL

```bash
sudo -u postgres psql
```

to change password for postgres user

```bash
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'your_new_password';"
```

## NGINX Web Server

Install Nginx

```bash
sudo apt install nginx
```

Edit the Nginx configuration

```bash
sudo nano /etc/nginx/sites-available/your_domain
```

```nginx
# API Application server
upstream app {
  server http://localhost:PORT;
  keepalive 64;
}

server {
  listen 80;
  listen [::]:80;

  # location of the web page folder
  root /var/www/your_domain/html;
  # name of default web pages
  index index.html index.htm index.nginx-debian.html;

  # this sets the payload size of incoming requests
  client_max_body_size 20M;

  server_name your_domain www.your_domain;

  location /images {
    root /home/your_project/server/public;
  }

  # Proxy setup for HTTP 1.1 APIs
  location /api {
    proxy_pass app;
    proxy_set_header Host $host;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_cache_bypass $http_updgrade;
  }

  # Setup for CSR Apps
  location / {
    root /var/www/your_domain/html;
    try_files $uri $uri/ /index.html;
  }
}
```

A cleaner nginx configuration file:

```nginx
upstream app {
  server http://localhost:PORT;
  keepalive 64;
}

server {
  listen 80;
  listen [::]:80;

  root /var/www/your_domain/html;
  index index.html index.htm index.nginx-debian.html;

  client_max_body_size 20M;

  server_name your_domain www.your_domain;

  location /images {
    root /home/your_project/server/public;
  }

  location /api {
    proxy_pass app;
    proxy_set_header Host $host;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_cache_bypass $http_updgrade;
  }

  location / {
    root /var/www/your_domain/html;
    try_files $uri $uri/ /index.html;
  }
}
```

Create a symbolic link to the configuration

```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Check The configuration syntax using

```bash
sudo nginx -t
```

Restart nginx

```bash
sudo systemctl restart nginx
```

## Generate SSL Certificate (Using Certbot)

Intall Certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

Make sure your nginx configuration is correct

```bash
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
...
server_name example.com www.example.com;
...
```

Obtain an SSL Certificat

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Setup Auto-renewal

```bash
sudo systemctl status certbot.timer
```

Test the renewal process

```bash
sudo certbot renew --dry-run
```

## Make pm2 startup on system reboot

type the following command and follow the instructions given

```bash
pm2 startup
```

then save the current app list using

```bash
pm2 save
```

