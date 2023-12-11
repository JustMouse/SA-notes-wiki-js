---
title: ДемоЭкзамен 2024 (СиСА-Профиль)
description: 
published: true
date: 2023-12-11T21:21:50.332Z
tags: 
editor: markdown
dateCreated: 2023-12-05T23:48:18.509Z
---

# Решение Демонстрационного Экзамена 2024 <br>Сетевое и Системное Администрирование (Профиль)
*Версия от 12.12.2023*
## Ссылки
[Официальная документация 2023](https://bom.firpo.ru/Public/83)
### Загрузка решения в виде документа
- [PDF](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.pdf)
- [Word](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.word)
- [HTML](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.html)
- [MD](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.md)
## Предисловие
1. Модули настоящего задания очень сильно связаны друг с другом. Например, это касается VPN (Модуль 3. Пункт 7) и Контроллера домена (Модуль 2. Пункт 3). Несмотря на то, что они находятся в разных модулях логичнее сначала настроить всю сетевую связанность, в том числе и VPN, а уже после Контроллер домена.
2. По идеи задания временное подключение используется для быстрого ввода устройства CLI в домен. Тем не менее в данном решении данное подключение не осуществляется, так как было принято решение предоставить устройству CLI функциональный VPN до HQ-R, что обосновано необходимостью для студентов навыками в умениями в данной дисциплине.
3. Настоящее задание предполагает, что в стенд не имеет доступа к Интернету, тем не менее в данном решение Интернет подключён к устройству ISP для скачивание пакетов с общедоступного репозитория.
4. Выбор операционной системы зависит от образовательной организации. В данном решении предполагается использования только [**RedOS 7.3.3 Minimal installation**](https://files.red-soft.ru/redos/7.3/x86_64/iso/redos-MUROM-7.3.3-20230815.0-Everything-x86_64-DVD1.iso) и [**RedOS 7.3.3 Workstation (MATE)**](https://files.red-soft.ru/redos/7.3/x86_64/iso/redos-MUROM-7.3.3-20230815.0-Everything-x86_64-DVD1.iso)
## Предлагаемая таблица IP-адресов

| Устройство | IP-адрес             | Сеть     |
| ---------- | -------------------- | -------- |
| ISP        | 192.168.106.X (DHCP) | Internet |
|            | 172.16.10.1/30       | ISP-HQ   |
|            | 172.16.20.1/30       | ISP-BR   |
|            | 192.168.111.1/24     | ISP-CLI  |
| HQ-R       | 172.16.10.2/30       | ISP-HQ   |
|            | 192.168.10.1/26      | HQ-NET   |
| BR-R       | 172.16.20.2/30       | ISP-BR   |
|            | 192.168.20.1/28      | BR-NET   |
| hq-srv     | 192.168.10.5/26      | HQ-NET   |
| BR-SRV     | 192.168.20.5/28      | BR-NET   |
| CLI        | 192.168.111.2/24     | ISP-CLI  |



## Топлогия

![demo24-topology.png](/demo24-topology.png)

## Модуль 1. Базовая настройка

1. Выполните базовую настройку всех устройств:

   a. Присвоить имена в соответсвии с топологией. На **ВСЕХ** машинах. Пункт проверяется на всех устройствах

```bash
redos$ hostnamectl set-hostname [Имя] # Например: hostnamectl set-hostname ISP
redos$ hostnamectl set-hostname [Имя.домен] # Используйте в локальных сетях организации, таким образом:
# hq-srv: hq-srv.hq.work
# BR-R: br-r.branch.work
# И т.д.
```

   b. Рассчитайте IP-адресацию IPv4 и IPv6. *см. Таблицу IP-адресов.*

   c. Пул адресов для BRANCH - не более 16 адресов. **Это маска 28**

   d. Пул адресов для HQ - не более 64 адресов. **Это маска 26**

   **Продолжение следующих пунктов** требует *базовой связанности* не указанной в задании. Убедитесь, что все машины выход в Интернет и на устройствах **ISP, HQ-R, BR-R** настроен **NAT (PAT)**.

```bash
# Настройка iptables на маршрутизаторах
redos$ dnf install iptables-services
redos$ iptables -t nat -A POSTROUTING -o ens* -j MASQUERADE # Вместо '*', используется интерфейс в сторону ближе к Интернету
redos$ systemctl enable iptables # Включаем автозагрузку
redos$ iptables-save > /etc/sysconfig/iptables # Сохраняем конфигурацию
#===#
# Можно посмотреть или удалить правило iptables
redos$ iptables -t nat -D POSTROUTING 1 # Удаляет правило POSTROUTING ПЕРВОЕ
redos$ iptables -t nat -F # Удалить все правила таблицы nat
redos$ iptables -t nat -S # Посмотреть все правила таблицы nat
redos$ iptables -S # Посмотреть все правила таблицы FILTER
```

```bash
# Настройка включение маршрутизации на маршрутизаторах
redos$ nano /etc/sysctl.conf
--- # Дописать строку
net.ipv4.ip_forward=1
---
redos$ sysctl -p # После ввода команды должна появиться только что написанная команда. Если это не так, то проверьте на опечатки. Сделать данную настройку нужно на ISP, HQ-R, BR-R
```

```bash
# Настройка сетевых интерфейсов
redos$ nmtui # Открывает псевдографика с настройками
redos$ Изменить подключение > [Интерфейс] Изменить > [Поменяйте Имя профиля, Конфигурацию IPv4: Выберите между Автоматически и Вручную (DHCP и Static), Укажите IP-адрес с маской, Укажите шлюз, если он не настраивается по DHCP, Установите Серверы DNS 2 шткуи, если они не настраиваются по DHCP: Они должны быть 192.168.10.5(hq-srv), 8.8.8.8(Google DNS) Именно в таком порядке]
# Включите и выключите интерфейсы, если они не обновляются автоматически после изменений. Или перезагрузите машину
```

2. Настройте внутреннюю динамическую маршрутизацию по средствам **FRR**. Это будет **OSPF** между туннелями HQ-R и BR-R. Сначала должен быть настроен **туннель**.

### VPN

```bash
HQ-R$ dnf install wireguard-tools frr # Wireguard - VPN, FRR - OSPF
HQ-R$ cd /etc/wireguard
HQ-R$ wg genkey | tee HQ-R.key | wg pubkey > HQ-R.pub # .key - приватный ключ; .pub - публичный ключ
HQ-R$ wg genkey | tee BR-R.key | wg pubkey > BR-R.pub # На HQ-R должен быть приватный HQ-R и публичный BR-R, а на BR-R наоборот
HQ-R$ nano wg.conf
--- # Представлен весь файл
[Interface]
PrivateKey = ? # Вместо '?' вставляем приватный ключ HQ-R
Address = 10.0.0.1/29
ListenPort = 51820
Table = off
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ BR-R
AllowedIPs = 10.0.0.2/32, 192.168.20.0/28, 0.0.0.0/0
---
HQ-R$ cat HQ-R.key >> wg.conf # Вставляем в файл ДЛИННУЮ строку-ключ с помощью '>>'. После вырезаем строку Ctrl+K и вставляем Ctrl+U
HQ-R$ cat BR-R.pub >> wg.conf
HQ-R$ wg-quick up wg
HQ-R$ systemctl enable wg-quick@wg
HQ-R$ nano /etc/ssh/sshd_config
---
PermitRootLogin yes;
---
HQ-R$ systemctl restart sshd
BR-R$ dnf install wireguard-tools frr
BR-R$ scp root@172.16.10.2:/etc/wireguard/* /etc/wireguard/ # Переносим ключики
BR-R$ nano /etc/wireguard/wg.conf # Заменяем ключи
--- # Представлен весь файл
[Interface]
PrivateKey = ? # Вместо '?' вставляем приватный ключ BR-R
Address = 10.0.0.2/29
Table = off
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ HQ-R
Endpoint = 172.16.10.2:51820
AllowedIPs = 10.0.0.1/32, 192.168.10.0/26, 0.0.0.0/0
---
BR-R$ cat HQ-R.pub >> wg.conf # Вставляем в файл ДЛИННУЮ строку-ключ с помощью '>>'. После вырезаем строку Ctrl+K и вставляем Ctrl+U
BR-R$ cat BR-R.key >> wg.conf
BR-R$ nano /etc/ssh/sshd_config
---
BR-R$ wg-quick up wg
BR-R$ systemctl enable wg-quick@wg
#===ПРОВЕРКА===#
HQ-R$ wg show
BR-R$ wg show
HQ-R$ ping 10.0.0.2
HQ-R$ ping 192.168.20.1
BR-R$ ping 10.0.0.1
BR-R$ ping 192.168.10.1
```

### OSPF (FRR)

```bash
#===HQ-R===#
HQ-R$ apt install frr
HQ-R$ nano /etc/frr/daemons
--- # В файле поменять следующую строку
ospfd=yes
---
HQ-R$ systemctl restart frr
HQ-R$ vtysh
HQ-R$ $ conf t
HQ-R$ (conf)$ ip forwarding
HQ-R$ (conf)$ router ospf
HQ-R$ (conf-router)$ network 192.168.10.0/26 area 0
HQ-R$ (conf-router)$ network 10.0.0.0/29 area 0
HQ-R$ (conf-router)$ neighbor 10.0.0.2
HQ-R$ (conf-router)$ exit
HQ-R$ (conf)$ interface wg
HQ-R$ (conf-if)$ ip ospf network broadcast
HQ-R$ (conf-if)$ end
HQ-R$ $ wr
HQ-R$ $ exit
HQ-R$ systemctl enable frr
HQ-R$ systemctl restart frr
#===BR-R===#
BR-R$ apt install frr
BR-R$ nano /etc/frr/daemons
--- # В файле поменять следующую строку
ospfd=yes
---
BR-R$ systemctl restart frr
BR-R$ vtysh
BR-R$ $ conf t
BR-R$ (conf)$ ip forwarding
BR-R$ (conf)$ router ospf
BR-R$ (conf-router)$ network 192.168.20.0/28 area 0
BR-R$ (conf-router)$ network 10.0.0.0/29 area 0
BR-R$ (conf-router)$ neighbor 10.0.0.1
HQ-R$ (conf-router)$ exit
HQ-R$ (conf)$ interface wg
HQ-R$ (conf-if)$ ip ospf network broadcast
HQ-R$ (conf-if)$ end
BR-R$ $ wr
BR-R$ $ exit
BR-R$ systemctl enable frr
BR-R$ systemctl restart frr
#===ПРОВЕРКА===#
--- # Проверка с HQ-R
HQ-R$ ip ro # Должен быть маршрут с ospf меткой до сети 192.168.20.0/28
HQ-R$ vtysh
HQ-R$ $ show ip ospf neighbor # Должен быть IP-адрес соседа, сам сосед с режимом(state) Full/Backup или Full/BR
HQ-R$ $ show ip route # Должен быть маршрут с типом "O>*"
--- # Проверка с BR-R
BR-R$ ip ro # Должен быть маршрут с ospf меткой до сети 192.168.10.0/26
BR-R$ vtysh
BR-R$ $ show ip ospf neighbor # Должен быть IP-адрес соседа, сам сосед с режимом(state) Full/Backup или Full/BR
BR-R$ $ show ip route # Должен быть маршрут с типом "O>*"
--- # Проверка с WEB-L с учётом работающего туннеля
hq-srv$ ping 192.168.20.5 # Хорошая проверка. Она проверяет сразу sysctl, iptables, frr. Если что-то не работает проверьте ping на более близкой дистанции. С HQ-R на BR-SRV или BR-R на BR-SRV
```

### DHCP

3. Настройте автоматическое распределение IP-адресов на роутере HQ-R. Имеется ввиду **DHCP** на HQ-R в сторону HQ-NET, то есть для машины hq-srv.

```bash
HQ-R$ dnf install dhcp-server
HQ-R$ nano /etc/dhcp/dhcpd.conf # Если плохо c памятью или медленно пишете, то можно заменить dhcpd.conf файлом-примером
Опционально: HQ-R$ cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
--- # Должны быть
option domain-name "hq.work";
option domain-name-servers 192.168.10.5, 8.8.8.8;
subnet 192.168.10.0 netmask 255.255.255.192 {
	range 192.168.10.2 192.168.10.20;
	option routers 192.168.10.1;
} # Не забыть скобку
host hq-srv { # Резервация IP-адреса
	hardware ethernet xx:xx:xx:xx:xx:xx; # Используйте MAC-адрес на hq-srv
	fixed-address 192.168.10.5;
}
---
HQ-R$ systemctl enable dhcpd # Убедитесь, что на интерфейсе хоста есть IP-адрес в сеть, в которую вы раздаёте IP-адреса.
HQ-R$ systemctl restart dhcpd
#===ПРОВЕРКА===#
hq-srv$ nmtui # Выключите и включите интерфейс. Он должен получать IP-адрес 192.168.10.5, если другой, то проверьте введённый MAC-адрес
#===ТШУТ===#
# Проверьте работу службы, если не работает
redos$ journalctl -e # Обычно dhcpd указывает конкретную строку с ошибкой. Как исправите перезагрузите службу
redos$ systemctl status dhcpd # Посмотреть состоянии службы
```

4. Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

| Имя учётной записи | Пароль   | На каких устройствах | Логин         |
| ------------------ | -------- | -------------------- | ------------- |
| Admin              | P@ssw0rd | CLI hq-srv HQ-R      | admin         |
| Branch admin       | P@ssw0rd | BR-SRV BR-R          | branch_admin  |
| Network admin      | P@ssw0rd | HQ-R BR-R BR-SRV     | network_admin |

```bash
redos$ useradd -c Admin admin # Аргумент после '-c' Имя учётной записи в таблицы. Сам же Логин должен быть в нижнем регистре, а пробелы заменены '_'
redos$ useradd -c "Branch admin" branch_admin
redos$ useradd -c "Network admin" network_admin
redos$ passwd admin # Смените пароль на пользователе, чтобы в него можно было авторизоваться
```

5. Измерьте пропускную способность сети между двумя узлами **HQ-R** и **ISP** по средствам утилиты **iperf3**. Предоставьте описание пропускной способности канала со скриншотами

```bash
redos$ dnf install iperf3 # Установка на двух устройствах
ISP$ iperf3 -s # Включаем iperf3 в роли сервера
HQ-R$ iperf3 -c 172.16.10.1 # Включаем iperf3 в роли клиента и соединяемся с сервером
# Заскриньте 5 строк со скоростью сети с помощью прикладных программа на компьютере или с помощью OpenNebula
```

<img src="Demo24-screen1.png" style="zoom:50%;" />

6. Составьте backup скрипты для сохранения конфигурации сетевых устройств, а именно **HQ-R** и **BR-R**. Продемонстрируйте их работу. **Данный пункт можно** выполнить только после выполнение пункта с **VPN и FRR** [2]

```bash

```

### SSH 2222

7. Настройте подключение по **SSH** для удалённого конфигурирования устройства **hq-srv** по порту **2222**. Учтите, что вам необходимо перенаправить трафик на этот порт по средствам контролирования трафика. Контроль трафика - это межсетевой экран, то есть **iptables**. Сделать настройку нужно на **HQ-R**, так как он виден в сети **Internet**

```bash
HQ-R$ iptables -t nat -A PREROUTING -i ens3 -j DNAT -p tcp --dport 2222 --to-destination 192.168.10.5:22
HQ-R$ iptables-save > /etc/sysconfig/iptables # Не забудьте сохранить правило
hq-srv$ dnf install openssh-server
hq-srv$ nano /etc/ssh/sshd_config
--- # Раскомментируйте и измените следующую строку
PermitRootLogin yes
---
hq-srv$ systemctl enable sshd
hq-srv$ systemctl restart sshd
#===ПРОВЕРКА===#
ISP$ ssh root@172.16.10.2 -p 2222 # Проверка с ISP. Вы должны согласиться с ключом и зайти в shell hq-srv
```
8. Настройте контроль доступа до hq-srv по SSH со всех устройств, кроме CLI

```bash
HQ-R$ iptables -A FORWARD -i ens3 -s 192.168.111.0/24 -p tcp --dport 2222 -j DROP
HQ-R$ iptables-save > /etc/sysconfig/iptables # Не забудьте сохранить правило
```

## Модуль 2.  Организация сетевого администрирования

###  BIND (ОПЦИОНАЛЬНО!!!)

1. Настройте **DNS-сервер** на сервере **hq-srv**. Нужно создать две прямых зоны с A записями и две обратные зоны с PTR записям. Прямые **"hq.work" и "branch.work"**. Обратные **"10.168.192.in-addr.arpa" и "20.168.192.in-addr.arpa"**

```bash
hq-srv$ dnf install bind
hq-srv$ nano /etc/named.conf
--- # Следующие строки должны быть, некоторые нужно изменить, некоторые добавить
listen-on port 53 { any; };
allow-query { any; };
dnssec-validation no;
zone "hq.work" IN {
	type master;
	file "hq.work";
}
zone "branch.work" IN {
	type master;
	file "branch.work";
}
zone "10.168.192.in-addr.arpa" IN { # Обратные зоны будут записаны внутрь базы данных зоны с помощью оператора $ORIGIN
	type master;
	file "hq.work"; # Указываем на ту же базу данных зоны, в ней также будет храниться обратная
}
zone "20.168.192.in-addr.arpa" IN {
	type master;
	file "branch.work";
}
```

Далее копируем базу данных (файлы) зоны по пути **/var/named/** с пустого примера и редактируем их

```bash
hq-srv$ cd /var/named
hq-srv$ cp named.empty hq.work
hq-srv$ nano hq.work
--- # Представлен полный файл
# Пользуйте горячими клавишами: ALT+R - замена. F6 - поиск
$TTL 3H
@	IN	SOA	hq.work	root.hq.work.	(
									1		; serial
									1D		; refresh
									1H		; retry
									1W		; expire
									3H )	; minimum
@		NS	hq.work.
@		A	192.168.10.5
hq-r	A	192.168.10.1
hq-srv	A	192.168.10.5

$ORIGIN 10.168.192.in-addr.arpa
5		PTR	hq-srv.hq.work.
1		PTR	hq-r.hq.work.
---
hq-srv$ cp hq.work branch.work
hq-srv$ nano branch.work
--- # Представлен полный файл
$TTL 3H
@	IN	SOA	branch.work	root.branch.work.	(
									1		; serial
									1D		; refresh
									1H		; retry
									1W		; expire
									3H )	; minimum
@		NS	branch.work.
@		A	192.168.10.5
br-r	A	192.168.20.1
br-srv	A	192.168.20.5

$ORIGIN 20.168.192.in-addr.arpa
5		PTR	br-srv.branch.work.
1		PTR	br-r.branch.work.
---
hq-srv$ systemctl restart named
#===ПРОВЕРКА===#
hq-srv$ named-checkconz -z # Проверяем загружены ли зоны. Должны быть следующие строки:
"zone hq.work/IN: loaded serial 1"
"zone 10.168.192.in-addr.arpa/IN: loaded serial 1"
"zone branch.work/IN: loaded serial 1"
"zone 20.168.192.in-addr.arpa/IN: loaded serial 1"
hq-srv$ journalctl -e # Если что смотрим журнал на наличие ошибок
hq-srv$ cat /etc/resolv.conf # Сервер получает параметры DNS с DHCP на HQ-R. Убедитесь, что у вас ПЕРВЫЙ параметр это:
"nameserver 192.168.10.5" # То есть Ваш hq-srv
hq-srv$ ping hq-r.hq.work
hq-srv$ ping br-r.branch.work
HQ-R$ cat /etc/resolv.conf # В "nmtui" установите DNS-сервер 192.168.10.5, а уже потом 8.8.8.8. В файле должен быть параметр:
"nameserver 192.168.10.5"
hq-srv$ ping hq-r.hq.work
hq-srv$ ping br-r.branch.work
```

2. Настройте синхронизацию времени между сетевыми устройствами по протоколу NTP. Будем использовать как сервер пакет **chrony**.

   a. Сервером будет **HQ-R со стратумом 5**

   b. Используем Loopback как сервер. 127.0.0.1

   c. Все устройства должны синхронизировать время с HQ-R

   d. Все устройства должны иметь правильный часовой пояс (UTC +3), то есть **Europe/Moscow**

```bash
HQ-R$ dnf install chrony # Вообще, он должен быть уже установлен
HQ-R$ nano /etc/chrony.conf
--- # Должны быть следующие строки. Некоторые можно раскомментировать
# ВСЕ ПАРАМЕТРЫ "server" должны быть закомментированы!!! Кроме того, что ниже.
local stratum 5
allow 0.0.0.0/0
server 127.0.0.1 #По заданию
---
HQ-R$ systemctl restart chronyd
HQ-R$ systemctl enable chronyd
redos$ timedatectl set-timezone Europe/Moscow # ВСЕ УСТРОЙСТВА!!! БЕЗ ИСКЛЮЧЕНИЙ!!!
#===НА КЛИЕНТАХ===#
redos$ timedatectl set-timezone Europe/Moscow # ВСЕ УСТРОЙСТВА!!! БЕЗ ИСКЛЮЧЕНИЙ!!!
redos$ nano /etc/chrony.conf
---
# ВСЕ ПАРАМЕТРЫ "server" должны быть закомментированы!!! Только один, описанные ниже, должен быть активным
server 192.168.10.1
---
redos$ systemctl restart chronyd
#===ПРОВЕРКА===#
redos$ chronyc sources # На всех устройствах должен быть один сервер, запись в выводе команды с параметром "Reach" 1 или больше.
"MS	NAME/IP STRATUM Poll Reach LastRx Last sample"
"192.168.10.1	0	6	1	-	+0ns[	+0ns] +/-	0ns"
```

##### FreeIPA

3. Настройте сервер домена выбор, его типа обоснуйте, на базе hq-srv через web интерфейс, выбор технологий обоснуйте. **Обоснование**: Бесплатный, общеизвестный, так как много где используется, удобный за счёт веб-интерфейса, гибкий, так как позволяет опубликовать любую службу в домен. **Нужно поднять контроллер домена** на **hq-srv** и ввести две машины в домен.

   a. Ввести две машины в домен **BR-SRV** и **CLI**

   b. Организуйте отслеживание подключения к домену

```bash
hq-srv$ dnf makecache
hq-srv$ systemctl disable --now systemd-timesyncd # Нужно отключить службу не chrony. Вообще, сервис по умолчанию должен не работать, но на всякий случай отключите
hq-srv$ timedatectl set-timezone Europe/Moscow # Если ещё не установлена корректный часовой пояс, то установите
# Перед установкой FreeIPA убедитесь в след. пунктах:
1) Установлен статический IP-адрес с корректной маской и шлюзом
hq-srv$ nmtui # Если нужно установить
2) Использованно полное доменной имя с !!нижним регистром!! # НИЖНИЙ РЕГИСТР ОБЯЗАТЕЛЕН
hq-srv$ hostnamectl set-hostname hq-srv.hq.work
3) В файле /etc/hosts имеется полное имя
hq-srv$ nano /etc/hosts
---
192.168.10.5	hq-srv.hq.work hq-work
---
4) В файле /etc/resolv.conf имеется отсылка, но не в Интернет # Создатели задания не учли, что hq.work - реально существующий домен в Интернете
hq-srv$ nano /etc/resolv.conf
---
search hq.work
nameserver 192.168.10.5
---
# Если выше 4 пункта выполняются, то можно приступить к установке
hq-srv$ dnf install bind bind-dyndb-ldap ipa-server ipa-server-dns ipa-server-trust-ad
hq-srv$ ipa-server-install --mkhomedir # Должно быть установлено 2ГБ Оперативной памяти и 2 Виртуальных ядра
"Do you want to configure integrated DNS (BIND)? [no]:" yes
"Server host name [hq-srv.hq.work]:" # По умолчанию просто Enter
"Please confirm the domain name [hq.work]:"
"Please provide a realm name [HQ.WORK]:"
"Directory Manager P@ssw0rd:" P@ssw0rd
"Password (confirm):" P@ssw0rd
"IPA admin password:" P@ssw0rd
"Password (confirm):" P@ssw0rd
"Checking DNS domain hq.work..." # Не должно быть связи с Интернетом, так как hq-work реально есть в нем, иначе вылетит с ошибкой. Проверьте resolv.conf, в нём не должно быть 8.8.8.8
"Do you want to configure DNS forwarders? [yes]:"
"Do you want to configure these servers as DNS forwarders? [yes]:" no
"Enter an IP adresss for a DNS forwarder, or press Enter to skip:" 8.8.8.8
"Enter an IP adresss for a DNS forwarder, or press Enter to skip:"
"Do you want to configure the reverse zone? [yes]:"
"NetBIOS domain name [HQ]:
"Do you want to configure chrony with NTP server or pool address? [no]:" yes
"Enter NTP source server addresses separated by comma...:" 192.168.10.1
"Enter NTP source pool adress...:"
"Continue to configure the system with these values? [no]:" yes
# В конце будет общая настройка, после "yes" идём выполнять другие пункты, пока выполняется установка (она не быстрая)
# Setup complete - FreeIPA успешно установлен
```

![](FreeIPA-final.png)

```bash
hq-srv$ kinit admin # Проверяем доступность домена, если успешно, то всё работает#
# === ДЛЯ КЛИЕНТА ===
# Клиентами, а точнее устройствами в домене выступают CLI и BR-SRV
# Для CLI нужно создать соединение с HQ-SRV (На нём FreeIPA). Это можно сделать двумя способами:
# 1) Проброс портов. На самом деле нехорошо, так как это локальный домен и хочется не пускать из Интернета кого попало
# 2) VPN. Хороший способ, требующий только установки на CLI и создать ключ на HQ-R
# FreeIPA по умолчанию обрабатывает только соседние сети по DNS. Поправить это можно добавив настройку в файл
hq-srv$ nano /etc/named/ipa-options-ext.conf
--- # Добавить снизу
allow-query { any; };
---
# === CLI. VPN ===
hq-r$ cd /etc/wireguard
hq-r$ wg genkey | tee cli.key | wg pubkey > cli.pub
hq-r$ nano wg.conf
--- # Дописываем ниже
[Peer]
PublicKey = ? # Сюда публичный ключ CLI. cli.pub
AllowedIPs = 10.0.0.3/32, 192.168.111.0/24
---
cli$ dnf install wireguard-tools
cli$ scp root@172.16.10.2:/etc/wireguard/* /etc/wireguard/
cli$ cd /etc/wiregaurd
cli$ nano /etc/wg.conf
--- # Представлен весь файл
[Interface]
PrivateKey = ? # Вместо '?' вставляем приватный ключ BR-R
Address = 10.0.0.3/29
Table = off
DNS = 192.168.10.5
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ HQ-R
Endpoint = 172.16.10.2:51820
AllowedIPs = 10.0.0.1/32, 192.168.10.0/26, 0.0.0.0/0
---
cli$ wg-quick up wg # Поднимаем
cli$ wg show # Проверяем
cli$ systemctl enable wg-quick@wg # Автозагрузка
# Если CLI имеет доступ к HQ-SRV, то можно зайти на ВЕБ-сайт FreeIPA по https:// который указан в консоли после завершение установки FreeIPA
# === CLI. IPA ===
cli$ dnf install ipa-client
cli$ ipa-client-install -d --mkhomedir --enable-dns-updates
"Continue to configure the system with these values? [no]:"
"User authorized to enroll computers: admin"
# === BR-SRV ===
cli$ dnf install ipa-client
cli$ ipa-client-install -d --mkhomedir --enable-dns-updates
"Continue to configure the system with these values? [no]:"
"User authorized to enroll computers: admin"
```

##### SMB

4. Реализуйте файловый SMB или NFS (выбор обоснуйте) сервер на базе сервера HQ-SRV. Будем работать с SMB, так как там проше разграничить пользователей

   a. Общие папки:

   ​	i. Branch_Files - для пользователя Branch admin

   ​	ii. Network - для пользователя Network

   ​	iii. Admin_Files - для пользователя Admin

5. Сконфигурируйте веб-сервер LMS Apache на сервере BR-SRV:

   a. На главной странице должен отражаться номер места

   b. Используйте базу днных MariaDB

   c. Создайте пользователей в соответствии с таблицей, пароли у всех пользователей "P@ssw0rd"

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |

6. Запустите сервис MediaWiki используя docker на сервере **HQ-SRV**

   a. Установите Docker и Docker Compose

```bash
hq-srv$ dnf install docker-compose
```

