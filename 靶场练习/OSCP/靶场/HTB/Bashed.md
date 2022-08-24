# Bashed

## 信息收集

nmap 收集一下信息：

```
sudo nmap -sV 10.10.10.68
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-24 04:12 CDT
Nmap scan report for 10.10.10.68
Host is up (0.43s latency).
Not shown: 998 closed ports
PORT   STATE    SERVICE VERSION
53/tcp filtered domain
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.33 seconds
```

看起来也是一个突破口在网页的目标。

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-24T09-18-44.264Z.png)

看着东西很多，实际上页面能访问的只有index.html、single.html。

先dirsearch扫一下目录，结果如下：

```
Target: http://10.10.10.68/

[04:21:37] Starting:                                                   
[04:21:42] 301 -  308B  - /php  ->  http://10.10.10.68/php/
[04:21:43] 301 -  307B  - /js  ->  http://10.10.10.68/js/
[04:21:52] 403 -  297B  - /.ht_wsr.txt
[04:21:52] 403 -  300B  - /.htaccess.bak1
[04:21:52] 403 -  300B  - /.htaccess.orig
[04:21:52] 403 -  302B  - /.htaccess.sample
[04:21:52] 403 -  300B  - /.htaccess.save
[04:21:52] 403 -  300B  - /.htaccess_orig
[04:21:52] 403 -  301B  - /.htaccess_extra
[04:21:52] 403 -  298B  - /.htaccess_sc
[04:21:52] 403 -  298B  - /.htaccessBAK
[04:21:52] 403 -  299B  - /.htaccessOLD2
[04:21:52] 403 -  290B  - /.htm
[04:21:52] 403 -  298B  - /.htaccessOLD
[04:21:52] 403 -  291B  - /.html
[04:21:52] 403 -  296B  - /.htpasswds
[04:21:52] 403 -  300B  - /.htpasswd_test
[04:21:52] 403 -  297B  - /.httr-oauth
[04:21:56] 403 -  290B  - /.php
[04:21:56] 403 -  291B  - /.php3
[04:22:08] 200 -    8KB - /about.html
[04:22:44] 200 -    0B  - /config.php
[04:22:46] 200 -    8KB - /contact.html
[04:22:48] 301 -  308B  - /css  ->  http://10.10.10.68/css/
[04:22:51] 301 -  308B  - /dev  ->  http://10.10.10.68/dev/
[04:22:51] 200 -    1KB - /dev/
[04:22:59] 301 -  310B  - /fonts  ->  http://10.10.10.68/fonts/
[04:23:04] 301 -  311B  - /images  ->  http://10.10.10.68/images/
[04:23:04] 200 -    2KB - /images/
[04:23:08] 200 -    3KB - /js/
[04:23:25] 200 -  939B  - /php/
[04:23:39] 403 -  299B  - /server-status
[04:23:39] 403 -  300B  - /server-status/
[04:23:54] 301 -  312B  - /uploads  ->  http://10.10.10.68/uploads/
[04:23:54] 200 -   14B  - /uploads/
```

不过返回来看页面上这篇文章，有一些关键信息：

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-24T09-29-53.097Z.png)

再结合靶机的名字，不难猜测这里就是突破口。根据地址去github找[源码](https://github.com/Arrexel/phpbash)，大概意思就是这个项目可以传一个可视化的shell上去。

回到目录扫描记录，挨个访问状态200的目录，一般来说可以先挑存储体积比较大的或者目录名字比较敏感的看。

这里访问到/dev/的时候，发现路径下已经部署了phpbash.php，那肯定毫不客气先进shell看看。

![image](../../../../../../Downloads/Firefox_Screenshot_2022-08-24T09-50-14.874Z.png)

这个shell是个比较低的权限。

```
www-data@bashed:/var/www/html/dev# whoami
www-data
```

这个shell可以直接拿到user.txt。

然后做一下提权，就能拿到root.txt。

```
www-data@bashed
:/home/arrexel# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
www-data@bashed
:/home/arrexel# sudo -u scriptmanager bash -i

bash: cannot set terminal process group (822): Inappropriate ioctl for device
bash: no job control in this shell
scriptmanager@bashed:/home/arrexel$ exit
```