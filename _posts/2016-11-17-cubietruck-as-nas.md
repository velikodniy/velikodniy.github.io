---
layout: post
title:  NAS на базе Cubietruck
---

О CubieTruck
============

Пару лет назад я купил [CubieTruck](http://www.cubieboard.org/) —
одноплатный компьютер на основе SoC архитектуры ARM. Тогда он подкупил меня
тем, что имел аппаратную поддержку SATA, и к нему можно было подключить
жёсткий диск напрямую без переходников, как в RaspberryPi.

Основные характеристики:

* AllWinnerTech SOC A20, ARM® Cortex™-A7 Dual-Core, ARM® Mali400 MP2 Complies with OpenGL ES 2.0/1.1
* 2GB DDR3 @480MHz (960MTPS)
* Видеовыходы: HDMI & VGA 1080p
* 10M/100M/1G Ethernet
* Wifi+BT беспроводные интерфейсы с антенной на плате
* SATA 2.0 интерфейс, поддерживающий 2.5″ HDD, (для 3.5″ HDD потребуется дополнительное 12 V питание)
* Хранение данных: icroSD
* 2 × USB HOST, 1 × OTG, 1 × Toslink (SPDIF Optical), 1 × IR, 4 × LED, 1 аудио, 3 кнопки
* Питание: DC 5 V @ 2.5 A с HDD, поддержка литиевых батареек для RTC
* 54 пина, поддерживающих I2S, I2C, SPI, CVBS, LRADC x2, UART, PS2, PWMx2, TS/CSI, IRDA, LINEIN&FMIN&MICIN, TVINx4 with 2.0 pitch connectors
* Размер платы: 11 cm × 8 cm × 1.4 mm, хорошо подходит подходит для установки 2.5″ HDD

Об этом компьютере не раз писали в Интернете:

* [Cubietruck. Уютный, домашний сервер](https://habrahabr.ru/post/225371/)
* [Обсуждение Cubietruck (от создателей Cubieboard)](https://habrahabr.ru/post/186576/)

Хотя сейчас есть одноплатные компьютеры и помощнее при меньшей цене,
эта система всё ещё актуальна.

С самого начала предполагалось, что плата будет работать без монитора
как [NAS](https://ru.wikipedia.org/wiki/NAS). Она бесшумная, шум
создаёт только жёсткий диск.

В качестве операционной системы поддерживается как Linux, так и
Android. Естественно, для NAS лучше подходит первый.

Ниже описаны шаги, позволяющие построить файлохранилище и
автоматическую торренто-качалку. Конечно, можно было бы добавить ещё
много чего, возможности этого компьютера очень велики.

Предполагается, что читатель немного знаком с работой в командной
строке Linux.

Питание
=======

Вопрос питания довольно важный, если к плате подключен жёсткий
диск. Сначала я питал его двух зарядных устройств: одно было
подключено к основному разъёму питния, а второе к microUSB. Они могут
использоваться одновременно. Но это неудобно и возникали сложности с
отключением от питания.

Поэтому я купил в Китае блок адаптер, способный выдавать 3 А, с
разъёмом USB. Его оказалось достаточно для запуска платы и старта
жёсткого диска. Кстати, как я понял, большой ток нужен лишь для старта,
во рвемя работы диск поребляет меньше.

Установка базовой системы
=========================

[Предлагаемые официально
образы](http://docs.cubieboard.org/tutorials/a10-cubieboard_lubuntu_server_releases)
основаны на Lubuntu (ядро
[Linaro](https://ru.wikipedia.org/wiki/Linaro) 13.06) и, мягко говоря,
устарели. Если хочется установить свежее программное обеспечение, придётся повозится.
После одной из попыток обновиться пакеты у меня система и вовсе перестала загружаться.
Я уж не стал искать причины, а попробовал найти альтернативы.

Оказывается, существует дистрибутив
[Armbian](https://www.armbian.com/) (бывший Cubian), поддерживающий
широкий спектр одноплатных компьютеров на ARM. Он основан на актуальных версиях
Ubuntu и Debian и постоянно обновляется.

[Образы для Cubiebtruck (Cubieboard 3)](http://www.armbian.com/cubietruck/)

Для «безголового сервера» больше подходит ветка «Mainline». Если же
нужен компьютер с монитором и поддержкой мультимедиа, то нужно
устанавливать ветку «Legacy».

Для каждой ветки можно выбрать дистрибутив. Педполагается, что в
дальнейшем используется образ Xenial Server, хотя разница в настройке
будет незначительной.

Установка образа
----------------

Про это довольно много написано в Интернете. Установка выполняется
очень просто. На «большом компьютере» скачиваем образ и записываем его
побайтно на карточку:

~~~ bash
sudo dd bs=1M if=filename.img of=/dev/sdX status=progress
sync
~~~

Не забудьте заменить `/dev/sdX` на имя устройства, соответствующего карточке!

Команда `dd` предназначена для побайтового копирования файлов блоками
указанного размера. Это размер задаётся параметром `bs`. В нашем
случае он равен 1 мегабайту. `if` и `of` — входной и выходной файлы
соответственно. А параметр `status=progress` заставляет во время
работы выводить информация о том, сколько байт уже скопировано, чтобы
мы не скучали.

Вставляем карточку и включаем систему. Первый запуск образа дольше
обычного, так как создаётся swap и образ «растягивается» на всю
карточку.

Первый запуск нужно проводить подключив монитор и клавиатуру, чтобы следить
за диагностическими сообщениями и, если нужно, что-то подправить.

Хотя, хоть я это и не пробовал, наверное, можно после загрузки сразу
подключиться через [SSH](https://ru.wikipedia.org/wiki/SSH). Главное
узнать IP-адрес — он по умолчанию определяется динамически через DHCP.

Первоначальная настройка
========================

По умолчанию вход выполняется от имени суперпользователя (`root`) с паролем `1234` (ха!).

При первом запуске система сама запрашивает логин и пароль, а также
создаёт пользователя. И это правильно — лучше не сидеть от имени `root`, чтобы что-то
не сломать и не подвергать компьютер лишнему риску.

Я создал пользователя `vadim`.

Сразу обновим систему:

~~~ bash
sudo apt update
sudo apt upgrade
~~~

Зададим имя нашему компьютеру (я назвал его archive):

~~~ bash
sudo hostnamectl set-hostname archive
~~~

Запуск команд без пароля
------------------------

Чтобы пользователю `vadim` можно было запускать команды от имени
суперпользователя без запроса пароля, нужно подправить файл
`/etc/sudoers`. Его нужно открыть на редактирование командой
`visudo`. Добавим строчку:

~~~
vadim ALL=(ALL) NOPASSWD:ALL
~~~

Кто-то может счесть это дырой в безопасности. Однако, если
злоумышленник сможет зайти от имени пользователя `vadim`, то это будет
меньшей из проблем, так как он получит доступ к файлам. В противном
же случае эта настройка на безопасность никак не влияет.

Локализация
-----------

Устанавливаем пакет с переводами:

~~~ bash
sudo apt install language-pack-ru
~~~

Установка системной локали (язык, кодировка и разные настройки, связанные с ними):

~~~ bash
sudo localectl set-locale LANG=ru_RU.UTF-8
~~~

Настройка часового пояса:

~~~ bash
sudo timedatectl set-timezone Europe/Chisinau
~~~

Для переключения раскладки клавиатуры на русский язык правим файл
`/etc/default/keyboard`:

~~~
XKBLAYOUT="us,ru"
XKBVARIANT=","
XKBOPTIONS="grp:alt_shift_toggle"
~~~

После этого раскладка будет переключаться по Alt+Shift.

Затем:

~~~ bash
sudo dpkg-reconfigure kbd
~~~

Если требуется тонкая настройка шрифтов в консоли:

~~~ bash
sudo dpkg-reconfigure console-setup
~~~

Последние шаги нужно выполнять только если предполагается сидеть в
терминале через подключенный монитор. При работе через SSH это ни на
что не влияет.

Полезные программы
==================

Тут каждый ставит то, что хочет. Например, для установки пакетов я
использую aptitude. Но здесь он уже есть из коробки.

Поэтому доустановим лишь [Midnight
Commander](https://ru.wikipedia.org/wiki/Midnight_Commander) и
текстовый редактор [jed](http://www.jedsoft.org/jed/) (легкий вариант моего любимого emacs):

~~~ bash
sudo apt install mc jed
~~~

Настройка сети
==============

Чтобы было удобнее подключаться, зададим нашему NAS статический
IP-адрес. (Его нужно задавать так, чтобы он находился в той же
подсети, что и остальные домашние компьютеры, и чтобы он не попадал в
диапазон раздаваемых через DHCP адресов. Поэтому у вас адрес может
быть и другим.)

У меня шлюз (в его роли выступает роутер) имеет адрес 192.168.0.254.

Итак, зададим статический IP 192.168.0.1 со шлюзом 192.168.0.254.

Есть два варианта настройки сети. (Спойлер: второй проще и скорее всего вам нужен именно он.)

1 способ. Настройка сети через systemd
--------------------------------------

Создаём файл `/etc/systemd/network/wired.network` с содержимым:

~~~
[Match]
Name=eth0

[Network]
Address=192.168.0.1
Gateway=192.168.0.254
DNS=8.8.8.8
~~~

Включаем автонастройку сети через systemd

~~~ bash
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd
~~~

После этого systemd будет добавлять к указанному интерфейсу
статический адрес. Но и адрес, полученный через DHCP тоже будет
работать. Это бывает удобно.

2 способ. Настройка сети через /etc/network/interfaces
------------------------------------------------------

Настройка сводится к правке `/etc/network/interfaces`:

~~~
allow-hotplug eth0
no-auto-down eth0
iface eth0 inet static
    address 192.168.0.1
    netmask 255.255.255.0
    gateway 192.168.0.254

auto lo
iface lo inet loopback
~~~

Для настройки DNS добавим в `/etc/resolv.conf` строчки:

~~~
nameserver 8.8.8.8
nameserver 4.4.4.4
~~~

Это DNS от Google. Можете указать и свои. У меня, например, на самом
деле прописан адрес роутера, который умеет кэшировать DNS-запросы.

После настройки нужно перезапустить сетевую подсистему:

~~~ bash
sudo systemctl restart networking
~~~

Или же можно просто перезагрузиться.

Настройка WiFi
--------------

Если не хочется тянуть к NAS сетевой провод, можно использовать
WiFi. Настраивается так же, как и проводная сеть.

Удалённый доступ
================

В дальнейшем планируется подключаться к NAS через SSH. Чтобы не
вводить каждый раз пароль настроим доступ по ключу.

Проверка состояния сервера SSH (он установлен из коробки):

~~~ bash
sudo systemctl status ssh
~~~

Все дальнейшие действия выполняются на большом компьютере (который будет подключаться).

Генерация ключа:

~~~ bash
ssh-keygen -f ~/.ssh/archive
~~~

Пароль можно оставить пустым. (Тогда, главное, ключ никому не давайте!)

Добавление ключа в связку:

~~~ bash
ssh-add ~/.ssh/archive
~~~

Копирование ключа на сервер:

~~~ bash
ssh-copy-id -i ~/.ssh/archive.pub vadim@192.168.0.1
~~~

Теперь к NAS можно подключиться без пароля:

~~~ bash
ssh vadim@192.168.0.1
~~~

Жёсткий диск
============

Для начала создадим системную группу для работы с жёстким диском. Все
пользователи, от имени которых работают демоны (системный фоновые
процессы), должны быть включены в эту группу, так как у диска будут
права этой группы.

~~~ bash
sudo addgroup --system data
~~~

Включаем основного пользователя в эту группу:

~~~ bash
sudo usermod -a -G data vadim
~~~

Создание директории для монтирования жёсткого диска:

~~~ bash
sudo mkdir /mnt/data
~~~

Для автоматического монтирования добавляем диск в `/etc/fstab`:

~~~
/dev/sda1    /mnt/data    ext4    defaults    0    2
~~~

Здесь `/dev/sda1` — имя устройства, связанного с жёстким
диском. Скорее всего оно будет так называться, хотя гарантий нет.

Вместо `/dev/sda1` можно указать UUID, который отображается командой blkid.

У каталогов, с которыми будут работать разные программы, должны быть права

~~~
vadim:data 2775
~~~

* 2 — SGID, чтобы для внутренних файлов использовались права группы владельца каталога.
* 7 — Пользователь может всё.
* 7 - Группа пользователя может всё.
* 5 — Остальные могут только читать и смотреть содержимое каталога.

Если на диске уже есть файлы, для смены прав доступа могут быть полезны команды:

~~~ bash
find . -type d -exec chmod 2775 {} +
find . -type f -exec chmod 2664 {} +
~~~

Они находят все файлы и подкаталоги в текущем (не забудьте в него
перейти) и меняют права.

Установка владельца для всех файлов и каталогов в текущем:

~~~ bash
sudo chown -R vadim:data ПУТЬ
~~~

Торренты
========

Настроим автоматическую закачку торрентов по расписанию.

Установка Transmission
----------------------

В качестве «качалки» будет испоьзовать Transmission. У него есть
вариант без графического интерфейса. Управлять программой можно через
браузер или с помощью приложений, которые подключаются к нему по сети.

Я, например, использую
[Torrnado](https://play.google.com/store/apps/details?id=com.gabordemko.torrnado)
для Android.

Установка демона:

~~~ bash
sudo apt install transmission-daemon
~~~

Добавляем демона к группе data:

~~~ bash
sudo usermod -a -G data debian-transmission
~~~

Для настройки демона нужно сперва его остановить:

~~~ bash
sudo systemctl stop transmission-daemon
~~~

Конфигурация демона находится в файле
`/etc/transmission-daemon/settings.json`. Исправим следующие
параметры:

~~~
"download-dir": "/mnt/data/downloads",
"incomplete-dir": "/mnt/data/downloads",
"rpc-whitelist-enabled": false,
"port-forwarding-enabled": true,
"umask": 2,
~~~

Здесь `umask=2` (или же 002) вместо 18 (=022) по умолчанию задаёт
файлам права 774 вместо 744. Этот параметр задаёт биты маски прав
доступа, которые не устанавливаются.

Пользователь и пароль по умолчанию: transmission:transmission. Чтобы
задать пароль нужно вписать его в поле rpc-password.

Для пользователя и пароля vadim:PASSW0RD:

~~~
    "rpc-password": "PASSW0RD",
    "rpc-username": "vadim",
~~~

После перезапуска Transmission сам его хеширует и в следующий раз в
конфигурации вместо пароля будет хеш.

Выключить пароль можно опцией `rpc-authentication-required`.

Если поменять URL для RPC, то некоторые программы могут не
работать. Будьте внимательны!

Запуск Transmission:

~~~ bash
sudo systemctl start transmission-daemon
~~~

Установка Monitorrent
---------------------

Раньше я пользовался [Flexget](https://flexget.com/) — это необычайно
мощная качалка файлов, которая, пожалуй может всё. Но поддавшись
модным тенденциям перешёл на
[Monitorrent](https://habrahabr.ru/post/305574/). У него чуть лучше
поддержка популярных трекеров, но он не умеет, например, скачаивать
премьеры. И с уведомлениями у него пака не очень. Но это всё
поправимо.

Проще всего устанавливать его через
[Docker](https://ru.wikipedia.org/wiki/Docker). Для этого нужно
установить сам Docker.

~~~ bash
sudo apt install docker.io
~~~

Теперь устанавливаем образ с Monitorrent:

~~~ bash
sudo docker pull werwolfby/armhf-alpine-monitorrent
~~~

Просмотр образов Docker:

~~~ bash
sudo docker images
~~~

Создание контейнера без его запуска:

~~~ bash
sudo docker create -p 6687:6687 -v /mnt/data/monitorrent.db:/var/www/monitorrent/monitorrent.db:rw werwolfby/armhf-alpine-monitorrent /usr/bin/python server.py
~~~

Здесь при помощи параметра -v файл хоста (база настроек) отображается
в контейнер. Это нужно, чтобы при обновлении или перезапуске
контейнера база не стёрлась.


Следующей командой можно получить список контейнеров. Для каждого
указан его ID (хеш) и кодовое имя. Для запуска и других операций можно
использовать кодовое имя.

~~~ bash
sudo docker ps -a
~~~

Контейнер переименуем в monitorrent:

~~~ bash
sudo docker rename ИМЯ monitorrent
~~~

Для автоматического запуска требуется service-файл `/etc/systemd/system/monitorrent.service`:

~~~
[Unit]
Description=Monitorrent
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a monitorrent
ExecStop=/usr/bin/docker stop -t 2 monitorrent

[Install]
WantedBy=default.target
Запуск и настройка автостарта после перезагрузки:
systemctl daemon-reload
systemctl start monitorrent.service
systemctl enable monitorrent.service
~~~

Затем настраивается сам Monitorrent по адресу
http://192.168.0.1:6687. Не забуьте задать пароль на вход в
Monitorrent.

Samba
=====

Для доступа к файлам с компьютеров под управлением Windows нужна Samba.

Установка:

~~~ bash
sudo apt install samba
~~~

Системный пользователь для входа:

~~~ bash
sudo adduser --system --home /var/lib/samba/home samba
sudo usermod -a -G data samba
~~~

Добавление пользователя в Samba: 

~~~ bash
sudo pdbedit -a -u samba
~~~

Пример конфигурации `/etc/samba/smb.conf`:

~~~
[global]
workgroup = HOME
netbios name = ARCHIVE 
server string = %h server (Samba, Ubuntu) 

log file = /var/log/samba/log.%m 
max log size = 1000 
syslog = 0 
panic action = /usr/share/samba/panic-action %d 

server role = standalone server 

passdb backend = tdbsam 
obey pam restrictions = yes 

unix password sync = yes 
passwd program = /usr/bin/passwd %u 
passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* . 
pam password change = yes 

map to guest = bad user 

create mask = 0664 
force create mode = 0664 
directory mask = 0775 
force directory mode = 0775 
read only = no 

[downloads] 
path = /mnt/data/downloads 
comment = Загруженные файлы 
~~~

Можно добавть ещё несколько каталогов и использовать их для хранения
или обмена файлами между компьютерами в локальной сети.

Dejadup
=======

Для бэкапов я использую DejaDup. Он может пересылать бэкапы через SSH
(точнее через SFTP). Для этого создадим пользователя, от имени
которого он будет подключаться.


Добавление пользователя:

~~~ bash
sudo adduser --system --home /var/lib/dejadup --shell /bin/sh dejadup
~~~

Пользователь в группе nogroup, без шелла sftp работать не будет.

Файлы хранятся в /mnt/data/backups. Нужно дать права на эту директорию пользователю dejadup:

~~~ bash
sudo chown -R dejadup:data /mnt/data/backups
~~~

Что дальше?
===========

К сожалению, многие вещи не были освещены:

* Syncthing — автоматическая синхронизация файлов между многими устройствами.
* MPD-сервер — подключив колонки к CubieTruck можно крутить на нём музыку, управляя вопроизведением с компьютера или телефона.
* DLNA — автоматическое перекодирования мультимедия для воспроизведения с устройств.
* VPN — использование CubieTruck как VPN-сервера.
* и многое другое.

Может быть, позже я дополню этот пост.