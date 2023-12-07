---
title: Iptables
description: Iptables описание и работа с ним
published: true
date: 2023-12-07T08:45:28.931Z
tags: nat, межсетевой экран
editor: markdown
dateCreated: 2023-12-07T08:09:23.665Z
---

# Iptables

Это межсетевой экран
## Таблицы
В Iptables существуют по умолчанию 3 таблицы:
1. **filter**
2. **nat**
3. **mangle**
В таблице **filter** устанавливаются правила, которые принимаются самим хостом, или пакеты, которые маршрутизируются через устройство.
В таблице **nat** устанавливаются правила, которые взаимодействуют с пакетами, используя технологию NAT.
## Правила
Чтобы установить правила iptables 
## Настройка iptables на RedOS
Для включение автозагрузки правил необходимо установить пакет iptables-serives:
```bash
dnf install iptables-services
```
> Вместе с iptables-servies устанавливаются блокирующие правила Reject в таблицу filter, их можно посмотреть
{.is-warning}

```bash
iptables -S
```
Чтобы удалить все правила в траблице filter, используем Flush:
```bash
iptables -F
```
Далее, включаем iptables в автозагрузку и дополнительно включаем --now
```bash
systemctl enable --now iptables
```
## Настройка PAT (MASQUERADE)
Чтобы установить правило MASQUERADE, Перегруженный NAT, или просто PAT, необходимо добавить соответствующие правило:
```bash
iptables -t nat -A POSTROUTING -o ens* -j MASQUERADE
```
Посмотреть данное правило в таблице nat можно следующей командой
```bash
iptables -t nat -S
```