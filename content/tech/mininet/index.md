---
title: "Mininet实验"
description: 
date: 2022-05-08T18:43:18+08:00
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

> Mininet是由一些虚拟的终端节点、交换机、路由器连接而成的一个网络仿真器，它采用轻量级的虚拟化技术使得系统可以和真实网络相媲美。 Mininet可以很方便地创建一个支持SDN的网络：host就像真实的电脑一样工作，可以使用ssh登录，启动应用程序，程序可以向以太网端口发送数据包，数据包会被交换机、路由器接收并处理。有了这个网络，就可以灵活地为网络添加新的功能并进行相关测试，然后轻松部署到真实的硬件环境中。

## 0x1 Mininet源码安装与验证

- Mininet架构按datapath的运行权限不同，分为kernel datapath和userspace datapath两种，其中kernel datapath把分组转发的逻辑编译进入Linux内核，效率非常高；userspace datapath把分组转发逻辑实现为一个应用程序，叫做ofdatapath，效率虽不及kernel datapath，但更为灵活，更容易重新编译。

- `ofprotocol`进程，它负责管理和维护同一控制器之间的安全信道。

安装方法：在本地安装Mininet源代码。该安装方法在安装过程中可以设置Open vSwitch的版本

进入`mininet/util`使用`./install.sh [options]`安装

- `-a` 完整安装
- `-s [dir]`指定目录安装（默认在home下）
- `-nfv`安装Mininet核心文件及依赖、OpenFlow和Open vSwitch。

安装完成后使用指令

`# mn --version`查看版本号，安装成功

## 0x2 Mininet拓扑构建与命令使用

`Miniedit`可视化，直接在界面上编辑任意拓扑，生成python自定义拓扑脚本，使用Mininet可视化界面方便了用户自定义拓扑创建，为不熟悉python脚本的使用者创造了更简单的环境，界面直观，可操作性强。

Mininet 2.2.0+内置`miniedit` 。在`mininet/examples`下提供`miniedit.py`脚本，执行脚本后显示可视化界面，可自定义拓扑及配置属性。
`MiniEdit`使用主要分三个步骤：`Miniedit`启动→自定义创建拓扑，设置设备信息→运行拓扑并生成拓扑脚本。

Mininet支持创建的网络拓扑为：minimal、single、linear和tree。

- minimal：创建一个交换机和两个主机相连的简单拓扑。默认无—topo参数的情况下就是这样。其内部实现就是调用了single,2对应的函数。
- single,n：设置一个交换机和n个主机相连的拓扑。
- linear,n：创建n个交换机，每个交换机只连接一个主机，并且所有交换机成线型排列。
- tree,depth=n,fanout=m：创建深度为n，每层树枝为m的树型拓扑。因此形成的拓扑的交换机个数为（mn-1）/（m-1），主机个数为mn。
- —custom：在上述已有拓扑的基础上，Mininet支持自定义的拓扑，使用一个简单的Python API即可。—custom需和—topo一起使用，如mn —custom file.py —topo mytopo。

#### Mininet启动参数

- `--topo = TOPO`指定拓扑类型，例如`mn --topo=single,n=5`

- **--switch** 定义网络拓扑要使用的交换机，后面可以接的参数有：**ovsk**、**ovsbr**、**ivs**、**lxbr**、**user**，前面三种均为OVS型交换机，后面两种分别为内核型(linux bridge)和用户型(user)交换机。

- --controller 一般我们不用mininet自带的控制器，而是自己制定一个远程控制器，代码如下：如果--ip和--port省略的话，则默认使用本地ip地址，端口默认使用6653或6633端口号
  
  ```
   mn --controller = remote,
        --ip = [控制器的IP地址]
        --port = [控制器的端口号]
        

#### Mininet中的常用命令

| 命令     | 作用                           |
| -------- | ------------------------------ |
| help     |                                |
| xterm    | 指定节点打开xterm              |
| iperf    | 进行iperf tcp测试              |
| iperfudp | 进行udp测试                    |
| net      | 显示网络连接情况               |
| pingpair | 前两个主机互相ping（不能指定） |
| link     | 禁用或启用两个节点之间的链路   |
| pingall  |                                |
| py       | 执行python表达式               |
| sh       | 运行shell命令                  |
| quit     | 退出                           |
| nodes    | 列出所有节点信息               |
| links    | 查看链路健壮性信息             |

使用

`mn --custom [file.py] --topo mytopo`创建自定义拓扑、

执行`mn -c`清除释放Mininet构造的交换机和主机。

可以通过miniedit可视化界面创建自定义拓扑并保存为`python`脚本

## 0x3 调用API拓展自定义拓扑

topo包含的函数有

- addHost(“host name”): 添加主机
- addSwitch(“sw name”): 添加交换机
- addLink(node,node): 添加链路
- attach(port):添加端口

打开mininet后

- `py net.addHost('h3')`添加主机
- `py net.addLink(s3,net.get('h3'))`在s3和h3之间添加一条链路
- `py s3.attach('s3-eth3')`添加接口
- `py net.get(‘h3’).cmd(‘ifconfig h3-eth0 10.3’)
  `给h3配置ip地址

## 0x4 Mininet可视化构建网络拓扑

使用Miniedit。

## 0x5  Mininet流表应用实战1——手动添加流表

执行`dpctl dump-flows`查看交换机流表信息。

若拓扑里没有sdn控制器，也没有流表，主机之间ping不通。

`dpctl add-flow in_port=1,actions=output:2`

`dpctl del-flows`删除所有流表

## 0x7多数据中心网络拓扑流量宽带实验