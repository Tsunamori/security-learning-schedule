# Beep

## 信息收集

```
└─$ sudo nmap -sV 10.10.10.7         
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-28 00:54 CDT
Nmap scan report for 10.10.10.7
Host is up (0.30s latency).
Not shown: 987 closed tcp ports (reset)
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 4.3 (protocol 2.0)
25/tcp    open     smtp       Postfix smtpd
53/tcp    filtered domain
80/tcp    open     http       Apache httpd 2.2.3
110/tcp   open     pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
111/tcp   open     rpcbind    2 (RPC #100000)
143/tcp   open     imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
443/tcp   open     ssl/http   Apache httpd 2.2.3 ((CentOS))
993/tcp   open     ssl/imap   Cyrus imapd
995/tcp   open     pop3       Cyrus pop3d
3306/tcp  open     mysql      MySQL (unauthorized)
4445/tcp  open     upnotifyp?
10000/tcp open     http       MiniServ 1.570 (Webmin httpd)
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 230.10 seconds
```

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-28_01-06-06.png)

80/443 访问后是一个登录框。首先尝试了默认密码 eLaStIx. 2oo7 没有成功。看时间写的是-2022,查组件漏洞都没有这么新的，就可以放弃这一点了。

扫一下目录。


