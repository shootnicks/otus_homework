# 25 Основы сбора и хранения логов  

## Описание домашнего задания:  

  [✔] В вагранте поднимаем 2 машины web и log  
  [✔] На web поднимаем nginx  
  [✔] На log настраиваем центральный лог сервер на любой системе на выбор: rsyslog  
  [✔] Настраиваем аудит, следящий за изменением конфигов nginx  

Все критичные логи с web должны собираться и локально и удаленно.  
Все логи с nginx должны уходить на удаленный сервер (локально только критичные).  
Логи аудита должны также уходить на удаленную систему.  
Формат сдачи ДЗ - vagrant + ansible  

## Комментарий к моему выполнению:  
Домашняя работа расчитана на выполнение в Debian 12.  

## Выполнение задания:  

Переходим в папку с дз (в моём случае это /opt/otus_homework/25_Basics_of_collecting_and_storing_logs) и запускаем проект:
```bash
vagrant up
```
Результатом выполнения команды vagrant up станет созданные 2 виртуальных машины **web** и **log**, с **Debian12** и настроенными сервисами через Ansible.  

### Нагенирируем событий для log файлов.  

Зайдем на машину **web** и повысим свои полномочия до root:
```bash
vagrant ssh web
```

```bash
sudo -i
```

Зайдем на nginx 3 раза:
```bash
curl localhost; curl localhost; curl localhost
```

Сгенерируем ошибку nginx. Переместим индексную страницу в папку /var/www/ :
```bash
mv /var/www/html/index.nginx-debian.html /var/www/
```
Зайдем на nginx 2 раза:
```bash
curl localhost; curl localhost
```
Вернем индексную страницу nginx обратно:
```bash
mv /var/www/index.nginx-debian.html /var/www/html/
```

Создадим и удалим файл с именем 'test' в каталоге /etc/nginx:
```bash
touch /etc/nginx/test; rm /etc/nginx/test
```

В результате с сервера **web** на rsyslog сервера **log** будет переданы логи nginx (access и error) и auditd (логи auditd записи в папку /etc/nginx)  
Проверим, что сервер log корректно принял логи и разместил из в папке **/var/log/rsyslog/web**  
Для этого зайдем на машину **log** и повысим свои полномочия до root:
```bash
vagrant ssh log
```
```bash
sudo -i
```

В папке /var/log/rsyslog/web создались 3 файла логов: nginx_access.log, nginx_error.log и auditd.log
```bash
ls -la /var/log/rsyslog/web
drwxr-xr-x 2 root root 4096 Feb 26 16:28 .
drwxr-xr-x 4 root root 4096 Feb 26 16:27 ..
-rw-r----- 1 root adm   993 Feb 26 16:28 auditd.log
-rw-r----- 1 root adm   910 Feb 26 16:28 nginx_access.log
-rw-r----- 1 root adm   424 Feb 26 16:28 nginx_error.log
```

Посмотрим access-логи Nginx:
```bash
cat /var/log/rsyslog/web/nginx_access.log
2025-02-26T16:27:55+00:00 web nginx_access: 127.0.0.1 - - [26/Feb/2025:16:27:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1"
2025-02-26T16:27:55+00:00 web nginx_access: 127.0.0.1 - - [26/Feb/2025:16:27:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1"
2025-02-26T16:27:55+00:00 web nginx_access: 127.0.0.1 - - [26/Feb/2025:16:27:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1"
2025-02-26T16:28:09+00:00 web nginx_access: 127.0.0.1 - - [26/Feb/2025:16:28:09 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.88.1"
2025-02-26T16:28:09+00:00 web nginx_access: 127.0.0.1 - - [26/Feb/2025:16:28:09 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.88.1"
```

Посмотрим error-логи Nginx:
```bash
cat /var/log/rsyslog/web/nginx_error.log
2025-02-26T16:28:09+00:00 web nginx_error: 2025/02/26 16:28:09 [error] 24301#24301: *6 directory index of "/var/www/html/" is forbidden, client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
2025-02-26T16:28:09+00:00 web nginx_error: 2025/02/26 16:28:09 [error] 24301#24301: *7 directory index of "/var/www/html/" is forbidden, client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
```

Посмотрим audit-логи auditd (внесение изменений в папке /etc/nginx)
```bash
cat /var/log/rsyslog/web/auditd.log
2025-02-26T16:28:21+00:00 web auditd type=SYSCALL msg=audit(1740587301.071:234): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7ffdb80497ed a2=941 a3=1b6 items=2 ppid=24413 pid=24425 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=9 comm="touch" exe="/usr/bin/touch" subj=unconfined key="nginx-config-change"#035ARCH=x86_64 SYSCALL=openat AUID="vagrant" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
2025-02-26T16:28:21+00:00 web auditd type=SYSCALL msg=audit(1740587301.083:235): arch=c000003e syscall=263 success=yes exit=0 a0=ffffff9c a1=555c6bccdbf0 a2=0 a3=7fb536fcff80 items=2 ppid=24413 pid=24426 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=9 comm="rm" exe="/usr/bin/rm" subj=unconfined key="nginx-config-change"#035ARCH=x86_64 SYSCALL=unlinkat AUID="vagrant" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
```

Nginx и rsyslog сервера **web** корректно отправляют логи на rsyslog сервера **log**.