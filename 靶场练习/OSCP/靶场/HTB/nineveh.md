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

443端口进来之后就只有一张ninevehForAll.png。F12看了也没有其它提示或者内容，那就直接gobuster来扫。

可能有价值的信息是：

```
#http
/department           (Status: 301) [Size: 315] [--> http://10.10.10.43/department/]
/server-status        (Status: 403) [Size: 299]  
# https
/db                   (Status: 301) [Size: 309] [--> https://10.10.10.43/db/]
/server-status        (Status: 403) [Size: 300]                              
/secure_notes         (Status: 301) [Size: 319] [--> https://10.10.10.43/secure_notes/]
```

访问后可以看到一个登录口，而且还有一个报错，可能是提示。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-09-20_03-55-46.png)

这里还是得去找相应的组件漏洞，由于版本号只能看到是1.9.x，所以相应的漏洞都要看一看，1.9.3的漏洞，是弱口令引起的RCE。这里登录口的口令不是默认的admin，但是依然是个弱口令（password123），离谱的是我没爆出来。（爆出来的WP用的是seclists的twitter-banned.txt，此外WP也补充了一下，现在的HTB box是不涉及暴力破解的，这个box比较老了）

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-09-20_20-41-50.png)

登录进来是这个样子。按照1.9.3漏洞的描述，直接创建一个新的数据库，然后编辑数据库插入一句话。引用一个[利用样例](https://blog.51cto.com/penright/1116853)
