# 05 Дисковая подсистема 


## Домашнее задание
Научиться использовать утилиту для управления программными RAID-массивами в Linux

Статус "Принято" ставится при выполнении следующего условия:

* [ ] сдан измененный Vagrantfile;
* [✔] сдан скрипт для создания рейда;
* [✔] сдан конф для автосборки рейда при загрузке.

## Комментарий к моему выполнению:
Playbook расчитан на установку на Debian 12. Буранов на уроке сказал, что можно не использовать Vagrant.

Начальный вид блочных устройств:

```bash
root@debian-test5:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    5G  0 disk
├─sda1   8:1    0  487M  0 part /boot
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  4,5G  0 part /
sdb      8:16   0    1G  0 disk
sdc      8:32   0    1G  0 disk
sdd      8:48   0    1G  0 disk
sde      8:64   0    1G  0 disk
```

## Выполнение задания

Скачиваем проект:

  ```bash
  git clone https://github.com/shootnicks/otus_homework.git
  ```

Переходим в папку домашнего задания

  ```bash
  cd ./otus_homework/05_Disk_subsystem
  ```

Необходимо скорректировать файлы
* hosts.ini
  *прописать файлы хостов*
* ansible.cfg
  *Скорректировать переменную **ansible_ssh_private_key_file***
*  unit.yml
  *Скорректировать переменную **hosts***

#### 💻 Нужно запустить playbook:

  ```bash
  ansible-playbook mdadm.yml
  ```

Итоговый вид блочных устройств:

```bash
root@debian-test5:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
sda      8:0    0    5G  0 disk
├─sda1   8:1    0  487M  0 part   /boot
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  4,5G  0 part   /
sdb      8:16   0    1G  0 disk
└─md10   9:10   0    2G  0 raid10 /mnt/raid10
sdc      8:32   0    1G  0 disk
└─md10   9:10   0    2G  0 raid10 /mnt/raid10
sdd      8:48   0    1G  0 disk
└─md10   9:10   0    2G  0 raid10 /mnt/raid10
sde      8:64   0    1G  0 disk
└─md10   9:10   0    2G  0 raid10 /mnt/raid10
```

## Готово


