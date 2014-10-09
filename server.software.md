# Состав серверного ПО 
> предварительный список, используемый на dev-сервере

##Необходимое ПО:

- ОС Linux Ubuntu 14.04
- nginx-naxsi 1.4.6
- nginx-naxsi-ui 1.4.6
- nodejs 0.10.25
- npm 1.3.10
- mongodb 2.4.9
- openjdk-7
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

### Установка и настройка rserve
Предварительно необходимо установить пакет _r-cran-rserve_:

	~# R
	> library(Rserve)
	> Rserve()

Создаем исполняемый файл для запуска сервера _/usr/local/bin/Rserve_:

	#!/bin/bash
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
