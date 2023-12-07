---
title: Iptables
description: Iptables описание и работа с ним
published: true
date: 2023-12-07T08:14:12.314Z
tags: nat, межсетевой экран
editor: markdown
dateCreated: 2023-12-07T08:09:23.665Z
---

# Iptables

Это межсетевой экран

## Настройка iptables на RedOS
Для включение автозагрузки правил необходимо установить пакет iptables-serives:
```bash
dnf install iptables-services
```
Вместе с iptables-servies устанавливаются блокирующие правила Reject в таблицу filter, их можно посмотреть
```bash
iptables -S
```
Чтобы удалить их использую Flush:
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