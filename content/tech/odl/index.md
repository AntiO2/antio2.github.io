---
title: "OpenDaylight实验"
description: 
date: 2022-05-20T18:44:49+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
 - 网络
categories:
 - 笔记
---



## 安装

找到安装包文件

```bash
# unzip lithium.zip
# cd distribution-karaf-0.3.0-Lithium
```

启动控制器

`#./bin/karaf`直接启动

或者

```bash
#./bin/start
#./bin/client -u karaf
```

后者即使退出控制台，控制器依然运行

安装OpenDaylight组件

```bash
> feature:install odl-restconf
> feature:install odl-l2switch-switch
> feature:install odl-openflowplugin-all
> feature:install odl-dlux-all
> feature:install odl-mdsal-all
> feature:install odl-adsal-northbound
```

使用

`>feature:list -i`查看所有已安装组件

访问

`http://[ODL_host_ip\]:8080/index.html`OpenDaylightWeb界面

## 使用OpenDaylight界面下发流表

| **字段名称**                            | **说明**                                                     |
| :-------------------------------------- | :----------------------------------------------------------- |
| in_port=port                            | 传递数据包的端口的OpenFlow端口编号                           |
| dl_vlan=vlan                            | 数据包的VLAN Tag值，范围是0-4095，0xffff代表不包含VLAN Tag的数据包 |
| dl_src=<MAC> dl_dst= <MAC>              | 匹配源或者目标的MAC地址 01:00:00:00:00:00/01:00:00:00:00:00 代表广播地址 00:00:00:00:00:00/01:00:00:00:00:00 代表单播地址 |
| dl_type=ethertype                       | 匹配以太网协议类型，其中： dl_type=0x0800 代表IPv4协议； dl_type=0x086dd 代表IPv6协议； dl_type=0x0806 代表ARP协议； |
| nw_src=ip[/netmask] nw_dst=ip[/netmask] | 当 dl_typ=0x0800 时，匹配源或者目标的IPv4地址，可以使IP地址或者域名 |
| table=number                            | 指定要使用的流表的编号，范围是0-254。在不指定的情况下，默认值为0。通过使用流表编号，可以创建或者修改多个Table中的Flow。 |

## OpenDaylight SFC项目基础

SFC核心组件如下： 

- Classification：根据初始化的（配置好的）policy匹配数据流进行封装，然后转入到Service Function Chain中
- Service Function(SF)：负责对收到的数据包进行特定功能的处理。作为一个逻辑上的组件，SF在具体实现的上可以是一个虚拟的元素，或者是嵌入在具体网络设备上的某种功能。常见的SF有：防火墙(firewall)，WAN设备加速器，深层报文检测(Deep Packet Inspection,DPI)，NAT等等。 
- Service Function Forwarder(SFF)：主要负责Service Function Chaining上的流量转发控制。 Service Function Chain(SFC)：SFC定义了一个抽象的Service Function有序集合。经过分类后的包要依次去遍历集合中的Service Function。比如：用户可以配置firewall->qos->dpi三种服务来构建一条SFC。 Rendered Service Path(RSP)：数据包实际行走的路径。 
- Service Function Path(Service Function Path)：SFP是一个逻辑概念，它是介于SFC和RSP之间的一层抽象，有时候会将SFP与SFC等同。

## 基于RESTCONF的拓扑查询

OpenDaylight的拓扑RESTful API对应的子资源点有两个分别为CONFIG和OPERATIONAL，CONFIG主要是拓扑的配置信息，OPERATIONAL主要是运行时的拓扑信息。每种类型的拓扑中包含两个模块的拓扑信息，flow模块和ovsdb模块。在OpenDaylight没有安装ovsdb模块时，ovsdb拓扑是不展示的。
在CONFIG类型中的拓扑包含ovsdb模块的配置信息如配置的网桥、端口、隧道等，flow模块中包含node、link以及流表的配置信息。
在OPERATIONAL类型中flow模块包含node信息，以及link信息。ovsdb拓扑包含ovsdb的配置信息，端口的流量信息。ovsdb的配置信息中包含当前连接的控制器信息、和控制器通信的 OpenFlow协议版本信息、bridge配置信息等。

### 交换机

```bash
ovs-vsctl show #查看信息
ovs-vsctl del-controller br-sw 
ovs-vsctl set-controller br-sw tcp:30.0.1.3:6633
```

连接控制器