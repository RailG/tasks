Hosting
====================

# Task

Текст задания как есть (но далее будем корректировать):

```
1. ОС:Ubuntu  
2. Админ панель? возможность работать и смотреть графики 
3. Установить пакеты Dotplant2 YII2
http://docs.dotplant.ru/ru/setup-example.html установить на домен youfhe.ru 
установить DotPlant2
4. Установить пакеты.

nginx
node npm
redis 
redis-desktop-manager http://redisdesktop.com/download
ssh доступ по ip http://rubuntu.com/1029/%D1%81%D0%B4%D0%B5%D0%BB%D0%B0%D1%82%D1%8C-%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF-%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE
добавить 87.117.167.118
#rabitmq
git
composer

postgerSQL

5. Настройка
почтовые почтовые уведомления

настройка backup mysql http://sourceforge.net/projects/automysqlbackup/
настройка backup сайта крон больше 3 дней


gunzip youfh_2015-03-10_06h47m.Tuesday.sql.gz
mysql -u root -p youfh < youfh_2015-04-08_11h53m.Wednesday.sql
tar -cvzf mylinker_02_07.tar.gz /var/www/mylinker
tar -xvf youfh_19_05_afternoon.tar.gz -C /var/www/archive/
```


# Порядок выполнения задания

# 1.1. Проверка установки nginx

Есть подозрение что `nginx` уже установлен или установлен какой-то и из веб-серверов (т.к. имеем возможность открывать ссылку: https://188.166.17.183:8083). Проверим это:

```
$ service nginx status
  * nginx is running
```

Т.е. видим что сервер `nginx` существует и запущен.

```
$ service apache2 status
  * apache2 is running
```

Т.е. сервер `apache2` тоже существует и запущен.

Если мы собираемся использовать только сервер `nginx`, тогда отключим службу `apache2`:

```
$ service apache2 stop
  * Stopping web server apache2
$ service apache2 status
  * apache2 is not running
```

Так же можем отключим автозагрузку службы `apache2`:

```
$ apt-get install sysv-rc-conf
```

```
$ sysv-rc-conf apache2 off
```

Проверить режимы автозагрузки можно командой:

```
$ sysv-rc-conf --list
```

Чтобы обратно влкючить автозагрузку службы `apache2` необходимо выполнить (включим т.к. точно не знаем будем ли полностью отказыватся от службы):

```
$ sysv-rc-conf apache2 on
```

PS. Также можно посмотреть какие службы установлены по ссылке: https://188.166.17.183:8083/list/server/ (т.е. службы присоединеные к "Vesta Control panel").

# 1.2. Обзор Nginx

* Заметка
  * Основные настройки `nginx` хранятся в файле `/etc/nginx/nginx.conf`.
  * Настройки сайта (по ip: 188.166.17.183), храняться в файле `/etc/nginx/conf.d/188.166.17.183.conf`.
  * Настройки для "Vesta Control panel", хранятся в файле `/etc/nginx/conf.d/vesta.conf`.
  
Но возникает вопрос? Где же прописан порт `8083`. Попробуем это найти:

```
$ grep -rl '8083' /etc/nginx
```

Результат поиска `null`. 

Есть подозрение что тут замешан iptables/firewall. Попробуем это проверить:

```
$ iptables -L
```

```
$ cat /etc/iptables.rules
```

Т.е. действительно видем записи:

```
-A INPUT -p tcp -m tcp --dport 8083 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 8083 -j ACCEPT
```

Но тут нет перенаправления. Откуда же взялся порт `8083`?
Заходим на сайт vestcap.com по сслыке [http://vestacp.com/docs/](http://vestacp.com/docs/). Видим что этот порт занять и имеет специальне назначение для "Vesta Control panel". Следовательно в конфигурационных файлах `nginx` этот порт мы не должны видеть, т.к. он прописан совсем по другому.

Тажке упоминание об этом порте `8083` есть в файле `/etc/roundcube/plugins/password/config.inc.php`.



# 1.3. Проверка и установка PostgerSQL

Проверим статус службы `postgresql`:
```
$ service postgresql status
postgresql: unrecognized service
```

Проверим установлен ли пакет `postgresql`:

```
$ dpkg -s postgresql | grep Status
```

Предварительно также читаем: [http://vestacp.com/docs/](http://vestacp.com/docs/) (How to set up PostgreSQL on a Debian or Ubuntu).

Установим пакет `postgresql`:

```
$ apt-get install postgresql postgresql-contrib phppgadmin
Setting up phppgadmin (5.1-1) ...
 * Reloading web server apache2         * 
 * Apache2 is not running
```

Видим, что `phppgadmin` спрашивает `Apache2`. Поэтому включим эту службу (см. п. 1.1).

```
$ cp /etc/postgresql/*/main/pg_hba.conf /etc/postgresql/*/main/pg_hba.conf.src
$ wget http://c.vestacp.com/0.9.8/debian/pg_hba.conf -O /etc/postgresql/*/main/pg_hba.conf
```

```
$ service postgresql restart
```

```
$ su - postgres
psql -c "ALTER USER postgres WITH PASSWORD '<password>'"
exit
```

```
$ vi /usr/local/vesta/conf/vesta.conf
...
DB_SYSTEM='mysql,pgsql'
```

```
$ v-add-database-host pgsql localhost postgres <password>
```

```
$ cp /etc/phppgadmin/config.inc.php /etc/phppgadmin/config.inc.php.src
$ wget http://c.vestacp.com/0.9.8/debian/pga.conf -O /etc/phppgadmin/config.inc.php
$ mkdir /etc/apache2/conf.d.src
$ cp /etc/apache2/conf.d/phppgadmin /etc/apache2/conf.d.src/phppgadmin
$ wget http://c.vestacp.com/0.9.8/debian/apache2-pga.conf -O /etc/apache2/conf.d/phppgadmin
```

```
$ service apache2 restart
```

# 1.4. MySQL

Изменим пароль (данный по умолчанию) для пользователя `root`:

```
/bin/mysqladmin -u root password '<password>'
```


# 2. Создание пользователя

```
$ adduser <username>
```

```
$ passwd <username>
	password: <password>
```

Но необходимо также учесть чтобы пользователи отображались в "Vesta Control panel". 
Лучший вариант добавить пользователей через саму панель. Но остается вопрос как это сделать, если пользователь уже существуе?

Добавим пользователья командой:

```
$ v-add-user production <password> <email> 
```

Пропишем пользователя `production` в sudo:

```
$ visudo

...

# User privilege specification
root 		ALL=(ALL:ALL) 	ALL
production 	ALL=(ALL) 	ALL 
```



# 3.1. Установка CMS "Dotplant2"

Вся установка будет производится от пользователя `admin`. 
Доументация по установке [http://docs.dotplant.ru/ru/setup-example.html](http://docs.dotplant.ru/ru/setup-example.html) 


```
$ sudo apt-get install nginx php5-fpm php5-gd php5-json mysql-server php5-mysql /
	 php5-cli php5-memcached memcached php5-curl php5-intl git
```

```
$ sudo apt-cache search php5 imagick
$ sudo apt-get install php5-imagick
$ sudo apt-cache search php5 apc
$ sudo apt-get install php5-apcu
```

```
$ cp /etc/php5/fpm/php.ini /etc/php5/fpm/php.ini.src
$ sudo vi /etc/php5/fpm/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/apache2/php.ini /etc/php5/apache2/php.ini.src
$ sudo vi /etc/php5/apache2/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/cli/php.ini /etc/php5/cli/php.ini.src
$ sudo vi /etc/php5/cli/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/cgi/php.ini /etc/php5/cgi/php.ini.src
$ sudo vi /etc/php5/cgi/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
# mysql -u root -p
CREATE DATABASE dotplant2 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'dotplant2'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON dotplant2.* TO 'dotplant2'@'localhost';
FLUSH PRIVILEGES;
```

Склонируем гит репозиторий и обновим зависимости:

```
$ cd ~/web/youfhe.ru/public_html
$ git clone https://github.com/DevGroup-ru/dotplant2.git
$ cd dotplant2/application
$ php ../composer.phar global require "fxp/composer-asset-plugin:~1.0"
# php ../composer.phar install --prefer-dist --optimize-autoloader
```

Проверяем тербования:

```
$ php requirements.php
```

Правим файл конфигурации для хоста `nginx`:

```
$ sudo mkdir /etc/nginx/conf.d.src
$ sudo cp /etc/nginx/conf.d/188.166.17.183.conf /etc/nginx/conf.d.src/188.166.17.183.conf 
$ sudo vi /etc/nginx/conf.d/188.166.17.183.conf 

server {
    listen 188.166.17.183:80 default;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    server_name _;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location ~ /\.ht {
       deny all;
    }
}
```

Но это может не сработать, т.к. контроль так же у нас происходить через `Vesta Control panel`. Поэтому будем править конфигурационные файлы в самой `Vesta`:

```
$ sudo rm /etc/nginx/conf.d/188.166.17.183.conf
$ sudo cp /home/admin/conf/web/nginx.conf /home/admin/conf/web/nginx.conf.src
$ sudo vi /home/admin/conf/web/nginx.conf

server {
    listen 188.166.17.183:80 default;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    server_name youfhe.ru www.youfhe.ru;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location ~ /\.ht {
       deny all;
    }
}

```


$ mkdir /etc/nginx/conf.d.src
$ sudo cp /etc/nginx/conf.d/188.166.17.183.conf /etc/nginx/conf.d.src/188.166.17.183.conf 


Отредактируем конфигурацию `PHP-fpm`:

```
$ sudo vi /etc/php5/fpm/php.ini
cgi.fix_pathinfo = 1
```

Перезагрузим службы `nginx` и `php5-fpm`:

```
$ sudo service nginx restart
$ sudo service php5-fpm restart
```

Установим базовые настройки CMS:

```
./installer
```


-------------------

server {
    listen 80;

    # NOTE: Replace with your path here
    root /home/igor/workspace/myapp/frontend/web;
    index index.php;

    # NOTE: Replace with your hostname
    server_name localhost;

    
location ~ \.php$ {
   root           /home/igor/workspace/myapp/frontend/web;
   try_files $uri =404;
   fastcgi_pass unix:/run/php-fpm/www.sock;
   fastcgi_index index.php;
   fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
   include fastcgi_params;
}
}


vi /etc/php-fpm.d/www.conf 
vi /etc/php5/fpm/pool.d/www.conf 


touch /etc/php5/fpm/pool.d/www_nginx.conf
[www]
php_admin_value[cgi.fix_pathinfo]=0

server {
    listen 188.166.17.183:80 default;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    server_name youfhe.ru www.youfhe.ru;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
    	root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        include fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
       deny all;
    }
}



