# 19 Docker: основы работы с контейнеризацией  

## Описание домашнего задания:  
  [✔] Установите Docker на хост машину  
  [✔] Установите Docker Compose - как плагин, или как отдельное приложение  
  [✔] Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)  
  [✔] Определите разницу между контейнером и образом  
  [✔] Вывод опишите в домашнем задании.  
  [✔] Ответьте на вопрос: Можно ли в контейнере собрать ядро?  

## Комментарий к моему выполнению:
Домашняя работа расчитана на выполнение в Debian 12.  

## Выполнение задания:  

### Установите Docker на хост машину.  
```bash
apt update &&  apt install -y docker curl
```

### Установите Docker Compose - как плагин, или как отдельное приложение.  
```bash
apt install -y docker-compose
```

### Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx):  

Создадим папку для нашего образа и зайдем в нее (В моём случае я создаду папку в папке c текущим ДЗ):
```bash
mkdir -p /opt/otus_homework/19_Docker_basics_of_working_with_containerization/otus_docker_nginx;\
cd /opt/otus_homework/19_Docker_basics_of_working_with_containerization/otus_docker_nginx
```

Создадим Dockerfile:
```bash
echo 'FROM nginx:alpine
COPY ./index.html /usr/share/nginx/html/index.html' > Dockerfile
```

Создадим index.html:
```bash
echo '<h1><center>Если видете этот текст, значит тестовый docker контейнер запустился.</center></h1>' > index.html
```

Создадим кастомизированный контейнер:
```bash
docker build -t otusdockernginx .
```

Результат выполнения команды:
```bash
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:alpine
alpine: Pulling from library/nginx
f18232174bc9: Pull complete
ccc35e35d420: Pull complete
43f2ec460bdf: Pull complete
984583bcf083: Pull complete
8d27c072a58f: Pull complete
ab3286a73463: Pull complete
6d79cc6084d4: Pull complete
0c7e4c092ab7: Pull complete
Digest: sha256:4ff102c5d78d254a6f0da062b3cf39eaf07f01eec0927fd21e219d0af8bc0591
Status: Downloaded newer image for nginx:alpine
 ---> 1ff4bb4faebc
Step 2/2 : COPY ./index.html /usr/share/nginx/html/index.html
 ---> 0c9421ff1b25
Successfully built 0c9421ff1b25
Successfully tagged otusdockernginx:latest
```

Посмотрим Docker-образы в системе:
```bash
docker images
```

Результат выполнения команды:
```bash
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
otusdockernginx   latest    0c9421ff1b25   3 minutes ago   47.9MB
nginx             alpine    1ff4bb4faebc   2 weeks ago     47.9MB
```

Запустим наш контейнер:
```bash
docker run -d --name webserver -p 8080:80 otusdockernginx
```

Результат выполнения команды:
```bash
735b723f911c70697e5d84d902c695836ebf1167daf78935a8269cced70b775c
```

Проверим запущенные контейнеры:
```bash
docker ps
```

Результат выполнения команды:
```bash
CONTAINER ID   IMAGE             COMMAND                  CREATED              STATUS              PORTS                                   NAMES
735b723f911c   otusdockernginx   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver
```

Попробуем загрузить страницу из нашего контейнера:

```bash
curl 127.0.0.1:8080
```

Результат выполнения команды:
```html
<h1><center>Если видете этот текст, значит тестовый docker контейнер запустился.</center></h1>
```

Зарегистрируемся на Docker Hub.  
Далее, выполним команду для подключения к Docker Hub:  

```bash
docker login
```

Результат выполнения команды:
```bash
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: andriyanovpg
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Выполним тегирование образа:
```bash
docker tag otusdockernginx andriyanovpg/otusdockernginx:latest
```

Выполним загрузку Docker-образа на Docker Hub:
```bash
docker push andriyanovpg/otusdockernginx:latest
```

Результат выполнения команды:
```bash
The push refers to repository [docker.io/andriyanovpg/otusdockernginx]
c6d643a735eb: Pushed
c18897d5e3dd: Mounted from library/nginx
9af9e76ea07f: Mounted from library/nginx
f1f70b13aacc: Mounted from library/nginx
252b6db79fae: Mounted from library/nginx
c9ce8cb4e76a: Mounted from library/nginx
8f3c313eb124: Mounted from library/nginx
c1761f3c364a: Mounted from library/nginx
08000c18d16d: Mounted from library/nginx
latest: digest: sha256:90c3f01f2595cf9664d08c175b938484d716c8007e954cdf2b4361f066fc5e18 size: 2196
```

Остановим запущенный докер:
```bash
docker stop webserver
```

Удалим все docker контейнеры:
```bash
docker rmi -f $(docker images -q)
```

Результат выполнения команды:
```bash
Untagged: andriyanovpg/otusdockernginx:latest
Untagged: andriyanovpg/otusdockernginx@sha256:90c3f01f2595cf9664d08c175b938484d716c8007e954cdf2b4361f066fc5e18
Deleted: sha256:0c9421ff1b25e58333eea0dce82061a674c76095727e3968622f9c417f930d0b
Deleted: sha256:a7aa16cce7e1f80ea9a59864c150ccdc301ff9d2f2ac67956bfdaf7975906df3
Deleted: sha256:85709b1869812616189f9b3347689a2b63817c0c319dfdd725465413ad70623d
Deleted: sha256:ee406d9dd9157f2d93cc60a5439054de89e11af5685bc6672f7c929cd9fe5a90
Deleted: sha256:bce08bd50eaa0b9c02d46240fd82c4ad6882b3bf5520be0dd681817bc085bc88
Deleted: sha256:f78f7efda4aaff92ba06ba89433f71b12d043975aa1b4d51c6aaa37048346713
Deleted: sha256:92c3896abdd8c4a2f9bec09d462c9186bef2341ba004a51e3ef328346c0cbe53
Deleted: sha256:055c9cb94079eafd5f7c3ce0ffdd84f7aa6ae6db16eb7c11999720b80cd58360
Deleted: sha256:5f9ae7b1f5994f699cebbcca01fdfaf0836c2beef0bf5390aff17744d642ee15
Deleted: sha256:08000c18d16dadf9553d747a58cf44023423a9ab010aab96cf263d2216b8b350
```

Скачаем наш образ с Docker Hub:
```bash
docker pull andriyanovpg/otusdockernginx
```

Результат выполнения команды:
```bash
Using default tag: latest
latest: Pulling from andriyanovpg/otusdockernginx
f18232174bc9: Pull complete
ccc35e35d420: Pull complete
43f2ec460bdf: Pull complete
984583bcf083: Pull complete
8d27c072a58f: Pull complete
ab3286a73463: Pull complete
6d79cc6084d4: Pull complete
0c7e4c092ab7: Pull complete
36a5b32664df: Pull complete
Digest: sha256:90c3f01f2595cf9664d08c175b938484d716c8007e954cdf2b4361f066fc5e18
Status: Downloaded newer image for andriyanovpg/otusdockernginx:latest
docker.io/andriyanovpg/otusdockernginx:latest
```

Запустим докер, скаченный с Docker Hub:
```bash
docker run -d --name webserv -p 8080:80 andriyanovpg/otusdockernginx
```

Посмотрим запущенные контейнеры:
```bash
docker ps
```

Результат выполнения команды:
```bash
CONTAINER ID   IMAGE                          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
7a1782960075   andriyanovpg/otusdockernginx   "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserv
```

Попробуем загрузить страницу из нашего контейнера:
```bash
curl 127.0.0.1:8080
```

Результат выполнения команды:
```html
<h1><center>Если видете этот текст, значит тестовый docker контейнер запустился.</center></h1>
```  

### Определите разницу между контейнером и образом.  
Docker образ — это статический файл, который содержит все необходимые компоненты для запуска приложения  
Docker контейнер — это запущенное приложение работающее на основе Docker-образа  

### Вывод опишите в домашнем задании.  

Docker удобен тем, что можно запустить приложение на системе с отсутствующими зависимостями.

### Ответьте на вопрос: Можно ли в контейнере собрать ядро?   

В docker контейнере можно собрать ядро, если его запустить с расширенными правами (--privileged)