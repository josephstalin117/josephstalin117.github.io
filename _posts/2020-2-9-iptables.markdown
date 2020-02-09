---
layout: post
title:  "iptables"
date:   2020-02-09 20:35:35 +0800
categories: command
---

#### 表(tables)
1. raw 表： 用于配置数据包，raw 中的数据包不会被系统跟踪。其优先级最高，设置 raw 时一般是为了不再让 iptables 做数据包的链接跟踪处理，提高性能
2. filter 表： 为 iptables 默认的表，在操作时如果没有指定使用哪个表，iptables 默认使用 filter 表来执行所有的命令。filter 表根据预定义的一组规则过滤符合条件的数据包。在 filter 表中只允许对数据包进行接收、丢弃的操作，而无法对数据包进行更改
3. nat 表： 即 Network Address Translation，主要是用于网络地址转换（例如：端口转发），该表可以实现一对一、一对多、多对多等 NAT 工作
4. mangle 表： 主要用于对指定包的传输特性进行修改。某些特殊应用可能需要改写数据包的一些传输特性，例如更改数据包的 TTL 和 TOS 等
5. security 表： 用于强制访问控制网络规则（例如： SELinux）

#### 链(chains)

1. INPUT： 处理入站数据包，当接收到访问本机地址的数据包(入站)时，应用此链中的规则
2. OUTPUT： 处理出站数据包，当本机向外发送数据包(出站)时，应用此链中的规则
3. FORWARD： 处理转发数据包，当接收到需要通过本机发送给其他地址的数据包(转发)时，应用此链中的规则
4. PREROUTING： 在对数据包作路由选择之前，应用此链中的规则
5. POSTROUTING： 在对数据包作路由选择之后，应用此链中的规则

Filter表包含三种内建链
1. INPUT 链 – 处理来自外部的数据
2. OUTPUT 链 – 处理向外发送的数据
3. FORWARD 链 – 将数据转发到本机的其他网卡设备上

NAT表包含三种内建链
1. PREROUTING 链 – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标 IP 地址（destination ip address），通常用于 DNAT(destination NAT)
2. POSTROUTING 链 – 处理即将离开本机的数据包。它会转换数据包中的源 IP 地址（source ip address），通常用于 SNAT（source NAT）
3. OUTPUT 链 – 处理本机产生的数据包

#### 规则(rules)

规则分别指定了源地址、目的地址、传输协议（如TCP、UDP、ICMP）和服务类型（如HTTP、FTP和SMTP）等。当数据包与规则匹配时，iptables就根据规则所定义的方法来处理这些数据包，如放行（accept）、拒绝（reject）和丢弃（drop）等。配置防火墙的主要工作就是添加、修改和删除这些规则。  

规则由 匹配条件 和 处理动作 组成。匹配条件 又分为 基本匹配条件 与 扩展匹配条件。基本匹配条件如：源地址 Source IP，目标地址 Destination IP；扩展匹配条件通常以模块的形式存在，这些模块可以按需安装，源端口 Source Port, 目标端口 Destination Port。  

处理动作(target) 也分为基本动作和扩展动作。以下列举一些常用的动作：
1. ACCEPT： 允许数据包通过
2. DROP： 直接丢弃数据包，不给任何回应信息，这时候客户端会感觉自己的请求泥牛入海了，过了超时时间才会有反应
3. QUEUE： 将数据包移交到用户空间
4. RETURN： 停止执行当前链中的后续规则，并返回到调用链(The Calling Chain)中
5. REJECT： 拒绝数据包通过，必要时会给数据发送端一个响应的信息，客户端刚请求就会收到拒绝的信息
6. DNAT： 目标地址转换
7. SNAT： 源地址转换，解决内网用户用同一个公网地址上网的问题
8. MASQUERADE： 是 SNAT 的一种特殊形式，适用于动态的、临时会变的 ip 上
9. REDIRECT： 在本机做端口映射
10. LOG： 记录日志信息，除记录外不对数据包做任何其他操作，仍然匹配下一条规则

#### 数据包流向
```
                              XXXXXXXXXXXXXXXXXX
                            XXX     Network    XXX
                              XXXXXXXXXXXXXXXXXX
                                      +
                                      |
                                      v
+-------------+              +------------------+
|table: filter| <---+        | table: nat       |
|chain: INPUT |     |        | chain: PREROUTING|
+-----+-------+     |        +--------+---------+
      |             |                 |
      v             |                 v
[local process]     |           ****************          +--------------+
      |             +---------+ Routing decision +------> |table: filter |
      v                         ****************          |chain: FORWARD|
****************                                          +------+-------+
Routing decision                                                  |
****************                                                  |
      |                                                           |
      v                         ****************                  |
+-------------+       +------>  Routing decision  <---------------+
|table: nat   |       |         ****************
|chain: OUTPUT|       |               |
+-----+-------+       |               |
      |               |               v
      v               |      +-------------------+
+--------------+      |      | table: nat        |
|table: filter | +----+      | chain: POSTROUTING|
|chain: OUTPUT |             +--------+----------+
+--------------+                      |
                                      v
                              XXXXXXXXXXXXXXXXXX
                            XXX    Network     XXX
                              XXXXXXXXXXXXXXXXXX
```

由图可知，当一个数据包进入计算机的网络接口时，数据首先进入 POSTROUTING 链，然后内核根据路由表决定数据包的目标。若数据包的目的地址是本机，则将数据包送往 INPUT 链进行规则匹配，当数据包进入 INPUT 链后，系统的任何进程都可以收到它，本机上运行的程序可以发送该数据包，这些数据包会经过 OUTPUT 链，再从 POSTROUTING 链发出；若数据包的目的地址不是本机，则检查内核是否允许转发，若允许，则将数据包送 FORWARD 链进行规则匹配，若不允许，则丢弃该数据包。若是主机本地进程产生并准备发出的包，则数据包被送往 OUTPUT 链进行规则匹配。

#### iptables 使用方法
```
iptables [-t 表]
         命令选项
         [链]
         [匹配选项]
         [操作选项]
```

命令选项
```
-A --append 在指定链的末尾添加一条新的规则
-D --delete 删除指定链中的某一条规则，按规则序号或内容确定要删除的规则
-I --insert 在指定链中插入一条新的规则，默认在链的开头插入
-R --replace 修改、替换指定链中的一条规则，按规则序号或内容确定
-F --flush 清空指定链中的所有规则，默认清空表中所有链的内容
-N --new 新建一条用户自己定义的规则链
-X --delete-chain 删除指定表中用户自定义的规则链
-P --policy 设置指定链的默认策略
-F, --flush 清空指定链上面的所有规则，如果没有指定链，清空表上所有链的所有规则
-Z, --zero 把指定链或表中的所有链上的所有计数器清零
-L --list 列出指定链中的所有的规则进行查看，默认列出表中所有链的内容
-S --list-rules 以原始格式列出链中所有规则
-v --verbose 查看规则列表时显示详细的信息
-n --numeric 用数字形式显示输出结果，如显示主机的 IP 地址而不是主机名
--line-number 查看规则列表时，同时显示规则在链中的顺序号
```
匹配选项
```
-i --in-interface 匹配输入接口，如 eth0，eth1
-o --out-interface 匹配输出接口
-p --proto 匹配协议类型，如 TCP、UDP 和 ICMP等
-s --source 匹配的源地址
--sport 匹配的源端口号
-d --destination 匹配的目的地址
--dport 匹配的目的端口号
-m --match 匹配规则所使用的过滤模块
```

过滤模块  

匹配封包来源网络接口的硬件地址，这个参数不能用在 OUTPUT 和 POSTROUTING 规则链上，因为封包要送到网卡后，才能由网卡驱动程序透过 ARP 通讯协议查出目的地的 MAC 地址，所以 iptables 在进行封包匹配时，并不知道封包会送到哪个网络接口去。
```
-m mac --mac-source
iptables -A INPUT -m mac --mac-source 00:00:00:00:00:01
```

匹配封包是否被表示某个号码，当封包被匹配成功时，可以透过 MARK 处理动作，将该封包标示一个号码，号码最大不可以超过 4294967296。
```
-m mac --mark
iptables -t mangle -A INPUT -m mark --mark 1
```

操作选项  
一般为`-j`处理动作 的形式，处理动作包括ACCEPT、DROP、RETURN、REJECT、DNAT、SNAT等。不同的处理动作可能还有额外的选项参数，如指定DNA、SNAT动作则还需指定`--to`参数用以说明要装换的地址，指定`REDIRECT动作则需指定`--to-ports`参数用于说明要跳转的端口。


#### 常用命令
查看帮助
```
iptables --help             # 查看 iptables 的帮助
iptables -m 模块名 --help    # 查看指定模块的可用参数
iptables -j 动作名 --help    # 查看指定动作的可用参数
```

查看规则
```
iptables -nvL
iptables -t nat -nvL

# 显示规则序号
iptables -nvL INPUT --line-numbers
iptables -t nat -nvL --line-numbers
iptables -t nat -nvL PREROUTING --line-numbers

# 查看规则的原始格式
iptables -t filter -S
iptables -t nat -S
iptables -t mangle -S
iptables -t raw -S
```

清除所有规则
```
iptables -F  # 清空表中所有的规则
iptables -X  # 删除表中用户自定义的链
iptables -Z  # 清空计数

iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X
iptables -t security -F
iptables -t security -X
```

增加、删除、修改规则
```
# 增加一条规则到最后
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# 注：以下几条操作都需要使用规则的序号，需要使用 -L --line-numbers 参数先查看规则的顺序号

# 添加一条规则到指定位置
iptables -I INPUT 2 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# 删除一条规则
iptabels -D INPUT 2

# 修改一条规则
iptables -R INPUT 3 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
```

开放指定端口
```
# 允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的或相关连接的通行
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 允许所有本机向外的访问
iptables -A OUTPUT -j ACCEPT

# 允许 22,80,443 端口的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dports 80,443 -j ACCEPT

# 如果有其他端口的需要开放，则同上
iptables -A INPUT -p tcp --dport 8000:8010 -j ACCEPT

# 允许 ping
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# 禁止其他未允许的规则访问
iptables -A INPUT -j REJECT
iptables -A FORWARD -j REJECT

# 注：以上操作需要安装顺序，确保规则顺序正确
```

目的地址转换，需要开启转发功能
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```
目的地址转换
```
# 把从 eth0 进来要访问 TCP/80 的数据包的目的地址转换到 192.168.1.18
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 80 -j DNAT --to 192.168.1.18

# 把从 123.57.172.149 进来要访问 TCP/80 的数据包的目的地址转换到 192.168.1.118:8000
iptables -t nat -A PREROUTING -p tcp -d 123.57.172.149 --dport 80 -j DNAT --to 192.168.1.118:8000
```

源地址转换
```
# 最典型的应用是让内网机器可以访问外网：
# 将内网 192.168.0.0/24 的源地址修改为 1.1.1.1 (可以访问互联网的机器的 IP)
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to 1.1.1.1

# 将内网机器的源地址修改为一个 IP 地址池
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to 1.1.1.1-1.1.1.10
```

持久化
```
# 保存当前规则
iptables-save > iptables.20190721

# 恢复备份规则
iptables-restore < iptables.20190721
```





