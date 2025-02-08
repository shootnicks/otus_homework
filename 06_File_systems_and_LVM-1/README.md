# 06 Файловые системы и LVM - 1

## Описание домашнего задания:
  [✔] Уменьшить том под / до 8GB.  
  [✔] Выделить том под /home.  
  [✔] Выделить том под /var - сделать в mirror.  
  [✔] /home - сделать том для снапшотов.  
  [✔] Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).  
  [✔] Работа со снапшотами:  
    - сгенерить файлы в /home/;  
    - снять снапшот;  
    - удалить часть файлов;  
    - восстановится со снапшота.  

### Дополнительные задания:
   [ ] *На дисках попробовать поставить btrfs/zfs — с кешем, снапшотами и разметить там каталог /opt.

## Комментарий к моему выполнению:
Работа выполнялась на Debian 12. Система развёрнута вручную, без vagrant. Почти для всех действий необходимы высокие полномочия, поэтому выполняются из под root-а.

## Выполнение задания:
Изначалный вывод команд `df -hT` и `lsblk` на тестовой системе.

```bash
root@debian-test-lvm-01:~# df -hT
Файловая система         Тип      Размер Использовано  Дост Использовано% Cмонтировано в
udev                     devtmpfs   960M            0  960M            0% /dev
tmpfs                    tmpfs      197M         536K  197M            1% /run
/dev/mapper/VG_ROOT-root xfs         19G         1,8G   17G           10% /
tmpfs                    tmpfs      984M            0  984M            0% /dev/shm
tmpfs                    tmpfs      5,0M            0  5,0M            0% /run/lock
/dev/sda1                ext2       937M         108M  782M           13% /boot
tmpfs                    tmpfs      197M            0  197M            0% /run/user/1000
```

```bash
root@debian-test-lvm-01:~# lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   20G  0 disk
├─sda1             8:1    0  953M  0 part /boot
└─sda2             8:2    0 19,1G  0 part
  ├─VG_ROOT-root 254:0    0 18,1G  0 lvm  /
  └─VG_ROOT-swap 254:1    0  972M  0 lvm  [SWAP]
sdb                8:16   0   10G  0 disk
sdc                8:32   0    2G  0 disk
sdd                8:48   0    1G  0 disk
sde                8:64   0    1G  0 disk
sr0               11:0    1 1024M  0 rom
```

---
### Уменьшим том под / до 8G:


\# Устанавливаем пакет xfsdump:

```bash
root@debian-test-lvm-01:~# apt update -y && apt install -y xfsdump
```

\# Подготовим вреенный том для раздела:

```bash
root@debian-test-lvm-01:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

\# Создадим Volume group vg_root на разделе /dev/sdb:

```bash
root@debian-test-lvm-01:~# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
```

\# Создадим Logical Volume lv_root в Volume Group vg_root:

```bash
root@debian-test-lvm-01:~# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
```

\# Создадим файловую систему xfs на созданном Logical volume lv_root:

```bash
root@debian-test-lvm-01:~# mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

\# Смонтируем раздел /dev/vg_root/lv_root в /mnt:

```bash
mount /dev/vg_root/lv_root /mnt
```

\# Скопируем все данные с раздела / в /mnt:

```bash
root@debian-test-lvm-01:~# xfsdump -J - /dev/VG_ROOT/root | xfsrestore -J - /mnt
xfsdump: xfsrestore: using file dump (drive_simple) strategy
using file dump (drive_simple) strategy
xfsrestore: version 3.1.11 (dump format 3.0)
xfsdump: version 3.1.11 (dump format 3.0)
xfsrestore: searching media for dump
xfsdump: level 0 dump of debian-test-lvm-01:/
xfsdump: dump date: Fri Feb  7 18:13:39 2025
xfsdump: session id: 09c23120-ce5c-40ba-a808-929c05fdf4ec
xfsdump: session label: ""
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 1722481984 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: debian-test-lvm-01
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/VG_ROOT-root
xfsrestore: session time: Fri Feb  7 18:13:39 2025
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: 926ab8ae-4f0d-4e72-b9b4-61eae0ae8111
xfsrestore: session id: 09c23120-ce5c-40ba-a808-929c05fdf4ec
xfsrestore: media id: 9c1acb52-f01d-41e6-b2e2-fc8e5df186cc
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 4292 directories and 39657 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 1681907648 bytes
xfsdump: dump size (non-dir files) : 1660781000 bytes
xfsdump: dump complete: 13 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 13 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```

\# Проверим, что содержимое раздела / скопировалось в /mnt:

```bash
root@debian-test-lvm-01:~# ls -la /mnt
итого 8
drwxr-xr-x 17 root root  298 фев  7 18:13 .
drwxr-xr-x 17 root root  298 фев  6 17:39 ..
lrwxrwxrwx  1 root root    7 фев  7 18:13 bin -> usr/bin
drwxr-xr-x  2 root root    6 фев  6 17:36 boot
drwxr-xr-x  4 root root  182 фев  6 17:37 dev
drwxr-xr-x 69 root root 4096 фев  7 18:04 etc
drwxr-xr-x  3 root root   23 фев  6 17:41 home
lrwxrwxrwx  1 root root   30 фев  7 18:13 initrd.img -> boot/initrd.img-6.1.0-30-amd64
lrwxrwxrwx  1 root root   30 фев  7 18:13 initrd.img.old -> boot/initrd.img-6.1.0-22-amd64
lrwxrwxrwx  1 root root    7 фев  7 18:13 lib -> usr/lib
lrwxrwxrwx  1 root root    9 фев  7 18:13 lib64 -> usr/lib64
drwxr-xr-x  3 root root   33 фев  6 17:36 media
drwxr-xr-x  2 root root    6 фев  6 17:37 mnt
drwxr-xr-x  2 root root    6 фев  6 17:37 opt
drwxr-xr-x  2 root root    6 мар 29  2024 proc
drwx------  3 root root   70 фев  6 17:45 root
drwxr-xr-x  2 root root    6 фев  6 17:42 run
lrwxrwxrwx  1 root root    8 фев  7 18:13 sbin -> usr/sbin
drwxr-xr-x  2 root root    6 фев  6 17:37 srv
drwxr-xr-x  2 root root    6 мар 29  2024 sys
drwxrwxrwt  8 root root  250 фев  7 18:03 tmp
drwxr-xr-x 12 root root  133 фев  6 17:37 usr
drwxr-xr-x 11 root root  139 фев  6 17:37 var
lrwxrwxrwx  1 root root   27 фев  7 18:13 vmlinuz -> boot/vmlinuz-6.1.0-30-amd64
lrwxrwxrwx  1 root root   27 фев  7 18:13 vmlinuz.old -> boot/vmlinuz-6.1.0-22-amd64
```


\# Сконфигурируем grub для того, чтобы при старте перейти в новый /  
Сымитируем текущий root, сделаем в него chroot и обновим grub:

```bash
for i in /proc/ /sys/ /dev/ /run/ /boot/; \
do mount --bind $i /mnt/$i; done
chroot /mnt/
```

\# Поищем строки содержащие старый / раздел в файлах /etc/fstab и /boot/grub/grub.cfg

```bash
root@debian-test-lvm-01:/# cat /etc/fstab | grep VG_ROOT
/dev/mapper/VG_ROOT-root /               xfs     defaults        0       0
/dev/mapper/VG_ROOT-swap none            swap    sw              0       0
root@debian-test-lvm-01:/# cat /boot/grub/grub.cfg | grep VG_ROOT
        linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro single
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/VG_ROOT-root ro single
```

\# Выполним замену VG_ROOT-root на vg_root-lv_root в файлах /etc/fstab и /boot/grub/grub.cfg и проверим что замена произошла:  

```bash
root@debian-test-lvm-01:/# sed -i 's/VG_ROOT-root/vg_root-lv_root/g' /etc/fstab && cat /etc/fstab | grep vg_root-lv_root
/dev/mapper/vg_root-lv_root /               xfs     defaults        0       0
root@debian-test-lvm-01:/# sed -i 's/VG_ROOT-root/vg_root-lv_root/g' /boot/grub/grub.cfg && cat /boot/grub/grub.cfg | grep vg_root-lv_root
        linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro single
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/vg_root-lv_root ro single
```

\# Обновим образ initrd

```bash
root@debian-test-lvm-01:/# update-initramfs -c -k all
update-initramfs: Generating /boot/initrd.img-6.1.0-22-amd64
update-initramfs: Generating /boot/initrd.img-6.1.0-30-amd64
```

\# перезапустим сервер:

```bash
root@debian-test-lvm-01:/# exit
exit
root@debian-test-lvm-01:~# reboot

The system will reboot now!
```

\# Изменим размер старой VG и вернем на него /

\# Удаляем старый LV размером в 18.3G и создаём новый на 8G

```bash
root@debian-test-lvm-01:~# lvremove /dev/VG_ROOT/root -y
  Logical volume "root" successfully removed.
```

\# Создадим новый lvm раздел root

```bash
root@debian-test-lvm-01:~# lvcreate -n VG_ROOT/root -L 8G /dev/VG_ROOT
WARNING: xfs signature detected on /dev/VG_ROOT/root at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VG_ROOT/root.
  Logical volume "root" created.
```

\# Создадим файловую систему xfs на разделе /dev/VG_ROOT/root

```bash
root@debian-test-lvm-01:~# mkfs.xfs /dev/VG_ROOT/root
meta-data=/dev/VG_ROOT/root      isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

\# Смонтируем раздел /dev/VG_ROOT/root в /mnt

```bash
root@debian-test-lvm-01:~# mount /dev/VG_ROOT/root /mnt
```

\# Скопируем все данные с раздела / в /mnt

```bash
root@debian-test-lvm-01:~# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsdump: using file dump (drive_simple) strategy
xfsrestore: using file dump (drive_simple) strategy
xfsdump: version 3.1.11 (dump format 3.0)
xfsrestore: version 3.1.11 (dump format 3.0)
xfsrestore: searching media for dump
xfsdump: level 0 dump of debian-test-lvm-01:/
xfsdump: dump date: Sat Feb  8 10:27:46 2025
xfsdump: session id: 5c17dfea-914b-447e-9342-a9047a590d38
xfsdump: session label: ""
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 1730996032 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: debian-test-lvm-01
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/vg_root-lv_root
xfsrestore: session time: Sat Feb  8 10:27:46 2025
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: e9ec9acb-c9cf-4188-ad54-ce5c22b7ccd5
xfsrestore: session id: 5c17dfea-914b-447e-9342-a9047a590d38
xfsrestore: media id: 953ecb7c-0b5b-479e-a1c4-e86cade367a9
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 4292 directories and 39665 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 1690459064 bytes
xfsdump: dump size (non-dir files) : 1669274648 bytes
xfsdump: dump complete: 11 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 11 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```

\# Сконфигурируем grub для того, чтобы при старте перейти в новый /

\# Сымитируем текущий root, сделаем в него chroot и обновим grub:

```bash
root@debian-test-lvm-01:~# for i in /proc/ /sys/ /dev/ /run/ /boot/; \
do mount --bind $i /mnt/$i; done
chroot /mnt/
```

\# Поищем строки содержащие старый / раздел в файлах /etc/fstab и /boot/grub/grub.cfg

```bash
root@debian-test-lvm-01:/# cat /etc/fstab | grep vg_root
/dev/mapper/vg_root-lv_root /               xfs     defaults        0       0
root@debian-test-lvm-01:/# cat /boot/grub/grub.cfg | grep vg_root
        linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/vg_root-lv_root ro single
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/vg_root-lv_root ro single
```

\# Выполним замену vg_root-lv_root на VG_ROOT-root в файлах /etc/fstab** и /boot/grub/grub.cfg и проверим что замена произошла:  

```bash
root@debian-test-lvm-01:/# sed -i 's/vg_root-lv_root/VG_ROOT-root/g' /etc/fstab && cat /etc/fstab | grep VG_ROOT-root
/dev/mapper/VG_ROOT-root /               xfs     defaults        0       0
root@debian-test-lvm-01:/# sed -i 's/vg_root-lv_root/VG_ROOT-root/g' /boot/grub/grub.cfg && cat /boot/grub/grub.cfg | grep VG_ROOT-root
        linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-30-amd64 root=/dev/mapper/VG_ROOT-root ro single
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-22-amd64 root=/dev/mapper/VG_ROOT-root ro single
```

\# Обновим образ initrd

```bash
root@debian-test-lvm-01:/# update-initramfs -c -k all
update-initramfs: Generating /boot/initrd.img-6.1.0-22-amd64
update-initramfs: Generating /boot/initrd.img-6.1.0-30-amd64
```

\# перезапустим сервер:

```bash
root@debian-test-lvm-01:/# exit
exit
root@debian-test-lvm-01:~# reboot

The system will reboot now!
```

\# Проверим что раздел / уменьшился до 8 GB:

```bash
root@debian-test-lvm-01:~# df -h | grep -e Filesystem -e Файловая -e VG_ROOT-root
Файловая система         Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/VG_ROOT-root   8,0G         1,8G  6,3G           22% /
root@debian-test-lvm-01:~# lsblk | grep -e NAME -e VG_ROOT-root
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
  └─VG_ROOT-root  254:2    0    8G  0 lvm  /
```


### Выделить том под /home

```bash
root@debian-test-lvm-01:~# lvcreate -n home -L 2G /dev/VG_ROOT
  Logical volume "home" created.
root@debian-test-lvm-01:~# mkfs.xfs /dev/VG_ROOT/home
meta-data=/dev/VG_ROOT/home      isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
root@debian-test-lvm-01:~# mount /dev/VG_ROOT/home /mnt/
root@debian-test-lvm-01:~# cp -aR /home/* /mnt/
root@debian-test-lvm-01:~# rm -rf /home/*
root@debian-test-lvm-01:~# umount /mnt
root@debian-test-lvm-01:~# mount /dev/VG_ROOT/home /home/
root@debian-test-lvm-01:~# echo "/dev/VG_ROOT/home /home xfs defaults 0 0" >> /etc/fstab
root@debian-test-lvm-01:~# mount -a
```

\# Проверим что /home отдельный раздел:

```bash
root@debian-test-lvm-01:~# df -h | grep -e Filesystem -e Файловая -e VG_ROOT-home
Файловая система         Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/VG_ROOT-home   2,0G          47M  1,9G            3% /home
root@debian-test-lvm-01:~# lsblk | grep -e NAME -e VG_ROOT-home
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
  └─VG_ROOT-home  254:3    0    2G  0 lvm  /home
```

### Выделить том под /var - сделать в mirror

```bash
root@debian-test-lvm-01:~# pvcreate /dev/sdd /dev/sde -y
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
root@debian-test-lvm-01:~# vgcreate VG_VAR /dev/sdd /dev/sde
  Volume group "VG_VAR" successfully created
root@debian-test-lvm-01:~# lvcreate -L 900M -m1 -n var VG_VAR
  Logical volume "var" created.
root@debian-test-lvm-01:~# mkfs.ext4 /dev/VG_VAR/var
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 230400 4k blocks and 57600 inodes
Filesystem UUID: 387a3e1d-bb68-4f1a-b30e-43a141e07d3c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@debian-test-lvm-01:~# systemctl daemon-reload && mount /dev/VG_VAR/var /mnt
root@debian-test-lvm-01:~# cp -aR /var/* /mnt/
root@debian-test-lvm-01:~# umount /mnt
root@debian-test-lvm-01:~# echo "/dev/mapper/VG_VAR-var /var ext4 defaults 0 0" >> /etc/fstab
root@debian-test-lvm-01:~# mount -a
```

\# Проверяем результат:

```bash
root@debian-test-lvm-01:~# df -h | grep -e Filesystem -e Файловая -e VG_VAR
Файловая система         Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/VG_VAR-var     868M         303M  505M           38% /var
root@debian-test-lvm-01:~# lsblk | grep -e NAME -e VG_VAR-var -e sdd -e sde
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd                     8:48   0    1G  0 disk
├─VG_VAR-var_rmeta_0  254:4    0    4M  0 lvm
│ └─VG_VAR-var        254:8    0  900M  0 lvm  /var
└─VG_VAR-var_rimage_0 254:5    0  900M  0 lvm
  └─VG_VAR-var        254:8    0  900M  0 lvm  /var
sde                     8:64   0    1G  0 disk
├─VG_VAR-var_rmeta_1  254:6    0    4M  0 lvm
│ └─VG_VAR-var        254:8    0  900M  0 lvm  /var
└─VG_VAR-var_rimage_1 254:7    0  900M  0 lvm
  └─VG_VAR-var        254:8    0  900M  0 lvm  /var
```


### Работа со снапшотами

\# Создадим несколько файлов в /home:

```bash
root@debian-test-lvm-01:~# touch /home/test_file{1..20}
```

\# Снимем снапшот c раздела /home:  

```bash
root@debian-test-lvm-01:~# lvcreate -L 100MB -s -n home_snap /dev/VG_ROOT/home
  Logical volume "home_snap" created.
```

\# Удалим часть файлов из /home:

```bash
rm -f /home/test_file{11..20} 
```

\# Восстановим из снапшота:  

```bash
root@debian-test-lvm-01:~# umount /home
root@debian-test-lvm-01:~# lvconvert --merge /dev/VG_ROOT/home_snap
  Merging of volume VG_ROOT/home_snap started.
  VG_ROOT/home: Merged: 100,00%
root@debian-test-lvm-01:~# systemctl daemon-reload && mount /dev/VG_ROOT/home /home
```

\# Проверяем результат:

```bash
ls -la /home
root@debian-test-lvm-01:~# ls -la /home
итого 4
drwxr-xr-x  3 root      root      4096 фев  8 12:57 .
drwxr-xr-x 17 root      root       298 фев  8 10:56 ..
drwx------  2 shootnick shootnick  111 фев  6 18:08 shootnick
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file1
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file10
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file11
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file12
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file13
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file14
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file15
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file16
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file17
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file18
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file19
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file2
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file20
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file3
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file4
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file5
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file6
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file7
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file8
-rw-r--r--  1 root      root         0 фев  8 12:57 test_file9
```

\# Перезапустим сервер и проверим что все разделы автомонтируются при запуске:

```bash
reboot
```

\# Проверяем результат:

```bash
root@debian-test-lvm-01:~# df -hT
Файловая система         Тип      Размер Использовано  Дост Использовано% Cмонтировано в
udev                     devtmpfs   960M            0  960M            0% /dev
tmpfs                    tmpfs      197M         600K  197M            1% /run
/dev/mapper/VG_ROOT-root xfs        8,0G         1,8G  6,3G           22% /
tmpfs                    tmpfs      984M            0  984M            0% /dev/shm
tmpfs                    tmpfs      5,0M            0  5,0M            0% /run/lock
/dev/mapper/VG_ROOT-home xfs        2,0G          47M  1,9G            3% /home
/dev/sda1                ext2       937M         108M  782M           13% /boot
/dev/mapper/VG_VAR-var   ext4       868M         311M  497M           39% /var
tmpfs                    tmpfs      197M            0  197M            0% /run/user/1000
```
