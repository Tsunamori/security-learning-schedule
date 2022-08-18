# 信息收集

## nmap
nmap 参数详解 https://www.freebuf.com/sectool/264941.html

（以下记录一些HTB中用过的方便再次查找）
标准服务检测：nmap -sV ip
扫描所有65535端口：nmap -p- ip


# 端口和用到的端口命令

## port 21 ftp
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

## port 23

1. port 23 telnet: telnet {IP} (该说不说这种默认口令port23真的已经很少很少了吧)

## port 139、445
https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb
```
For instance, on Windows, SMB can run directly over TCP/IP without the need for NetBIOS over TCP/IP. This will use, as you point out, port 445. On other systems, you’ll find services and applications using port 139. This means that SMB is running with NetBIOS over TCP/IP**.**
```
1. port 139
2. port 445 SMB :
	* smbclient : 用于连接和传输文件的脚本（linux based）
	* 不知道用户账密时：使用访客或匿名模式查看分享文件夹，`smbclient -L {target_IP}`
	* 探知未配置访问控制的分享文件`smbclient \\\\{target_IP}\\{foldername}`

## port 6379
1. port 6379 Redis:
	* redis 未授权访问 https://blog.csdn.net/qq_41210745/article/details/103309137 
	* `info 获取redis信息`， `select {数字} 切换到指定数据库`，`keys * 列出所有key名称`，`get {key} 获取某个key的内容`