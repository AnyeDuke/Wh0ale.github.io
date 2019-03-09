---
layout:     post
title:      wireshark技巧
date:       2019-3-9
author:     Wh0ale
header-img: img/1.jpg
catalog: true
tags:
    - wireshark
---







这篇文章献给即将为两会通宵几个晚上的自己

# 0x01 基础操作

## **1.1数据过滤**

ip过滤

```
ip.addr==x.x.x.x，ip.src== x.x.x.x,ip.dst== x.x.x.x
```

协议过滤

HTTP、HTTPS、SMTP、ARP等

端口过滤

```
tcp.port==21、udp.port==53
```

组合过滤

```
ip.addr==x.x.x.x && tcp.port==21、tcp.port==21 or udp.port==53
```

## **1.2数据统计**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxc32qmhj20qo0j4wgp.jpg)



**IP统计：**在菜单中选择Statistics，然后选择Conversation，就可以统计出所在数据包中所有通信IP地址，包括IPV4和IPV6。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxo29ub1j20qq0ghdh7.jpg)



**端口统计：**同IP统计，点击TCP可以看到所有TCP会话的IP、端口包括数据包数等信息，且可以根据需求排序、过滤数据。UDP同理。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxq8zj5gj20qq0ghq4a.jpg)



## 1.3搜索功能

WireShark具备强大的搜索功能，在分析中可快速识别出攻击指纹。Ctrl+F弹出搜索对话框。

Display Filter：显示过滤器，用于查找指定协议所对应的帧。

Hex Value：搜索数据中十六进制字符位置。

String：字符串搜索。Packet list：搜索关键字匹配的Info所在帧的位置。Packet details：搜索关键字匹配的Info所包括数据的位置。Packet bytes：搜索关键字匹配的内容位置。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxyx3141j20qn0jedi4.jpg)



## 1.4Follow TCP Stream

对于TCP协议，可提取一次会话的TCP流进行分析。点击某帧TCP数据，右键选择Follow TCP Stream，就可以看到本次会话的文本信息，还具备搜索、另存等功能。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy12s0b5j20ub0gztdo.jpg)



## 1.5HTTP头部分析

对于HTTP协议，WireShark可以提取其URL地址信息。

在菜单中选择Statistics，选择HTTP，然后选择Packet Counter（可以过滤IP）,就可以统计出HTTP会话中请求、应答包数量。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy2itwbtj20ma0dngm8.jpg)

在菜单中选择Statistics，选择HTTP，然后选择Requests（可以过滤IP）,就可以统计出HTTP会话中Request的域名，包括子域名。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy52rm4sj20mm0gvdhx.jpg)

在菜单中选择Statistics，选择HTTP，然后选择Load Distribution（可以过滤IP）,就可以统计出HTTP会话的IP、域名分布情况，包括返回值。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy5ferq0j20mp0cagn6.jpg)



# 0x02 WireShark分析攻击行为步骤

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy6rdouwj20ka0jtmyn.jpg)



## 2.1肉鸡邮件服务器

肉鸡也称傀儡机，是指可以被黑客远程控制的机器。一旦成为肉鸡，就可以被攻击者随意利用，如：窃取资料、再次发起攻击、破坏等等。下面将利用WireShark一起学习一种肉鸡的用途：广告垃圾邮件发送站。

### 2.1.1发现问题

在对某企业服务器群进行安全检测时发现客户一台服务器（10.190.214.130）存在异常，从其通信行为来看应该为一台空闲服务器。 经过一段时间的抓包采集，对数据进行协议统计发现，基本均为SMTP协议。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyaz6arvj20ys0g4jxb.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wybmqumkj20q10dm0u6.jpg)

SMTP协议为邮件为邮件传输协议。正常情况下出现此协议有两种情况：

```
1、用户发送邮件产生。
2、邮件服务器正常通信产生。
```

该IP地址属于服务器，所以肯定非个人用户利用PC机发送邮件。

那这是一台邮件服务器？如果是，为什么仅有SMTP协议，POP3、HTTP、IMAP等等呢？

带着疑问我们统计了一下数据的IP、端口等信息：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wych4xg4j20sx0ibdjv.jpg)

统计信息表明：所有通信均是与61.158.163.126（河南三门峡）产生的SMTP协议，且服务器（10.190.214.130）开放了TCP25端口，它的的确确是一台邮件服务器。

到这，很多安全分析人员或监控分析软件就止步了。原因是IP合理、逻辑也合理、SMTP协议很少有攻击行为，以为是一次正常的邮件通信行为。那么很可惜，你将错过一次不大不小的安全威胁事件。

职业的敏感告诉我，它不是一台合理的邮件服务器。这个时候需要用到应用层的分析，看一看它的通信行为。继续看看SMTP登陆过程的数据。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wydh8c4fj20xh06h76l.jpg)

从数据看出，邮箱登陆成功，右键`Follow TCPStream`可以看见完整登陆信息。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wye79553j20js058wet.jpg)



```
334 VXNlcm5hbWU6          // Base64解码为：“Username:”
YWRtaW4=  //用户输入的用户名，Base Base64解码为：“admin”
334 UGFzc3dvcmQ6         //Base64解码为：“Password:”
YWRtaW4=  //用户输入的密码，Base Base64解码为：“admin”
235 Authentication successful.  //认证成功
MAIL FROM:<admin@system.mail>  //邮件发送自……
```

这段数据表明：61.158.163.126通过SMTP协议，使用用户名admin、密码admin，成功登陆邮件服务器10.190.214.30，邮件服务器的域名为@system.mail，且利用admin@system.mail发送邮件。

一看用户名、密码、邮箱，就发现问题了：

> 1、admin账号一般不会通过互联网登陆进行管理。
>
> 2、“二货”管理员才会把admin账号设为密码。
>
> 3、域名@system.mail与客户无任何关系。

很显然，这是一台被控制的邮件服务器—“肉鸡邮件服务器”。

题外插一个刚遇到的伪造邮件例子：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyk6mm7uj20iq09ngmh.jpg)



### 2.1.2行为跟踪

发现问题了，下一步跟踪其行为，这个肉鸡服务器到底是干什么的。查看Follow TCPStream完整信息可发现：这是一封由admin@system.mail群发的邮件，收件人包括： www651419067@126.com、wyq0204@yahoo.com.cn、zhaocl1@163.com等10个人（带QQ的邮箱暂时抹掉，原因见最后），邮件内容不多。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyllls6vj20p90iwdi9.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wylt0k1nj20pb0iw77g.jpg)

为看到完整邮件内容，我们可以点击Save As存为X.eml，用outlook等邮件客户端打开。

![enter image description here](http://www.anquan.us/static/drops/full/d484829fa9f06826050549e552d6d0bafb9e5409.jpg)







































































