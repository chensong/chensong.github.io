---
layout: post
title: Kubernetes 异常 cpu.cfs_period_us
categories: [Blog, k8s]
description: Kubernetes 异常 cpu.cfs_period_us
keywords: kubernetes, k8s
---

## 1. 异常

### 1.1 现象

我部署的一个 pod 突然没有再次出现并挂在“CrashLoopBackOff”中
pod 无法运行，提示 CrashLoopBackOff, ContainerCannotRun

```sh
# kubectl describe pod 显示此错误消息：
OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:402: container init caused \"process_linux.go:367: setting cgroup config for procHooks process caused \\\"failed to write 200000 to cpu.cfs_quota_us: write /sys/fs/cgroup/cpu,cpuacct/container.slice/kubepods/burstable/pod6ba8075b-132e-11e9-ab2e-246e9674888c/a5f752a5a36fafeab7f16beb4763521cf2370efc3ba961e85a8ac1faef721b48/cpu.cfs_quota_us: invalid argument\\\"\"": unknown
```

### 1.2 原因与修复

当前环境操作系统: Ubuntu 16.04.6 LTS 4.4.0-154-generic
根据参考文章中的内容得知，这是一个 kernel 问题，升级内核即可

开启了自动更新的系统，重启后就会升级内核：4.4.0-210-generic
问题就解决了

网上还有一个 fix-cfs.py 的临时修复脚本可以试用，推荐升级内核

## 参考文章

* <https://github.com/kubernetes/kubernetes/issues/72878>
* <https://lore.kernel.org/lkml/20191004001243.140897-1-xueweiz@google.com/>
