---
title: Установка
description: 
published: true
date: 2023-12-05T23:35:58.964Z
tags: 
editor: markdown
dateCreated: 2023-12-05T23:32:35.053Z
---

# FreeIPA
**Окружение**
- **ОС:** РЕД ОС 7.3.3
- **Конфигурация:** Сервер минимальный
- **Версия ПО:**

## Подготовка сервера
### Настройка синхронизации
Откройте файл конфигурации:
```bash
nano /etc/chrony.conf
```

## Установка пакетов FreeIPA
### RedOS, RedHat

```bash
dnf install install bind bind-dyndb-ldap ipa-server ipa-server-dns ipa-server-trust-ad
```
## Интерактивная установка
```bash
ipa-server-install --mkhomedir
```
```bash
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