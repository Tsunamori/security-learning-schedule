# shocker

## 信息收集

```
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-23 02:50 CDT
Nmap scan report for 10.10.10.56
Host is up (0.43s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE VERSION
53/tcp   filtered domain
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.17 seconds
```

看一下80端的网页。

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-23T07-59-24.881Z.png)

F12也没找到什么内容，这个图片的路径是ip/bug.jpg。

dirsearch有看到一个`/cgi-bin/` 这个一开始没能引起注意，虽然是403但是还是要