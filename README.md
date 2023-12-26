# Домашнее задание № 9 по теме: "Инициализация системы. Systemd". К курсу Administrator Linux. Professional

## Задание

- Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig или в /etc/default).
- Установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).
- Дополнить unit-файл httpd (он же apache2) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.

## Задание 1.

- Отключить SELinux

```bash
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
reboot
```

- Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig

```bash
touch /etc/sysconfig/watchlog
vi /etc/sysconfig/watchlog

cat /etc/sysconfig/watchlog
```

```bash
# Configuration file for my watchlog service
# Place it to /etc/sysconfig

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```

- Создать скрипт /opt/watchlog.sh

```bash
cat /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

chmod +x /opt/watchlog.sh
```

- Создать юнит для сервиса /etc/systemd/system/watchlog.service

```bash
cat /etc/systemd/system/watchlog.service

[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```

- Создать юнит таймера /etc/systemd/system/watchlog.timer

```bash
cat /etc/systemd/system/watchlog.timer

[Unit]
Description=Run watchlog script every 30 second
Requires=watchlog.service

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
```

- Стартуем таймер

```bash
systemctl start watchlog.timer
```

- Результат

```bash
tail -f /var/log/messages

Dec 26 17:01:24 lesssystemd systemd[1]: Started Session 4 of user vagrant.
Dec 26 17:01:24 lesssystemd systemd-logind[734]: New session 4 of user vagrant.
Dec 26 17:01:35 lesssystemd systemd[1]: systemd-hostnamed.service: Succeeded.
Dec 26 17:04:11 lesssystemd systemd[911]: Starting Mark boot as successful...
Dec 26 17:04:11 lesssystemd systemd[911]: Started Mark boot as successful.
Dec 26 17:07:51 lesssystemd systemd[1]: Started Run watchlog script every 30 second.
Dec 26 17:07:51 lesssystemd systemd[1]: Starting My watchlog service...
Dec 26 17:07:51 lesssystemd root[2881]: Tue Dec 26 17:07:51 UTC 2023: I found word, Master!
Dec 26 17:07:51 lesssystemd systemd[1]: watchlog.service: Succeeded.
Dec 26 17:07:51 lesssystemd systemd[1]: Started My watchlog service.
Dec 26 17:09:11 lesssystemd systemd[1]: Starting My watchlog service...
Dec 26 17:09:11 lesssystemd root[2889]: Tue Dec 26 17:09:11 UTC 2023: I found word, Master!
Dec 26 17:09:11 lesssystemd systemd[1]: watchlog.service: Succeeded.
Dec 26 17:09:11 lesssystemd systemd[1]: Started My watchlog service.

systemctl stop watchlog.timer
```

## Задание 2.

- Установить spawn-fcgi из epel-репозитория

```bash
yum install epel-release -y
yum install yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```

- Конфиг сервиса spawn-fcgi

```bash
cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```

- Юнит-файл сервиса spawn-fcgi

```bash
cat /etc/systemd/system/spawn-fcgi.service

[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```

- Запустить и проверить статус сервиса

```bash
systemctl start spawn-fcgi.service

systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2023-12-26 17:29:20 UTC; 7s ago
 Main PID: 3853 (php-cgi)
    Tasks: 33 (limit: 5978)
   Memory: 19.3M
   CGroup: /system.slice/spawn-fcgi.service
           ├─3853 /usr/bin/php-cgi
           ├─3854 /usr/bin/php-cgi
           ├─3855 /usr/bin/php-cgi
           ├─3856 /usr/bin/php-cgi
           ├─3857 /usr/bin/php-cgi
           ├─3858 /usr/bin/php-cgi
           ├─3859 /usr/bin/php-cgi
           ├─3860 /usr/bin/php-cgi
           ├─3861 /usr/bin/php-cgi
           ├─3862 /usr/bin/php-cgi
           ├─3863 /usr/bin/php-cgi
           ├─3864 /usr/bin/php-cgi
           ├─3865 /usr/bin/php-cgi
           ├─3866 /usr/bin/php-cgi
           ├─3867 /usr/bin/php-cgi
           ├─3868 /usr/bin/php-cgi
           ├─3869 /usr/bin/php-cgi
           ├─3870 /usr/bin/php-cgi
           ├─3871 /usr/bin/php-cgi
           ├─3872 /usr/bin/php-cgi
           ├─3873 /usr/bin/php-cgi
           ├─3874 /usr/bin/php-cgi
           ├─3875 /usr/bin/php-cgi
           ├─3876 /usr/bin/php-cgi
           ├─3877 /usr/bin/php-cgi
           ├─3878 /usr/bin/php-cgi
           ├─3879 /usr/bin/php-cgi
           ├─3880 /usr/bin/php-cgi
           ├─3881 /usr/bin/php-cgi
           ├─3882 /usr/bin/php-cgi
           ├─3883 /usr/bin/php-cgi
           ├─3884 /usr/bin/php-cgi
           └─3885 /usr/bin/php-cgi

Dec 26 17:29:20 lesssystemd systemd[1]: Started Spawn-fcgi startup service by Otus.
```

## Задание 3.

- Дополнить unit-файл httpd (он же apache2) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.

- Используем шаблон юнита

```bash
cat /usr/lib/systemd/system/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

# Добавить файл с переменными окружения
EnvironmentFile=/etc/sysconfig/httpd-%I

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

- Файлы окружения

```bash
cat /etc/sysconfig/httpd-first
OPTIONS=-f conf/first.conf

cat /etc/sysconfig/httpd-second
OPTIONS=-f conf/second.conf
```

- Конфигурационные файлы

```bash
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf
```

- Модифицировать следующие параметры в second.conf

```bash
PidFile /var/run/httpd-second.pid
Listen 8080
```

- Запустить и проверить

```bash
systemctl start httpd@first
systemctl start httpd@second

ss -tnulp | grep httpd
tcp   LISTEN 0      128                *:8080            *:*    users:(("httpd",pid=4232,fd=4),("httpd",pid=4231,fd=4),("httpd",pid=4230,fd=4),("httpd",pid=4229,fd=4),("httpd",pid=4227,fd=4))
tcp   LISTEN 0      128                *:80              *:*    users:(("httpd",pid=4009,fd=4),("httpd",pid=4008,fd=4),("httpd",pid=4007,fd=4),("httpd",pid=4006,fd=4),("httpd",pid=4003,fd=4))
```

