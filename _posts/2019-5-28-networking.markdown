---
layout: post
title:  networking
date:   2019-5-28 20:35:35 +0800
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

# 查看主机端口开启情况(UDP端口)
nmap -p 161 -sU 192.168.1.100

# 查看1-200间的端口是否开放
nmap -p 1-200 192.168.255.130

# windows
arp -a

# 查看arp缓存表获取局域网IP地址
cat /proc/net/arp
```

proxychains
```
# install
sudo apt install proxychains

# 修改配置
vim /etc/proxychains.conf
socks4 127.0.0.1 9095 -> socks5 127.0.0.1 1080

# 使用
proxychains4 wget http://xxx.com/xxx.zip
```

route
```
# 查看路由表
route

# 添加默认路由
route add default gw 172.18.3.254

# 删除默认路由
route del default gw 82.17.68.254

# 添加静态路由
route add -net 192.168.4.0/24 gw 60.12.105.145 dev eth1

# 删除静态路由
route del -net 192.168.100.0/24 gw 192.168.100.1
```

ip
```
# 查看ip地址
ip addr

# 删除静态路由
ip route del 192.0.2.0/26 dev eth0

# 添加静态路由 via指网关地址
ip route add 192.0.2.0/26 via 192.0.2.1 dev eth0

# 添加默认路由
ip route add default via 192.168.0.1 dev eth0
```

traceroute
```
# 追踪网络数据包的路由
traceroute baidu.com
```

iftop查看网卡流量
```
# 合并为一行
t

# 查看端口
p

# 暂停刷新
P

# 直接显示IP, 不进行DNS反解析
iftop -n

# 监控eth1网卡
iftop -i eth1

# 显示某个网段进出封包流量
iftop -F 192.168.1.0/24 or 192.168.1.0/255.255.255.0
```

nethogs查看进程流量
```
# 查看eth0的流量
nethogs eht0
```

查看DNS服务器
```
cat /etc/resolv.conf
```

静态路由设置
```
/etc/netplay/01-network-manager.yaml

# 应用设置
sudo netplan apply

# 设置静态路由
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
      addresses: [192.168.1.110/24, ]
      gateway4:  192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
```

v2ray
```
curl https://install.direct/go.sh | sudo bash

# server config
{
	"log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [{
    "port": 123,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "testid",
          "level": 1,
          "alterId": 64
        }]
    },
    "streamSettings":{
	     "network":"mkcp",
    	 "kcpSettings":{
		    "uplinkCapacity": 5,
		    "downlinkCapacity": 100,
		    "congestion": true,
		    "header": {
          "type": "none"
		    }
	     }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}


# client config
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [{
    // Port to listen on. You may need root access if the value is less than 1024.
    "port": 1081,

    // IP address to listen on. Change to "0.0.0.0" to listen on all network interfaces.
    //"listen": "127.0.0.1",
    "listen": "0.0.0.0",

    // Tag of the inbound proxy. May be used for routing.
    "tag": "socks-inbound",

    // Protocol name of inbound proxy.
    "protocol": "socks",

    // Settings of the protocol. Varies based on protocol.
    "settings": {
      "auth": "noauth",
      "udp": false,
      //"ip": "127.0.0.1"
      "ip": "172.18.3.1"
    },

    // Enable sniffing on TCP connection.
    "sniffing": {
      "enabled": true,
      // Target domain will be overriden to the one carried by the connection, if the connection is HTTP or HTTPS.
      "destOverride": ["http", "tls"]
    }
  }],

  // List of outbound proxy configurations.
  "outbounds": [{
    // Protocol name of the outbound proxy.
    "protocol": "vmess",

    // Settings of the protocol. Varies based on protocol.
    "settings": {
    	"vnext": [{
        	"address": "ip", 
        	"port": 123, 
        	"users": [{ "id": "d821" }]
      }]
    },

     "streamSettings": {
        "network": "mkcp",
        "kcpSettings": {
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": true,
          "header": {
            "type": "none"
          }
        }
      },

    // Tag of the outbound. May be used for routing.
    "tag": "direct"
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],

  // You may add other entries to the configuration, but they will not be recognized by V2Ray.
  "other": {}
}

```