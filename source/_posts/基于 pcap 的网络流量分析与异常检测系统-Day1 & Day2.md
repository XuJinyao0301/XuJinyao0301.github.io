---
title: 基于 pcap 的网络流量分析与异常检测系统
categories: 
- Programming
tags: 
- Network
- Python
date: 2026-03-27 14:23:23
description:
- 这篇文章记录了自己做小项目的过程
cover:
---
## Day 1

第一天主要是安装一些必要的工具
比如：python pip Wireshark

然后利用Wireshark抓包，并且把找到的包保存下来

## Day 2

第二天的核心任务是：
学会在 Wireshark 里看懂最基础的包信息。

![](/images/Wireshark01.png)

以第一行为例：
```
Source: 111.62.113.50 
Destination: 10.15.39.136 
Protocol: TCP 
Length: 66
```
Source：源地址
Destination：目的地址
Protocol：协议
Length：长度

我们可以先把它粗略理解成：

有一台地址是 111.62.113.50 的机器，给 10.15.39.136 发了一个 TCP 包，这个包大小是 66 字节。


再看第二行，它使用的协议是 TLSv1.2 
TLS 指的是 加密传输层协议
它的包长度是2878，要比上面的那个TCP的包长度大很多

一般来说
小包通常更像是在做连接控制或确认，而不是在传很多真正的网页内容
大包则是在传真正的数据

### 标志位

在 TCP 中，标志位用于指示 TCP 连接的特定状态或提供一些额外的有用信息

SYN：发起连接
ACK：确认
FIN：结束连接
RST：重置连接

我们来看几个例子

```1
Source: 10.15.39.136
Destination: 223.6.6.6
Protocol: TCP
Length: 66
Info: 3376 -> 443 [SYN] Seq=0 Win=65535 Len=0 MSS=1460 WS=256 SACK_PERM
```
[SYN]
表示：我要开始建立连接了

我们现在可以把这个包粗略理解成一句话：
我的电脑想和对方服务器的 443 端口建立一个 TCP 连接。

```2
Source: 10.15.39.136
Destination: 223.6.6.6
Protocol: TCP
Length: 54
Info: 3376 -> 443 [ACK] Seq=1 Ack=1 Win=65280 Len=0
```
[SYN,ACK]  
表示“我收到了，也同意建立连接”

```3
Source: 223.6.6.6
Destination: 10.15.39.136
Protocol: TCP
Length: 66
Info: 443 -> 3376 [SYN, ACK] Seq=0 Ack=1 Win=42340 Len=0 MSS=1440 SACK_PERM WS=2048
```
443 -> 3376
表示这个包是从 443 端口 发到 3376 端口
其中 443 很常见，通常和 HTTPS 有关

[ACK]
表示这是一个 确认包
你现在可以粗略理解成：
“我收到了，你继续发。”

Len=0
表示这个包没有携带真正的数据内容
所以这个包更像是在“确认通信状态”，不是在传网页正文


现在这三步就完整了：

3376 -> 443 [SYN]
电脑说：我想连接你
443 -> 3376 [SYN, ACK]
服务器说：我收到了，也同意连接
3376 -> 443 [ACK]
电脑说：好，收到

这三步合起来，就是：
TCP 三次握手完成，连接建立成功。

## Day 3

