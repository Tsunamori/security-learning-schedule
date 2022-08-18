### 信息收集

首先nmap看一下端口，这里没有扫全部65535端口，因为速度慢太多了
`sudo nmap -sV ip`
得到如下信息：
```
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-17 04:48 CDT
Nmap scan report for 10.10.10.3
Host is up (0.35s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.07 seconds
```
### 寻找入口
接下来就是先根据端口服务看看有没有什么可能的漏洞。

**port 21**，尝试匿名者模式
```
sudo ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:tsunamori): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```
dir不能列目录，使用ls
```
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls -a
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
```
看起来没什么可以利用的东西。入口应该不在这里。

**port 22**
https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh

这里SSH没有更多信息，想要打入口只能暴力破解，不符合考试思路，所以大概率不是入口点，可以看下一个了。

**port 139、445**
这里两个port功能差不多，都是关于SMB的不同实现方式。
尝试了SMB的匿名连接，没成功。
```
sudo smbclient -L 10.10.10.3                                   
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```
不使用暴力破解的话，就只能根据SMB的服务类型去搜相应的exploit。
google 关键词 `netbios-ssn Samba smbd exploit`找到一篇[利用](https://amolblog.com/139-tcp-open-netbios-ssn-samba-smbd-3-x-4-x/)。

参照上文尝试做一下利用。
首先收集信息
```
sudo nmap -T4 -A -p 139,445 10.10.10.3                         
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-18 02:32 CDT
Nmap scan report for 10.10.10.3
Host is up (0.55s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Linux 2.6.23 (92%), Belkin N300 WAP (Linux 2.6.30) (92%), Control4 HC-300 home controller (92%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (92%), Dell Integrated Remote Access Controller (iDRAC5) (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Citrix XenServer 5.5 (Linux 2.6.18) (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

Host script results:
|_clock-skew: mean: 2h01m27s, deviation: 2h49m44s, median: 1m25s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2022-08-18T03:34:00-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 139/tcp)
HOP RTT       ADDRESS
1   598.38 ms 10.10.16.1
2   598.61 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.48 seconds
```
这里直接MSF打没成功，往下一步尝试。
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 192.168.163.128:4444 
[*] Exploit completed, but no session was created.
```
尝试1：enum4linux 没有找到workgroup，转下一尝试。
尝试2：smbmap 这里能看到只有tmp是有读写权限的。
```
smbmap -H 10.10.10.3
[+] IP: 10.10.10.3:445  Name: 10.10.10.3                                        
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        tmp                                                     READ, WRITE     oh noes!
        opt                                                     NO ACCESS
        IPC$                                                    NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$                                                  NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
```
于是使用smbclient尝试连接tmp文件，连接失败。
```
smbclient //10.10.10.3/tmp                                           
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```
这里查了一下，并不是存在访问控制，而是[本地smb的一些配置问题](https://unix.stackexchange.com/questions/562550/smbclient-protocol-negotiation-failed)，解决办法，打开`/etc/samba/smb.conf `，在`global setting`下添加如下两句：
```
client min protocol = CORE
client max protocol = SMB3
```
再次尝试连接，连接成功。
```
smbclient //10.10.10.3/tmp                                            1 ⨯
Enter WORKGROUP\tsunamori's password:  //这里直接回车
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> 
```
获取tmp下文件内容
```
smb: \> dir
  .                                   D        0  Thu Aug 18 03:03:27 2022
  ..                                 DR        0  Sat Oct 31 01:33:58 2020
  orbit-makis                        DR        0  Wed Aug 17 05:25:31 2022
  5576.jsvc_up                        R        0  Wed Aug 17 04:21:25 2022
  .ICE-unix                          DH        0  Wed Aug 17 04:20:22 2022
  vmware-root                        DR        0  Wed Aug 17 04:20:44 2022
  .X11-unix                          DH        0  Wed Aug 17 04:20:47 2022
  gconfd-makis                       DR        0  Wed Aug 17 05:25:31 2022
  .X0-lock                           HR       11  Wed Aug 17 04:20:47 2022
  vgauthsvclog.txt.0                  R     1600  Wed Aug 17 04:20:20 2022

                7282168 blocks of size 1024. 5385916 blocks available
smb: \> pwd
Current directory is \\10.10.10.3\tmp\
```
能读取的只有最后两个文件。