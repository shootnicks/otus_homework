# 15 Управление процессами

## Описание домашнего задания:  
  [✔] написать свою реализацию ps ax используя анализ /proc  
Результат ДЗ - рабочий скрипт который можно запустить

## Выполнение задания:

Создадим скрипт в папке  **/usr/bin/**   
Скрипт назовем: **myps**
```bash
cat > /usr/bin/myps
```
1. Скопируйте и вставьте текст скрипта:
```bash
#!/bin/bash

# Функция для отображения процесса
ps_simple() {
    # Заголовок
    echo "PID	TTY	STAT	TIME	COMMAND"
    
    # Перебираем все каталоги в /proc, которые содержат только числа (PID)
    for pid_dir in /proc/[0-9]*; do
        pid=$(basename "$pid_dir")  # Получаем PID
        # Проверяем, существует ли файл status для процесса
        if [ -f "$pid_dir/status" ]; then
            # Извлекаем имя процесса и его состояние
            process_name=$(grep -s "Name:" "$pid_dir/status" | awk '{print $2}')
            process_state=$(grep -s "State:" "$pid_dir/status" | awk '{print $2}')
            
            # Извлекаем время процессора (используем stat файл для более подробной информации)
            process_time=$(awk '{print $14 + $15}' "$pid_dir/stat")
            
            # Извлекаем информацию о терминале (TTY)
            tty=$(ps -o tty= -p "$pid" | tr -d ' ')
            
            # Если TTY пустой, это значит, что процесс не имеет терминала
            if [ -z "$tty" ]; then
                tty="?"
            fi
            
            # Печатаем PID, TTY, состояние процесса, время процессора и команду
            echo "$pid	$tty	$process_state	$process_time	$process_name"
        fi
    done
}

# Запуск функции
ps_simple

```
2. Нажмите комбинацию кнопок: **ctrl+d**

3. Сделаем скрипт исполняемым: 
```bash
chmod +x /usr/bin/myps
```
4. Проверим работу скрипта:
```bash
myps
```
<details>
<summary> результат выполнения команды: </summary>

```
PID TTY      STAT   TIME COMMAND
1       ?       S       152     systemd
10      ?       I       0       mm_percpu_wq
1008    ?       S       0       lightdm
1012    ?       S       9       systemd
1013    ?       S       0       (sd-pam)
1028    ?       S       20      pulseaudio
1029    ?       S       791     lightdm-gtk-gre
1030    ?       S       29      rtkit-daemon
1037    ?       S       1       dbus-daemon
1042    ?       S       0       at-spi-bus-laun
1048    ?       S       0       dbus-daemon
1050    ?       S       2       gvfsd
1086    ?       S       0       lightdm
1088    ?       S       0       at-spi2-registr
1095    ?       S       3       cups-browsed
11      ?       I       0       rcu_tasks_kthread
12      ?       I       0       rcu_tasks_rude_kthread
13      ?       I       0       rcu_tasks_trace_kthread
137     ?       I       8       kworker/3:1H-kblockd
138     ?       I       12      kworker/2:1H-kblockd
14      ?       S       11      ksoftirqd/0
143     ?       I       172     kworker/2:2-mm_percpu_wq
15      ?       I       235     rcu_preempt
16      ?       S       9       migration/0
160     ?       I       0       ata_sff
162     ?       S       0       scsi_eh_0
163     ?       I       0       scsi_tmf_0
164     ?       S       0       scsi_eh_1
165     ?       I       0       scsi_tmf_1
166     ?       S       0       scsi_eh_2
167     ?       I       0       scsi_tmf_2
169     ?       I       207     kworker/3:2-events
170     ?       S       0       card0-crtc0
171     ?       S       0       card0-crtc1
172     ?       S       0       card0-crtc2
18      ?       S       0       cpuhp/0
19      ?       S       0       cpuhp/1
1918    ?       I       69      kworker/0:1-events
1995    ?       I       51      kworker/1:1-events
2       ?       S       1       kthreadd
20      ?       S       16      migration/1
2036    ?       I       12      kworker/u8:0-events_unbound
2056    ?       S       7       sshd
2059    ?       S       8       systemd
2060    ?       S       0       (sd-pam)
2076    ?       S       19      pulseaudio
2081    ?       S       36      sshd
2088    pts/0   S       1       bash
21      ?       S       11      ksoftirqd/1
2118    ?       S       1       dbus-daemon
2119    ?       I       0       kworker/2:0-cgroup_destroy
2152    ?       I       0       kworker/0:2
2154    pts/0   S       23      sudo
2155    pts/1   S       0       sudo
2156    pts/1   S       19      bash
2162    ?       S       82      sshd
2165    ?       S       6       systemd
2166    ?       S       0       (sd-pam)
218     ?       S       45      jbd2/sdb1-8
2185    ?       S       1       sh
219     ?       I       0       ext4-rsv-conver
2203    ?       S       170     code-e54c774e0a
2234    ?       S       0       sh
2238    ?       S       355     node
2258    ?       S       981     node
2269    ?       S       48      node
2283    ?       S       52      node
23      ?       I       0       kworker/1:0H-events_highpri
2314    ?       S       102     node
2330    ?       S       28      node
2397    ?       I       4       kworker/u8:3-events_unbound
2398    ?       I       0       kworker/3:0-events
24      ?       S       0       cpuhp/2
25      ?       S       15      migration/2
26      ?       S       13      ksoftirqd/2
271     ?       S       76      systemd-journal
28      ?       I       0       kworker/2:0H-events_highpri
29      ?       S       0       cpuhp/3
298     ?       S       20      systemd-udevd
3       ?       I       0       rcu_gp
30      ?       S       16      migration/3
31      ?       S       9       ksoftirqd/3
33      ?       I       0       kworker/3:0H-events_highpri
344     ?       S       17      systemd-timesyn
38      ?       S       0       kdevtmpfs
39      ?       I       0       inet_frag_wq
396     ?       S       0       watchdogd
4       ?       I       0       rcu_par_gp
40      ?       S       0       kauditd
41      ?       S       0       khungtaskd
413     ?       S       0       irq/130-mei_me
42      ?       S       0       oom_reaper
43      ?       I       0       writeback
44      ?       S       94      kcompactd0
45      ?       S       0       ksmd
46      ?       S       41      khugepaged
47      ?       I       0       kintegrityd
48      ?       I       0       kblockd
49      ?       I       0       blkcg_punt_bio
5       ?       I       0       slub_flushwq
501     ?       I       0       cfg80211
507     ?       I       0       cryptd
53      ?       I       0       tpm_dev_wq
535     ?       S       139     irq/132-iwlwifi
54      ?       I       0       edac-poller
55      ?       I       0       devfreq_wq
56      ?       I       17      kworker/1:1H-kblockd
57      ?       S       0       kswapd0
6       ?       I       0       netns
63      ?       I       0       kthrotld
65      ?       S       0       irq/122-aerdrv
651     ?       I       7       kworker/u9:1-rb_allocator
66      ?       S       0       irq/123-aerdrv
662     ?       S       385     avahi-daemon
663     ?       S       5       bluetoothd
664     ?       S       4       cron
666     ?       S       407     dbus-daemon
6664    ?       S       7       sshd
6670    ?       S       1       sshd
6671    pts/2   S       4       bash
6674    pts/2   S       6       sudo
6676    pts/3   S       0       sudo
6677    pts/3   S       1       bash
6687    ?       I       0       kworker/1:0-events
6690    ?       I       1       kworker/u8:1-events_unbound
6692    ?       I       1       kworker/u8:2-events_unbound
67      ?       S       0       irq/124-aerdrv
674     ?       S       30      polkitd
677     ?       S       78      systemd-logind
68      ?       S       0       irq/124-pciehp
682     ?       S       308     udisksd
689     ?       S       0       avahi-daemon
69      ?       I       0       acpi_thermal_pm
71      ?       I       0       mld
715     ?       S       1281    NetworkManager
717     ?       S       382     wpa_supplicant
718     ?       S       9       ModemManager
72      ?       I       21      kworker/0:1H-kblockd
73      ?       I       0       ipv6_addrconf
759     ?       S       2       lightdm
764     ?       S       1       cupsd
765     ?       S       3       sshd
78      ?       I       0       kstrp
790     tty7    S       229     Xorg
791     tty1    S       0       agetty
8       ?       I       0       kworker/0:0H-events_highpri
83      ?       I       0       zswap-shrink
8380    ?       S       0       sleep
8381    pts/1   S       14      getpoclist
84      ?       I       0       kworker/u9:0-hci0
966     ?       I       0       iprt-VBoxWQueue
967     ?       S       0       iprt-VBoxTscThread
```
</details>