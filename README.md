
**Описание проекта.**
Для любой организации не будет лишним создать свою локальную сеть и настроить взаимодействие между устройствами в даной сети. Даный проект разрабатывается для того, чтобы попрактиковатся в создании и настройке различных сервисов и серверов. Цель проекта заключается в том, чтобы создать виртуальные машины (использовав облачный провайдер и любой продукт виртуализации), обеспечить сеть всем необходимым для достижения надежности и безопасности сети, Создать VPN-сервер (через который можно будет выходить в Интернет), написать различные скрипты и собрать deb-пакеты для быстрого развертывания того или иного сервиса.

**Требования к инфраструктуре**
1. Роутер
2. Компьютер
3. Брандмауэр и остальные методы защиты системы.
4. Резервное копирование системы и некоторых сервисов
5. Использование Yandex Cloud для создания VPN-сервера
6. Антивирусная защита
7. Системы мониторинга и управления ресурсами (Prometheus)
8. Обновление программного обеспечения

**Для дальнейшей работы нужно воспользоваться скриптом "tools.sh" в папке "scripts", который установит все необходимые инструменты и программы.**


**Как пользоваться и развернуть проект**
От использования VPN, клиент получит безопасное соединение, так как OpenVPN поддерживает безопасность SSL/TLS, мосты Ethernet, туннельную транспортировку TCP или UDP через прокси-серверы или NAT, поддержку динамических IP-адресов и DHCP, переносимость на большинство основных платформ ОС.
Для того чтобы развернуть проект нужно скачать данный архив и разархивировать его в систему. 
Далее необходимо:
1. Развернуть удостоверяющий центр для выдачи сертификатов (sudo dpkg -i easyrsa_0.1-1_all.deb) 
2. Развернуть VPN-сервер (sudo dpkg -i opvpn_0.1-1_all.deb)
3*. Если VPN-сервер уже установлен в системе, то можно разархивировать архив "clients_0.1.orig.tar.xz" в /etc/openvpn.
4. Установить на клиентский компьютер OpenVPN.
5. После установки воспользоватся файлом "2.ovpn" (Импорт -> импорт файла конфигурации -> вибираем "2.ovpn")
6. Происходит подключение.

**Мониторинг Prometheus**
Prometheus будем использовать для мониторинга и оповещения системы VPN-сервера и клиента. Prometheus собирает и хранит свои метрики в виде данных временных рядов, т. е. информация о метриках сохраняется с отметкой времени, в которой она была записана, а также с необязательными парами ключ-значение, называемыми метками. Оповещение с помощью Prometheus разделено на две части. Отправляют оповещения в Alertmanager. Затем Alertmanager управляет этими оповещениями, включая подавление, блокировку, агрегацию и отправку уведомлений с помощью таких методов, как электронная почта, системы уведомлений по вызову и платформы чата.
В данном случае все отчеты о ошибках на сервере будут отсылаться на электронную почту и хранится там. 

**Для того чтобы развернуть сервис мониторинга системы Promеrtheus нужно воспользоваться скриптом prometheus.sh. Далее:**
1. Находим в архиве файл с названием "prom_0.1-1_all.deb"
2. Дальше нужно воспользоваться уже созданым deb-пакетом для установки Prometheus. Выполняем команду - sudo dpkg -i prom_0.1-1_all.deb

После выполения этих этапов нужно перейти в: sudo vim /etc/nginx/sites-available/site-local.conf
В этом файле нужно прописать:
...
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	listen 8080 default_server;	
}
...
location /stub_status{
	stub_status;
	allow 127.0.0.1;
	deny all;
}
Вносим изменения в iptables с помощю скрипта iptables.sh (sudo ./iptables.sh) расположеном в папке "scripts"

**/*---  MySQL   ---*/**
С помощью файла, который использовали ранее ("tools.sh") в системе уже установлен MySQL и использован скрипт безопасности для MySQL.
Проверяем работоспособность (sudo service mysql status)

**/*---    Описание системы резервного копирования    ---*/**

1. Использование Yandex Disk, DropBox, OneDrive и т.д., для хранения backup-ов системы.
2. Может перестать работать MySql и все созданные базы данных пропадут. Создаем бэкап MySql с помощью встроенной функции mysqldump. Сам файл -> dumpsql.sql
	sudo mysqldump -u root mysql > /tmp/dump.sql
3. Если возникнет необходимость Создаем бэкап папки /etc/.   Файл -> "backetc.tar.gz"
	Файл - tar -czf archive.tar.gz /etc/ 

5. Бэкап всего диска: sudo tar czf /backup.tar.gz --exclude=/backup.tar.gz --exclude=/home --exclude=/media --exclude=/dev --exclude=/mnt --exclude=/proc --exclude=/sys --exclude=/tmp /
	Файл -> script/bckscript.sh

**/*---    Описание системы резервного копирования   ---*/**
Состав серверов проекта, схема взаимодействия серверов написана в файле "документ о системе для системного администратора.docx"
Идеи для улучшения и модернизации системы более детально расписаны в файле - "Документ идей для улучшения системы.xlsx"

**/*--- Oписание файлов и их назначение ---*/**
Папка "Скриншоты":
Скриншот с успешно изданным корневым сертификатом.png
Скриншот с успешным подключением клиента к VPN-серверу№1.png
Скриншот с успешным подключением клиента к VPN-серверу№2.png
Скриншот с созданными алертами и несколько дней исторических данных метрик.png
Пример алерта.png
Папка "scripts":
bckscript.sh - скрипт, который создает резервную копию папки /etc/; текстовый файл, в котором запишутся все установленные программы и бэкап БД.
chmod.sh - скрипт для изменения прав доступа к файлам и каталогам
firewall.sh - разрешения и запрет некоторых портов и соединений 
iptables.sh - дополнительные настройки для создания VPN-сервера
make_config.sh - дополнительные настройки для создания VPN-сервера
prometheus.sh - установка prometheus и остальных сервисов
rkhunter.sh - установка rkhunter (это сканер различных видов локальных (потенциальных) уязвимостей (бэкдоров, эксплоитов и руткитов) со своей регулярно обновляемой базой.)
Файлы:
2.ovpn - OpenVPN Config File для подключения VPN.
backuprograms.txt - backup всех установленых в системе програм
dumpsql.sql - backup БД.
Архивы и deb-пакеты:
clients_0.1.orig.tar.xz - архив клиетнской информации для /etc/openvpn/
easyrsa (deb-пакет):
easyrsa_0.1.orig.tar.xz
easyrsa_0.1-1.debian.tar.xz
easyrsa_0.1-1.dsc
easyrsa_0.1-1_all.deb
easyrsa_0.1-1_amd64.build
easyrsa_0.1-1_amd64.buildinfo
easyrsa_0.1-1_amd64.changes
keys (deb-пакет):
keys_0.1.orig.tar.xz
keys_0.1-1.debian.tar.xz
keys_0.1-1.dsc
keys_0.1-1_all.deb
keys_0.1-1_amd64.build
keys_0.1-1_amd64.buildinfo
keys_0.1-1_amd64.changes
opvpn (deb-пакет openvpn):
opvpn_0.1-1.debian.tar.xz
opvpn_0.1-1.dsc
opvpn_0.1-1_all.deb
opvpn_0.1-1_amd64.build
opvpn_0.1-1_amd64.buildinfo
opvpn_0.1-1_amd64.changes
prom (deb-пакет prometheus):

**Дополнительные комментарии:**
БД - база данных.

/*---    Руководство пользователя VPN    ---*/
На стороне клиента должна быть установлена программа OpenVpn.
1. Открываем данное приложение.
2. Нажимаем правой кнопкой мыши на графический интерфейс OpenVpn, выбираем "Импорт".
3. Выбираем из выпадающего списка пункт "Импорт файла конфигурации".
4. Воспользоватся файлом "2.ovpn" и нажимаем кнопку "Открыть".
5. Происходит подключение.
/*---    Руководство пользователя VPN    ---*/

/*---    Инструкция для самостоятельного получения сертификата    ---*/
1.	easyrsa gen-req server nopass
2.	easyrsa sign-req server server
3.	sudo cp pki/issued/server.crt /etc/openvpn/server/ - переместить сертификат в другую папку.
4.	sudo cp pki/ca.crt  /etc/openvpn/server/
5.	/usr/sbin/openvpn --genkey --secret ta.key  - генерируем ключ
6.	sudo cp ta.key /etc/openvpn/server/ - перемещаем созданный ключ в другую папку 
7.	./easyrsa gen-req client-1 nopass – генерируем ключ клиента
8.	cp pki/private/client-1.key ../clients/keys/
9.	./easyrsa sign-req client client-1 и далее настраиваем, то что предлагают заполнить
10.	cp ta.key ~/clients/keys/
11.	cp pki/issued/client-1.crt ~/clients/keys/
12.	cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
13.	sudo vim /etc/openvpn/server/server.conf – изменяем некоторые данные 
a.	tls-auth ta.key 0 меняем на tls-crypt ta.key
b.	ищем строки и добавляем: cipher AES-256-GCM auth SHA256
c.	dh none
d.	снимаем комментарий с: user nobody и group noqroup
14.	sudo vim /etc/sysctl.conf – переходим в этот файл и снимаем комментарий со строки net.ipv4.ip_forward=1 и выполняем команду «sudo sysctl -p»
15.	cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/clients/base.conf и делаем на подобии пункта /

*---    Инструкция для самостоятельного получения сертификата    ---*/