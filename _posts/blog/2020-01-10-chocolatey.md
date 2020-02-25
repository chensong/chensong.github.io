---
layout: post
title: scoop, chocolatey 包管理工具
categories: [Blog, scoop, chocolatey]
description: 包管理工具
keywords: chocolatey
---

## chocolatey

### 1、配置安装目录

```powershell
# 修改默认安装目录
[environment]::SetEnvironmentvariable("ChocolateyInstall", "D:\Chocolatey", "User")
```

### 2、安装

```powershell
# 修改权限
Set-ExecutionPolicy RemoteSigned -scope CurrentUser

# install
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')
# or shorter
iwr -useb chocolatey.org/install.ps1 | iex


# 还原权限
Set-ExecutionPolicy Undefined -scope CurrentUser
```

### 3、配置 proxy

```cmd
# 配置代理
choco config set proxy http://127.0.0.1:1080

# 删除代理
choco config unset proxy
```

### 4、常用软件

```cmd

```