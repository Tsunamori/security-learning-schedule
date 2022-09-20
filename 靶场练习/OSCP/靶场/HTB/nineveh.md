# nineveh

## 信息收集

nmap:

```
└─$ sudo nmap -sV 10.10.10.43
Starting Nmap 7.91 ( https://nmap.org ) at 2022-09-20 03:43 CDT
Nmap scan report for 10.10.10.43
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.33 seconds
```

80端口是默认页面，而443有证书信任弹窗，问题不大，可以强行访问。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-09-20_03-46-02.png)

进来之后就只有一张ninevehForAll.png。F12看了也没有其它提示或者内容，那就直接dirsearch来扫。

可能有价值的信息是：

```
[03:49:47] 200 -   11KB - /db/index.php
[03:49:48] 200 -   11KB - /db/
```

访问后可以看到一个登录口，而且还有一个报错，可能是提示。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-09-20_03-55-46.png)

这里还是得去找相应的组件漏洞，由于版本号只能看到是1.9.x，所以相应的漏洞都要看一看