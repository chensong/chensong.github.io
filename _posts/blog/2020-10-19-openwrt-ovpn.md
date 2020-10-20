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
1. openvpn 网络模式，默认是 net30：表示掩码30位，有地址浪费，还有 P2P 模式，当然还有 subnet 这是比较推荐的
    ```
    option topology 'subnet'
    option push 'topology subnet'
    ```
1. 添加静态 IP 分配文件 option ifconfig_pool_persist '/etc/openvpn/ipp.txt'
    ```sh
    # 文件格式
    client1,10.0.100.10
    client2,10.0.100.20
    ```
1. 或者添加静态 IP 分配文件目录 option client_config_dir '/etc/openvpn/ccd'
    ```sh
    # 文件格式，每个客户端一个文件，文件名使用 CN 值，例如 
    # ccd/client1
    ifconfig-push 10.0.100.10 10.0.100.9
    # ccd/client2
    ifconfig-push 10.0.100.30 10.0.100.29
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

## 3、详细配置书面

### 3.1 server 端配置
```sh
#使用那种协议，可选的有，udp tcp-server tcp-client
    proto udp  
    port 1194

#tun路由模式，tap桥模式，据说tun效率高于tap，但是tun只能转发IP数据，tap是二层可以封装任何协议，window下只有tap模式
    dev tun

#该参数能防止密码被缓存到内存中
    auth-nocache    

# openvpn网络模式，默认是net30：表示掩码30位，有地址浪费，还有P2P模式，当然还有subnet这是比较推荐的
    topology subnet

#push表示推送，即将配置推送给客户端，让客户端也使用subnet模式
    push "topology subnet"

#定义分配给客户端的IP段，服务端自己默认使用第一个可以地址
    server 172.16.0.0 255.255.255.0

#在openvpn重启时,再次连接的客户端将依然被分配和以前一样的IP地址
    #ipp.txt文件格式 "nanchang,172.16.0.22"  一行一个
    ifconfig-pool-persist /etc/openvpn/beihai/ipp.txt

#认证相关的信息
    ca /etc/openvpn/ca/pki/ca.crt
    cert /etc/openvpn/ca/pki/issued/beihai.crt
    key /etc/openvpn/beihai/pki/private/beihai.key
    #dh 是server必须要有的，客户端可以不要，它定义了如何进行密钥交换
    dh /etc/openvpn/beihai/pki/dh.pem
    ##防DDOS攻击，openvpn控制通道的tls握手进行保护，服务器端0,客户端1
    tls-auth /etc/openvpn/beihai/ta.key 0

#待openvpn初始化完成后，将其降级为nobody权限运行
    user nobody
    group nobody

    persist-key #通过keepalive检测超时后，重新启动VPN，不重新读取keys，保留第一次使用的keys
    persist-tun #通过keepalive检测超时后，重新启动VPN，一直保持tun或者tap设备是linkup的，否则网络连接会先linkdown然后linkup
    comp-lzo        #允许数据压缩，如果启用了客户端配置文件也需要有这项
    keepalive 5 20  #表示每隔5秒ping一下客户端/服务端，若是20秒内无响应，认为down，随即重启openvpn（强烈开启）
    max-clients 100 #最大客户端并发连接数量    

    push "dhcp-option DNS 8.8.8.8"  #指定客户端使用的主DNS
    push "dhcp-option DNS 8.8.4.4"  #指定客户端使用的备DNS
    client-to-client            #默认客户端间不能互访，开启客户端互访，tap模式会形成广播域，tun不会
    duplicate-cn                    #支持一个证书多个客户端登录使用，建议不启用

#开启对客户端进行细粒度控制（该目录需要手动创建，名字为客户端的证书辨识名）
    client-config-dir /etc/openvpn/beihai/ccd

#给自己本地添加一个到隧道的路由
    route 192.168.29.0 255.255.255.0

#推送一条路由信息给客户端
#推送路由，若是推送失败，需要检查server 是否设置正常，该故障我遇到过，设置ifconfig-pool了，发现推送失效
    push "route 192.168.11.0 255.255.255.0"

#重定向默认网关
为什么要重定向网关：vpn客户端是经常出差的，网络环境不安全，希望它将所有流量传到公司，经公司出口
    #其中包含的flags有"local autolocal def1 bypass-dhcp bypass-dns block-local ipv6 !ipv4"（多个标志之间用空格分隔），
    #推荐使用def1，它使用0.0.0.0/1和128.0.0.0/1而不是0.0.0.0/0来覆盖默认网关，即有新路由也保留原始默认网关，只是优先匹配而已
    #block-local 是表示当客户端拨入后，阻断其除与本地网关的访问外，本地的其他IP都不允许访问
    push "redirect-gateway def1 bypass-dhcp"

#记录日志，每次重新启动openvpn后追加原有的log信息
    log-append /var/log/openvpn.log

#日志记录级别，可选0-9，0只记录错误信息，4能记录普通的信息，5和6在连接出现问题时能帮助调试，9显示所有信息，甚至连包头等信息都显示（像tcpdump） 
    verb 3

#状态文件：定期(默认60s)把状态信息写到该文件，以便自己写程序计费或者进行其他操作（需要关闭selinux）
    status /var/log/openvpn-status.log

#相同信息的数量，如果连续出现 20 条相同的信息，将不记录到日志中。
    mute 20
```

CCD配置文件语法，CCD能对客户端进行细粒度控制
```sh
    ifconfig-push    ：向客户端推送虚拟IP地址，它将会覆盖掉 --config-pool 的动态分配地址
    local
    remote-netmask    ：他们的设置相当于--config 选项，用于在客户端机器上远程执行vpn通道的配置
    push
    push-reset
    iroute  ：用于从服务器向特定的客户端生成内部路由，不管客户端从哪里拨入，总是将某个特定子网从服务器路由到客户端，同时需要在服务器端使用 --route 指令添加该系统路由
    config

#事例
    cat /etc/openvpn/beihai/ccd/nanchang

ifconfig-push 172.16.0.88 255.255.255.0    #表示给客户端分配一个特定IP地址
push "redirect-gateway def1 bypass-dhcp"   #重定向默认网关
```
