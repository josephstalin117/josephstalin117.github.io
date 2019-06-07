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