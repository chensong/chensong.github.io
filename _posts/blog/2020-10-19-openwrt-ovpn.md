---
layout: post
title: openwrt 配置 openvpn
categories: [Blog, openwrt, openvpn]
description: openwrt 配置 openvpn
keywords: openwrt, openvpn
---

## 1、安装
```sh
# Install packages
# 也可在 web ui 安装： 系统 > Software
opkg update
opkg install openvpn-openssl openvpn-easy-rsa luci-i18n-openvpn-zh-cn
```

## 2、配置 Server

1. 生成证书
    ```sh
    mkdir -p /etc/openvpn
    cd /etc/openvpn

    # Remove and re-initialize the PKI directory
    easyrsa init-pki
    
    # Generate DH parameters
    easyrsa gen-dh
    
    # Create a new CA
    easyrsa build-ca nopass
    
    # Generate a key pair and sign locally for a server
    easyrsa build-server-full server nopass
    
    # Generate a key pair and sign locally for a client
    easyrsa build-client-full client1 nopass
    easyrsa build-client-full client2 nopass
    
    # Generate TLS PSK
    OVPN_PKI="/etc/easy-rsa/pki"
    openvpn --genkey --secret ${OVPN_PKI}/tc.pem
    ```

1. 在 web ui 操作，按模板添加，模板名称至少 4 个字符
1. server bridge: 使用 tap 网卡，不支持 Android 客户端
1. server multi-client: 使用 tun 网卡，支持 Android 客户端
1. 在此选择 tun 模式
1. 选择证书文件 ca.crt dh.pem server.crt server.key
1. 添加客户端通信 option client_to_client '1'
1. 添加端口 option port '1194'
1. 添加协议 option proto 'tcp-server'
1. 添加静态 IP 分配文件 option ifconfig_pool_persist '/etc/openvpn/ipp.txt'
    ```
    # 文件格式
    client1,10.0.100.10
    client2,10.0.100.20
    # tap 模式正常，tun 模式会报错，错误关键词：subnet30 (255.255.255.252)
    # 以下对应 ip 分别为 10 30 50 70 90
    client1,10.0.100.8
    client2,10.0.100.28
    client3,10.0.100.48
    client4,10.0.100.68
    client5,10.0.100.88

    client5,10.0.100.[ip-2]
    ```
1. 或者添加静态 IP 分配文件目录 option client_config_dir '/etc/openvpn/ccd'
    ```
    # 文件格式，每个客户端一个文件，文件名使用 CN 值，例如 ccd/client1
    # 同上，tap 模式正常，tun 模式使用以下配置
    # client1
    ifconfig-push 10.0.100.10 10.0.100.9
    # client2
    ifconfig-push 10.0.100.30 10.0.100.29
    # ip = 4a+2
    ifconfig-push 10.0.100.[4a+2] 10.0.100.[4a+1]
    ```
1. 配置样例 /etc/config/openvpn
    ```
    config openvpn 'server_multi_client'
            option dev 'tun'
            option comp_lzo 'yes'
            option mssfix '1420'
            option keepalive '10 60'
            option verb '3'
            option ca '/etc/openvpn/ca.crt'
            option dh '/etc/openvpn/dh.pem'
            option cert '/etc/openvpn/server.crt'
            option key '/etc/openvpn/server.key'
            option client_to_client '1'
            option port '1194'
            option proto 'tcp-server'
            option ifconfig_pool_persist '/etc/openvpn/ipp.txt'
            option server '10.0.100.0 255.255.255.0'
            option enabled '1'

    config openvpn 'server_bridge'
            option dev 'tap'
            option mssfix '1420'
            option keepalive '10 60'
            option verb '3'
            option comp_lzo 'yes'
            option ca '/etc/openvpn/ca.crt'
            option cert '/etc/openvpn/server.crt'
            option key '/etc/openvpn/server.key'
            option port '1194'
            option dh '/etc/openvpn/dh.pem'
            option proto 'tcp-server'
            option server_bridge '192.168.1.2 255.255.255.0 192.168.1.201 192.168.1.210'
            option client_to_client '1'
            option ifconfig_pool_persist '/etc/openvpn/ipp.txt'
    ```

## 2、配置 Client

```
client
dev tun
;dev-node "client1"
proto tcp
remote d2play.jios.org 4343
comp-lzo
resolv-retry infinite
nobind
persist-key
persist-tun
;http-proxy-retry # retry on connection failures
;http-proxy proxy 3128
mute-replay-warnings
verb 3

;ca ca.crt # ..\\config\\ca.crt
;cert ca.crt # ..\\config\\client1.crt
;key ca.key #..\\config\\client1.key
;auth-user-pass test01.key # ..\\config\\test01.key

;redirect-gateway def1 bypass-dhcp
;dhcp-option DNS 42.120.21.30


<ca>
[copy ca.crt]
</ca>

<cert>
[copy issued/client1.crt]
</cert>

<key>
[copy private/client1.key]
</key>

```