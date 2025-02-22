# 18 SELinux - когда все запрещено

## Описание домашнего задания:  
1. Запустить nginx на нестандартном порту 3-мя разными способами:  
  [✔] переключатели setsebool;  
  [✔] добавление нестандартного порта в имеющийся тип;  
  [✔] формирование и установка модуля SELinux.  

2. Обеспечить работоспособность приложения при включенном selinux.  

  [✔] развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems;  
  [✔] выяснить причину неработоспособности механизма обновления зоны (см. README);  
  [✔] предложить решение (или решения) для данной проблемы;  
  [✔] выбрать одно из решений для реализации, предварительно обосновав выбор;  
  [✔] реализовать выбранное решение и продемонстрировать его работоспособность.  

## Выполнение задания:

### 1. Запустить nginx на нестандартном порту 3-мя разными способами:  

Переходим в папку с дз (в моём случае это /opt/otus_homework/18_SELinux_-_when_everything_is_forbidden/nginx) и запускаем проект:
```bash
vagrant up
```

Результатом выполнения команды vagrant up станет созданная виртуальная машина с установленным nginx, который работает на порту TCP 4881. Порт TCP 4881 уже проброшен до хоста. SELinux включен.  
Во время развёртывания стенда попытка запустить nginx завершится с ошибкой:  
```bash
    selinux: Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
    selinux: ● nginx.service - The nginx HTTP and reverse proxy server
    selinux:    Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
    selinux:    Active: failed (Result: exit-code) since Tue Fri 2025-02-21 14:22:38 UTC; 14ms ago
    selinux:   Process: 13765 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
    selinux:   Process: 13764 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    selinux:
    selinux: Feb 21 14:22:38 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
    selinux: Feb 21 14:22:38 selinux nginx[13765]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    selinux: Feb 21 14:22:38 selinux nginx[13765]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
    selinux: Feb 21 14:22:38 selinux nginx[13765]: nginx: configuration file /etc/nginx/nginx.conf test failed
    selinux: Feb 21 14:22:38 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
    selinux: Feb 21 14:22:38 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
    selinux: Feb 21 14:22:38 selinux systemd[1]: Unit nginx.service entered failed state.
```

- Примечание: Данная ошибка появляется из-за того, что SELinux блокирует работу nginx на нестандартном порту.  
Зайдите в виртуальную машину (box):  

```bash
vagrant ssh
```

Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```bash
sudo -i
```

#### **Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool:**  

```bash
setsebool -P nis_enabled 1
systemctl restart nginx
systemctl status nginx
```

Результат выполнения команд, Nginx теперь работает:
```bash
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2025-02-21 15:27:16 UTC; 5s ago
  Process: 32730 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 32728 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 32727 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 32732 (nginx)
   CGroup: /system.slice/nginx.service
           ├─32732 nginx: master process /usr/sbin/nginx
           └─32734 nginx: worker process

Feb 21 15:27:16 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 21 15:27:16 selinux nginx[32728]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 21 15:27:16 selinux nginx[32728]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 21 15:27:16 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

**Вернём запрет работы nginx на порту 4881 обратно. Для этого отключим nis_enabled:**
```bash
setsebool -P nis_enabled 0
```

#### **Разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип:**
```bash
semanage port -a -t http_port_t -p tcp 4881
systemctl restart nginx
systemctl status nginx
```

Результат выполнения команд, Nginx снова работает:
```bash
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2025-02-21 18:21:13 UTC; 12ms ago
  Process: 366 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 364 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 363 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 369 (nginx)
   CGroup: /system.slice/nginx.service
           ├─369 nginx: master process /usr/sbin/nginx
           └─373 nginx: master process /usr/sbin/nginx

Feb 21 18:21:13 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 21 18:21:13 selinux nginx[364]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 21 18:21:13 selinux nginx[364]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 21 18:21:13 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

**Удалим нестандартный порт (4881) из имеющегося типа:**
```bash
semanage port -d -t http_port_t -p tcp 4881
```

#### **Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux:**

```bash
semodule -i nginx.pp
systemctl restart nginx
systemctl status nginx
```

Результат выполнения команд, Nginx снова работает:
```bash
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2025-02-21 18:41:32 UTC; 12ms ago
  Process: 451 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 449 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 448 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 453 (nginx)
   CGroup: /system.slice/nginx.service
           ├─453 nginx: master process /usr/sbin/nginx
           └─456 nginx: master process /usr/sbin/nginx

Feb 21 18:41:32 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 21 18:41:32 selinux nginx[449]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 21 18:41:32 selinux nginx[449]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 21 18:41:32 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

Прерываем ssh соединение набрав:
```bash
exit  
```

#### 1 ДЗ Готово.

### 2. Обеспечить работоспособность приложения при включенном selinux:

Переходим в папку с дз (в моём случае это /opt/otus_homework/18_SELinux_-_when_everything_is_forbidden/dns) и запускаем проект:
```bash
vagrant up
```

Результатом выполнения команды vagrant up станет созданные 2 виртуальных машины: **ns01** и **client**.   
   
Посмотреть имена виртуальных машин можно командой:  
```bash
vagrant status
```

Результат выполнения команды:  
```bash
Current machine states:

ns01                      running (virtualbox)
client                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

    
**Зайдите в виртуальную машину (box) с именем: client**
```bash
vagrant ssh client
```
  - Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```bash
sudo -i
```

**Попробуем внести изменения в зону:**
```bash
nsupdate -k /etc/named.zonetransfer.key
server 192.168.50.10
zone ddns.lab
update add www.ddns.lab. 60 A 192.168.50.15
send
```

Результат выполнения команд:  
```bash
update failed: SERVFAIL
```


Выйдем из редактора зон:
```bash
quit
```

Изменения внести не получилось. Давайте посмотрим логи SELinux, чтобы понять в чём может быть проблема.   
Для этого воспользуемся утилитой **audit2why**:
```bash
cat /var/log/audit/audit.log | audit2why
```
Видим, что на клиенте отсутствуют ошибки.   

Не закрывая сессию на клиенте, подключимся к серверу ns01 и проверим логи SELinux:   
Для этого, подключимся к машине с Vagrant в соседнем окне и выполним команду:
```bash
cd /opt/otus/SELinux/dns && vagrant ssh ns01
```
  - Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```bash
sudo -i
```
**Проверим логи SELinux:**
```bash
cat /var/log/audit/audit.log | audit2why
```

Результат выполнения команды:  
```bash
type=AVC msg=audit(1732088140.214:2333): avc:  denied  { create } for  pid=16132 comm="isc-worker0000" name="named.ddns.lab.view1.jnl" scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:etc_t:s0 tclass=file permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.

type=AVC msg=audit(1732088371.059:2334): avc:  denied  { create } for  pid=16132 comm="isc-worker0000" name="named.ddns.lab.view1.jnl" scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:etc_t:s0 tclass=file permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.
```


В логах мы видим, что ошибка в контексте безопасности. Вместо типа **named_t** используется тип **etc_t**   
   
Проверим данную проблему в каталоге **/etc/named**:
```bash
ls -laZ /etc/named
```

Результат выполнения команды:  
```bash
drw-rwx---. root named system_u:object_r:etc_t:s0       .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:etc_t:s0   dynamic
-rw-rw----. root named system_u:object_r:etc_t:s0       named.50.168.192.rev
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab.view1
-rw-rw----. root named system_u:object_r:etc_t:s0       named.newdns.lab
```


  - Примечание: Тут мы также видим, что контекст безопасности неправильный. Проблема заключается в том, что конфигурационные файлы лежат в другом каталоге.

**Посмотреть в каком каталоги должны лежать, файлы, чтобы на них распространялись правильные политики SELinux можно с помощью команды:**
```bash
semanage fcontext -l | grep named
```

Результат выполнения команды:  
```bash
/etc/rndc.*                                        regular file       system_u:object_r:named_conf_t:s0
/var/named(/.*)?                                   all files          system_u:object_r:named_zone_t:s0
/etc/unbound(/.*)?                                 all files          system_u:object_r:named_conf_t:s0
/var/run/bind(/.*)?                                all files          system_u:object_r:named_var_run_t:s0
/var/log/named.*                                   regular file       system_u:object_r:named_log_t:s0
/var/run/named(/.*)?                               all files          system_u:object_r:named_var_run_t:s0
/var/named/data(/.*)?                              all files          system_u:object_r:named_cache_t:s0
/dev/xen/tapctrl.*                                 named pipe         system_u:object_r:xenctl_t:s0
/var/run/unbound(/.*)?                             all files          system_u:object_r:named_var_run_t:s0
/var/lib/softhsm(/.*)?                             all files          system_u:object_r:named_cache_t:s0
/var/lib/unbound(/.*)?                             all files          system_u:object_r:named_cache_t:s0
/var/named/slaves(/.*)?                            all files          system_u:object_r:named_cache_t:s0
/var/named/chroot(/.*)?                            all files          system_u:object_r:named_conf_t:s0
/etc/named\.rfc1912.zones                          regular file       system_u:object_r:named_conf_t:s0
/var/named/dynamic(/.*)?                           all files          system_u:object_r:named_cache_t:s0
/var/named/chroot/etc(/.*)?                        all files          system_u:object_r:etc_t:s0
/var/named/chroot/lib(/.*)?                        all files          system_u:object_r:lib_t:s0
/var/named/chroot/proc(/.*)?                       all files          <<None>>
/var/named/chroot/var/tmp(/.*)?                    all files          system_u:object_r:named_cache_t:s0
/var/named/chroot/usr/lib(/.*)?                    all files          system_u:object_r:lib_t:s0
/var/named/chroot/etc/pki(/.*)?                    all files          system_u:object_r:cert_t:s0
/var/named/chroot/run/named.*                      all files          system_u:object_r:named_var_run_t:s0
/var/named/chroot/var/named(/.*)?                  all files          system_u:object_r:named_zone_t:s0
/usr/lib/systemd/system/named.*                    regular file       system_u:object_r:named_unit_file_t:s0
/var/named/chroot/var/run/dbus(/.*)?               all files          system_u:object_r:system_dbusd_var_run_t:s0
/usr/lib/systemd/system/unbound.*                  regular file       system_u:object_r:named_unit_file_t:s0
/var/named/chroot/var/log/named.*                  regular file       system_u:object_r:named_log_t:s0
/var/named/chroot/var/run/named.*                  all files          system_u:object_r:named_var_run_t:s0
/var/named/chroot/var/named/data(/.*)?             all files          system_u:object_r:named_cache_t:s0
/usr/lib/systemd/system/named-sdb.*                regular file       system_u:object_r:named_unit_file_t:s0
/var/named/chroot/var/named/slaves(/.*)?           all files          system_u:object_r:named_cache_t:s0
/var/named/chroot/etc/named\.rfc1912.zones         regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot/var/named/dynamic(/.*)?          all files          system_u:object_r:named_cache_t:s0
/var/run/ndc                                       socket             system_u:object_r:named_var_run_t:s0
/dev/gpmdata                                       named pipe         system_u:object_r:gpmctl_t:s0
/dev/initctl                                       named pipe         system_u:object_r:initctl_t:s0
/dev/xconsole                                      named pipe         system_u:object_r:xconsole_device_t:s0
/usr/sbin/named                                    regular file       system_u:object_r:named_exec_t:s0
/etc/named\.conf                                   regular file       system_u:object_r:named_conf_t:s0
/usr/sbin/lwresd                                   regular file       system_u:object_r:named_exec_t:s0
/var/run/initctl                                   named pipe         system_u:object_r:initctl_t:s0
/usr/sbin/unbound                                  regular file       system_u:object_r:named_exec_t:s0
/usr/sbin/named-sdb                                regular file       system_u:object_r:named_exec_t:s0
/var/named/named\.ca                               regular file       system_u:object_r:named_conf_t:s0
/etc/named\.root\.hints                            regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot/dev                              directory          system_u:object_r:device_t:s0
/etc/rc\.d/init\.d/named                           regular file       system_u:object_r:named_initrc_exec_t:s0
/usr/sbin/named-pkcs11                             regular file       system_u:object_r:named_exec_t:s0
/etc/rc\.d/init\.d/unbound                         regular file       system_u:object_r:named_initrc_exec_t:s0
/usr/sbin/unbound-anchor                           regular file       system_u:object_r:named_exec_t:s0
/usr/sbin/named-checkconf                          regular file       system_u:object_r:named_checkconf_exec_t:s0
/usr/sbin/unbound-control                          regular file       system_u:object_r:named_exec_t:s0
/var/named/chroot_sdb/dev                          directory          system_u:object_r:device_t:s0
/var/named/chroot/var/log                          directory          system_u:object_r:var_log_t:s0
/var/named/chroot/dev/log                          socket             system_u:object_r:devlog_t:s0
/etc/rc\.d/init\.d/named-sdb                       regular file       system_u:object_r:named_initrc_exec_t:s0
/var/named/chroot/dev/null                         character device   system_u:object_r:null_device_t:s0
/var/named/chroot/dev/zero                         character device   system_u:object_r:zero_device_t:s0
/usr/sbin/unbound-checkconf                        regular file       system_u:object_r:named_exec_t:s0
/var/named/chroot/dev/random                       character device   system_u:object_r:random_device_t:s0
/var/run/systemd/initctl/fifo                      named pipe         system_u:object_r:initctl_t:s0
/var/named/chroot/etc/rndc\.key                    regular file       system_u:object_r:dnssec_t:s0
/usr/share/munin/plugins/named                     regular file       system_u:object_r:services_munin_plugin_exec_t:s0
/var/named/chroot_sdb/dev/null                     character device   system_u:object_r:null_device_t:s0
/var/named/chroot_sdb/dev/zero                     character device   system_u:object_r:zero_device_t:s0
/var/named/chroot/etc/localtime                    regular file       system_u:object_r:locale_t:s0
/var/named/chroot/etc/named\.conf                  regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot_sdb/dev/random                   character device   system_u:object_r:random_device_t:s0
/etc/named\.caching-nameserver\.conf               regular file       system_u:object_r:named_conf_t:s0
/usr/lib/systemd/systemd-hostnamed                 regular file       system_u:object_r:systemd_hostnamed_exec_t:s0
/var/named/chroot/var/named/named\.ca              regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot/etc/named\.root\.hints           regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot/etc/named\.caching-nameserver\.conf regular file       system_u:object_r:named_conf_t:s0
/var/named/chroot/lib64 = /usr/lib
/var/named/chroot/usr/lib64 = /usr/lib
```

  - Примечание: из всего вывода команды мы смотрим на данную строку:   
`/var/named(/.*)?                                   all files          system_u:object_r:named_zone_t:s0`

Изменим тип контекста безопасности для каталога **/etc/named**:
```bash
chcon -R -t named_zone_t /etc/named
```
   
Посмотрим на папку /etc/named:
```bash
ls -laZ /etc/named
```

Результат выполнения команды:  
```bash
drw-rwx---. root named system_u:object_r:named_zone_t:s0 .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:named_zone_t:s0 dynamic
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.newdns.lab
```


**Перейдем на клиента (client) и попробуем снова внести изменения:**
```bash
nsupdate -k /etc/named.zonetransfer.key
server 192.168.50.10
zone ddns.lab
update add www.ddns.lab. 60 A 192.168.50.15
send
quit
```
Теперь мы не наблюдаем ошибку.   

**Проверим применились ли изменения:**
```bash
dig www.ddns.lab
```

Результат выполнения команды:  
```bash
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.16 <<>> www.ddns.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19558
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.                  IN      A

;; ANSWER SECTION:
www.ddns.lab.           60      IN      A       192.168.50.15

;; AUTHORITY SECTION:
ddns.lab.               3600    IN      NS      ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.           3600    IN      A       192.168.50.10

;; Query time: 1 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Fri Feb 21 08:18:05 UTC 2025
;; MSG SIZE  rcvd: 96
```


Видим, что изменения применились.   

**Перезагрузим оба хоста `init 6`, зайдем на них и ещё раз сделаем запрос с помощью dig:**
```bash
dig @192.168.50.10 www.ddns.lab
```

Результат выполнения команды:  
```bash
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.16 <<>> @192.168.50.10 www.ddns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30098
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.                  IN      A

;; ANSWER SECTION:
www.ddns.lab.           60      IN      A       192.168.50.15

;; AUTHORITY SECTION:
ddns.lab.               3600    IN      NS      ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.           3600    IN      A       192.168.50.10

;; Query time: 1 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Fri Feb 21 08:54:25 UTC 2025
;; MSG SIZE  rcvd: 96
```


**Видим, что настройки сохранились.**   

  - Доп информация:
Для того, чтобы вернуть правила обратно, можно ввести команду на сервере (ns01):
```bash
restorecon -v -R /etc/named
```

Результат выполнения команды:  
```bash
restorecon reset /etc/named context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/named.dns.lab.view1 context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/named.dns.lab context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/dynamic context unconfined_u:object_r:named_zone_t:s0->unconfined_u:object_r:etc_t:s0
restorecon reset /etc/named/dynamic/named.ddns.lab context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/dynamic/named.ddns.lab.view1 context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/dynamic/named.ddns.lab.view1.jnl context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/named.newdns.lab context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
restorecon reset /etc/named/named.50.168.192.rev context system_u:object_r:named_zone_t:s0->system_u:object_r:etc_t:s0
```

  - Примечание: после выполнения команды мы снова не сможем вносить изменения в DNS-зону.