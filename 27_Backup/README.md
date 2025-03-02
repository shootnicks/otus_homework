# 27 Резервное копирование   

## Описание домашнего задания:  

  [✔] Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client.  

Настроить удаленный бекап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:  
  [✔] директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB;  
  [✔] репозиторий для резервных копий должен быть зашифрован ключом или паролем - на ваше усмотрение;  
  [✔] имя бекапа должно содержать информацию о времени снятия бекапа;  
  [✔] глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех.  
  [✔] Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;  
  [✔] резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;  
  [✔] написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на ваше усмотрение;  
  [✔] настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.

Формат сдачи ДЗ - vagrant + ansible  

## Комментарий к моему выполнению:  
Домашняя работа расчитана на выполнение в Debian 12.  

## Выполнение задания:  

Переходим в папку с дз (в моём случае это /opt/otus_homework/27_Backup) и запускаем проект:
```bash
vagrant up; \
ping 192.168.56.11 -c 1; \
ping 192.168.56.12 -c 1; \
ANSIBLE_CONFIG=./provisioning/ansible.cfg ansible-playbook -i provisioning/hosts.ini provisioning/setup_all.yml
```
Мне не удалось добавить в Vagrantfile запуск playbook-а так, чтобы он запускался после создания всех ВМ. Поэтому запуск playbook выполняется отдельно.

Результатом выполнения команды vagrant up станет созданные 2 виртуальных машины **backup** и **client**, с **Debian12** и настроенными сервисами через Ansible.  

### Нагенирируем событий по резервному копированию подождав минут 20-30. Затем...  

Зайдем на машину **client** и повысим свои полномочия до root:
```bash
vagrant ssh client
```

```bash
sudo -i
```

Для проверки репозитория понадобится парольная фраза: #!Otus2025!#
```bash
root@client:~# borg list borg@192.168.56.11:/var/backup
Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Enter passphrase for key ssh://borg@192.168.56.11/var/backup:
client-2025-03-02_06:48:37           Sun, 2025-03-02 06:48:39 [fdb63a24384d363307f04e248bef8425e48eab38bcf4317fa77b96e3aeb612b9]
client-2025-03-02_07:09:48           Sun, 2025-03-02 07:09:50 [36e5f0b066ee6f5781b69a668f4b10cb172b559b693d1d3f9779705a09ad4136]
```

Также посмотрим содержимого архива:
```bash
root@client:~# borg list borg@192.168.56.11:/var/backup::client-2025-03-02_06:48:37
Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Enter passphrase for key ssh://borg@192.168.56.11/var/backup:
drwxr-xr-x root   root          0 Sun, 2025-03-02 06:47:48 etc
-rw-r--r-- root   root       3040 Thu, 2023-05-25 15:54:35 etc/adduser.conf
-rw-r--r-- root   root       1706 Thu, 2023-05-25 15:54:35 etc/deluser.conf
drwxr-xr-x root   root          0 Sun, 2025-01-26 14:39:55 etc/apt
drwxr-xr-x root   root          0 Sun, 2025-01-26 14:40:11 etc/apt/apt.conf.d
-rw-r--r-- root   root        399 Thu, 2023-05-25 14:11:37 etc/apt/apt.conf.d/01autoremove
-rw-r--r-- root   root        182 Sun, 2023-01-08 21:50:51 etc/apt/apt.conf.d/70debconf
-rw-r--r-- root   root        307 Sun, 2021-03-28 11:06:44 etc/apt/apt.conf.d/20listchanges
-rw-r--r-- root   root         80 Sat, 2022-12-31 20:59:00 etc/apt/apt.conf.d/20auto-upgrades
-rw-r--r-- root   root       7338 Sat, 2022-12-31 20:59:00 etc/apt/apt.conf.d/50unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/auth.conf.d
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/keyrings
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/preferences.d
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/sources.list.d
drwxr-xr-x root   root          0 Sun, 2025-01-26 14:39:26 etc/apt/trusted.gpg.d
...
```

Логи:  
```bash
journalctl -t borg
Mar 02 06:48:38 client borg[12930]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 02 06:48:39 client borg[12930]: ------------------------------------------------------------------------------
Mar 02 06:48:39 client borg[12930]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 02 06:48:39 client borg[12930]: Archive name: client-2025-03-02_06:48:37
Mar 02 06:48:39 client borg[12930]: Archive fingerprint: fdb63a24384d363307f04e248bef8425e48eab38bcf4317fa77b96e3aeb612b9
Mar 02 06:48:39 client borg[12930]: Time (start): Sun, 2025-03-02 06:48:39
Mar 02 06:48:39 client borg[12930]: Time (end):   Sun, 2025-03-02 06:48:39
Mar 02 06:48:39 client borg[12930]: Duration: 0.59 seconds
Mar 02 06:48:39 client borg[12930]: Number of files: 430
Mar 02 06:48:39 client borg[12930]: Utilization of max. archive size: 0%
Mar 02 06:48:39 client borg[12930]: ------------------------------------------------------------------------------
Mar 02 06:48:39 client borg[12930]:                        Original size      Compressed size    Deduplicated size
Mar 02 06:48:39 client borg[12930]: This archive:                1.49 MB            649.36 kB            647.65 kB
Mar 02 06:48:39 client borg[12930]: All archives:                1.49 MB            648.76 kB            694.46 kB
Mar 02 06:48:39 client borg[12930]:                        Unique chunks         Total chunks
Mar 02 06:48:39 client borg[12930]: Chunk index:                     412                  422
Mar 02 06:48:39 client borg[12930]: ------------------------------------------------------------------------------
Mar 02 06:48:40 client borg[12932]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 02 06:53:55 client borg[12943]: ------------------------------------------------------------------------------
Mar 02 06:53:55 client borg[12943]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 02 06:53:55 client borg[12943]: Archive name: client-2025-03-02_06:53:53
Mar 02 06:53:55 client borg[12943]: Archive fingerprint: 82c11b9b3809534794bc3e30b5e2b5e14237db4c6d55b29b7b7bd62948bbe4ca
Mar 02 06:53:55 client borg[12943]: Time (start): Sun, 2025-03-02 06:53:55
Mar 02 06:53:55 client borg[12943]: Time (end):   Sun, 2025-03-02 06:53:55
Mar 02 06:53:55 client borg[12943]: Duration: 0.12 seconds
Mar 02 06:53:55 client borg[12943]: Number of files: 430
Mar 02 06:53:55 client borg[12943]: Utilization of max. archive size: 0%
Mar 02 06:53:55 client borg[12943]: ------------------------------------------------------------------------------
Mar 02 06:53:55 client borg[12943]:                        Original size      Compressed size    Deduplicated size
Mar 02 06:53:55 client borg[12943]: This archive:                1.49 MB            649.36 kB                602 B
Mar 02 06:53:55 client borg[12943]: All archives:                2.97 MB              1.30 MB            695.06 kB
Mar 02 06:53:55 client borg[12943]:                        Unique chunks         Total chunks
Mar 02 06:53:55 client borg[12943]: Chunk index:                     413                  844
Mar 02 06:53:55 client borg[12943]: ------------------------------------------------------------------------------
...
```
