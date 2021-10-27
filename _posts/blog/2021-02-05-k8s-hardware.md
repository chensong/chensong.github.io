---
layout: post
title: Kubernetes 硬件
categories: [Blog, kubernetes, k8s]
description: Kubernetes 硬件
keywords: Kubernetes, k8s
---

## 1. 主机选型

本文介绍构建Kubernetes集群时该如何选择ECS类型以及选型的注意事项。

### 集群规划

目前在创建Kubernetes集群时，存在着使用很多小规格ECS的现象，这样做有以下弊端：

* 小规格Woker ECS的网络资源受限。
* 如果一个容器基本可以占用一个小规格ECS，此ECS的剩余资源就无法利用（构建新的容器或者是恢复失败的容器），在小规格ECS较多的情况下，存在资源浪费。

使用大规格ECS的优势：

* 网络带宽大，对于大带宽类的应用，资源利用率高。
* 容器在一台ECS内建立通信的比例增大，减少网络传输。
* 拉取镜像的效率更高。因为镜像只需要拉取一次就可以被多个容器使用。而对于小规格的ECS拉取镜像的次数就会增多，若需要联动ECS伸缩集群，则需要花费更多的时间，反而达不到立即响应的目的。

### 选择Master节点规格

通过容器服务创建的Kubernetes集群，Master节点上运行着etcd、kube-apiserver、kube-controller等核心组件，对于Kubernetes集群的稳定性有着至关重要的影响，对于生产环境的集群，必须慎重选择Master规格。Master规格跟集群规模有关，集群规模越大，所需要的Master规格也越高。

说明 您可从多个角度衡量集群规模，例如节点数量、Pod数量、部署频率、访问量。这里简单的认为集群规模就是集群里的节点数量。

对于常见的集群规模，可以参见如下的方式选择Master节点的规格（对于测试环境，规格可以小一些。下面的选择能尽量保证Master负载维持在一个较低的水平上）。

节点规模      | Master规格
--------------|-----------
1~5个节点     | 4核8 GB（不建议2核4 GB）
6~20个节点    | 4核16 GB
21~100个节点  | 8核32 GB
100~200个节点 | 16核64 GB

选择Worker节点规格

* ECS规格要求：CPU大于等于4核，且内存大于等于8 GiB。
* 确定整个集群的日常使用的总核数以及可用度的容忍度。

  例如：集群总的核数有160核，可以容忍10%的错误。那么最小选择10台16核ECS，并且高峰运行的负荷不要超过160*90%=144核。如果容忍度是20%，那么最小选择5台32核ECS，并且高峰运行的负荷不要超过160*80%=128核。这样就算有一台ECS出现故障，剩余ECS仍可以支持现有业务正常运行。

* 确定CPU：Memory比例。对于使用内存比较多的应用例如Java类应用，建议考虑使用1:8的机型。

## 参考文章

* <https://help.aliyun.com/document_detail/98886.html>
* <https://kubernetes.io/zh/docs/setup/best-practices/cluster-large/>
* <https://v1-20.docs.kubernetes.io/zh/docs/setup/best-practices/cluster-large>
* <https://cloud.google.com/compute/all-pricing#n1_machine_types>
* <https://zh.wikipedia.org/wiki/Google%E8%AE%A1%E7%AE%97%E5%BC%95%E6%93%8E>
* <https://blog.csdn.net/weixin_42715225/article/details/109725742>
