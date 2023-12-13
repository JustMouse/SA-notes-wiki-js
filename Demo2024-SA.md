---
title: ДемоЭкзамен 2024 (СиСА-Профиль)
description: 
published: true
date: 2023-12-13T04:50:29.234Z
tags: 
editor: markdown
dateCreated: 2023-12-05T23:48:18.509Z
---

# Решение Демонстрационного Экзамена 2024

Сетевое и Системное Администрирование (Профиль)  
*Версия от 12.12.2023*

## Ссылки

[Официальная документация 2023](https://bom.firpo.ru/Public/83)

### Загрузка решение в виде документа (Компактная версия)

-   [PDF](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.pdf)
-   [Word](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.word)
-   [HTML](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.html)
-   [MD](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-101.md)

### Загрузка решения в виде документа (Версия сайта)

-   [PDF](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-wiki-101.pdf)
-   [Word](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-wiki-101.word)
-   [HTML](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-wiki-101.html)
-   [MD](http://wiki.kptlo.ru:8080/Demo2024-SA/Demo24-wiki-101.md)

## Предисловие

1.  Модули настоящего задания очень сильно связаны друг с другом. Например, это касается VPN (Модуль 3. Пункт 7) и Контроллера домена (Модуль 2. Пункт 3). Несмотря на то, что они находятся в разных модулях логичнее сначала настроить всю сетевую связанность, в том числе и VPN, а уже после Контроллер домена.
2.  По идеи задания временное подключение используется для быстрого ввода устройства CLI в домен. Тем не менее в данном решении данное подключение не осуществляется, так как было принято решение предоставить устройству CLI функциональный VPN до HQ-R, что обосновано необходимостью для студентов обладать данным навыком в дисциплине.
3.  Настоящее задание предполагает, что в стенд не имеет доступа к Интернету, тем не менее в данном решение Интернет подключён к устройству ISP для скачивание пакетов с общедоступного репозитория.
4.  Выбор операционной системы зависит от образовательной организации. В данном решении предполагается использования только [**RedOS 7.3.3 Minimal installation**](https://files.red-soft.ru/redos/7.3/x86_64/iso/redos-MUROM-7.3.3-20230815.0-Everything-x86_64-DVD1.iso) и [**RedOS 7.3.3 Workstation (MATE)**](https://files.red-soft.ru/redos/7.3/x86_64/iso/redos-MUROM-7.3.3-20230815.0-Everything-x86_64-DVD1.iso)

## Топология

![](/demo24-topology.png)

## Таблица IP-адресов

| Устройство | IPv4 | IPv6 | Сеть |
| --- | --- | --- | --- |
| ISP | 192.168.106.X (DHCP) | None | Internet |
|  | 172.16.10.1/30 | 2001:172:10::1/126 | ISP-HQ |
|  | 172.16.20.1/30 | 2001:172:20::1/126 | ISP-BR |
|  | 192.168.111.1/24 | 2001:111::1/120 | ISP-CLI |
| HQ-R | 172.16.10.2/30 | 2001:172:10::2/126 | ISP-HQ |
|  | 192.168.10.1/26 | 2001:192:10::1/122 | HQ-NET |
|  | 10.0.0.1/29 | 2001:10::1/125 | VPN |
| BR-R | 172.16.20.2/30 | 2001:172:20::2/126 | ISP-BR |
|  | 192.168.20.1/28 | 2001:192:20::1/123 | BR-NET |
|  | 10.0.0.2/29 | 2001:10::2/125 | VPN |
| HQ-SRV | 192.168.10.5/26 | 2001:192:10::5/122 | HQ-NET |
| BR-SRV | 192.168.20.5/28 | 2001:192:20::5/123 | BR-NET |
| CLI | 192.168.111.2/24 | 2001:111::1/120 | ISP-CLI |
|  | 10.0.0.3/29 | 2001:10::3/125 | VPN |

## Оглавление

[Выполните базовую настройку всех устройств:](#h-1-выполните-базовую-настройку-всех-устройств)

## Модуль 1. Базовая настройка

### 1. Выполните базовую настройку всех устройств: 

#### a. Присвоить имена в соответствии с топологией.

Присвоить имена в соответсвии с топологией. На **ВСЕХ** машинах. Пункт проверяется на всех устройствах

```bash
redos$ hostnamectl set-hostname [Имя] # Например: hostnamectl set-hostname ISP
redos$ hostnamectl set-hostname [Имя.домен...] # Используйте в локальных сетях организации, таким образом, чтобы автоматически заполнить параметр 'search' поисковый домен
```
**Примеры:**
HQ-SRV: hq-srv.hq.work
HQ-R: hq-r.hq.work
BR-SRV: br-srv.branch.work
BR-R: br-r.branch.work
CLI: cli
ISP: isp

Если хочется отобразить имя устройство на текущей консоли, то можно или перезайти под учётную запись, или перезагрузить оболочку `bash`
```bash
redos$ exec bash
```
##### b. Рассчитайте IP-адресацию IPv4 и IPv6.
*см. [Таблицу IP-адресов](#таблица-ip-адресов)*
#### c. Пул адресов для BRANCH - не более 16 адресов. **Это маска 28**
#### d. Пул адресов для HQ - не более 64 адресов. **Это маска 26**

> Количество хостов для сети с маской подсети IPv4: /26 = 62, /28 = 30. IPv6: /123 = 62 /123 = 30

> **Продолжение следующих пунктов** (Пункт .2) требует *базовой связанности* предполагаемая в задании. Убедитесь, что все машины выходят в Интернет (В рамках данного решения) и на устройствах **ISP, HQ-R, BR-R** настроен **NAT (MASQUERADE)** для каждого филиала
{.is-warning}

На RedOS 7.3.3 уже заранее предустановлена утилита управления межсетевым экраном `netfilter` в виде пакета [`Iptables`](/Iptables). Любые изменение внесённые с помощью команды `iptables` будут работать только до перезагрузки устройства, поэтому необходимо включить автозагрузку правил. Для этого устанавливаем пакет `iptables-services` там где это необхоидмо и, используя `systemd` и команду `systemctl enable iptables`, включаем автозапуск сервиса. Все правила по умолчанию хранятся по пути `/etc/sysconfig/iptables`, поэтому любые изменение важно сохранить именно в этот файл.
```bash
redos$ dnf install iptables-services
redos$ systemctl enable iptables
# Для ISP, HQ-R, BR-R
```
По уиолчанию пакет `iptables-servies` добавляет блокирующие правила в таблицу `filter`. Удалить их можно следующей командой:
```bash
redos$ iptables -F
# Для ISP, HQ-R, BR-R
```
Можно добавлять необходимые правила. Для начала настроем трансляцию IP-адерсов для локальных сетей и на ISP для выхода в Интернет.
```bash
redos$ iptables -t nat -A POSTROUTING -o <Интерфейс> -j MASQUERADE
# Для ISP, HQ-R, BR-R
```
> Для лучшего понимание флагов и работы Iptables в целом, предлагается ознакомиться со соответсвующей документации в Интернете или на [страницы данной Вики](/Iptables)
{.is-info}

После внесения изменения можно посмотреть набор правил в таблице `filter` и `nat`. Или, при ошибке, удалить правило:
```bash
redos$ iptables -t nat -D POSTROUTING 1 # Удаляет правило POSTROUTING ПЕРВОЕ
redos$ iptables -t nat -F # Удалить все правила таблицы nat
redos$ iptables -t nat -S # Посмотреть все правила таблицы nat
redos$ iptables -S # Посмотреть все правила таблицы filter
```
Сохраним изменения таблиц, добавленные правила, в указанный файл:
```bash
redos$ iptables-save > /etc/sysconfig/iptables # Сохраняем конфигурацию
# Для ISP, HQ-R, BR-R
```
Далее включаем маршрутизацию трафика на устройствах выполняющие роль маршрутизаторов:
```bash
# Настройка включение маршрутизации на маршрутизаторах
redos$ nano /etc/sysctl.conf
# Для ISP, HQ-R, BR-R

```

Дописываем строку в файл:
```bash
net.ipv4.ip_forward=1 # IPv4
net.ipv6.conf.all.forwarding=1 # IPv6
```
Готово. Перечитаем конфиг-файл и введём изменения:
```bash
redos$ sysctl -p
````
После ввода команды должна появиться только что написанная команда в файл команда. Если это не так, то проверьте на опечатки. Сделать данную настройку нужно на `ISP`, `HQ-R`, `BR-R`
В RedOS 7.3.3 по умолчанию используется пакет `NetworkManager` для работы с сетевыми сервисами. В данном пакете есть удобный интерфейс из псевдографики для работы с ним
```bash
# Настройка сетевых интерфейсов
redos$ nmtui # Открывает псевдографика с настройками
```

> Учтите, что для установки пакетов нужно установить публичный DNS-сервер, например `8.8.8.8`. При этом, настройка должна иметь временный характер, так как после установки FreeIPA все устройство в локальных сетях должны отправлять DNS-запросы в первую очередь к контроллеру домена
{.is-warning}

Далее добавляем IP-адрес, маску подсети, шлюз, DNS-сервера, как минимум будет IP-адрес контроллера домена, и Поисковый домен для автодополнение DNS-запросов:
```bash
redos$ Изменить подключение > [Интерфейс] Изменить > [Поменяйте Имя профиля, Конфигурацию IPv4: Выберите между Автоматически и Вручную (DHCP и Static).
# Укажите IP-адрес с маской, Укажите шлюз, если он не настраивается по DHCP, Установите Серверы DNS 2 шткуи, если они не настраиваются по DHCP: Они должны быть 192.168.10.5(HQ-SRV), 8.8.8.8(Google DNS). Именно в таком порядке
# Включите и выключите интерфейсы, если они не обновляются автоматически после изменений. Или перезагрузите машину
```
**Пример на HQ-R:**
![demo24-isp-nmtui.png](/demo24-isp-nmtui.png)
**Пример:**
HQ-R:
- IPv4: `172.16.10.2/30, 192.168.10.1/26`
- Шлюз: `172.16.10.1`
- DNS: `192.168.10.5`
- Поисковый домен: `hq.work`

### 2.  Настройте внутреннюю динамическую маршрутизацию по средствам **FRR**. Выберите и обоснуйте выбор протокола динамической маршрутизации из расчёта, что в дальнейшем сеть будет масштабироваться

Это будет **OSPF** в VPN туннеле `HQ-R` и `BR-R`. Сначала должен быть настроен **туннель**.
> Несмотря на то, что в здании не уточняется, использование `ISP` при выполнении этого пункта, тем не менее по-настоящему **внутренней** она будет при использование исключительно локальных сети, в том числе и туннеля

Для настройки OSPF используется пакет `frr`, реализующий на устройстве динамическую маршрутизацию.
Необходимо настроить OSPF-связанность через туннель. Для начало он должен быть. В данном случае самым простым по настройки будет `WireGuard`, использующий метод шифрования `ChaCha20`, обычную пару ключей
Установим `WireGuard` и `frr` на устройствах `HQ-R`,`BR-R`:
```bash
hq-r$ dnf install wiregaurd-tools frr
# Устанавливаем на BR-R и HQ-R
```
Далее, создадим пары ключей для устройств в папке конфигов `WireGuard` для удобного доступа
> В дальнейшем устройство CLI такуже будет подключено к VPN. Поэтому, чтобы не тратить время преднастроем сервер-VPN заранее
```bash
hq-r$ cd /etc/wireguard
hq-r$ wg genkey | tee HQ-R.key | wg pubkey > HQ-R.pub # .key - приватный ключ; .pub - публичный ключ
hq-r$ wg genkey | tee BR-R.key | wg pubkey > BR-R.pub # На HQ-R должен быть приватный HQ-R и публичный BR-R, а на BR-R наоборот
hq-r$ wg genkey | tee CLI.key | wg pubkey > CLI.pub # Клиент
```
Создаём и настраиваем туннель в файле `/etc/wireguard/wg.conf`:
```bash
hq-r$ nano /etc/wireguard/wg.conf
```
```bash
[Interface]
PrivateKey = ? # Вместо '?' вставляем приватный ключ HQ-R
Address = 10.0.0.1/29, 2001:10::1/125
ListenPort = 51820
Table = off # Отключаем статическую маршрутизацию для работы OSPF

# BR-R
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ BR-R
AllowedIPs = 10.0.0.2/32, 192.168.20.0/28, 2001:192:20::/123
PersistentKeepAlive = 5 # Поддержка туннеля

# CLI
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ CLI
AllowedIPs = 10.0.0.2/32, 192.168.111.0/24
PersistentKeepAlive = 5 # Поддержка туннеля
```
Импортируем ключи и расставляем по нужным местам. Используем `nano`, поэтому удобно вырезать <kbd>CTRL</kbd>+<kbd>K</kbd> и вставить <kbd>CTRL</kbd>+<kbd>U</kbd>:
```bash
hq-r$ cat HQ-R.key >> wg.conf # Вставляем в файл ДЛИННУЮ строку-ключ с помощью дозаписи '>>'. 
hq-r$ cat BR-R.pub >> wg.conf
hq-r$ cat CLI.pub >> wg.conf
```
Включаем интерфейс и включаем автозагрузку:
```bash
HQ-R$ wg-quick up wg
HQ-R$ systemctl enable wg-quick@wg
```
Приготовимся к переносу ключей через `scp`:
```bash
hq-r$ nano /etc/ssh/sshd_config
```
```bash
PermitRootLogin yes;
Port 2220;
```
```bash
hq-r$ systemctl restart sshd
```
Перенесём ключи с HQ-R:
```bash
br-r$ scp root@172.16.10.2:/etc/wireguard/* /etc/wireguard/ # Переносим ключики
br-r$ nano /etc/wireguard/wg.conf # Заменяем ключи
```
```bash
[Interface]
PrivateKey = ? # Вместо '?' вставляем приватный ключ BR-R
Address = 10.0.0.2/29, 2001:10::2/125
Table = off
[Peer]
PublicKey = ? # Вместо '?' вставляем публичный ключ HQ-R
Endpoint = 172.16.10.2:51820
AllowedIPs = 10.0.0.1/32, 192.168.10.0/26, 2001:192:10::/122
PersistentKeepAlive = 5 # Поддержка туннеля
```
```bash
br-r$ cat HQ-R.pub >> wg.conf # Вставляем в файл ДЛИННУЮ строку-ключ с помощью '>>'.
br-r$ cat BR-R.key >> wg.conf
```
```bash
BR-R$ wg-quick up wg
BR-R$ systemctl enable wg-quick@wg
```
Проверить работу туннеля можно так:
```bash
hq-r$ wg show
br-r$ wg show
hq-r$ ping 10.0.0.2
hq-r$ ping 2001:10::2
br-r$ ping 10.0.0.1
hq-r$ ping 2001:10::1
```
Настреваем `frr` и `OSPF` в частности:
```bash
hq-r$ nano /etc/frr/daemons
```
```bash
ospfd=yes
```
```bash
hq-r$ systemctl restart frr
hq-r$ vtysh
hq-r$ $ conf t
hq-r$ (conf)$ ip forwarding
hq-r$ (conf)$ router ospf
hq-r$ (conf-router)$ network 192.168.10.0/26 area 0
hq-r$ (conf-router)$ network 10.0.0.0/29 area 0
hq-r$ (conf-router)$ neighbor 10.0.0.2
hq-r$ (conf-router)$ exit
hq-r$ (conf)$ interface wg
hq-r$ (conf-if)$ ip ospf network broadcast
hq-r$ (conf-if)$ end
hq-r$ $ wr
hq-r$ $ exit
```
```bash
hq-r$ systemctl enable frr
hq-r$ systemctl restart frr
#===br-r===#
br-r$ apt install frr
br-r$ nano /etc/frr/daemons
--- # В файле поменять следующую строку
ospfd=yes
---
br-r$ systemctl restart frr
br-r$ vtysh
```
```bash
br-r$ $ conf t
br-r$ (conf)$ ip forwarding
br-r$ (conf)$ router ospf
br-r$ (conf-router)$ network 192.168.20.0/28 area 0
br-r$ (conf-router)$ network 10.0.0.0/29 area 0
br-r$ (conf-router)$ neighbor 10.0.0.1
hq-r$ (conf-router)$ exit
hq-r$ (conf)$ interface wg
hq-r$ (conf-if)$ ip ospf network broadcast
hq-r$ (conf-if)$ end
br-r$ $ wr
br-r$ $ exit
```
```bash
br-r$ systemctl enable frr
br-r$ systemctl restart frr
#===ПРОВЕРКА===#
--- # Проверка с hq-r
hq-r$ ip ro # Должен быть маршрут с ospf меткой до сети 192.168.20.0/28
hq-r$ vtysh
hq-r$ $ show ip ospf neighbor # Должен быть IP-адрес соседа, сам сосед с режимом(state) Full/Backup или Full/BR
hq-r$ $ show ip route # Должен быть маршрут с типом "O>*"
--- # Проверка с br-r
br-r$ ip ro # Должен быть маршрут с ospf меткой до сети 192.168.10.0/26
br-r$ vtysh
br-r$ $ show ip ospf neighbor # Должен быть IP-адрес соседа, сам сосед с режимом(state) Full/Backup или Full/BR
br-r$ $ show ip route # Должен быть маршрут с типом "O>*"
--- # Проверка с WEB-L с учётом работающего туннеля
hq-srv$ ping 192.168.20.5 # Хорошая проверка. Она проверяет сразу sysctl, iptables, frr. Если что-то не работает проверьте ping на более близкой дистанции. С hq-r на BR-SRV или br-r на BR-SRV
```

### DHCP

1.  Настройте автоматическое распределение IP-адресов на роутере HQ-R. Имеется ввиду **DHCP** на HQ-R в сторону HQ-NET, то есть для машины hq-srv.

```plaintext
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

1.  Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

| Имя учётной записи | Пароль | На каких устройствах | Логин |
| --- | --- | --- | --- |
| Admin | P@ssw0rd | CLI hq-srv HQ-R | admin |
| Branch admin | P@ssw0rd | BR-SRV BR-R | branch\_admin |
| Network admin | P@ssw0rd | HQ-R BR-R BR-SRV | network\_admin |

```plaintext
redos$ useradd -c Admin admin # Аргумент после '-c' Имя учётной записи в таблицы. Сам же Логин должен быть в нижнем регистре, а пробелы заменены '_'
redos$ useradd -c "Branch admin" branch_admin
redos$ useradd -c "Network admin" network_admin
redos$ passwd admin # Смените пароль на пользователе, чтобы в него можно было авторизоваться
```

1.  Измерьте пропускную способность сети между двумя узлами **HQ-R** и **ISP** по средствам утилиты **iperf3**. Предоставьте описание пропускной способности канала со скриншотами

```plaintext
redos$ dnf install iperf3 # Установка на двух устройствах
ISP$ iperf3 -s # Включаем iperf3 в роли сервера
HQ-R$ iperf3 -c 172.16.10.1 # Включаем iperf3 в роли клиента и соединяемся с сервером
# Заскриньте 5 строк со скоростью сети с помощью прикладных программа на компьютере или с помощью OpenNebula
```

1.  Составьте backup скрипты для сохранения конфигурации сетевых устройств, а именно **HQ-R** и **BR-R**. Продемонстрируйте их работу. **Данный пункт можно** выполнить только после выполнение пункта с **VPN и FRR** \[2\]

### SSH 2222

1.  Настройте подключение по **SSH** для удалённого конфигурирования устройства **hq-srv** по порту **2222**. Учтите, что вам необходимо перенаправить трафик на этот порт по средствам контролирования трафика. Контроль трафика - это межсетевой экран, то есть **iptables**. Сделать настройку нужно на **HQ-R**, так как он виден в сети **Internet**

```plaintext
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

1.  Настройте контроль доступа до hq-srv по SSH со всех устройств, кроме CLI

```plaintext
HQ-R$ iptables -A FORWARD -i ens3 -s 192.168.111.0/24 -p tcp --dport 2222 -j DROP
HQ-R$ iptables-save > /etc/sysconfig/iptables # Не забудьте сохранить правило
```

## Модуль 2. Организация сетевого администрирования

1.  Настройте **DNS-сервер** на сервере **hq-srv**. Нужно создать две прямых зоны с A записями и две обратные зоны с PTR записям. Прямые **"hq.work" и "branch.work"**. Обратные **"10.168.192.in-addr.arpa" и "20.168.192.in-addr.arpa"**

Настройте синхронизацию времени между сетевыми устройствами по протоколу NTP. Будем использовать как сервер пакет **chrony**.

a. Сервером будет **HQ-R со стратумом 5**

b. Используем Loopback как сервер. 127.0.0.1

c. Все устройства должны синхронизировать время с HQ-R

d. Все устройства должны иметь правильный часовой пояс (UTC +3), то есть **Europe/Moscow**

```plaintext
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

Настройте сервер домена выбор, его типа обоснуйте, на базе hq-srv через web интерфейс, выбор технологий обоснуйте. **Обоснование**: Бесплатный, общеизвестный, так как много где используется, удобный за счёт веб-интерфейса, гибкий, так как позволяет опубликовать любую службу в домен. **Нужно поднять контроллер домена** на **hq-srv** и ввести две машины в домен.

a. Ввести две машины в домен **BR-SRV** и **CLI**

b. Организуйте отслеживание подключения к домену

```plaintext
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

Реализуйте файловый SMB или NFS (выбор обоснуйте) сервер на базе сервера HQ-SRV. Будем работать с SMB, так как там проше разграничить пользователей

a. Общие папки:

​ i. Branch\_Files - для пользователя Branch admin

​ ii. Network - для пользователя Network

​ iii. Admin\_Files - для пользователя Admin

Сконфигурируйте веб-сервер LMS Apache на сервере BR-SRV:

a. На главной странице должен отражаться номер места

b. Используйте базу днных MariaDB

c. Создайте пользователей в соответствии с таблицей, пароли у всех пользователей "P@ssw0rd"


Запустите сервис MediaWiki используя docker на сервере **HQ-SRV**

a. Установите Docker и Docker Compose

```bash
hq-srv$ dnf install docker-compose
```