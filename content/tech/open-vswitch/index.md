---
title: "Open vSwitch"
description: 
date: 2022-05-24T21:03:29+08:00
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

## Open vSwitch网桥管理



据包可以在内核中执行操作。
ovs-vsctl关于网桥管理的常用命令如下：

| 命令                         | 含义                             |
| :--------------------------- | :------------------------------- |
| init                         | 初始化数据库（前提数据分组为空） |
| show                         | 打印数据库信息摘要               |
| add-br BRIDGE                | 添加新的网桥                     |
| del-br BRIDGE                | 删除网桥                         |
| list-br                      | 打印网桥摘要信息                 |
| list-ports BRIDGE            | 打印网桥中所有port摘要信息       |
| add-port BRIDGE PORT         | 向网桥中添加端口                 |
| del-port [BRIDGE] PORT       | 删除网桥上的端口                 |
| get-controller BRIDGE        | 获取网桥的控制器信息             |
| del-controller BRIDGE        | 删除网桥的控制器信息             |
| set-controller BRIDGE TARGET | 向网桥添加控制器                 |

## OVS流表管理

ovs-ofctl是Open vSwitch提供的命令行工具。在没有配置OpenFlow控制器的模式下，用户可以使用ovs-ofctl命令通过OpenFlow协议连接Open vSwitch来创建、修改或删除Open vSwitch中的流表项，并对Open vSwitch的运行状况进行动态监控。ovs-ofctl关于流表管理的常用命令如下表所示。



| 命令                   | 含义                       |
| ---------------------- | -------------------------- |
| show Switch            | 输出OpenFlow信息           |
| dump-ports SWITCH PORT | 输出端口统计信息           |
| dump-ports-desc SWITCH | 输出端口描述信息           |
| dump-flows SWITCH      | 输出交换机所有流表项       |
| dump-flows SWITCH FLOW | 输出交换机中匹配的流表项   |
| add-flow SWITCH FLOW   | 添加流表项                 |
| add-flows SWITCH FILE  | 在文件中向交换机添加流表项 |
| mod-flows SWITCH FLOW  | 修改交换机的流表项         |
| del-flows SWITCH FLOW  | 删除交换机流表项           |

常见的流表操作

| 操作                         | 说明                                          |
| ---------------------------- | --------------------------------------------- |
| output:port                  | 输出数据包到指定端口.port指OpenFlow的端口编号 |
| mod_vlan_vid                 | 修改数据包中的VLANtag                         |
| strip_vlan                   | 移除数据包中的VLAN tag                        |
| mod_dl_src/mod_dl_dest       | 修改源或者目标的MAC地址信息                   |
| mod_nw_src/mod_nw_dst        | 修改源或者目标IPv4地址                        |
| resubmit:port                | 替换流表的in_port字段,并重新进行匹配          |
| load:valuse->dst[start..end] | 写数据到指定的字段                            |

流表项作为ovs-ofctl的值,采用`字段=值`的格式,用空格或者逗号分开。

| 字段名称                                 | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| in_port=port                             | 传递数据包端口的OpenFLow端口号                               |
| dl_vlan=vlan                             | 数据包的VLAN Tag值，范围是0~4095,0xffff代表不包含VLAN tag数据包 |
| dl_src=<MAC>dl_dst=<MAC>                 | 匹配源或者目标的MAC地址                                      |
| dl_type=ethertype                        | 匹配以太网协议类型,dl_type=0x0800代表IPv4,dl_type=0x086dd代表IPv6,dl_type=0x0806代表ARP |
| nw_src=ip[/netmask]  nw_dst=ip[/netmask] | 可以使用IP或者域名                                           |
| nw_proto=proto                           | 代表协议编号                                                 |
| table=number                             | 指定流表编号，范围是0~254，在不指定的情况下，默认值为0，通过使用流表编号，可以创建或者修改多个Table中的Flow |
| reg<idx>=value[/mask]                    | 交换机中寄存器的值，当一个数据包进入交换机，所有的寄存器都被清零，用户可以通过Action的指令修改寄存器中的值 |

## OpenFlow建立连接交互流程学习

连接设置 OF交换机与控制器连接之前，有3个参数需要提前设置，包括控制器IP地址、控制器端口号以及传输协议。 多控制器 OF-CONFIG协议提供交换机同时与多控制器连接的参数配置。 OpenFlow逻辑交换机 OpenFlow v1.3.1协议规定了与OpenFlow逻辑交换机有关的各种OpenFlow资源。OF-CONFIG协议必须支持对这些OpenFlow资源的配置，如对OpenFlow逻辑交换机进行端口和队列等资源的配置。 连接中断 当交换机与控制器失去连接时，有Fail Secure Mode（失败安全模式）和Fail Standalone Mode（失败独立模式）两种模式可选择，OF-CONFIG协议可预先为OF交换机配置连接失效后进入的模式。 加密 为安全考虑，OF交换机与控制器第一次建立连接时，双方均进行身份认证，OF-CONFIG协议提供用户配置，两者以TLS建立连接的身份认证方式。 队列 OF-CONFIG协议提供对OF交换机队列最小速率（Minrate）、最大速率（Maxrate）以及自定义速率（Experimenter）3个参数的配置。 端口 虽然OpenFlow协议本身对交换机端口参数可以进行部分配置，但不够全面系统。而端口属性配置是网络配置中必不可少的一项，OF-CONFIG协议提供以下4种属性的配置，包括禁止接收、禁止转发、禁止Packet-in消息以及管理状态等，同时也可对端口速率、双工、铜介质、光纤介质、自动协商、暂停以及非对称暂停等参数进行配置。同时，在数据中心网络等网络虚拟化环境中，OF-CONFIG协议还支持逻辑端口的配置，目前版本的OF-CONFIG协议可以支持IPinGRE、VxLAN以及NVGRE，之后的OF-CONFIG版本可能会支持其他类型的隧道。 能力发现 OpenFlow v1.3.1协议规范了多种虚拟交换机能力特性，如多种Action类型。虽然配置这些能力超出了OF-CONFIG协议的范围

| **项目**   | **说明**                                                     |
| :--------- | :----------------------------------------------------------- |
| 控制器连接 | 支持在交换机上配置控制器参数，包括IP地址、端口号及连接使用的传输协议等 |
| 多控制器   | 支持多个控制器的参数配置                                     |
| 逻辑交换机 | 支持对逻辑交换机（即OF交换机的实例）的资源（如队列、端口等）设置，且支持带外设置 |
| 连接中断   | 支持故障安全，故障脱机等两种应对模式的设置                   |
| 加密传输   | 支持控制器和交换机之间TLS隧道参数的设置                      |
| 队列       | 支持队列参数的配置，包括最小速率、最大速率、自定义参数等     |
| 端口       | 支持交换机端口的参数和特征的配置，即使OpenFlow v1.2中并没有要求额外的配置方式 |
| 能力发现   | 发现虚拟交换机的能力特性                                     |

2、 实际操作运维的设计需求
除了上述为支持OpenFlow协议而制定的设计需求外，为便于交换机的操作运维，OF-CONFIG v1.1.1协议必须支持以下几种场景：

- 支持OF交换机被多个OpenFlow配置点配置；
- 支持一个OpenFlow配置点管理多个OF交换机；
- 支持一个OpenFlow逻辑交换机被多个控制器控制；
- 支持配置OpenFlow逻辑交换机的端口和队列；
- 支持OpenFlow逻辑交换机的能力发现；
- 支持配置隧道如IPinGRE、VxLAN以及NVGRE。
  3、 交换机管理协议需求
  OF-CONFIG v1.1.1协议定义了OF交换机与OpenFlow配置点间的通信标准，主要包括交换机的管理协议部分以及数据模型的设计。其中数据模型部分将在下一节具体介绍，交换机的管理协议部分则有以下设计需求：
- 保障安全性，支持完整、私有以及认证，支持对交换机和配置点双向认证；
- 支持配置请求和应答的可靠传输；
- 支持由配置点或者交换机进行连接设置；
- 能够承载局部交换机配置以及大范围交换机配置；
- 支持配置点在交换机配置参数以及接收来自交换机的配置参数；
- 支持在交换机创建、更改以及删除配置信息，并支持报告配置成功的结果以及配置失败的错误码；
- 支持独立发送配置请求，并支持交换机到配置点的异步通知；
- 支持记忆能力、可伸展性以及报告其自身属性和能力。

