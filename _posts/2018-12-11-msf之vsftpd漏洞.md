---
layout:     post
title:      msf之vsftpd漏洞
date:       2018-12-11
author:     Wh0ale
header-img: img/109399.jpg
catalog: true
tags:
    - msf
    - vsftpd
---

## msf之vsftpd漏洞

最近写ELK 规则提到了vsftpd漏洞，经过查询资料发现有一个简单的复现过程于是想写篇文章记录一下

首先介绍下vsftpd

## 0x01vsftpd介绍

vsftpd 是一个 UNIX 类操作系统上运行的服务器的名字，它可以运行在诸如 Linux, BSD, Solaris, HP-UX 以及 IRIX 上面。它支持很多其他的 FTP 服务器不支持的特征。

在vsftpd.conf中有如下内容定义了日志的记录方式：

```
# 表明FTP服务器记录上传下载的情况
xferlog_enable=YES 
# 表明将记录的上传下载情况写在xferlog_file所指定的文件中， 即xferlog_file选项指定的文件中
xferlog_std_format=YES 
xferlog_file=/var/log/xferlog 
# 启用双份日志。在用xferlog文件记录服务器上传下载情况的同时，
# vsftpd_log_file所指定的文件，即/var/log/vsftpd.log也将用来 记录服务器的传输情况
dual_log_enable=YES
vsftpd_log_file=/var/log/vsftpd.log
```

vsftpd的两个日志文件分析如下：

**/var/log/xferlog**

记录内容举例

```
Thu Sep 6 09:07:48 2007 7 192.168.57.1 4323279 /home/student/phpMyadmin-2.11.0-all-languages.tar.gz b -i r student ftp 0 * c
```

**/var/log/vsftpd.log**

记录内容举例

```
Tue Sep 11 14:59:03 2007 [pid 3460]    CONNECT: Client "127.0.0.1"
Tue Sep 11 14:59:24 2007 [pid 3459] [ftp] OK LOGIN;Client "127.0.0.1" ,anon password ”?" /var/log/xferlog日志文件中数据的分析和参数说明 
```

[![img](http://blog.51cto.com/attachment/201102/023711797.jpg)](http://blog.51cto.com/attachment/201102/023711797.jpg)

**FTP数字代码的意义**

```
110 重新启动标记应答。
120 服务在多久时间内ready。
125 数据链路端口开启，准备传送。
150 文件状态正常，开启数据连接端口。
200 命令执行成功。
202 命令执行失败。
211 系统状态或是系统求助响应。
212 目录的状态。
213 文件的状态。
214 求助的讯息。
215 名称系统类型。
220 新的联机服务ready。
221 服务的控制连接端口关闭，可以注销。
225 数据连结开启，但无传输动作。
226 关闭数据连接端口，请求的文件操作成功。
227 进入passive mode。
230 使用者登入。
250 请求的文件操作完成。
257 显示目前的路径名称。
331 用户名称正确，需要密码。
332 登入时需要账号信息。
350 请求的操作需要进一部的命令。
421 无法提供服务，关闭控制连结。
425 无法开启数据链路。
426 关闭联机，终止传输。
450 请求的操作未执行。
451 命令终止:有本地的错误。
452 未执行命令:磁盘空间不足。
500 格式错误，无法识别命令。
501 参数语法错误。
502 命令执行失败。
503 命令顺序错误。
504 命令所接的参数不正确。
530 未登入。 
532 储存文件需要账户登入。
550 未执行请求的操作。
551 请求的命令终止，类型未知。
552 请求的文件终止，储存位溢出。 
553 未执行请求的的命令，名称不正确。
```

vsftpd-2.3.4早期版本存在恶意的后门，在钟馗之眼上目前骇客以收到如此的主机，不过很多的服务器都已经被修复过，但总有漏网之鱼，有兴趣的小伙伴不妨去试试

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfilp52x4j20np0lfgmb.jpg)

## 0×02前言

```
  vsftpd-2.3.4早期版本存在恶意的后门，在钟馗之眼上目前骇客以收到如此的主机，不过很多的服务器都已经被修复过，但总有漏网之鱼，有兴趣的小伙伴不妨去试试，
```

## 0×03工具

```
 metasploit
 nmap  
```

## 0x04漏洞探测

```
 先经过nmap –script=vuln扫一遍，得出我们的目标主机的ftp是可以匿名登录的，并且版本也是2.3.4，当然这个主机 是metasploitable2
```

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfilwz2d7j20nf0280sl.jpg)

下一步，还等什么，看一下msf中有没有相关可以用的exp啊，这里我省略了搜索的过程，大家如果不知道怎么搜，可以搜他的服务，或者是他的版本，或者是其他的一些关键词，

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfim30nvej20nt0dfmxd.jpg)

针对 vsftp的2.3.4版本的exp只有这个，那我们就简单的试一下，老样子，还是看一下他需要的参数，然后我们再给他添加一些，基本上就可以了，喜欢细致的我，在说一遍这里的参数，其中 rhost 表示目标主机的ip地址，rport当然就是目标主机的端口啦，这里默认给出的是21，当然，任何端口都是可以进行修改的，所以大家在实际使用过程中要注意好端口哦

```
 这里的目标主机的ip是  10.0.10.104  在新版本的msf中，允许不添加payload参数，可有系统自动给出，所以这里我就不直接写上了
```

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfim7v9hlj20q00knwic.jpg)

## 0×05攻击

```
攻击的话，直接使用参数exploit就可以了，如果其中发生报错，我们就需要根据每一条去解决，
```

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfimdy2p6j20xc0cgju9.jpg)

执行到这里，基本上拿到了shell，而且是root权限，那我们就可以创建一个用户，用普通用户正大光明的登陆进去，然后在，想办法提权，

## 0×06分析

```
首先 msfconsole通过发送ack的探测包，确定了目标服务器的21端口的服务版本，通过返回的数据包可以明确的看到具体的版本，这里我也建议大家使用namp的时候用ack的扫描方式和syn的扫描方式，速度快而且准确，
```

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfimj5ooaj20xc0cgju9.jpg)

接着，msf通过匿名登录到目标服务器上，并尝试触发恶意后门，

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfimrilmmj20ov0azdh3.jpg)
![图片描述](https://segmentfault.com/img/bVSoDG?w=1249&h=359)

```
恶意后门被触发成功，并且反弹了一个shell给8这台机器，上半文是以前做的日志，下面抓包的图是现在截获的，所以大家就不要好奇了，同样在攻击者这边每次执行命令之后，都会发送到受害者这边，我们在wireshack截获的数据包中可以清晰的看到
```

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfimxtl7ij20v3034q3t.jpg)

这是刚刚触发的图，我们可以清楚的看到，这是而已代码执行，而没有建立会话，如果是会话会有提示符的，而这个没有，也就是说他可以使用当前的root用户执行任意命令，比如说添加一个用户

![](https://ws1.sinaimg.cn/large/b6de3d7dgy1fzfinlucyyj20t40a9n08.jpg)





klion大佬的叙述：https://klionsec.github.io/2017/12/09/vsftp-secfig/

