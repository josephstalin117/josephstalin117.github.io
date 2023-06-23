---
layout: post
title:  "zerotier"
date:   2023-6-23 20:35:35 +0800
categories: command
---


# 安装MOON

下载zerotier-one
```
curl -s https://install.zerotier.com | sudo bash
```

进入zerotier-one程序所在的目录
```
cd /var/lib/zerotier-one
```

生成moon.json配置文件
```
sudo zerotier-idtool initmoon identity.public > moon.json
```

生成文件
```
 "id": "1c110b9ac2",
 "objtype": "world",
 "roots": [
  {
   "identity": "9c960b9ac2:0:daca38dfc5f3",
   "stableEndpoints": []
  }
 ],
 "signingKey": "676f0c29eb8d6f2f00ce22ee2082b3ec",
 "signingKey_SECRET": "39de9f7ab16d0adb035276b7281f73344",
 "updatesMustBeSignedBy": "676f0c29eb8d6f2f00ce22ee",
 "worldType": "moon"
}
```

这里我们需要根据自己服务器的公网静态IP，修改`stableEndpoints`那一行格式如下，其中`11.22.33.44`为你的公网IP，`9993`是默认的端口号：
```
"stableEndpoints": [ "11.22.33.44/9993" ]
```

根据`moon.json`文件生成真正需要的签名文件`.moon`：
```
sudo zerotier-idtool genmoon moon.json
输出
wrote 0000009c960b9ac2.moon (signed world with timestamp 1280398410930)
```

移动`.moon`签名文件到`moons.d`目录下并且重启服务
```
cd /var/lib/zerotier-one
mkdir moons.d
mv 000000*.moon moons.d
service zerotier-one restart
```


# 其他客户端使用

本地加入`.moon`签名文件，并重启服务器
```
#For linux
# 一般是 /var/lib/zerotier-one 目录下
cd /var/lib/zerotier-one
mkdir moons.d
# 将 .moon 文件移动到 moons.d 目录下

#For windows
直接拷贝文件至C:\ProgramData\ZeroTier\One\moons.d目录下即可，通常默认配置均为这个路径，不然请从服务中找到文件路径
```

# 检测连接是否成果
```
#for linux and windows(windows需要用管理员模式启动cmd输入)
zerotier-cli listpeers
```

# 加入网络
```
# 加入网络
sudo zerotier-cli join network_id

# 退出网络
sudo zerotier-cli leave network_id
```
