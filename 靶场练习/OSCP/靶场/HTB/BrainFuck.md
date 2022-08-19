# BrainFuck

## 信息收集
Nmap过一下

```
sudo nmap -sV 10.10.10.17      
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-19 03:24 CDT
Nmap scan report for 10.10.10.17
Host is up (0.79s latency).
Not shown: 995 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp     Postfix smtpd
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 144.34 seconds
```

## 寻找入口

**port 22**
按照之前的经验来看，基本不用先考虑这个port。

**port 25**
看起来没有什么突破点，留一下。

**port 110**
看起来没有什么突破点，留一下。

**port 143**
看起来没有什么突破点，留一下。

**port 443**
https?直接访问看一下，是nginx的默认页面。dirsearch扫不出东西。

