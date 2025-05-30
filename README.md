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

```bash
# SET HOSTNAME

sudo hostnamectl set-hostname mail.your-domain.com
sudo nano /etc/hosts
127.0.0.1       mail.your-domain.com localhost
hostname -f

```


### Создание нового пользователя:
```
adduser anon
usermod -aG sudo anon
su anon


sudo visudo
# Defaults env_reset => Defaults env_reset,pwfeedback

```

### Наводим секьюрность
```bash

# Запрещаем root по ssh меняем порт:
sudo nano /etc/ssh/sshd_config

  PermitRootLogin no
  Port 23

sudo service ssh restart


# Fail2ban для блокировки брутфорс-атак
sudo apt update
sudo apt install fail2ban

sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = 14500  # Укажите ваш SSH-порт, если вы изменили его
maxretry = 5
bantime = 600  # 10 минут блокировки после 5 неудачных попыток
findtime = 600

sudo systemctl enable fail2ban
sudo systemctl start fail2ban

sudo fail2ban-client status sshd

# Ограничение числа подключений с помощью UFW

sudo ufw limit 14500/tcp


# Установка 2FA

sudo apt install libpam-google-authenticator
google-authenticator
sudo nano /etc/pam.d/sshd
auth required pam_google_authenticator.so
sudo nano /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
sudo systemctl restart ssh


```



### Установка зависимостей:
```zsh
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

npm install pm2 -g 

pm2 start npm -- start
pm2 restart 0


# Автозапуск, вернёт команду
pm2 startup
# run app
pm2 start ecosystem.config.cjs --watch

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


# Debian client
sudo apt install wireguard resolvconf
sudo nano /etc/wireguard/wg0.conf
sudo systemctl restart wg-quick@wg0
sudo systemctl enable wg-quick@wg0.service

```

# Second WireGuard tunnel
```bash
nano /etc/wireguard/wg1.conf

[Interface]
PrivateKey = <PrivateKey2>
Address = 10.0.1.1/24
ListenPort = 51821

[Peer]
PublicKey = <PublicKey2>
Endpoint = <Server2_IP>:51821
AllowedIPs = 0.0.0.0/0

sudo wg-quick up wg1

# Для проверки запущенных туннелей
sudo wg

# Автоконнект
sudo systemctl enable wg-quick@wg1
```


### Установка сервера outline

```text
(В ИНТЕРФЕЙСЕ МЕНЕДЖЕРА ЕСТЬ ИНСТРУКЦИЯ)
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


### MatterMost

```bash
# Подготовка базы данных
nano /etc/postgresql/{version}/main/postgresql.conf

# listen_addresses = '*'
sudo systemctl restart postgresql


nano /etc/postgresql/9.4/main/pg_hba.conf
# from
local   all             all                        peer
host    all             all         ::1/128        ident
# to
local   all             all                        trust
host    all             all         ::1/128        trust

sudo systemctl reload postgresql


# CREATE DB mattermost:mmuser<mmuser-password>

# Получаем последнюю версию
wget https://releases.mattermost.com/10.0.1/mattermost-10.0.1-linux-amd64.tar.gz

tar -xvzf mattermost*.gz
sudo mv mattermost /opt
sudo mkdir /opt/mattermost/data
sudo useradd --system --user-group mattermost


sudo chown -R mattermost:mattermost /opt/mattermost
sudo chmod -R g+w /opt/mattermost

sudo nano /lib/systemd/system/mattermost.service

```

```text
[Unit]
Description=Mattermost
After=network.target

[Service]
Type=notify
ExecStart=/opt/mattermost/bin/mattermost
TimeoutStartSec=3600
KillMode=mixed
Restart=always
RestartSec=10
WorkingDirectory=/opt/mattermost
User=mattermost
Group=mattermost
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target
```

```bash
sudo cp /opt/mattermost/config/config.json /opt/mattermost/config/config.defaults.json
sudo nano /opt/mattermost/config/config.json
# "postgres://mmuser:<mmuser-password>@<host-name-or-IP>:5432/mattermost?sslmode=disable&connect_timeout=10"

sudo systemctl start mattermost
curl http://localhost:8065
sudo systemctl enable mattermost.service


sudo cat /etc/nginx/sites-enabled/mattermost
```

```text
upstream backend {
    server localhost:8065;
    keepalive 32;
    }

server {
    server_name domainname.ru www.domainname.ru;

    location ~ /api/v[0-9]+/(users/)?websocket$ {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        client_max_body_size 50M;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;
        proxy_buffers 256 16k;
        proxy_buffer_size 16k;
        client_body_timeout 60s;
        send_timeout 300s;
        lingering_timeout 5s;
        proxy_connect_timeout 90s;
        proxy_send_timeout 300s;
        proxy_read_timeout 90s;
        proxy_pass http://localhost:8065;
    }

    location / {
        client_max_body_size 50M;
        proxy_set_header Connection "";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;
        proxy_buffers 256 16k;
        proxy_buffer_size 16k;
        proxy_read_timeout 600s;
        proxy_http_version 1.1;
        proxy_pass http://localhost:8065;
    }
}
```



### Установка NFS-сервера

```bash
sudo apt update
sudo apt install nfs-kernel-server

# systemctl start nfs-server
# systemctl enable nfs-server
```

```bash
sudo mkdir -p /mnt/shared

sudo chown nobody:nogroup /mnt/shared
sudo chmod 777 /mnt/shared
```

```bash
sudo nano /etc/exports
/mnt/shared 192.168.1.0/24(rw,sync,no_subtree_check)

# /mnt/shared — путь к каталогу, который вы расшариваете.
# 192.168.1.0/24 — сеть, которая имеет доступ к ресурсу.
# rw — разрешение на чтение и запись.
# sync — синхронная запись (гарантирует, что изменения сразу записываются на диск).
# no_subtree_check — отключает проверку подкаталогов (ускоряет работу).
```

```bash
sudo exportfs -a

# systemctl restart nfs-server
sudo systemctl restart nfs-kernel-server

sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw allow from 192.168.1.0/24 to any port 111

# На сервере тоже лучше завести аналогичного пользователя
sudo groupadd -g 1030 mattermost
sudo useradd -u 1020 -g 1030 mattermost

# Просмотр пользователей
cat /etc/passwd

# Просмотр групп
cat /etc/group
```


```bash
# Подключение с клиента

sudo apt update
sudo apt install nfs-common

sudo mount 192.168.1.10:/mnt/shared /mnt
# sudo mount 10.0.0.4:/STORAGE-2 /opt/mattermost/storage

# nano /etc/fstab
192.168.1.10:/mnt/shared /mnt nfs defaults 0 0
```



```bash
# WIHTIG

# Делать миграции данных от пользователя
sudo su - mattermost


```

### SSH with socks5
```bash

ssh -p 22 -f -C2qTnN -D 1080 user@192.168.1.1

ssh — выполняет подключение по SSH к удаленному серверу.
-p 22 — указывает порт для подключения, в данном случае порт 22 (стандартный для SSH).
-f — запускает SSH-сессию в фоновом режиме, после того как подключение установлено.
-C — включает сжатие данных, чтобы увеличить скорость передачи данных по SSH.
-2 — заставляет SSH использовать протокол версии 2.
-q — подавляет вывод ошибок SSH, делая его "тихим".
-T — отключает выделение псевдотерминала на сервере, поскольку он здесь не нужен.
-n — перенаправляет стандартный ввод из /dev/null, чтобы избежать возможных блокировок.
-N — указывает SSH не выполнять удаленные команды; он только устанавливает соединение.
-D 1080 — запускает SOCKS-прокси-сервер на локальном порту 1080.

ps aux | grep "ssh -p 22 -f -C2qTnN -D 1080 root@serv"
kill <PID>
```


```bash
sudo nano /etc/gdm3/greeter.dconf-defaults

# Automatic suspend
# =================
[org/gnome/settings-daemon/plugins/power]
# - Time inactive in seconds before suspending with AC power
#   1200=20 minutes, 0=never
# sleep-inactive-ac-timeout=1200
# - What to do after sleep-inactive-ac-timeout
#   'blank', 'suspend', 'shutdown', 'hibernate', 'interactive' or 'nothing'
sleep-inactive-ac-type='nothing'
# - As above but when on battery
# sleep-inactive-battery-timeout=1200
sleep-inactive-battery-type='nothing'

```


```bash
sudo cat /etc/fstab              

#//192.168.60.248/storage-t/ /home/user/STORAGE-T/ cifs credentials=/home/user/.access.txt,uid=1000,gid=1000 0 0

cat .access.txt 

username=user
password=123456


### Компиляция Python из исходников

```bash

sudo apt update && sudo apt install -y build-essential libssl-dev \
  zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl \
  llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev \
  liblzma-dev python3-openssl git

cd /usr/src
sudo wget https://www.python.org/ftp/python/X.Y.Z/Python-X.Y.Z.tgz
sudo tar xvf Python-X.Y.Z.tgz
cd Python-X.Y.Z
sudo ./configure --enable-optimizations
sudo make -j$(nproc)

# Важно: Используйте make altinstall, а не make install, чтобы не перезаписать python3
sudo make altinstall

pythonX.Y --version

```


### Docker

```bash
# Добавление пользователя в группу Docker
sudo usermod -aG docker $USER
newgrp docker

# Установить контейнер
sudo docker pull ubuntu:16.04

# Список контейнеров
sudo docker ps -a


# Запуск/ перезапуск контейнера
sudo docker start ubuntu16
sudo docker exec -it ubuntu16 /bin/bash


# Удалить контейнер
sudo docker rm ubuntu16


# Посмотреть список образов
sudo docker images

# Создать новый конттейнер из образа
sudo docker run -it --name new_ubuntu16 ubuntu:16.04 /bin/bash

# 
# 
# 
# 
# 
# 

```


================

```bash

sudo apt install mysql-server
sudo service mysql start
pip install --trusted-host pypi.python.org --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.tx


scp ./tehnosvar.sql ./local_settings.py ./media.tar.gz anon@172.17.0.2:/home/anon/tehnosvar-app


libmysqlclient-dev



# СРАБОТАЛО, НО ФАЙЛЫ В КОНТЕЙНЕРЕ ПРОПАЛИ
docker run -it -p 8000:8000 -v $(pwd):/tehnosvar-app --name tehnosvar-app tehnosvar-app bash

```

```zsh
# Отключить засыпание, если разлогинен.
sudo nano /etc/gdm3/greeter.dconf-defaults

```

```zsh
# GIT

# create a new repository on the command line
echo "# exx" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/GLSVAR-KZ/exx.git
git push -u origin main

#…or push an existing repository from the command line
git remote add origin https://github.com/GLSVAR-KZ/exx.git
git branch -M main
git push -u origin main

```


```zsh
# VirtualBox
sudo usermod -a -G vboxusers $USER
```


```zsh
# Custom
sudo nano /etc/fstab

UUID=462F325C5EF028C8   /STORE/HDD0   ntfs  defaults,uid=1000,gid=1000,umask=0022 0 2
UUID=730CE51142710AB7   /STORE/HDD1   ntfs  defaults,uid=1000,gid=1000,umask=0022 0 2
```



```txt
Windows Server - Общий доступ

Откройте services.msc и убедитесь, что запущены следующие службы:

DNS Client
Function Discovery Resource Publication
SSDP Discovery
UPnP Device Host
Если у вас русскоязычная версия, то службы называются так:

DNS-клиент
Публикация ресурсов обнаружения функций
Обнаружение SSDP
Узел универсальных PnP

```