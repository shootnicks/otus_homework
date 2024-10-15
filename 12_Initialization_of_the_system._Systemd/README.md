# 03 Автоматизация администрирования. Ansible-1


## Домашнее задание
1. [✔] Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. [✔] Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. [✔] Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## Комментарий к моему выполнению:
Playbook расчитан на установку на Debian. 

## Выполнение задания

Скачиваем проект:

  ```bash
  git clone https://github.com/shootnicks/otus_homework.git
  ```

Переходим в папку домашнего задания

  ```bash
  cd ./otus_homework/12_Initialization_of_the_system._Systemd
  ```

Необходимо скорректировать файлы
* hosts.ini
  *прописать файлы хостов*
* ansible.cfg
  *Скорректировать переменную **ansible_ssh_private_key_file***
*  unit.yml
  *Скорректировать переменную **hosts***

#### 💻 1. [✔] Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default). Нужно запустить playbook:

  ```bash
  ansible-playbook unit.yml --tags unit_monitoring
  ```

<details>
<summary><strong> 🔍 Проверить работу сервиса</strong></summary>

<details>
<summary><strong>Debian 12</strong></summary>

На целевом хосте необходимо выполнить команду

```bash
journalctl -n 20 -u watchlog.service
```
</details>
<details>
<summary><strong>Debian <11</strong></summary>

На целевом хосте необходимо выполнить команду

```bash
tail -n 20 /var/log/syslog | grep word
```
</details>
</details>

#### 💻 2. [✔] Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020). Нужно запустить playbook:

  ```bash
  ansible-playbook unit.yml --tags unit_spawn-fcgi
  ```

<details>
<summary><strong>🔍 Проверить работу сервиса</strong></summary>

На целевом хосте необходимо выполнить команду

```bash
systemctl status spawn-fcgi
```
</details>

#### 💻 3. [✔] Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно. Нужно запустить playbook:

  ```bash
  ansible-playbook unit.yml --tags unit_nginx
  ```

<details>
<summary><strong>🔍 Проверить работу сервиса</strong></summary>

На целевом хосте необходимо выполнить команду

```bash
systemctl status nginx@first
systemctl status nginx@second
```
</details>

## Готово
