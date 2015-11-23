title: Scapy notenote
date: 2022-12
categories: [Scapy]
tags: [Scapy,笔记]
---
文章摘要
<!--more-->
```
#/usr/bin/python
from scapy.all import *

ip = IP(src = "10.10.10.10", dst = "10.10.10.11")
SYN = TCP(sport = 1030, dport = 80, flags = "S", seq = 10)
SYNACK = sr1(ip/SYN)
my_ack = SYNACK.seq + 1
ACK = TCP(sport = 1030, dport = 80, flags = "A", seq =11, ack = my_ack)
send(ip/ACK)
palyload = "SEND TCP"
PUSH = TCP(sport = 1030, dport = 80, flags = "PA", seq=11, ack = my_ack)

```
