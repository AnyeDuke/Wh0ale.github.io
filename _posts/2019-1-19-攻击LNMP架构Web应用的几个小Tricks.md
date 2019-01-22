---
layout:     post
title:      攻击LNMP架构Web应用的几个小Tricks
date:       2019-1-19
author:     Wh0ale
header-img: img/1.jpg
catalog: true
---

## 0x01 拉取源码

题干比较简单，我们用浏览器打开，发现提示`ERR_INVALID_HTTP_RESPONSE`，说明这个端口并非HTTP服务。用nmap进行端口指纹识别：

```bash
nmap -sV -p 62231 2018.mhz.pw
```

[![15190539687211.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/a763c5e8-ec97-4424-ba37-e6c1d5ccac97.e0b1509d5c0e.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/a763c5e8-ec97-4424-ba37-e6c1d5ccac97.jpg)

可见，这是一个rsync服务，我们使用rsync命令查看目录，发现只有一个目录，名为`pwnhub_6670.git`，将其拉取下来：

[![15190541067152.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/740189b6-13df-4519-b8fc-2dc53bcb97fa.ffbe35cad4cf.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/740189b6-13df-4519-b8fc-2dc53bcb97fa.jpg)

查看`pwnhub_6670.git`目录，发现这是一个git裸仓库（[git bare repository](http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/)）。

Git裸仓库怎么还原呢？其实非常简单，我们平时用的Github、Gitlab上存的所有仓库其实都是裸仓库，比如我们拉取vulhub的源码，执行`git clone https://github.com/vulhub/vulhub.git`，其中`vulhub.git`其实就是Github服务器上的一个裸仓库。可见，裸仓库一般以”项目名称.git“为名。

Git支持通过本地文件、SSH、HTTPS或GIT协议拉取信息。我们既然已经用rsync将裸仓库下载到本地了，所以只需要`git clone pwnhub_6670.git`即可将裸仓库拉取下来，成为一个标准的仓库：

[![15190547400346.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/3094d63b-7c5b-432d-86f5-1ffb704f6324.fd73fd4bf0e8.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/3094d63b-7c5b-432d-86f5-1ffb704f6324.jpg)

仓库`pwnhub_6670`文件夹下的内容就是我们需要审计的源代码。



## 0x02 SQL注入漏洞挖掘

作为一个出题人，我耍了点小阳谋。我用了一个叫[speed](https://github.com/SpeedPHP/speed)的小众PHP框架，但改了核心文件名为`core.php`。就是为了防止大家去找这个框架本身的漏洞导致走偏方向，所有有漏洞的代码都出在我写的代码中。

拿到源码仓库，第一步先看看`git log`和branch、tags等信息，也许会暴漏一些目标的敏感信息。这里没有，于是我们就应该把目标放向代码本身。

目标网站`http://2018.mhz.pw`只有简单的注册、登录功能，有关输入的代码如下：

```php
<?php
escape($_REQUEST);
escape($_POST);
escape($_GET);

function escape(&$arg) {
    if(is_array($arg)) {
        foreach ($arg as &$value) {
            escape($value);
        }
    } else {
        $arg = str_replace(["'", '\\', '(', ')'], ["‘", '\\\\', '（', '）'], $arg);
    }
}

function arg($name, $default = null, $trim = false) {
    if (isset($_REQUEST[$name])) {
        $arg = $_REQUEST[$name];
    } elseif (isset($_SERVER[$name])) {
        $arg = $_SERVER[$name];
    } else {
        $arg = $default;
    }
    if($trim) {
        $arg = trim($arg);
    }
    return $arg;
}
```

escape是将GPR中的单引号、圆括号转换成中文符号，反斜线进行转义；arg是获取用户输入的`$_REQUEST`或`$_SERVER`。显然，这里`$_SERVER`变量没有经过转义，先记下这个点。

全局没其他值得注意的地方了，所以开始看controller的代码。

```php
<?php
function actionRegister(){
    if ($_POST) {
        $username = arg('username');
        $password = arg('password');

        if (empty($username) || empty($password)) {
            $this->error('Username or password is empty.');
        }

        $email = arg('email');
        if (empty($email)) {
            $email = $username . '@' . arg('HTTP_HOST');
        }

        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            $this->error('Email error.');
        }

        $user = new User();
        $data = $user->query("SELECT * FROM `{$user->table_name}` WHERE `username` = '{$username}'");
        if ($data) {
            $this->error('This username is exists.');
        }

        $ret = $user->create([
            'username' => $username,
            'password' => md5($password),
            'email' => $email
        ]);
        if ($ret) {
            $_SESSION['user_id'] = $user->lastInsertId();
        } else {
            $this->error('Unknown error.');
        }
    }

}
```

以上是注册功能的代码，比较简单。用户名和密码是必填项，邮箱如果没有填写，则自动设置为”用户名@网站域名“。最后将三者传入create方法，create方法其实就是拼接了一个INSERT语句。

值得注意的是，网站域名是从`arg('HTTP_HOST')`中获取，也就是从`$_REQUEST`或`$_SERVER`中获取。因为`$_SERVER`没有经过转义，我们只需要在HTTP头Host值中引入单引号，即可造成一个SQL注入漏洞。

但email变量经过了`filter_var($email, FILTER_VALIDATE_EMAIL)`的检测，我们首先要绕过之。



## 0x03 FILTER_VALIDATE_EMAIL绕过

这就是今天第一个trick。这个点早在当初PHPMailer的[CVE-2016-10033](https://www.leavesongs.com/PENETRATION/PHPMailer-CVE-2016-10033.html)就提到过。

RFC 3696规定，邮箱地址分为local part和domain part两部分。local part中包含特殊字符，需要如下处理：

1. 将特殊字符用`\`转义，如`Joe\'Blow@example.com`
2. 或将local part包裹在双引号中，如`"Joe'Blow"@example.com`
3. local part长度不超过64个字符

虽然PHP没有完全按照RFC 3696进行检测，但支持上述第2种写法。所以，我们可以利用之绕过`FILTER_VALIDATE_EMAIL`的检测。

因为代码中邮箱是用户名、@、Host三者拼接而成，但用户名是经过了转义的，所以单引号只能放在Host中。我们可以传入用户名为`"`，Host为`aaa'"@example.com`，最后拼接出来的邮箱为`"@aaa'"@example.com`。

这个邮箱是合法的：

[![15190588848420.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/c99fb7d6-6173-47b5-a213-ed97cfe984c8.56524021cd22.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/c99fb7d6-6173-47b5-a213-ed97cfe984c8.jpg)

这个邮箱包含单引号，将闭合SQL语句中原本的单引号，造成SQL注入漏洞。



## 0x04 绕过Nginx Host限制

这是今天第二个trick。

我们尝试向目标注册页面发送刚才构造好的用户名和Host：

[![15190591584407.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/ee68755f-cbc5-4455-b8d2-6cb1b26f9a4c.e5c5b2be1b86.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/ee68755f-cbc5-4455-b8d2-6cb1b26f9a4c.jpg)

直接显示404，似乎并没有进入PHP的处理过程。

这就回到问题的本质了，Host头究竟是做什么的？

众所周知，如果我们在浏览器里输入`http://2018.mhz.pw`，浏览器将先请求DNS服务器，获取到目标服务器的IP地址，之后的TCP通信将和域名没有关系。那么，如果一个服务器上有多个网站，那么Nginx在接收到HTTP包后，将如何区分？

这就是Host的作用：用来区分用户访问的究竟是哪个网站（在Nginx中就是Server块）。

如果Nginx发现我们传入的Host找不到对应的Server块，将会发送给默认的Server块，也就是我们通过IP地址直接访问的那个Nginx默认页面：

[![15190596270561.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/d1e480b6-6d19-4977-9830-967de376c22e.cbd0d7f4e8fb.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/d1e480b6-6d19-4977-9830-967de376c22e.jpg)

默认网站并没有`/main/register`这个请求的处理方法，所以自然会返回404。

这里给出解决这个问题的两个方法，也许还有更多新方法我没有想到，欢迎补充。

### 法1

Nginx在处理Host的时候，会将Host用冒号分割成hostname和port，port部分被丢弃。所以，我们可以设置Host的值为`2018.mhz.pw:xxx'"@example.com`，这样就能访问到目标Server块：

[![15190606828564.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/d3ea0303-2829-4be5-90c4-8eb48be82cd9.bdc4dfdd5131.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/d3ea0303-2829-4be5-90c4-8eb48be82cd9.jpg)

如上图，成功触发SQL报错。



### 法2

当我们传入两个Host头的时候，Nginx将以第一个为准，而PHP-FPM将以第二个为准。

也就是说，如果我传入：

```
Host: 2018.mhz.pw
Host: xxx'"@example.com
```

Nginx将认为Host为`2018.mhz.pw`，并交给目标Server块处理；但PHP中使用`$_SERVER['HTTP_HOST']`取到的值却是`xxx'"@example.com`。这样也可以绕过：

[![15190608786400.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/6b2e4a08-0436-40c2-a835-60be44b8e8e4.0a3ac76105eb.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/6b2e4a08-0436-40c2-a835-60be44b8e8e4.jpg)

这个方法我以前在某群里提到过，只有Nginx+PHP会出现这个问题，Apache的情况下将会是另一个样子，此处不展开讨论。



## 0x05 Mysql 5.7 INSERT注入方法

这是今天第三个trick。

既然已经触发了SQL报错，说明SQL注入近在眼前。通过阅读源码中包含的SQL结构，我们知道flag在flags表中，所以不废话，直接注入读取该表。

### 插入显示位

因为用户成功登录后，将会显示出该用户的邮箱地址，所以我们可以将数据插入到这个位置。发送如下数据包：

```json
POST /main/register HTTP/1.1
Host: 2018.mhz.pw
Host: '),('t123',md5(12123),(select(flag)from(flags)))#"@a.com
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: multipart/form-data; boundary=--------356678531
Content-Length: 176

----------356678531
Content-Disposition: form-data; name="username"

"a
----------356678531
Content-Disposition: form-data; name="password"

aaa
----------356678531--
```

可见，我闭合了INSERT语句，并插入了一个新用户`t123`，并将flag读取到email字段。登录该用户，获取flag：

[![15190615665771.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/7ddc6542-5f28-415d-b153-1925ecf7cb45.9e327f6dfbad.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/7ddc6542-5f28-415d-b153-1925ecf7cb45.jpg)

flag是支付宝红包口令。:)



### 报错注入

为了降低难度，我特地给出了Mysql的报错信息，没想到居然还增加了难度，这一点我没考虑到，还是 @burnegg 同学提出来的解决方法。

很多同学上来就测试报错注入，但这里有两个需要绕过的坑：

1. 由于邮箱的限制，注入语句长度需要小于64位
2. Mysql 5.7 默认开启严格模式，部分字符串连接语法将导致错误：`ErrorInfo: Truncated incorrect INTEGER value`

我们可以不使用字符串连接语法，而使用`<`、`>`、`=`等比较符号来触发漏洞：

[![15190631626996.jpg](https://www.leavesongs.com/media/attachment/2018/02/20/81a89362-f875-4c7d-b48b-1f3bdaa10d6b.83b87208c757.jpg)](https://www.leavesongs.com/media/attachment/2018/02/20/81a89362-f875-4c7d-b48b-1f3bdaa10d6b.jpg)

更多INSERT注入相关内容，可以阅读[MySQL Injection in Update, Insert and Delete](https://www.exploit-db.com/docs/english/41275-mysql-injection-in-update,-insert,-and-delete.pdf)。







转自：

<https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html>

最近看到P牛的文章，仔细研究了几天发现真的是个安全宝藏，受用很多。从webshell到php各种姿势的利用，都学到不少。最近在家准备好好学下代码审计了，长时间不写代码，一些库的使用都快生疏了。这个假期好好加油。