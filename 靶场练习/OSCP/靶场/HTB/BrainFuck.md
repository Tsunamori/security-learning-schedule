# BrainFuck

## 信息收集
Nmap过一下

```
sudo nmap -sV 10.10.10.17      
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-19 03:24 CDT
Nmap scan report for 10.10.10.17
Host is up (0.79s latency).
Not shown: 995 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp     Postfix smtpd
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 144.34 seconds

sudo nmap -T4 -A -p 22,25,110,143,443 10.10.10.17
[sudo] password for tsunamori: 
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-21 20:57 CDT
Nmap scan report for 10.10.10.17
Host is up (0.48s latency).
Not shown: 995 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) TOP USER CAPA UIDL PIPELINING AUTH-RESP-CODE RESP-CODES
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: ID Pre-login more ENABLE have IDLE SASL-IR IMAP4rev1 post-login LITERAL+ OK AUTH=PLAINA0001 LOGIN-REFERRALS listed capabilities
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.13 (92%), Linux 3.13 or 4.2 (92%), Linux 3.16 (92%), Linux 3.2 - 4.9 (92%), Linux 4.2 (92%), Linux 4.4 (92%), Linux 4.8 (92%), Linux 4.9 (91%), Linux 3.12 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   456.57 ms 10.10.16.1
2   456.61 ms 10.10.10.17

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 129.51 seconds
```

## 寻找入口

～～这里参考了[别人的WP](https://0xdf.gitlab.io/2022/05/16/htb-brainfuck.html)～～

这个题第一个需要注意的点是 SSL cert 的问题。

在前面nmap的扫描结果里，已经写了这个IP相关的域名和DNS解析：`brainfuck.htb www.brainfuck.htb sup3rs3cr3t.brainfuck.htb`，所以这里要在 **/etc/hosts** 里加上明确的 host 解析：
```
10.10.10.17 brainfuck.htb www.brainfuck.htb sup3rs3cr3t.brainfuck.htb
```

这里直接访问`https://10.10.10.17`会看到nginx 默认欢迎页，但如果输入`https://www.brainfuck.htb/`的话会跳转到一个wordpress页面。

上wpscan扫，这里中途尝试用dirsearch还是不行，wpscan有disable-tls-checks的参数可以用，估计差别在这里。

～～未完待续，从WP来说不是一个很难的题，只不过思路很重要，这个题后续我再回来自己打一遍～～