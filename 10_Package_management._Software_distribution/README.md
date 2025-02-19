# 10 Управление пакетами. Дистрибьюция софта

## Описание домашнего задания:  
  [✔] Создать свой RPM пакет.
  [✔] Создать свой репозиторий и разместить там ранее собранный RPM.

## Выполнение задания:
Переходим в папку с дз (в моём случае это /opt/otus_homework/10_Package_management._Software_distribution) и запускаем проект:
```bash
vagrant up
```
Результатом выполнения команды vagrant up станет созданная виртуальная машина, с Almalinux 9.  
Зайдите в виртуальную машину (box):  
```bash
vagrant ssh
```
Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```bash
sudo -i
```

### Создать свой RPM пакет (Возьмем пакет Nginx и соберем его с дополнительным модулем ngx_broli)

**Для данного задания нам понадобятся следующие установленные пакеты:**   
```bash
yum install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano
```
Возьмем пакет **Nginx** и соберем его с дополнительным модулем **ngx_broli**   
    
Загрузим SRPM пакет Nginx для дальнейшей работы над ним:
```bash
mkdir rpm; cd rpm; yumdownloader --source nginx
```
   - **Примечание:** При установке такого пакета в домашней директории создается дерево каталогов для сборки.   
Поставим все **зависимости** для сборки пакета **Nginx**:  
```bash
rpm -Uvh nginx*.src.rpm; yum-builddep -y nginx
```
Также нужно скачать исходный код модуля **ngx_brotli** — он потребуется при сборке:   
      
```bash
cd /root; git clone --recurse-submodules -j8 \
https://github.com/google/ngx_brotli
```  

<details>
<summary> результат выполнения команды: </summary>

```bash
Cloning into 'ngx_brotli'...
remote: Enumerating objects: 237, done.
remote: Counting objects: 100% (37/37), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 237 (delta 24), reused 21 (delta 21), pack-reused 200 (from 1)
Receiving objects: 100% (237/237), 79.51 KiB | 733.00 KiB/s, done.
Resolving deltas: 100% (114/114), done.
Submodule 'deps/brotli' (https://github.com/google/brotli.git) registered for path 'deps/brotli'
Cloning into '/root/ngx_brotli/deps/brotli'...
remote: Enumerating objects: 7700, done.
remote: Counting objects: 100% (1575/1575), done.
remote: Compressing objects: 100% (384/384), done.
remote: Total 7700 (delta 1310), reused 1191 (delta 1191), pack-reused 6125 (from 3)
Receiving objects: 100% (7700/7700), 36.54 MiB | 877.00 KiB/s, done.
Resolving deltas: 100% (5039/5039), done.
Submodule path 'deps/brotli': checked out 'ed738e842d2fbdf2d6459e39267a633c4a9b2f5d'
```
</details>

```bash
cd ngx_brotli/deps/brotli && mkdir out && cd out
``` 

**Соберем модуль ngx_brotli:**

```bash
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
``` 

<details>
<summary> результат выполнения команды: </summary>

```bash
-- The C compiler identification is GNU 11.5.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type is 'Release'
-- Performing Test BROTLI_EMSCRIPTEN
-- Performing Test BROTLI_EMSCRIPTEN - Failed
-- Compiler is not EMSCRIPTEN
-- Looking for log2
-- Looking for log2 - not found
-- Looking for log2
-- Looking for log2 - found
-- Configuring done (1.2s)
-- Generating done (0.0s)
CMake Warning:
  Manually-specified variables were not used by the project:

    CMAKE_CXX_FLAGS


-- Build files have been written to: /root/ngx_brotli/deps/brotli/out
```
</details>

```bash
cmake --build . --config Release -j 2 --target brotlienc
``` 

<details>
<summary> результат выполнения команды: </summary>

```bash
[  3%] Building C object CMakeFiles/brotlicommon.dir/c/common/constants.c.o
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/context.c.o
[ 10%] Building C object CMakeFiles/brotlicommon.dir/c/common/platform.c.o
[ 13%] Building C object CMakeFiles/brotlicommon.dir/c/common/dictionary.c.o
[ 17%] Building C object CMakeFiles/brotlicommon.dir/c/common/shared_dictionary.c.o
[ 20%] Building C object CMakeFiles/brotlicommon.dir/c/common/transform.c.o
[ 24%] Linking C static library libbrotlicommon.a
[ 24%] Built target brotlicommon
[ 27%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references.c.o
[ 31%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references_hq.c.o
[ 34%] Building C object CMakeFiles/brotlienc.dir/c/enc/bit_cost.c.o
[ 37%] Building C object CMakeFiles/brotlienc.dir/c/enc/block_splitter.c.o
[ 41%] Building C object CMakeFiles/brotlienc.dir/c/enc/brotli_bit_stream.c.o
[ 44%] Building C object CMakeFiles/brotlienc.dir/c/enc/cluster.c.o
[ 48%] Building C object CMakeFiles/brotlienc.dir/c/enc/command.c.o
[ 51%] Building C object CMakeFiles/brotlienc.dir/c/enc/compound_dictionary.c.o
[ 55%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment.c.o
[ 58%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment_two_pass.c.o
[ 62%] Building C object CMakeFiles/brotlienc.dir/c/enc/dictionary_hash.c.o
[ 65%] Building C object CMakeFiles/brotlienc.dir/c/enc/encode.c.o
[ 68%] Building C object CMakeFiles/brotlienc.dir/c/enc/encoder_dict.c.o
[ 72%] Building C object CMakeFiles/brotlienc.dir/c/enc/entropy_encode.c.o
[ 75%] Building C object CMakeFiles/brotlienc.dir/c/enc/fast_log.c.o
[ 79%] Building C object CMakeFiles/brotlienc.dir/c/enc/histogram.c.o
[ 82%] Building C object CMakeFiles/brotlienc.dir/c/enc/literal_cost.c.o
[ 86%] Building C object CMakeFiles/brotlienc.dir/c/enc/memory.c.o
[ 89%] Building C object CMakeFiles/brotlienc.dir/c/enc/metablock.c.o
[ 93%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict.c.o
[ 96%] Building C object CMakeFiles/brotlienc.dir/c/enc/utf8_util.c.o
[100%] Linking C static library libbrotlienc.a
[100%] Built target brotlienc
```
</details>

Далее, нужно поправить сам spec файл, чтобы Nginx собирался с необходимыми нам опциями: находим секцию с параметрами **configure** ( ***if ! ./configure \*** ) (**до условий %if**) и добавляем указание на модуль (не забудьте указать завершающий обратный слэш): ***--add-module=/root/ngx_brotli \***    
Сделаем это одной командой:

```bash
sed -i '/\x2E\x2Fconfigure/a\--add-module=/root/ngx_brotli \\'  ~/rpmbuild/SPECS/nginx.spec
```  

<details>
<summary> Краткое описание как работает данная команда: </summary>

```bash
ищем в файле ~/rpmbuild/SPECS/nginx.spec фразу:
./configure
(\x2D - код символа '.'; \x2E - код символа '/') (подробнее о символах ASCII смотрим тут: https://klondike-studio.ru/blog/sed-spetssimvoly/)
и добавляем после найденной строки следующую строку:
--add-module=/root/ngx_brotli \
(символ '\' мы экранируем, потому в конце команды имеем '\\')
```
</details>

проверяем, что строка ***--add-module=/root/ngx_brotli \*** добавилась корректно:

```bash
grep -A5 '/configure' ~/rpmbuild/SPECS/nginx.spec
```

<details>
<summary> результат выполнения команды: </summary>

```bash
if ! ./configure \
--add-module=/root/ngx_brotli \
    --prefix=%{_datadir}/nginx \
    --sbin-path=%{_sbindir}/nginx \
    --modules-path=%{nginx_moduledir} \
    --conf-path=%{_sysconfdir}/nginx/nginx.conf \
```
</details>

Видим, что искомая строка находится в правильном месте.

<details>
<summary> Доп. информация: </summary>

```
Альтерантивный (полуручной) вариант, как можно добавить (оставил это для себя, может пригодится когда-нибудь...):
Можно сделать так:
Смотрим после какой строки надо добавить вызов модуля:
cat ~/rpmbuild/SPECS/nginx.spec | grep -n './configure'
301:if ! ./configure \

*Добавляем вызов модуля 302 строкой:
sed -i 302i\ '--add-module=/root/ngx_brotli \\' ~/rpmbuild/SPECS/nginx.spec

проверяем что добавилось:
grep -A5 '/configure' ~/rpmbuild/SPECS/nginx.spec

Либо, меняем вручную: vi ~/rpmbuild/SPECS/nginx.spec
```
</details>

По этой ссылке https://nginx.org/ru/docs/configure.html можно посмотреть все доступные опции для сборки.    
    
**Теперь можно приступить к сборке RPM пакета:**

```bash
cd ~/rpmbuild/SPECS/ && rpmbuild -ba nginx.spec -D 'debug_package %{nil}'
```

<details>
<summary> Результат выполнения команды (строк очень много, потому привожу последние несколько строк): </summary>

```bash
Wrote: /root/rpmbuild/SRPMS/nginx-1.20.1-20.el9.alma.1.src.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-devel-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-core-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-stream-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-mail-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-http-perl-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-http-image-filter-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-mod-http-xslt-filter-1.20.1-20.el9.alma.1.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/noarch/nginx-all-modules-1.20.1-20.el9.alma.1.noarch.rpm
Wrote: /root/rpmbuild/RPMS/noarch/nginx-filesystem-1.20.1-20.el9.alma.1.noarch.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.gPn17b
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.1
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.1-20.el9.alma.1.x86_64
+ RPM_EC=0
++ jobs -p
+ exit 0
```
</details>

**Далее, убедимся, что пакеты создались:**

```bash
ll ~/rpmbuild/RPMS/x86_64/
```

<details>
<summary> результат выполнения команды: </summary>

```bash
total 1992
-rw-r--r--. 1 root root   36240 Dec 17 07:56 nginx-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root 1027906 Dec 17 07:56 nginx-core-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root  759831 Dec 17 07:56 nginx-mod-devel-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   19368 Dec 17 07:56 nginx-mod-http-image-filter-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   31012 Dec 17 07:56 nginx-mod-http-perl-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   18170 Dec 17 07:56 nginx-mod-http-xslt-filter-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   53784 Dec 17 07:56 nginx-mod-mail-1.20.1-20.el9.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   80460 Dec 17 07:56 nginx-mod-stream-1.20.1-20.el9.alma.1.x86_64.rpm
```
</details>

**Копируем пакеты в общий каталог и зайдем в него:**
```bash
cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/ && cd ~/rpmbuild/RPMS/x86_64
```

**Теперь можно установить наш пакет и убедиться, что nginx работает:**
```bash
yum -y localinstall *.rpm
```
<details>
<summary> Результат выполнения команды: </summary>

```bash
Last metadata expiration check: 2:29:30 ago on Wed 19 Feb 2025 05:36:25 AM UTC.
Dependencies resolved.
============================================================================================================================================================================================================================
 Package                                                        Architecture                              Version                                                     Repository                                       Size
============================================================================================================================================================================================================================
Installing:
 nginx                                                          x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     35 k
 nginx-all-modules                                              noarch                                    2:1.20.1-20.el9.alma.1                                      @commandline                                    7.2 k
 nginx-core                                                     x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                    1.0 M
 nginx-filesystem                                               noarch                                    2:1.20.1-20.el9.alma.1                                      @commandline                                    8.2 k
 nginx-mod-devel                                                x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                    742 k
 nginx-mod-http-image-filter                                    x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     19 k
 nginx-mod-http-perl                                            x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     30 k
 nginx-mod-http-xslt-filter                                     x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     18 k
 nginx-mod-mail                                                 x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     53 k
 nginx-mod-stream                                               x86_64                                    2:1.20.1-20.el9.alma.1                                      @commandline                                     79 k
Installing dependencies:
 almalinux-logos-httpd                                          noarch                                    90.5.1-1.1.el9                                              appstream                                        18 k

Transaction Summary
============================================================================================================================================================================================================================
Install  11 Packages

Total size: 2.0 M
Total download size: 18 k
Installed size: 9.5 M
Downloading Packages:
almalinux-logos-httpd-90.5.1-1.1.el9.noarch.rpm                                                                                                                                             386 kB/s |  18 kB     00:00
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                        31 kB/s |  18 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                    1/1
  Running scriptlet: nginx-filesystem-2:1.20.1-20.el9.alma.1.noarch                                                                                                                                                    1/11
  Installing       : nginx-filesystem-2:1.20.1-20.el9.alma.1.noarch                                                                                                                                                    1/11
  Installing       : nginx-core-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                          2/11
  Installing       : almalinux-logos-httpd-90.5.1-1.1.el9.noarch                                                                                                                                                       3/11
  Installing       : nginx-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                               4/11
  Running scriptlet: nginx-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                               4/11
  Installing       : nginx-mod-http-image-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                         5/11
  Running scriptlet: nginx-mod-http-image-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                         5/11
  Installing       : nginx-mod-http-perl-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                 6/11
  Running scriptlet: nginx-mod-http-perl-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                 6/11
  Installing       : nginx-mod-http-xslt-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                          7/11
  Running scriptlet: nginx-mod-http-xslt-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                          7/11
  Installing       : nginx-mod-mail-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                      8/11
  Running scriptlet: nginx-mod-mail-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                      8/11
  Installing       : nginx-mod-stream-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                    9/11
  Running scriptlet: nginx-mod-stream-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                    9/11
  Installing       : nginx-all-modules-2:1.20.1-20.el9.alma.1.noarch                                                                                                                                                  10/11
  Installing       : nginx-mod-devel-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                    11/11
  Running scriptlet: nginx-mod-devel-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                    11/11
  Verifying        : almalinux-logos-httpd-90.5.1-1.1.el9.noarch                                                                                                                                                       1/11
  Verifying        : nginx-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                               2/11
  Verifying        : nginx-all-modules-2:1.20.1-20.el9.alma.1.noarch                                                                                                                                                   3/11
  Verifying        : nginx-core-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                          4/11
  Verifying        : nginx-filesystem-2:1.20.1-20.el9.alma.1.noarch                                                                                                                                                    5/11
  Verifying        : nginx-mod-devel-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                     6/11
  Verifying        : nginx-mod-http-image-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                         7/11
  Verifying        : nginx-mod-http-perl-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                 8/11
  Verifying        : nginx-mod-http-xslt-filter-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                          9/11
  Verifying        : nginx-mod-mail-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                     10/11
  Verifying        : nginx-mod-stream-2:1.20.1-20.el9.alma.1.x86_64                                                                                                                                                   11/11

Installed:
  almalinux-logos-httpd-90.5.1-1.1.el9.noarch                nginx-2:1.20.1-20.el9.alma.1.x86_64             nginx-all-modules-2:1.20.1-20.el9.alma.1.noarch             nginx-core-2:1.20.1-20.el9.alma.1.x86_64
  nginx-filesystem-2:1.20.1-20.el9.alma.1.noarch             nginx-mod-devel-2:1.20.1-20.el9.alma.1.x86_64   nginx-mod-http-image-filter-2:1.20.1-20.el9.alma.1.x86_64   nginx-mod-http-perl-2:1.20.1-20.el9.alma.1.x86_64
  nginx-mod-http-xslt-filter-2:1.20.1-20.el9.alma.1.x86_64   nginx-mod-mail-2:1.20.1-20.el9.alma.1.x86_64    nginx-mod-stream-2:1.20.1-20.el9.alma.1.x86_64

Complete!

```
</details>

**Запустим, включим автозапуск и проверим статус nginx:**
```bash
systemctl start nginx && systemctl enable nginx && systemctl status nginx
```

<details>
<summary> результат выполнения команды: </summary>

```bash
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-02-19 08:08:54 UTC; 531ms ago
   Main PID: 49859 (nginx)
      Tasks: 2 (limit: 5573)
     Memory: 7.7M
        CPU: 76ms
     CGroup: /system.slice/nginx.service
             ├─49859 "nginx: master process /usr/sbin/nginx"
             └─49861 "nginx: worker process"

Dec 17 08:08:54 rpm1.local systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 17 08:08:54 rpm1.local nginx[49857]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 17 08:08:54 rpm1.local nginx[49857]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 17 08:08:54 rpm1.local systemd[1]: Started The nginx HTTP and reverse proxy server.

```
</details>

   - Примечание: мы будем использовать его для доступа к своему репозиторию.

---
### Создать свой репозиторий и разместить там ранее собранный RPM.

**Приступим к созданию своего репозитория.**   
Директория для статики у Nginx по умолчанию **/usr/share/nginx/html**.   
Создадим там каталог **repo**:
```bash
mkdir /usr/share/nginx/html/repo
```

**Копируем туда наши собранные RPM-пакеты:**
```bash
cp ~/rpmbuild/RPMS/x86_64/*.rpm /usr/share/nginx/html/repo/
```

**Инициализируем репозиторий командой:**
```bash
createrepo /usr/share/nginx/html/repo/
```
<details>
<summary> результат выполнения команды: </summary>

```bash
Directory walk started
Directory walk done - 10 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
```
</details>

Для прозрачности настроим в **NGINX** доступ к листингу каталога. В файле **/etc/nginx/nginx.conf** в блоке **server** добавим следующие директивы:   
	index index.html index.htm;   
	autoindex on;   
 **Сделаем это одной командой:**
 
```bash
sed -i   '/^ \+server \x7B/a\
        index index.html index.htm;\
        autoindex on;' /etc/nginx/nginx.conf
```

<details>
<summary> Краткое описание как работает данная команда: </summary>

```
ищем в файле /etc/nginx/nginx.conf следующую фразу:
Строка начинается с одного или более пробелов, далее фраза: server {
(\x7B - код символа '{'  ) (подробнее о символах ASCII смотрим тут: https://klondike-studio.ru/blog/sed-spetssimvoly/)
и добавляем после найденной строки следующие строки:
        index index.html index.htm;
        autoindex on;
```
</details>


проверяем, что искомые строки добавилась корректно:

```bash
grep -A5 'server {' /etc/nginx/nginx.conf
```

<details>
<summary> результат выполнения команды: </summary>

```conf
    server {
        index index.html index.htm;
        autoindex on;
        listen       80;
        listen       [::]:80;
        server_name  _;
--
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
```
</details>

Либо можете добавить искомые строки вручную внеся изменения в файл /etc/nginx/nginx.conf   
   
Проверяем синтаксис и перезапускаем **nginx**:
```bash
nginx -t
```

<details>
<summary> результат выполнения команды: </summary>

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
</details>

```bash
nginx -s reload
```

**Проверим что репозиторий доступен по http**:
```bash
curl -a http://localhost/repo/
```

<details>
<summary> результат выполнения команды: </summary>

```html
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          19-Feb-2025 08:20                   -
<a href="nginx-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-1.20.1-20.el9.alma.1.x86_64.rpm</a>              19-Feb-2025 08:20               36240
<a href="nginx-all-modules-1.20.1-20.el9.alma.1.noarch.rpm">nginx-all-modules-1.20.1-20.el9.alma.1.noarch.rpm</a>  19-Feb-2025 08:20                7357
<a href="nginx-core-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-core-1.20.1-20.el9.alma.1.x86_64.rpm</a>         19-Feb-2025 08:20             1027906
<a href="nginx-filesystem-1.20.1-20.el9.alma.1.noarch.rpm">nginx-filesystem-1.20.1-20.el9.alma.1.noarch.rpm</a>   19-Feb-2025 08:20                8439
<a href="nginx-mod-devel-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-devel-1.20.1-20.el9.alma.1.x86_64.rpm</a>    19-Feb-2025 08:20              759831
<a href="nginx-mod-http-image-filter-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-http-image-filter-1.20.1-20.el9.alma...&gt;</a> 19-Feb-2025 08:20               19368
<a href="nginx-mod-http-perl-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-http-perl-1.20.1-20.el9.alma.1.x86_64..&gt;</a> 19-Feb-2025 08:20               31012
<a href="nginx-mod-http-xslt-filter-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-http-xslt-filter-1.20.1-20.el9.alma.1..&gt;</a> 19-Feb-2025 08:20               18170
<a href="nginx-mod-mail-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-mail-1.20.1-20.el9.alma.1.x86_64.rpm</a>     19-Feb-2025 08:20               53784
<a href="nginx-mod-stream-1.20.1-20.el9.alma.1.x86_64.rpm">nginx-mod-stream-1.20.1-20.el9.alma.1.x86_64.rpm</a>   19-Feb-2025 08:20               80460
</pre><hr></body>
</html>

```
</details>

**Все готово для того, чтобы протестировать репозиторий**   
Добавим его в **/etc/yum.repos.d**:

```bash
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```

**Убедимся, что репозиторий подключился и посмотрим, что в нем есть:**
```bash
yum repolist enabled | grep otus
```

<details>
<summary> результат выполнения команды: </summary>

```bash
otus                             otus-linux
```
</details>

**Добавим пакет в наш репозиторий:**
```bash
cd /usr/share/nginx/html/repo/ && wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```
<details>
<summary> результат выполнения команды: </summary>

```bash
--2025-02-19 08:42:22--  https://repo.percona.com/yum/percona-release-latest.noarch.rpm
Resolving repo.percona.com (repo.percona.com)... 49.12.125.205, 2a01:4f8:242:5792::2
Connecting to repo.percona.com (repo.percona.com)|49.12.125.205|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27900 (27K) [application/x-redhat-package-manager]
Saving to: ‘percona-release-latest.noarch.rpm’

percona-release-latest.noarch.rpm                      100%[============================================================================================================================>]  27.25K  --.-KB/s    in 0.001s

2025-02-19 08:42:22 (40.7 MB/s) - ‘percona-release-latest.noarch.rpm’ saved [27900/27900]
```
</details>

**Обновим список пакетов в репозитории:**
```bash
createrepo /usr/share/nginx/html/repo/ && yum makecache && yum list | grep otus
```

<details>
<summary> результат выполнения команды: </summary>

```bash
Directory walk started
Directory walk done - 11 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
AlmaLinux 9 - AppStream                                                                                                                                                                     7.1 kB/s | 4.2 kB     00:00
AlmaLinux 9 - BaseOS                                                                                                                                                                        7.0 kB/s | 3.8 kB     00:00
AlmaLinux 9 - Extras                                                                                                                                                                        6.4 kB/s | 3.8 kB     00:00
otus-linux                                                                                                                                                                                  488 kB/s | 7.2 kB     00:00
Metadata cache created.
percona-release.noarch                               1.0-29                              otus
```
</details>

Так как Nginx у нас уже стоит, установим репозиторий **percona-release**:
```bash
yum install -y percona-release.noarch
```
<details>
<summary> результат выполнения команды: </summary>

```bash
Last metadata expiration check: 0:02:32 ago on Wed 19 Feb 2025 08:43:31 AM UTC.
Dependencies resolved.
============================================================================================================================================================================================================================
 Package                                                     Architecture                                       Version                                              Repository                                        Size
============================================================================================================================================================================================================================
Installing:
 percona-release                                             noarch                                             1.0-29                                               otus                                              27 k

Transaction Summary
============================================================================================================================================================================================================================
Install  1 Package

Total download size: 27 k
Installed size: 48 k
Downloading Packages:
percona-release-latest.noarch.rpm                                                                                                                                                           2.7 MB/s |  27 kB     00:00
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                       2.2 MB/s |  27 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                    1/1
  Installing       : percona-release-1.0-29.noarch                                                                                                                                                                      1/1
  Running scriptlet: percona-release-1.0-29.noarch                                                                                                                                                                      1/1
* Enabling the Percona Release repository
<*> All done!
* Enabling the Percona Telemetry repository
<*> All done!
* Enabling the PMM2 Client repository
<*> All done!
The percona-release package now contains a percona-release script that can enable additional repositories for our newer products.

Note: currently there are no repositories that contain Percona products or distributions enabled. We recommend you to enable Percona Distribution repositories instead of individual product repositories, because with the Distribution you will get not only the database itself but also a set of other componets that will help you work with your database.

For example, to enable the Percona Distribution for MySQL 8.0 repository use:

  percona-release setup pdps8.0

Note: To avoid conflicts with older product versions, the percona-release setup command may disable our original repository for some products.

For more information, please visit:
  https://docs.percona.com/percona-software-repositories/percona-release.html


  Verifying        : percona-release-1.0-29.noarch                                                                                                                                                                      1/1

Installed:
  percona-release-1.0-29.noarch

Complete!

```
</details>

Все прошло успешно. В случае, если нам потребуется обновить репозиторий (а это делается при каждом добавлении файлов) снова, то выполняем команду `createrepo /usr/share/nginx/html/repo/`.