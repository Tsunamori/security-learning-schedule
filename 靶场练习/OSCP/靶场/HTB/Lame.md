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
接下来就是先根据端口服务看看有没有什么可能的漏洞。