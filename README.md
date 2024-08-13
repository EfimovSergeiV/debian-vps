# Настройка VPS для Django/Nuxt

+ Настройка системы
  + Создание пользователя
  + Настройка шелл
  + Установка зависимостей
+ PostgreSQL
  + Установка из репозитория
  + Создание пользователя и базы данных
  + Восстановление из дампа
  + Создание резервного дампа 
  + Установка панели управления ( серверная )
+ MySQL
  + Установка из репозитория
  + Создание пользователя и базы данных
  + Восстановление из дампа
  + Создание резервного дампа 
+ Django
  + Создание виртуальной среды
  + Установка зависимостей пректа
  + Сбор статики и создание директорий
+ Gunicorn
  + Конфигурирование сокета
  + Запуск и управление
+ NodeJS
  + Установка сервера
  + Установка зависимостей и сборка проекта
+ PM2
  + Запуск и управление проектом
+ Nginx
  + Конфигурация для Django
  + Конфигурация для Nuxt
  + Управление
+ Firewall (UFW)
  + Установка и настройка
  + Управление


## Настройка пользователя и окружения

### Создание нового пользователя:
```
adduser anon
usermod -aG sudo anon
su anon
```

### Запрещаем root по ssh:
```
sudo nano /etc/ssh/sshd_config
    PermitRootLogin no

sudo service ssh restart
```

### Установка зависимостей:
```
sudo apt install python3-venv python3-pip python3-dev libpq-dev build-essential
```


## PostgreSQL

### Установка: 
> https://www.postgresql.org/download/linux/debian/

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql

adduser postgres sudo
```

### Восстановление миграций Django
```bash
python manage.py dbshell
TRUNCATE django_migrations;
python manage.py migrate --fake
```


### Создание базы данных и пользователя:
```
sudo -u postgres psql

create database name_db;
create user name_user with password 'password';
alter role name_user set client_encoding TO 'utf8';
alter role name_user set default_transaction_isolation to 'read committed';
alter role name_user set timezone to 'UTC';
grant all PRIVILEGES ON DATABASE name_db to name_user;

\q
```

### Создание/Восстановление данных из дампа:
```
sudo -u postgres pg_dump database > db.sql
sudo -u postgres psql -d newdatabase -f db.sql
sudo -u postgres psql -d glsvar_db -f 23-06-2023.sql
```

## Разворачивание Django приложения

### Клонирование и установка зависимостей приложения:
```
git clone https://github.com/login/repository.git
cd repository
python3 -m venv venv
source venv/bin/activate
```

### Создание/Восстановление данных из JSON:


====================================================================================================





## Установка и настройка Firewall (UFW)

> https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-debian-9-ru
```
sudo apt install ufw

sudo ufw app list  # Список профилей приложений
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'


sudo ufw enable
sudo ufw status
sudo ufw status verbose
```











Установка пакетов:
```
sudo apt install nginx python3-venv python3-pip python3-dev libpq-dev build-essential
```





```
sudo nano /etc/default/ufw
    IPV6=yes

sudo ufw delete allow 80
sudo ufw allow ssh or sudo ufw allow 22
sudo ufw allow http or sudo ufw allow 80 
sudo ufw allow https or sudo ufw allow 443
sudo ufw enable
sudo ufw status verbose
```


<!-- # PostgreSQL
Установка: https://www.postgresql.org/download/linux/debian/ 
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql

pip3 install psycopg2-binary
```
Настройка:
```
sudo -u postgres psql

CREATE DATABASE home_project;
CREATE USER home_project WITH PASSWORD 'password';

ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE myprojectuser SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
\q

.....
su - postgres
createdb --encoding UNICODE dbms_db --username postgres
exit
``` -->

# PostgreSQL

Manual: https://www.postgresql.org/download/linux/debian/

```bash
sudo apt install postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt install postgresql
```

```bash
sudo -u postgres psql

create database name_db;
create user name_user with password 'password';
alter role name_user set client_encoding TO 'utf8';
alter role name_user set default_transaction_isolation to 'read committed';
alter role name_user set timezone to 'UTC';
# grant all PRIVILEGES ON DATABASE name_db to name_user;
ALTER DATABASE name_db OWNER TO name_user;

\q
```


# ElasticSearch with Kibana

```bash
# sudo systemctl start elasticsearch

# cd /usr/share/elasticsearch/bin # ElasticSearch applications
# sudo nano /etc/elasticsearch/elasticsearch.yml

# sudo nano /etc/elasticsearch/jvm.options

# sudo service elasticsearch start
# sudo service elasticsearch status
```

```bash
pip install elasticsearch-dsl
pip install django-elasticsearch-dsl


sudo nano /etc/elasticsearch/elasticsearch.yml

# Enable security features
xpack.security.enabled: false



/etc/elasticsearch/jvm.options

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms128m
-Xmx128m

sudo systemctl restart elasticsearch.service
sudo systemctl enable elasticsearch.service
```



# MySQL
Установка из репозитория: 
```
sudo apt install python-dev default-libmysqlclient-dev 

wget https://dev.mysql.com/get/mysql-apt-config_0.8.18-1_all.deb
https://dev.mysql.com/get/Downloads/Connector-Python/mysql-connector-python-py3_8.0.26-1debian10_amd64.deb

sudo apt install ./mysql-apt-config_0.8.18-1_all.deb

sudo apt install mysql-server

sudo apt install ./mysql-connector-python-py3_8.0.26-1debian10_amd64.deb
```
Настрока:
```
mysql_secure_installation
```
Создание базы и восстановление из дампа:
```
# Создание базы данных и пользователя 
mysql -u root -p
create database dbase_name;

create user 'db_user'@'localhost' identified by 'pass';
GRANT ALL PRIVILEGES ON dbase_name. * to 'db_user'@'localhost';

# Восстановление из дампа
mysql -u db_user -p
use database_name;
source backup/dbase_name.sql

# Удаление базы данных
DROP DATABASE productsdb;
```



# Gunicorn
Установка:
```
sudo apt edit-sources
    deb http://ftp.debian.org/debian buster-backports main
    deb-src http://ftp.debian.org/debian buster-backports main
sudo apt update
sudo apt -t buster-backports install gunicorn
```
Настройка:
```
gunicorn --bind 0.0.0.0:8000 main.wsgi
sudo service gunicorn restart

sudo nano /etc/systemd/system/gunicorn.socket
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target


sudo nano /etc/systemd/system/gunicorn.service
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application

[Install]
WantedBy=multi-user.target


sudo systemctl daemon-reload
sudo systemctl enable gunicorn.service

```

# NGINX
Настройка
```
sudo nano /etc/nginx/sites-available/myproject.conf
sudo ln -s /etc/nginx/sites-available/myproject.conf /etc/nginx/sites-enabled
sudo nginx -t

sudo service nginx restart

sudo ufw allow 'Nginx Full'
```

Конфигурация для Django + Gunicorn + Nginx
```
server {
    listen 80;
    server_name sitename.ru www.sitename.ru;
    
    client_max_body_size 3G;

    location = /favicon.ico {
        alias /home/anon/cw-dapp/files/favicon.ico;
    }

    location = /robots.txt {  
        alias /home/anon/cw-dapp/files/robots.txt;
    }

    location /static/ {
        root /home/anon/szwei-django;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

Конфигурация для NuxtJS
```
map $sent_http_content_type $expires {
    "text/html"                 epoch;
    "text/html; charset=utf-8"  epoch;
    default                     off;
}

server {
    listen          45.67.56.50:80;             # the port nginx is listening on
    server_name     glsvar.ru www.glsvar.ru;    # setup your domain here

    gzip            on;
    gzip_types      text/plain application/xml text/css application/javascript;
    gzip_min_length 1000;

    location / {
        expires $expires;

        proxy_redirect                      off;
        proxy_set_header Host               $host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto  $scheme;
        proxy_read_timeout          1m;
        proxy_connect_timeout       1m;
        proxy_pass                          http://localhost:3000; # set the address of the Node.js instance here
    }
}

```

```txt
#map $sent_http_content_type $expires {
#    "text/html"                 epoch;
#    "text/html; charset=utf-8"  epoch;
#    default                     off;
#}

map $sent_http_content_type $expires {
    "text/html"                 1h;
    "text/html; charset=utf-8"  1h;
    default                     1h;
}


server {             # the port nginx is listening on
    server_name     glsvar.ru www.glsvar.ru;    # setup your domain here

    gzip            on;
    gzip_types      text/plain application/xml text/css application/javascript;
    gzip_min_length 1000;

    location / {
        expires $expires;

        proxy_redirect                      off;
        proxy_set_header Host               $host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto  $scheme;
        proxy_read_timeout          1m;
        proxy_connect_timeout       1m;
        proxy_pass                          http://localhost:3001; # set the address of the Node.js instance here
    }




    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.glsvar.ru-0001/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.glsvar.ru-0001/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot





}



server {
    if ($host = glsvar.ru) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = www.glsvar.ru) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name     glsvar.ru www.glsvar.ru;
    listen 80;
    return 404; # managed by Certbot




}

```




# Certbot SSL
Let’s Encrypt: 
```
sudo apt install python3-acme python3-certbot python3-mock python3-openssl python3-pkg-resources python3-zope.interface python3-pyparsing
sudo apt install python3-certbot-nginx
sudo certbot --nginx -d your_domain -d www.your_domain
```


# Supervisor



# NodeJS
Установка:
```bash
# curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
# sudo bash nodesource_setup.sh
# sudo apt install nodejs




# installs nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# download and install Node.js (you may need to restart the terminal)
nvm install 20
# verifies the right Node.js version is in the environment
node -v # should print `v20.16.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.1`
```

# PNPM
```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
# or
wget -qO- https://get.pnpm.io/install.sh | sh -
```


# PM2
Запуск VueJS приложения:
```bash
sudo npm install pm2 -g
pm2 start npm -- start
pm2 restart 0
```


# Управление Django проектом
Создание/Активация/Деактивация виртуального окружения
```
python3 -m venv venv
source venv/bin/activate
deactivate
```

# Управление VueJS проектом



# WireGuard VPN
# https://github.com/Nyr/wireguard-install решение попроще

```bash
sudo apt install wireguard
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
sudo chmod 600 /etc/wireguard/privatekey

ip a

sudo cat /etc/wireguard/privatekey

sudo nano /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <privatekey>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1

sudo sysctl -p

sudo ufw allow 51820/udp

sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
sudo systemctl status wg-quick@wg0.service

wg genkey | sudo tee /etc/wireguard/anon_privatekey | wg pubkey | sudo tee /etc/wireguard/anon_publickey

sudo nano /etc/wireguard/wg0.conf
[Peer]
PublicKey = <anon_publickey>
AllowedIPs = 10.0.0.2/32

sudo systemctl restart wg-quick@wg0 
sudo systemctl status wg-quick@wg0


CLIENT:

nano anon_wb.conf
[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = 10.0.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER-PUBKEY>
Endpoint = <SERVER-IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20

```


### Установка сервера outline

```text
https://github.com/Jigsaw-Code
https://gist.github.com/JohnyDeath/3f93899dc78f90cc57ae52b41ea29bac
```

```bash
sudo curl https://get.docker.com | sh
sudo wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh | sudo bash

sudo ufw allow 39885/tcp
sudo ufw allow 1586/tcp
sudo ufw allow 1586/udp

=================================================================================
If you have connection problems, it may be that your router or cloud provider
blocks inbound connections, even though your machine seems to allow them.

Make sure to open the following ports on your firewall, router or cloud provider:
- Management port 33055, for TCP
- Access key port 62162, for TCP and UDP

```
#### ОФФ
```bash
sudo bash -c "$(wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh)"
```