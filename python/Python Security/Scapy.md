# Scapy

## 1. 介绍

​        目前很多优秀的网络扫描和攻击工具都使用了这个模块。也可以在自己的程序中使用这个模块来实现对网络数据包的发送、监听和解析。相比起Nmap，这个模块更底层。你可以更直观地了解网络中的各种扫描和攻击行为，例如检测某个端口是否开放，你只须给Nmap提供一个端口号，而Nmap就会反馈一个开放或者关闭的结果，但是并不知道Nmap是怎么做到的。如果你想深入研究网络中的各种问题，Scapy无疑是一个更好的选择，你可以利用它来产生各种类型的数据包并发送出去，Scapy也只会把收到的数据包展示给你，并不会告诉你这意味着什么，一切都将由你来判断。

## 2. 基本用法

**安装：** Kali默认安装，既可以在python中调用它，也可以直接使用它。

```python
pip install scapy                #安装scapy
from scapy.all import *          #在python中调用scapy模块
```

###  (1).基本用法

> 在scapy中每一个协议就是一个类，需要实例化一个类后，才可以创建一个该协议的数据包

**例：** 创建一个发往“192.168.1.101”的IP类型数据包

```python
>>> ip=IP(dst="192.168.1.101")    
>>> ip
<IP  dst=192.168.1.101 |>

#dst既可以是一个IP也可以是一个范围
>>> ip=IP(dst="192.168.1.0/24")
>>> ip
<IP  dst=Net("192.168.1.0/24") |>
>>> [p for p in ip]                  #查看每一个数据包
[<IP  dst=192.168.1.0 |>, <IP  dst=192.168.1.1 |>, <IP  dst=192.168.1.2 |>, <IP  dst=192.168.1.3 |>, <IP  dst=192.168.1.4 |>, <IP  dst=192.168.1.5 |>, ...]
```

​          Scapy采用分层形式构造数据包，通常最下面的协议是Ether，然后是IP，再之后是TCP或者UDP。

   ```python
#构造一个ARP请求和应答包，使用Ether();下面产生了一个广播包
>>> Ether(dst="ff:ff:ff:ff:ff:ff")    
<Ether  dst=ff:ff:ff:ff:ff:ff |>
   ```

​          Scapy中的分层通过符号“/”实现，一个数据包是由多层协议组合而成的。这些协议之间可以使用“/”分开，按照协议由底而上的顺序从左向右排列。

```python
#构造一个TCp数据包
>>> Ether()/IP()/TCP()
<Ether  type=IPv4 |<IP  frag=0 proto=tcp |<TCP  |>>>

#构造一个HTTP数据包
>>> IP()/TCP()/"GET / HTTP/1.0\r\n\r\n"
<IP  frag=0 proto=tcp |<TCP  |<Raw  load='GET / HTTP/1.0\r\n\r\n' |>>>
```

​            Scapy中使用频率最高的类要数Ether、IP、TCP和UDP了。使用ls()函数可以查看一个类拥有的属性。

```python
>>> ls(IP())
version    : BitField  (4 bits)                  = 4               ('4')
ihl        : BitField  (4 bits)                  = None            ('None')
tos        : XByteField                          = 0               ('0')
len        : ShortField                          = None            ('None')
id         : ShortField                          = 1               ('1')
flags      : FlagsField                          = <Flag 0 ()>     ('<Flag 0 ()>')
frag       : BitField  (13 bits)                 = 0               ('0')
ttl        : ByteField                           = 64              ('64')
proto      : ByteEnumField                       = 0               ('0')
chksum     : XShortField                         = None            ('None')
src        : SourceIPField                       = '127.0.0.1'     ('None')
dst        : DestIPField                         = '127.0.0.1'     ('None')
options    : PacketListField                     = []              ('[]')
```

**使用实例：** 

```python
>>> IP(src="192.168.1.1",dst="192.168.1.2",ttl=32)
<IP  ttl=32 src=192.168.1.1 dst=192.168.1.2 |>
```



### (2). Scapy中的函数

​      1. send()和sendp()

​		除了这些对应着协议的类及其属性外，还需要一些可以完成各种功能的函数。需要注意的是，刚才使用IP()的作用是产生一个IP数据包，但是并没有将其发送出去。现在看看如何将产生的报文发送出去。Scapy中提供了多个用来完成发送数据包的函数，首先看一下send()和sendp()。这两个函数的区别是：send()工作在第三层，而sendp()工作在第二层。简单地说，send()是用来发送IP数据包的，而sendp()是用来发送Ether数据包的。

```python
>>> send(IP(dst="192.168.1.1")/ICMP())     #发送一个ICMP包
.
Sent 1 packets.

>>> sendp(Ether(dst="ff:ff:ff:ff:ff:ff"))   #发送一个Ether包
.
Sent 1 packets.
```

​		如果希望发送一个内容是随机填充的数据包，而且要保证这个数据包的正确性，那么可以使用fuzz()函数。例如可以使用如下命令创建一个发往192.168.1.101的TCP数据包：

```python
>>> IP(dst="192.168.1.1")/fuzz(TCP())
<IP  frag=0 proto=tcp dst=192.168.1.1 |<TCP  |>>
```

​		send()和sendp()特点是只发不收，也就是说，只会将数据包发送出去，但是没有能力处理该数据包的响应包。

​		2. sr(), sr1()和srp()

​		Scapy提供了3个用来发送和接收应答数据包的函数，分别是sr()、sr1()和srp()，其中，sr()和sr1()主要用于第三层，例如IP和ARP等；而srp用于第二层。

```python
>>> sr(IP(dst="110.242.68.66")/ICMP())
Begin emission:
Finished sending 1 packets.
.*
Received 2 packets, got 1 answers, remaining 0 packets
(<Results: TCP:0 UDP:0 ICMP:1 Other:0>, <Unanswered: TCP:0 UDP:0 ICMP:0 Other:0>)
```

​		sr()函数是Scapy的核心，它的返回值是两个列表：第一个列表是收到应答的包和对应的应答；第二个列表是未收到应答的包。所以可以使用两个列表来保存sr()的返回值.

```
>>> ans,unans=sr(IP(dst="110.242.68.66")/ICMP())
Begin emission:
Finished sending 1 packets.
.*
Received 2 packets, got 1 answers, remaining 0 packets
>>> ans.summary()
IP / ICMP 192.168.0.105 > 110.242.68.66 echo-request 0 ==> IP / ICMP 110.242.68.66 > 192.168.0.105 echo-reply 0
```

​		这里面使用ans和unans来保存sr()的返回值，因为发出去的是一个ICMP的请求数据包，而且也收到了一个应答包，所以这个发送的数据包和收到的应答包都被保存到ans列表中，使用ans.summary()可以查看两个数据包的内容，而unans列表为空。

​		sr1()函数跟sr()函数作用基本一样，但是只返回一个应答的包。只需要使用一个列表就可以保存这个函数的返回值。例如使用p来保存sr1(IP(dst="192.168.1.102")/ICMP())的返回值.

```python
>>> p=sr1(IP(dst="110.242.68.66")/ICMP())
Begin emission:
Finished sending 1 packets.
....*
Received 5 packets, got 1 answers, remaining 0 packets
>>> p
<IP  version=4 ihl=5 tos=0x0 len=28 id=1 flags= frag=0 ttl=50 proto=icmp chksum=0x149b src=110.242.68.66 dst=192.168.0.105 |<ICMP  type=echo-reply code=0 chksum=0x0 id=0x0 seq=0x0 |>>
```

​		利用sr1()函数测试目标的某个端口是否开放，采用半开扫描（SYN）

```python
>>> sr1(IP(dst="110.242.68.66")/TCP(dport=80,flags="S"))
Begin emission:
Finished sending 1 packets.
...*
Received 4 packets, got 1 answers, remaining 0 packets
<IP  version=4 ihl=5 tos=0x0 len=44 id=1 flags= frag=0 ttl=50 proto=tcp chksum=0x1486 src=110.242.68.66 dst=192.168.0.105 |<TCP  sport=http dport=ftp_data seq=1474046892 ack=1 dataofs=6 reserved=0 flags=SA window=8192 chksum=0x837f urgptr=0 options=[('MSS', 536)] |>>
```

​		3.sniff()		

​		另外一个十分重要的函数是sniff()，如果你使用过Tcpdump，那么对这个函数不会感到陌生。通过这个函数就可以在程序中捕获经过本机网卡的数据包。

```python
>>> sniff()
<Sniffed: TCP:273 UDP:57 ICMP:0 Other:0>
```

​		使用sniff()开始监听，但是捕获的数据包不会即时显示出来，只有使用Ctrl+C组合键停止监听时，才会显示捕获到的数据包。

这个函数最强大的地方在于可以使用参数filter过滤数据包：

```python
>>> sniff(filter="host 110.242.68.66")   #只捕获与110.242.68.66有关的数据包
<Sniffed: TCP:0 UDP:0 ICMP:8 Other:0>

>>> sniff(filter="icmp")                 #filter来过滤指定协议
<Sniffed: TCP:0 UDP:0 ICMP:8 Other:0>               

#同时满足多个条件可以使用“and”“or”等关系运算符来表达                    
>>> sniff(filter="host 110.242.68.66 and icmp")
<Sniffed: TCP:0 UDP:0 ICMP:8 Other:0> 
                   
```

​		另外两个很重要的参数是iface和count。iface可以用来指定所要进行监听的网卡；而count则用来指定监听到数据包的数量，达到指定的数量就会停止监听。

```python
>>>sniff(iface="em1")

>>>sniff(sount=3)

​```
设计一个综合性的监听器，它会在网卡eth0上监听源地址或者目的地址为192.168.1.102的ICMP数据包，当收到3个这样的数据包之后，就会停止监听。
​```
>>> sniff(filter="host 192.168.1.102 and icmp",count=3,iface="eth0")

```



## 3.Scapy模块的常用简单实例

​		使用Scapy实现一次ACK类型的端口扫描。例如，针对192.168.1.102的21、23、135、443、445这5个端口是否被屏蔽。