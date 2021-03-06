# Состав серверного ПО 
> предварительный список, используемый на dev-сервере

##Необходимое ПО:

- ОС Linux Ubuntu 14.04
- nginx-naxsi 1.4.6
- nginx-naxsi-ui 1.4.6
- nodejs 0.10.25
- nodejs-legacy ?
- npm 1.3.10
- postgresql 9.3
- redis-server 2.8.4
- openjdk-7 для Sencha Cmd
- ruby 1.9.1 для Sencha Cmd
- Sencha Cmd 5.0
- r-base 3.1.1
- r-cran-rserve 1.7
 
###Полезное ПО:

- mc
- htop
- p7zip
- unrar

## Пакеты R

- rserve
- RJSONIO
- pastecs
- car
- survival

### Установка и настройка rserve
Предварительно необходимо установить пакет _r-cran-rserve_:

	~# R
	> library(Rserve)
	> Rserve()

Создаем исполняемый файл для запуска сервера _/usr/local/bin/Rserve_:

	#!/bin/bash
	export LANG=ru_RU.UTF-8
	export LC_ALL=ru_RU.UTF-8
	
	/usr/lib/R/bin/R CMD /usr/lib/R/site-library/Rserve/libs/Rserve --vanilla --RS-conf /etc/Rserve.conf

Создаем простейший скрипт для автозапуска (правильнее переписать для запуска через upstart) сервиса _/etc/init.d/rserve_:

	#!/bin/sh
	
	RSERVE_CMD="/usr/local/bin/Rserve"	
	[ -x "$RSERVE_CMD" ] || exit 0
	
	# Get lsb functions
	. /lib/lsb/init-functions
	
	case "$1" in
	  start)
	    log_begin_msg "Starting Rserve services..."
	    start-stop-daemon --start --exec $RSERVE_CMD > /dev/null
	    log_end_msg $?
	    ;;
	  stop)
	    log_begin_msg "Stopping Rserve services..."
	    killall Rserve
	    log_end_msg $?
	    ;;
	  restart)
	    $0 stop
	    sleep 1
	    $0 start
	    ;;
	  *)
	    log_success_msg "Usage: /etc/init.d/rserve {start|stop|restart}"
	    exit 1
	esac

Конфигурационный файл _[/etc/Rserve.conf](http://rforge.net/Rserve/doc.html#conf)_

	workdir /home/rserve
	remote disable
	
	uid user
	gid user
	
	encoding utf8

### Установка Sails.js

	sudo npm -g install sails

### Настройка postgresql

В файл pg_hba.conf добавить разрешение на подключение из соответствующей сети:

	host	all	all	10.1.0.0/0	md5
	
В файл postgresql.conf добавить адреса, которые будет прослушивать сервис:

	listen_addresses='*'
	
Изменение пароле пользователя postgres:

	sudo su postgres
	psql postgres postgres
	ALTER USER postgres WITH PASSWORD 'новый пароль';
	
Добавление нового пользователя:

	sudo su postgres
	psql
	CREATE USER statca WITH PASSWORD 'пароль';

### Удаленный запуск компиляции клиента из MS Windows

_sencha.statca.bat statca.ui build_

	"c:\Program Files\putty\plink.exe" -P 10022 -l user -pw kalibotan -2 -4 127.0.0.1 "cd /srv/dev/%1/;/home/user/bin/Sencha/Cmd/5.0.0.160/sencha app %2"
