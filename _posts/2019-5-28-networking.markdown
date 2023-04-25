---
layout: post
title:  networking
date:   2019-5-28 20:35:35 +0800
categories: command
---

nmap
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

curl
```
# 只打印响应头部信息
curl -I http://baidu.com

# 将下载数据写入指定文件中
curl http://man.linuxde.net/test.iso -o filename.iso --progress

# HTTP POST 方式传送数据
curl -d 

# 使用代理
curl -x socks5://myproxy.com:8080 https://www.example.com

# ipv6 访问
curl -g -6 byr.pt
```

telnet
```
# 测试端口
telnet 101.199.97.65 62715

# 端口未打开
Trying 101.199.97.65...
telnet: connect to address 101.199.97.65: Connection refused

# 端口已打开
Trying 101.199.97.65...
Connected to 101.199.97.65.
Escape character is '^'.
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

# 删除默认网桥
ip link del docker0 down
```

dns查询
```
# 查询单个域名的DNS信息
dig baidu.com

# 使用指定的DNS服务器进行查询
dig @8.8.8.8 abc.filterinto.com
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

netstat查看套接字连接情况
```
# 列出当前所有连接
netstat -a

# 列出TCP协议连接
netstat -t

# 列出UDP协议连接
netstat -u

# 禁用反向域名解析
netstat -n

# 列出正在监听的套接字
netstat -tnl

# 获取进程名、进程号及用户ID
sudo netstat -nlpt

# 查看进程名和用户名
sudo netstat -ltpe

# 查看端口占用
netstat -tunlp | grep 端口号
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
v2ray --config=/etc/v2ray/config.json

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

iptables
```
# tcp
iptables -t nat -N V2RAY
iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN # 直连 192.168.0.0/16 
iptables -t nat -A V2RAY -p tcp -j RETURN -m mark --mark 0xff # 直连 SO_MARK 为 0xff 的流量(0xff 是 16 进制数，数值上等同与上面配置的 255)，此规则目的是避免代理本机(网关)流量出现回环问题
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345 # 其余流量转发到 12345 端口（即 V2Ray）
iptables -t nat -A PREROUTING -p tcp -j V2RAY # 对局域网其他设备进行透明代理
iptables -t nat -A OUTPUT -p tcp -j V2RAY # 对本机进行透明代理

# udp
ip rule add fwmark 1 table 100
ip route add local 0.0.0.0/0 dev lo table 100
iptables -t mangle -N V2RAY_MASK
iptables -t mangle -A V2RAY_MASK -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A V2RAY_MASK -p udp -j TPROXY --on-port 12345 --tproxy-mark 1
iptables -t mangle -A PREROUTING -p udp -j V2RAY_MASK
```

时间同步
```
apt-get update
apt-get install ntp ntpdate -y

yum install ntp ntpdate -y

service ntpd stop                 #停止ntp服务
ntpdate us.pool.ntp.org           #同步ntp时间
service ntpd start                #启动ntp服务
```

修改mac地址
```
# 闭网卡设备
ifconfig eth0 down

# 修改MAC地址
ifconfig eth0 hw ether MAC地址

# 重启网卡
ifconfig eth0 up

# /etc/rc.d/rc.local中添加下三行
ifconfig eth0 down
ifconfig eth0 hw ether 00:0C:18:EF:FF:ED
ifconfig eth0 up
```

acme.sh
```
# 生成证书
acme.sh  --issue -d mydomain.com   --standalone

# 安装证书
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

