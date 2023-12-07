---
title: Iptables
description: Iptables описание и работа с ним
published: true
date: 2023-12-07T08:11:12.084Z
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
Далее, включаем iptables в автозагрузку и дополнительно включаем --now
```bash
systemctl enable --now iptables
```