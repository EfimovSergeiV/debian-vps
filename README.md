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

CREATE DATABASE home_project;
CREATE USER home_project WITH PASSWORD 'password';

ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE myprojectuser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;

\q
```

### Создание/Восстановление данных из дампа:
```
sudo -u postgres pg_dump database > db.sql
sudo -u postgres psql -d newdatabase -f db.sql
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

    location = /favicon.ico { access_log off; log_not_found off; }
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
