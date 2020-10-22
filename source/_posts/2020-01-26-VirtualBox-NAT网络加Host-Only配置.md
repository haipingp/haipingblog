---
title: VirtualBox-NAT网络加Host-Only配置
date: 2020-01-26 23:30:09
categories:
- 虚拟机
---


## 步骤

* 在VirtualBox中新建虚拟机
<!-- more -->

* 全局工具中关于host-only的配置（一般安装好VirtualBox之后，在网络中会多出一个HostOnly的网络）：

![2020-01-26-1.png](https://haipingp.github.io/myPics/2020-01-26-1.png)

* 全局设定中关于NAT网络的设置：

![2020-01-28-2044.png](https://haipingp.github.io/myPics/2020-01-28-2044.png)

* 右键新建的虚拟机，点击设置，点击网络，网卡1和网卡2的设置如下：

![](https://haipingp.github.io/myPics/2020-01-28-2319.png)
![](https://haipingp.github.io/myPics/2020-01-28_23-19-41.png)

* 安装Ubuntu 18.04 Server

* ifconfig -a查看网络信息，其中`enp0s3`是NAT网络，`enp0s8`是Host-Only网络，需要在文件`/etc/netplan/50-cloud-init.yaml`作如下配置：

```
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            dhcp4: false
            dhcp6: false
            addresses: [192.168.56.101/24]
    version: 2
```

* 输入以下命令使配置生效，并通过ifconfig查看是否ip为配置的ip:

```
netplan apply
```

* 可以通过xshell访问`192.168.56.101:22`

