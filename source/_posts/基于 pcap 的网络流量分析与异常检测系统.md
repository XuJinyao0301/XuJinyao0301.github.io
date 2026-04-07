---
title: 基于 pcap 的网络流量分析与异常检测系统
categories: 
- Programming
tags: 
- Network
- Python
date: 2026-03-27 14:23:23
description:
- 这篇文章记录了自己做一个小项目的过程
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

### ICMP

ICMP：网际控制报文协议

```
Source: 10.15.37.10
Destination: 36.152.44.93
Protocol: ICMP
Length: 74
Info: Echo (ping) request id=0x0001, seq=27/6912, ttl=128
```

这是电脑发出去的一个ICMP请求包，也就是 ping request
它的意思是：“你能收到我吗？”

```
Source: 36.152.44.93
Destination: 10.15.37.10
Protocol: ICMP
Length: 74
Info: Echo (ping) reply id=0x0001, seq=27/6912, ttl=52
```

这是对方主机收到了电脑发出的 ping request，然后回了一个 ping reply
这说明：网络是通的，对方能回你

观察上面两个包，可以发现：request 和 reply 的 Source 和 Destination 正好反过来了
这就是一组最基础的“请求与响应”

再看到 ICMP reply包里，
```
ttl=52
```

ttl 表示这个包还能经过多少跳，ttl 是数据包里的一个字段，会随着包在网络中转发而变化
所以请求和恢复里的 ttl 可能不一样

```
request：ttl=128
reply：ttl=52
```
这说明请求包和回复包的 ttl 可以不一样

## Day 4

### 读取 pcap 文件

```python
from scapy.all import rdpcap

packets = rdpcap("data/test1.pcapng")

print("数据包总数：", len(packets))
```

运行结果：
```python
数据包总数： 87032
```

这一步说明：
Python 已经可以正常读取 pcapng 文件
后面的分析都可以建立在这个基础上继续做

### 打印概要信息

```python
from scapy.all import rdpcap

packets = rdpcap("data/test1.pcapng")

print("第1个包：")
print(packets[0].summary())
```

运行结果：
```python
第1个包：
Ether / IP / TCP 111.62.113.50:https > 10.15.39.136:13699 A
```
这说明第 1 个包是一个 TCP 包
并且它的标志位是 A
这里的 A 表示：
这是一个 ACK 包
也就是确认包
可以先粗略理解成：
“我收到了，你继续发”


接着，我们把这个包里的几个关键字段拆开打印出来

```python
from scapy.all import rdpcap, IP, TCP

packets = rdpcap("data/test1.pcapng")
pkt = packets[0]

print("源IP：", pkt[IP].src)
print("目的IP：", pkt[IP].dst)
print("源端口：", pkt[TCP].sport)
print("目的端口：", pkt[TCP].dport)
print("TTL：", pkt[IP].ttl)
print("IP层总长度：", pkt[IP].len)
print("TCP标志位：", pkt[TCP].flags)
```

运行结果：
```python
源IP： 111.62.113.50
目的IP： 10.15.39.136
源端口： 443
目的端口： 13699
TTL： 54
IP层总长度： 52
TCP标志位： A
```

### 判断这个包属于什么协议
```python
from scapy.all import rdpcap, IP, TCP, ICMP

packets = rdpcap("data/test1.pcapng")
pkt = packets[0]

print("是不是 IP 包：", pkt.haslayer(IP))
print("是不是 TCP 包：", pkt.haslayer(TCP))
print("是不是 ICMP 包：", pkt.haslayer(ICMP))
```

运行结果；
```
是不是 IP 包： True
是不是 TCP 包： True
是不是 ICMP 包： 0
```
这说明；
第一个包是 IP 包
也是 TCP 包
但不是 ICMP 包

所以它属于 TCP 通信
不是 ping 那种 ICMP 通信

### 对整份抓包做基础统计

先统计：
TCP 包有多少个
ICMP 包有多少个
携带实际数据的 TCP 包有多少个
纯 ACK 包有多少个

```python
from scapy.all import rdpcap, TCP, ICMP, Raw

packets = rdpcap("data/test1.pcapng")

tcp_count = 0
icmp_count = 0
tcp_with_data_count = 0
pure_ack_count = 0

for pkt in packets:
    if pkt.haslayer(TCP):
        tcp_count += 1

        if pkt.haslayer(Raw):
            tcp_with_data_count += 1

        if pkt[TCP].flags == "A" and not pkt.haslayer(Raw):
            pure_ack_count += 1

    if pkt.haslayer(ICMP):
        icmp_count += 1

print("数据包总数：", len(packets))
print("TCP包数量：", tcp_count)
print("ICMP包数量：", icmp_count)
print("携带实际数据的TCP包数量：", tcp_with_data_count)
print("纯ACK包数量：", pure_ack_count)
```

运行结果：
```python
数据包总数： 87032
TCP包数量： 86675
ICMP包数量： 0
携带实际数据的TCP包数量： 46605
纯ACK包数量： 39651
```
这个结果说明：
这份抓包几乎全是 TCP
没有 ICMP
同时也说明：
很多 TCP 包是在真正传数据
但也有很多 TCP 包只是做确认

### 统计 TCP 标志位

```python
from scapy.all import rdpcap, TCP

packets = rdpcap("data/test1.pcapng")

ack_count = 0
syn_count = 0
syn_ack_count = 0
fin_count = 0

for pkt in packets:
    if pkt.haslayer(TCP):
        flags = str(pkt[TCP].flags)

        if flags == "A":
            ack_count += 1
        elif flags == "S":
            syn_count += 1
        elif flags == "SA":
            syn_ack_count += 1

        if "F" in flags:
            fin_count += 1

print("ACK包数量：", ack_count)
print("SYN包数量：", syn_count)
print("SYN+ACK包数量：", syn_ack_count)
print("FIN包数量：", fin_count)
```

运行结果：
```python
ACK包数量： 70332
SYN包数量： 142
SYN+ACK包数量： 96
FIN包数量： 147
```

这说明这份抓包里
不仅有大量已经建立好的通信
也包含连接建立和关闭的过程

### 在 SYN 包里找异常

先把前十个 SYN 包打印出来

```python
from scapy.all import rdpcap, IP, TCP

packets = rdpcap("data/test1.pcapng")

count = 0

for pkt in packets:
    if pkt.haslayer(IP) and pkt.haslayer(TCP):
        if str(pkt[TCP].flags) == "S":
            count += 1
            print(
                count,
                pkt[IP].src,
                ":",
                pkt[TCP].sport,
                "->",
                pkt[IP].dst,
                ":",
                pkt[TCP].dport
            )

            if count == 10:
                break
```
运行结果里有一个很明显的现象；
```python
1 10.15.39.136 : 12180 -> 216.239.38.223 : 443
2 10.15.39.136 : 12183 -> 216.239.38.223 : 443
3 10.15.39.136 : 12180 -> 216.239.38.223 : 443
4 10.15.39.136 : 12183 -> 216.239.38.223 : 443
5 10.15.39.136 : 12180 -> 216.239.38.223 : 443
6 10.15.39.136 : 12183 -> 216.239.38.223 : 443
7 10.15.39.136 : 12180 -> 216.239.38.223 : 443
8 10.15.39.136 : 12183 -> 216.239.38.223 : 443
9 10.15.39.136 : 2044 -> 216.239.32.223 : 443
10 10.15.39.136 : 2047 -> 216.239.32.223 : 443
```

可以看到：
同一个连接请求反复出现了很多次

这里就要引入一个重要概念：

### 四元组

四元组指的是：
源 IP
目的 IP
源端口
目的端口

例如：
```python
10.15.39.136:12180 -> 216.239.38.223:443
```

这就是一个四元组
它可以用来标识一条TCP连接

因为：
IP 表示是哪两台主机在通信
端口表示主机上的具体应用或具体会话

### SYN 重传

SYN 重传指的是：
客户端发出了 SYN
但是在预期时间内没有收到对应的 SYN+ACK
所以又重新发送了一次 SYN
也就是说：
SYN 重传不一定等于最终连接失败
但它一定说明：
连接建立过程出现了异常，客户端正在重试
如果一直收不到 SYN+ACK
那就更像是真正的建连失败

这里我们让程序自动去分析整份抓包
思路是；
1.	找出所有 SYN
2.	按四元组统计它们出现的次数
3.	找出重复出现的 SYN 四元组
4.	再检查这些四元组有没有对应方向的 SYN+ACK

```python
from scapy.all import rdpcap, IP, TCP

packets = rdpcap("data/test1.pcapng")

syn_dict = {}
sa_dict = {}

for pkt in packets:
    if pkt.haslayer(IP) and pkt.haslayer(TCP):
        src_ip = pkt[IP].src
        dst_ip = pkt[IP].dst
        sport = pkt[TCP].sport
        dport = pkt[TCP].dport
        flags = str(pkt[TCP].flags)

        if flags == "S":
            key = (src_ip, dst_ip, sport, dport)
            syn_dict[key] = syn_dict.get(key, 0) + 1

        elif flags == "SA":
            key = (src_ip, dst_ip, sport, dport)
            sa_dict[key] = sa_dict.get(key, 0) + 1

problem_count = 0

print("重复 SYN 且没有对应 SYN+ACK 的四元组：")

for key, syn_count in syn_dict.items():
    if syn_count > 1:
        reverse_key = (key[1], key[0], key[3], key[2])
        sa_count = sa_dict.get(reverse_key, 0)

        if sa_count == 0:
            problem_count += 1
            print(key, "SYN =", syn_count, ", SYN+ACK =", sa_count)

print("这类四元组总数：", problem_count)
```

运行结果；
```python
重复 SYN 且没有对应 SYN+ACK 的四元组：
('10.15.39.136', '216.239.38.223', 12180, 443) SYN = 4 , SYN+ACK = 0
('10.15.39.136', '216.239.38.223', 12183, 443) SYN = 4 , SYN+ACK = 0
('10.15.39.136', '216.239.32.223', 2044, 443) SYN = 5 , SYN+ACK = 0
('10.15.39.136', '216.239.32.223', 2047, 443) SYN = 5 , SYN+ACK = 0
('10.15.39.136', '216.239.34.223', 13406, 443) SYN = 5 , SYN+ACK = 0
('10.15.39.136', '216.239.34.223', 13409, 443) SYN = 5 , SYN+ACK = 0
('10.15.39.136', '216.239.36.223', 12873, 443) SYN = 5 , SYN+ACK = 0
('10.15.39.136', '216.239.36.223', 12876, 443) SYN = 5 , SYN+ACK = 0
这类四元组总数： 8
```

这说明；
在这个抓包里一共找到了8组重复 SYN 且没有收到对应 SYN + ACK 的异常连接
这些连接都属于比较典型的：
疑似 SYN 重传/建连失败

### 导出分析结果

我们可以把这些异常连接保存成 CSV 文件

```python
from scapy.all import rdpcap, IP, TCP
import pandas as pd

packets = rdpcap("data/test1.pcapng")

syn_dict = {}
sa_dict = {}

for pkt in packets:
    if pkt.haslayer(IP) and pkt.haslayer(TCP):
        src_ip = pkt[IP].src
        dst_ip = pkt[IP].dst
        sport = pkt[TCP].sport
        dport = pkt[TCP].dport
        flags = str(pkt[TCP].flags)

        if flags == "S":
            key = (src_ip, dst_ip, sport, dport)
            syn_dict[key] = syn_dict.get(key, 0) + 1

        elif flags == "SA":
            key = (src_ip, dst_ip, sport, dport)
            sa_dict[key] = sa_dict.get(key, 0) + 1

results = []

for key, syn_count in syn_dict.items():
    if syn_count > 1:
        reverse_key = (key[1], key[0], key[3], key[2])
        sa_count = sa_dict.get(reverse_key, 0)

        if sa_count == 0:
            results.append({
                "src_ip": key[0],
                "dst_ip": key[1],
                "src_port": key[2],
                "dst_port": key[3],
                "syn_count": syn_count,
                "syn_ack_count": sa_count
            })

df = pd.DataFrame(results)
df.to_csv("output/syn_no_response.csv", index=False, encoding="utf-8-sig")

print("已保存到 output/syn_no_response.csv")
print(df)
```

运行结果：
```python
已保存到 output/syn_no_response.csv
         src_ip          dst_ip  src_port  dst_port  syn_count  syn_ack_count
0  10.15.39.136  216.239.38.223     12180       443          4              0
1  10.15.39.136  216.239.38.223     12183       443          4              0
2  10.15.39.136  216.239.32.223      2044       443          5              0
3  10.15.39.136  216.239.32.223      2047       443          5              0
4  10.15.39.136  216.239.34.223     13406       443          5              0
5  10.15.39.136  216.239.34.223     13409       443          5              0
6  10.15.39.136  216.239.36.223     12873       443          5              0
7  10.15.39.136  216.239.36.223     12876       443          5              0
```

这样我们在后面写项目报告的时候就可以直接引用这个结果文件，不用再重新统计一遍