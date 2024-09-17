# 01 С чего начинается Linux

## Описание домашнего задания
1) Запустить ВМ с помощью Vagrant.
2) Обновить ядро ОС из репозитория ELRepo.
3) Оформить отчет в README-файле в GitHub-репозитории.

## Комментарий к моему выполнению:
Заменил CentOS 8 на Debian 12. 


Действия выполняются изходя из того, что вы находитесь сейчас в папке "01_Where_does_Linux_start".

Запускаем ВМ (Vagrantfile был взят с github Николая Лавлинского):

    vagrant up
	
Подключаемся к ней:

    ssh vagrant@127.0.0.1 -p2222 -i ./.vagrant/machines/kernel-update/virtualbox/private_key

Скачиваем нужную нам версию ядра:

    wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.10.9.tar.xz
	
Скачивам и устанавливем нужные для сборки пакеты:

    sudo apt update
    sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison

Распаковываем файл с ядром и переходим в распакованную папку:

    tar xvf linux-6.10.9.tar.xz
    cd linux-6.10.9
	
Копируем текущую конфигурацию ядра:

    cp /boot/config-$(uname -r) .config
	
Чтобы внести изменения в файл конфигурации, выполните команду make:

    make defconfig
	
Компилируем ядро:

    make -j$(nproc)
	
Устанавливаем необходимые модули с помощью команды:

    sudo make modules_install
	
Устанавливаем ядро:

    sudo make install	
	
Перезагружаем:

    sudo reboot

Немного ждём перезагрузки системы и вновь подключаемся к ней:

    ssh vagrant@127.0.0.1 -p2222 -i ./.vagrant/machines/kernel-update/virtualbox/private_key

Проверяем версию ядра

    uname -r
	
Должна стать "6.10.9"

После выполнения задания выходим из ВМ:

    exit

И уничтожаем эту ВМ:

    vagrant destroy

## Готово
