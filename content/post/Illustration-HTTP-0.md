---
title: "《图解HTTP》读书笔记（零）"
date: 2017-12-15T18:55:02+08:00
toc: true
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["HTTP", "NOTES"]
categories: ["读书笔记"]
CreativeCommons: true
draft: false
---

## 前言

本文是[《图解HTTP》](https://book.douban.com/subject/25863515/)读书笔记的开篇，主要记录在学习《图解HTTP》一书中的思考和笔记。本文不会面面俱到，将集中在我认为值得记录的地方。话不多说，下面开始。

## Web及HTTP概览

HTTP协议及其他的一系列协议，如TCP/IP协议，都从一个问题出发，相隔遥远的我们如何交流知识？众所周知的WWW技术最初由CERN的Tim Berners Lee博士提出。即：

> 借助多文档之间相互关联形成的超文本（HyperText），连成可相互参阅的WWW（World Wide Web，万维网）

Web借助HTTP（HyperText Transfer Protocol）协议作为规范，完成从客户端（Client）到服务端（Server）的信息传递，故说Web是建立在HTTP协议之上的。

### 网络基础TCP/IP

在了解什么是HTTP之前，我们需要先了解什么是TCP/IP协议，从集合角度来说，HTTP是TCP/IP的一个子集，下面具体来看看TCP/IP协议。

#### TCP/IP协议族

计算机及网络设备的相互通信需要依据于一系列的事先定义好的协议（Protocol），这样的协议有很多，如ICMP，HTTP，FTP，SNMP，PPPoE，TCP，IP等，类似把互联网相关连的协议统称为TCP/IP协议，因为TCP/IP协议是其中应用最广泛的两种。

![](http://www.ituring.com.cn/figures/2014/PIC%20HTTP/05.d01z.005.png)

#### TCP/IP的分层

TCP/IP按层次分为：应用层，传输层，网络层和数据链路层。主要作用如下：

##### 应用层

应用层决定了向用户提供应用服务时通信的活动，例如，FTP和DNS服务，HTTP协议也位于其中。

##### 传输层

传输层对上层应用层，提供处于网络连接中两台计算机之间的数据传输，例如，TCP和UDP协议。

##### 网络层

网络层用来处理在网络上流动的数据包（网络传输的最小数据单位）。

##### 链路层

用来处理连接网络的硬件部分。包括控制操作系统，硬件的设备驱动，网卡及光纤等物理设备。主要处理硬件上的范畴。

#### TCP/IP通信传输流

一言以蔽之，利用TCP/IP协议族通信时，数据流是按照层次一级一级走的，客户端从应用层一直到链路层，服务端反之。并在每经过一层时，在数据报文上封装上该层的首部信息（客户端），或消去首部信息（服务端）。

![](http://www.ituring.com.cn/download/021jVNia3uoL)

### HTTP相关协议：IP、TCP和DNS

#### 负责传输的IP协议（网络层）

IP协议（Internet Protocol）和IP地址虽然相似但是并不相同，IP协议负责将数据包发送给想要通信的对象。怎么知道要发送给谁呢？需要IP地址和MAC地址（Media Access Control Address)的帮忙。IP地址指明节点被分配到的地址。MAC地址指网卡所属固件的地址，暂时可以理解为设备本身的硬件指纹。一般来说，IP地址可变换，MAC地址不可变换，但也可人为改变。

##### 使用ARP协议凭借MAC地址通信

前面说道，IP间通信依赖于MAC地址，而设备之间处于同一局域网（LAN）的情况很少。大多数时候需要多台设备和路由的跳转。这个时候，我们就需要根据ARP协议（Address Resolution Protocol），根据通信方的IP地址，反查对应的MAC地址，从而达到设备的中转。

##### 路由选择

由前面所述可知，数据的通信需要经过多级中转，如何知道在岔路口该怎么选择？需要借助路由器（Router）来判断这一部分，那一部分的数据该往哪传递。

![](http://www.ituring.com.cn/figures/2014/PIC%20HTTP/05.d01z.008.png)

#### 确保可靠性的TCP协议

TCP位于传输层，提供可靠的字节流服务。什么是字节流（Byte Stream）呢？比如我们想要传输1G大的数据，不可能一次性发送这么多，需要切割成以报文段（segment）为单位的数据包。这么多块数据包，如何保证完整的传输呢？

##### 首先确保能到达目的地

TCP协议采用了三次握手策略（three-way handshaking）来保证这一点。发送数据包后，三次握手过程中使用TCP标志（flag）----SYN（synchronize）和ACK（acknowledgement)。举个栗子：

> 小明：嗨，服务器能听到么？（先发送一个带SYN标志的数据包）
>
> 服务器：嗨，我听到了。（回传一个带SYN/ACK标志的数据包表示收到信息）
>
> 小明：OK，我知道你能听到我说话了。（再回传一个带ACK标志的数据包，代表握手结束）

这样，就保证这条通信线路是可靠的，可以开始发送数据了。如果三次握手不能完成，TCP协议会再次按相同顺序发送相同的数据包。除了三次握手策略，TCP协议还有其他保证通信可靠性的方法。

#### 负责域名解析的DNS服务

DNS（Domain Name System）和HTTP协议一样，同样位于应用层。提供域名和IP地址之间的解析工作。IP地址（仅就IPv4来说），格式形如xxx.xxx.xxx.xxx，xxx为0~255的数字。难以记忆，域名（eg:www.baidu.com)相对来说方便记忆。故我们使用了DNS协议，来通过域名解析出真正的IP地址，或由IP地址反查域名。

#### 各协议和HTTP协议的关系

当我们在浏览器地址栏输入网址时，发生了什么呢？如下所示：

![](http://www.ituring.com.cn/figures/2014/PIC%20HTTP/05.d01z.011.png)

#### URL和URI

关于URL和URI的区别经常让人头大，因为这两个概念太相像了。

URI (Uniform Resource Identifier)，统一资源标识符。简而言之，是标识某一互联网资源身份的一串字符。它可以描述这个资源是什么，获取它的方法是什么等等。

URL（Uniform Resource Locator），统一资源定位符。URL是URI的一个子集，也就是说，所有的URL都是URI，而不是所有的URI都是URL，也可能是URN（Uniform Resource Name），统一资源名称，在此不深入了。URL标识如何（How-to）获取某一互联网上的资源，故说所有的URL都是URI。
