---
{"dg-publish":true,"permalink":"/thm-rooms/decryptify/","tags":["tryhackme","decryptify","insecure-randomness","#offensivesecurity","ethicalhacking","#writeup","#marecini"],"created":"2026-03-31T17:31:09.113+02:00","updated":"2026-03-31T17:32:35.022+02:00","dg-note-properties":{"tags":["tryhackme","decryptify","insecure-randomness","#offensivesecurity","ethicalhacking","#writeup","#marecini"]}}
---


# Test note


## Description

## Reconnaissance

![nmapallpports.png](/img/user/Attachments/nmapallpports.png)

We now identify more specifically what services are running on these ports 
![nmap-specific-ports.png](/img/user/Attachments/nmap-specific-ports.png)


The source code on the homepage on the Apache server reveals there is an endpoint
* /api.php


![apipage.png](/img/user/Attachments/apipage.png)
And the front page shows us a standard login page 

![homepage.png](/img/user/Attachments/homepage.png)


## Enumeration

#### Fuzzing

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.112.167.252:1337/ -t 40 -x php,txt,json,asp -  
b 403-500 -o dirs.out
```

![gobuster.png](/img/user/Attachments/gobuster.png)

- **/logs** looks like an interesting endpoint to explore

The endpoint holds an **app.log** file which could be valuable.

![logsendpoiint.png](/img/user/Attachments/logsendpoiint.png)
Lets retrieve it
![down_app.log.png](/img/user/Attachments/down_app.log.png)

The content of the **app.log** file reveals an email and a code which is base64-encoded. Futhermore we see that this code seems to be a part of the login process.
![applogfile.png](/img/user/Attachments/applogfile.png)

**Deccrypting the code**

![decryptedb64.png](/img/user/Attachments/decryptedb64.png)

Encoded `MTM0ODMzNzEyMg==
Decoded`1348337122

It appears this is a unix timestamp. 
#### Brute-forcing API-endpoint

For the sake of it I decided to attempt brute-forcing the login with Hydra. 

```
hydra -l "" -P /usr/share/wordlists/rockyou.txt 10.112.156.53 -s 1337 http-post-form "/api.php:api-password=^PASS^:Incorrect  
password."
```

The port is specified since the service is running on a non-standard port. the username parameter is left blank since the `alpha@fake.thm` is **deactivated**

Using **DevTools** to explore relevant scripts **api.js** is found to have obfuscated code.
#### Obfuscated Code
```
function b(c,d){const e=a();return b=function(f,g){f=f-0x165;let h=e[f];return h;},b(c,d);}const j=b;function a(){const k=['16OTYqOr','861cPVRNJ','474AnPRwy','H7gY2tJ9wQzD4rS1','5228dijopu','29131EDUYqd','8756315tjjUKB','1232020YOKSiQ','7042671GTNtXE','1593688UqvBWv','90209ggCpyY'];a=function(){return k;};return a();}(function(d,e){const i=b,f=d();while(!![]){try{const g=parseInt(i(0x16b))/0x1+-parseInt(i(0x16f))/0x2+parseInt(i(0x167))/0x3*(parseInt(i(0x16a))/0x4)+parseInt(i(0x16c))/0x5+parseInt(i(0x168))/0x6*(parseInt(i(0x165))/0x7)+-parseInt(i(0x166))/0x8*(parseInt(i(0x16e))/0x9)+parseInt(i(0x16d))/0xa;if(g===e)break;else f['push'](f['shift']());}catch(h){f['push'](f['shift']());}}}(a,0xe43f0));const c=j(0x169);
```

#### Deobfuscated Code 

```
function b(c, d) {
  const e = a();
  return b = function (f, g) {
    f = f - 357;
    let h = e[f];
    return h;
  }, b(c, d);
}
const j = b;
function a() {
  const k = ["16OTYqOr", "861cPVRNJ", "474AnPRwy", "H7gY2tJ9wQzD4rS1", "5228dijopu", "29131EDUYqd", "8756315tjjUKB", "1232020YOKSiQ", "7042671GTNtXE", "1593688UqvBWv", "90209ggCpyY"];
  a = function () {
    return k;
  };
  return a();
}
(function (d, e) {
  const i = b, f = d();
  while (true) {
    try {
      const g = parseInt(i(363)) / 1 + -parseInt(i(367)) / 2 + parseInt(i(359)) / 3 * (parseInt(i(362)) / 4) + parseInt(i(364)) / 5 + parseInt(i(360)) / 6 * (parseInt(i(357)) / 7) + -parseInt(i(358)) / 8 * (parseInt(i(366)) / 9) + parseInt(i(365)) / 10;
      if (g === e) break; else f.push(f.shift());
    } catch (h) {
      f.push(f.shift());
    }
  }
}(a, 934896));
const c = j(361);

```

This piece of code stands out since it contains constants which appear to be either a **token/secret/API-key** or the likes. 
```

function a() {
  const k = ["16OTYqOr", "861cPVRNJ", "474AnPRwy", "H7gY2tJ9wQzD4rS1", "5228dijopu", "29131EDUYqd", "8756315tjjUKB", "1232020YOKSiQ", "7042671GTNtXE", "1593688UqvBWv", "90209ggCpyY"];
  a = function () {
    return k;
  };
  return a();
}

```
.

![api.js-script.png](/img/user/Attachments/api.js-script.png)



## Priv Escalation
## Payload Crafting

## Attack Pattern Analysis