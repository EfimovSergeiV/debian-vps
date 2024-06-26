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



# Наводим красоту
```bash
sudo hostnamectl set-hostname mail.your-domain.com
sudo nano /etc/hosts
127.0.0.1       mail.your-domain.com localhost
hostname -f

```


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


# PostgreSQL
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
# или
ALTER DATABASE name_db OWNER TO name_user;

\q

.....
su - postgres
createdb --encoding UNICODE dbms_db --username postgres
exit
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
```
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt install nodejs
```

# PM2
Запуск VueJS приложения:
```
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


# ElasticSearch with Kibana

```bash
sudo systemctl start elasticsearch

cd /usr/share/elasticsearch/bin # ElasticSearch applications
sudo nano /etc/elasticsearch/elasticsearch.yml
sudo service elasticsearch start
sudo service elasticsearch status
```


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


### Установка сервера Outline

```bash
# Install docker https://docs.docker.com/engine/install/debian/#install-using-the-repository
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
```
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# Install outline (link from Outline Manager)
sudo bash -c "$(wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh)"

#Make sure to open the following ports on your firewall, router or cloud provider:
#- Management port 23986, for TCP
#- Access key port 10029, for TCP and UDP
sudo ufw allow 23986/tcp
sudo ufw allow 10029/tcp
sudo ufw allow 10029/udp
```




## Mattermost

```text
Настройка Postgresql
https://docs.mattermost.com/install/prepare-mattermost-database.html
Установка на дебиан
https://docs.mattermost.com/install/install-debian.html
Корректируем из мана под юбунту
https://docs.mattermost.com/install/install-ubuntu.html
Настройка прокси NGINX
https://docs.mattermost.com/install/setup-nginx-proxy.html
(Местный сертбот из этого мана потом юзаем)
Upgrade mm тоже по этому ману успешный
https://docs.mattermost.com/upgrade/upgrading-mattermost-server.html
```


```bash
psql (16.2 (Debian 16.2-1.pgdg120+2))
Type "help" for help.

postgres=# CREATE DATABASE mattermost;
CREATE DATABASE
postgres=# CREATE USER mmuser WITH PASSWORD 'mmuser-password';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE mattermost to mmuser;
GRANT
postgres=# ALTER DATABASE mattermost OWNER TO mmuser;
ALTER DATABASE
postgres=# GRANT USAGE, CREATE ON SCHEMA PUBLIC TO mmuser;
GRANT
postgres=# \q

#######
sudo systemctl restart postgresql-{version}
```


```bash
/etc/postgresql/{version}/main/postgresql.conf
# Find the following line: #listen_addresses = 'localhost'
# Uncomment the line and change localhost to *: listen_addresses = '*'
# Restart PostgreSQL for the change to take effect by running:
sudo systemctl restart postgresql-{version}




/etc/postgresql/{version}/main/pg_hba.conf


local   all             all                        peer

host    all             all         ::1/128        ident

###

local   all             all                        trust

host    all             all         ::1/128        trust


sudo systemctl reload postgresql-{version}
psql --dbname=mattermost --username=mmuser --password

```


```bash
wget https://releases.mattermost.com/9.5.3/mattermost-9.5.3-linux-amd64.tar.gz 
tar -xvzf mattermost*.gz
sudo mv mattermost /opt

sudo mkdir /opt/mattermost/data
sudo useradd --system --user-group mattermost
sudo chown -R mattermost:mattermost /opt/mattermost
sudo chmod -R g+w /opt/mattermost
sudo touch /lib/systemd/system/mattermost.service
sudo nano /lib/systemd/system/mattermost.service
sudo cp /opt/mattermost/config/config.json /opt/mattermost/config/config.defaults.json
sudo nano /opt/mattermost/config/config.json                                          

sudo systemctl start mattermost
```
