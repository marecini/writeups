---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/binex/","created":"2026-04-04T18:56:56.357+02:00","updated":"2026-04-04T21:21:24.052+02:00","dg-note-properties":{}}
---

![](/img/user/Attachments/redteaming2.png)
## Description


## Objectives

Enumerate the machine and get an interactive shell. Exploit an SUID bit file, use GNU debugger to take advantage of a buffer overflow and gain root access by PATH manipulation.

## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap1.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

```
% sudo nmap 10.114.173.81 -oN 2nmap.txt -sV -p 22,139,445 -sC  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-04 19:04 CEST  
Nmap scan report for 10.114.173.81  
Host is up (0.091s latency).  
  
PORT    STATE SERVICE     VERSION  
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol  
2.0)  
| ssh-hostkey:  
|   2048 3f:36:de:da:2f:c3:b7:78:6f:a9:25:d6:41:dd:54:69 (RSA)  
|   256 d0:78:23:ee:f3:71:58:ae:e9:57:14:17:bb:e3:6a:ae (ECDSA)  
|_  256 4c:de:f1:49:df:21:4f:32:ca:e6:8e:bc:6a:96:53:e5 (ED25519)  
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)  
Service Info: Host: THM_EXPLOIT; OS: Linux; CPE: cpe:/o:linux:linux_kernel  
  
Host script results:  
| smb2-security-mode:  
|   3:1:1:  
|_    Message signing enabled but not required  
| smb2-time:  
|   date: 2026-04-04T17:04:16  
|_  start_date: N/A  
|_nbstat: NetBIOS name: THM_EXPLOIT, NetBIOS user: <unknown>, NetBIOS MAC: <unknow  
n> (unknown)  
| smb-security-mode:  
|   account_used: guest  
|   authentication_level: user  
|   challenge_response: supported  
|_  message_signing: disabled (dangerous, but default)  
| smb-os-discovery:  
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)  
|   Computer name: thm_exploit  
|   NetBIOS computer name: THM_EXPLOIT\x00  
|   Domain name: \x00  
|   FQDN: thm_exploit  
|_  System time: 2026-04-04T17:04:16+00:00  
  
Service detection performed. Please report any incorrect results at https://nmap.o  
rg/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 15.73 seconds
```
## Enumeration

### Swiss Armyknife Netexec 

It looks like SMB is running on the system. SMB is a good vector to target first. Let's start there.

`smbclient -N -L \\\\10.114.173.81`

![](/img/user/Attachments/smbclient.png)

Not much is being revealed other than 2 administrator shares. 

`netexec smb 10.114.173.81`

![](/img/user/Attachments/netexec.png)

Now it's getting very interesting. From netexec's results it is confirmed that 
1. smb null authentification is present -> `Null Auth: True`
2. SMB relay attacks is possible -> `signing: false`
3. EternalBlue can be used due to SMBv1 is on the target -> `SMBv1: True`

Worth also noting that SMB is running on Unix - Samba not a Windows system. 

**Logging in with Default Creds**
`netexec smb 10.114.173.81 -u 'admin' -p 'admin'`

The login works however the results show that the I only get a guest login likely with low privileges.

![](/img/user/Attachments/defaultlogin.png)

Since this room is about exploiting binaries perhaps SMB is not the intended attack path. 

--------

Moving on to other and better things...

### Anonymous Login with RPC

Anonymous login with rpcclient is possible however enumeration via this method yielded nothing

`rpcclient -U "" 10.114.173.81`

```
# Commands yielded nothing
rpcclient > enumdomgroups
rpcclient > enumdomusers
```

![](/img/user/Attachments/rpcanon.png)


### Enum4linux

Let's upgrade and use a tool to enumerate this host. 

First grab the next generation of enum4linux from [github](https://github.com/cddmp/enum4linux-ng)

Start up python virtual environment and enter the directory of enum4linux

```
# to install the tool
cd enum4linux-ng

# step 2 
pip install -r requirements.txt 

# step 3 
python3 enum4linux-ng.py TARGET

```

**SSH Dictionary Attack**
Interesting findings here reveals valuable intel. First off it is confirmed that performing a dictionary attack is a a viable option due to the minimum length of the password is set to **5** which narrows the scope. Also additional useful information is available to us. 

**No Account Lockout**
Second interesting finding is it is confirmed that there is `Lockout threshold: None` so brute-forcing techniques is definitely worth considering as the system wont lock me out.

**Guest Access**
As already discovered guest access is wide open.

![](/img/user/Attachments/enum4linux445.png)

-----------

### SSH - Let's get to Digging

Now initially I tried different ways to attack the SSH. But without a username or a password it seems pointless. So I ran eum4linux again targeting usernames.

`python3 /opt/enum4linux-ng/enum4linux-ng.py -R 1000 10.114.173.81 -u '' -p ''`

	-R flag -> enumerates for users specified by a bulk size
	
* RID 500 = Admin
* RID 501 = Guest
* RID 1000+ = regular users

Finally several users are revealed. 

![](/img/user/Attachments/enum4linux-enumusers.png)

```
# 4 users are found 
kel        (RID 1000)
des        (RID 1001)
tryhackme  (RID 1002)
noentry    (RID 1003)
```

```
# RID = Relative Identifier - The number which is assigned to a security principal
	
500  = Built-in Administrator
501  = Built-in Guest
544  = Administrators group
545  = Users group
1000+ = Regular users created on the system
```

Let's compile these users to a list

`echo "kel\ndes\ntryhackme\nnoentry\nnobody" > users`

Let's generate a filtered password list based off of rockyou.txt with awk from the password criteria discovered from earlier.

`awk 'length>=5 && length<=8' /usr/share/wordlists/rockyou.txt > filteredpw.txt`


`hydra -L users -P filteredpw.txt 10.114.173.81 ssh -t 4`


## Exploitation

## Post-exploitation

## Pwnage

First flag: 
Second flag: 

## Attack Pattern Analysis (APA)