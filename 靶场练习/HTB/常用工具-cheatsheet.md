# 信息收集

## nmap
nmap 参数详解 https://www.freebuf.com/sectool/264941.html

（以下记录一些HTB中用过的方便再次查找）
标准服务检测：nmap -sV ip
扫描所有65535端口：nmap -p- ip


# 见过的端口和用到的端口命令

1. port 21 ftp：
  * anonymous/任意新建一个密码
  * ftp 常用命令 https://www.cnblogs.com/answerThe/p/11463117.html
  * 查看文件-get {filename}
  * bye  退出ftp
1. port 23 telnet: telnet {IP} (该说不说这种默认口令port23真的已经很少很少了吧)
1. port 445 SMB :
	* smbclient : 用于连接和传输文件的脚本（linux based）
	* 不知道用户账密时：使用访客或匿名模式查看分享文件夹，`smbclient -L {target_IP}`
	* 探知未配置访问控制的分享文件`smbclient \\\\{target_IP}\\{foldername}`
1. port 6379 Redis:
	* redis 未授权访问 https://blog.csdn.net/qq_41210745/article/details/103309137 
	* `info 获取redis信息`， `select {数字} 切换到指定数据库`，`keys * 列出所有key名称`，`get {key} 获取某个key的内容`