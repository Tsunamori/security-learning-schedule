# Nibbles

## 信息收集

nmap收一下信息：

```
─$ sudo nmap -sV 10.10.10.75        
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-25 02:16 CDT
Nmap scan report for 10.10.10.75
Host is up (0.40s latency).
Not shown: 997 closed ports
PORT   STATE    SERVICE VERSION
22/tcp open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
53/tcp filtered domain
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.11 seconds
```

大概率还是web页面的突破口，收集一下信息。

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-25T07-20-02.190Z.png)

```
└─$ sudo python3 dirsearch.py -u 10.10.10.75                     130 ⨯

  _|. _ _  _  _  _ _|_    v0.4.2.8                                     
 (_||| _) (/_(_|| (_| )                                                
                                                                       
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 11374

Output File: /home/tsunamori/L/tools/dirsearch/reports/_10.10.10.75/_22-08-25_02-21-58.txt

Target: http://10.10.10.75/

[02:21:58] Starting:                                                   
[02:22:13] 403 -  297B  - /.ht_wsr.txt
[02:22:13] 403 -  300B  - /.htaccess.bak1
[02:22:13] 403 -  302B  - /.htaccess.sample
[02:22:13] 403 -  300B  - /.htaccess.orig
[02:22:13] 403 -  300B  - /.htaccess_orig
[02:22:13] 403 -  301B  - /.htaccess_extra
[02:22:13] 403 -  300B  - /.htaccess.save
[02:22:13] 403 -  298B  - /.htaccess_sc
[02:22:13] 403 -  298B  - /.htaccessBAK
[02:22:13] 403 -  298B  - /.htaccessOLD
[02:22:13] 403 -  299B  - /.htaccessOLD2
[02:22:13] 403 -  290B  - /.htm
[02:22:13] 403 -  291B  - /.html
[02:22:13] 403 -  300B  - /.htpasswd_test
[02:22:13] 403 -  296B  - /.htpasswds
[02:22:13] 403 -  297B  - /.httr-oauth
[02:22:17] 403 -  290B  - /.php
[02:22:17] 403 -  291B  - /.php3
[02:24:05] 403 -  300B  - /server-status/
[02:24:05] 403 -  299B  - /server-status
```

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-25_02-44-49.png)

目录并没有发现什么有意思的东西，但是查看源码时看到了提示，访问/nibbleblog/。

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-25T07-47-25.417Z.png)

