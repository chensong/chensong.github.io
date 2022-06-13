---
layout: post
title: Linux 修改网卡固定 IP
categories: [Blog, linux, vim]
description: Linux 修改网卡固定 IP, vim 粘贴格式错乱
keywords: linux, vim
---

## Linux 修改网卡固定 IP

```sh
sudo vi /etc/hostname 
hostname

sudo vi /etc/netplan/50-cloud-init.yaml 
sudo netplan apply

sudo reboot
```

```yaml
# /etc/netplan/50-cloud-init.yaml 修改前
network:
    ethernets:
        ens33:
            dhcp4: true
    version: 2
```

```yaml
# /etc/netplan/50-cloud-init.yaml 修改后
network:
    ethernets:
        ens33:
            dhcp4: no
            addresses: [192.168.208.201/24]
            gateway4: 192.168.208.2
            nameservers:
                addresses: [192.168.208.2]
    version: 2
```

## vim 复制粘贴格式错乱

### vim进入 paste 模式，命令如下

```txt
:set paste

进入 paste 模式之后，再按 i 进入插入模式，进行复制、粘贴就很正常了。
```

### 命令模式下，输入

```txt
:set nopaste

解除 paste 模式。
```

### paste 模式主要帮我们做了如下事情

* textwidth设置为0
* wrapmargin设置为0
* set noai
* set nosi
* softtabstop设置为0
* revins重置
* ruler重置
* showmatch重置
* formatoptions使用空值
