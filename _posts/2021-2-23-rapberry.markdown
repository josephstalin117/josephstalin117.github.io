---
layout: post
title:  "respberry"
date:   2021-2-23 20:35:35 +0800
categories: command
---

#### 安装ssh

`/boot`中新建`/ssh`文件夹


#### 配置wifi

`/boot`新建`wpa_supplicant.conf`文件
```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="WiFi-A"
psk="12345678"
key_mgmt=WPA-PSK
priority=1
}
```

#### 查看树莓派版本

```
getconf LONG_BIT        # 查看系统位数
uname -a            # kernel 版本
/opt/vc/bin/vcgencmd  version   # firmware版本
strings /boot/start.elf  |  grep VC_BUILD_ID    # firmware版本
cat /proc/version       # kernel
cat /etc/os-release     # OS版本资讯
cat /etc/issue          # Linux distro 版本
cat /etc/debian_version     # Debian版本编号
```

#### 系统更新
```
sudo apt-get update
```

