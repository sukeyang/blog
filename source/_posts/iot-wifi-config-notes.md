---
title: 智能硬件配网/组网方案整理笔记
date: 2016-09-19 01:48:39
tags: [物联网, 智能硬件]
categories: 技术相关
---

今年的工作主要是和智能硬件相关，上半年在配网这方面花费了不少功夫，与硬件同事共同努力，兼容性稳定性都有较大提升。在此做一个大致的整理，叙述我所了解的各种配网的大致原理、流程。


名词解释：配网指的是，让智能硬件接入网络的过程。可以是互联网可以是局域网，应该没有太严格的定义吧。比较常见的有Wifi、蓝牙、ZigBee这些。ZigBee和手机端没有什么关联，蓝牙设备的搜索发现有标准协议，有线网关就不用说了插线就可以了，让Wifi通讯的智能硬件联网是一个有点难度的问题。

## WiFi配网

以小米空气净化器为例，按照APP的添加设备流程，输入家里路由器的名称、密码，点击下一步，净化器就连上路由器了。记得第一次配网成功的时候，心里是百思不得其解，作了好多猜测：
	
- 内置蓝牙模块？当时并没有开蓝牙
- 声波传输了ssid和密码？好像也并没有开声音
- 手机连上了设备的AP热点，然后发送完ssid、密码再切回路由器？iOS并不提供切换wifi的api（其实这就是AP配网了）

很久以后才搞明白，原来这就是传说中的黑科技。。一键配网

### 一键配网

SmartConfig、EasyConnect、Esptouch、Airkiss……whatever, 各个厂商有自己不同的实现和名称，大致原理是差不多的

首先，通过硬件复位，让设备处于待配网状态。此时设备处于混杂模式，会捕获范围内所有网络数据包，进行分析。

然后在APP中输入路由器密码，点击开始，发送配网信息（SSID和密码）。设备从数据包中过滤出有用信息并解码，得到正确的SSID和密码，连接至路由器。

>从802.11帧格式分析中获知,无线信号监听方的角度来说,不管无线信道有没有加密,DA、SA、LENGTH 、LLC、SNAP、FCS字段总是暴露的，因此信号监听方可以从这6个字段获取有效信息.从发送方讲,由于操作系统的限制,如果采用广播只剩下LENGTH发送方可通过改变其所需要发送数据包的长度进行控制.所以只要指定出一套利用长度编码的通讯协议,就可利用数据包的Lenght字段进行数据传递;

具体原理可见松哥博客：[wifi一键配网smartconfige 原理及应用](http://blog.csdn.net/flyingcys/article/details/49283273)

**优点**：用户操作方便，配网速度快，效率高。
**缺点**：部分手机与低端路由器不兼容，广播包在发送或是接收的过程中被丢弃了。

时序图：

![](/imgs/smartconfig.svg)

### AP配网

AP配网操作起来复杂的多，对用户很不友好，一般作为备选方案使用。

首先通过硬件复位，设备会开启AP热点，等待手机端连接。
然后手机连接wifi到设备热点，在APP中输入路由器名称、密码，点击开始。数据通过局域网发送到设备端。设备收到数据后主动断开热点，连接路由器。这时手机wifi被断开，也会自动去连接路由器。等手机和设备都连上以后，配网就成功了。

**优点**：兼容性好，基本所有手机、路由器都能支持
**缺点**：操作复杂，上了年纪的人估计真的不会用。。

时序图：

![](/imgs/apconfig.svg)


## Mesh组网方案

Wifi Mesh网络的配网也可以使用SmartConfig。最终每个设备都运行在sta+ap模式，类似路由器中继/桥接的感觉。多个设备都配置上路由器之后，他们会互相进行协商，只留一个设备作为根节点连接到路由器，另外的设备作为子节点，形成一个树状图的结构。根节点会对数据包进行转发。BLE也可以用于Mesh组网。

![](/imgs/mesh.jpg)

这样的网络拓扑结构适合覆盖大面积的开放区域，因为二级三级节点不用直连路由器。同时网络流量可以做负载均衡等等，不会给根节点带来太大压力。传输功率、功耗、信号干扰方面也会有提升。

缺点吧。。感觉出问题的时候调试会很困难，链路太长了😂。。每个环节都有可能会出问题，不好定位。响应上可能也会略慢些。

[Wikipedia - Wireless mesh network
](https://en.wikipedia.org/wiki/Wireless_mesh_network)

----
还在和硬件同事联调中，到时候再来补充:-)