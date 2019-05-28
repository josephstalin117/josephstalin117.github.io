---
layout: post
title:  php
date:   2019-5-16 20:35:35 +0800
categories: command
---

#### nmap

```
# 扫描局域网ip(进行ping扫描)
nmap -sP 192.168.9.0/24

# SYN扫描（需要root权限）
namp -sS 172.17.148.0/24

# UDP扫描（需要root权限，且速度慢）
nmap -sU 172.17.148.0/24

# 扫描指定主机的开放端口，系统版本等信息
nmap -A 172.17.148.168 

# windows
arp -a

# 查看arp缓存表获取局域网IP地址
cat /proc/net/arp
```