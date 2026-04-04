---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/binex/","created":"2026-04-04T18:56:56.357+02:00","updated":"2026-04-04T19:26:02.142+02:00","dg-note-properties":{}}
---

![](/img/user/Attachments/redteaming2.png)
## Description


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

It looks like an AD-environment and SMB is running on the system. SMB is a good vector to target. Let's start there.

`smbclient -N -L \\\\10.114.173.81`

![](/img/user/Attachments/smbclient.png)

Not much is being revealed other than 2 administrator shares. 

`netexec smb 10.114.173.81`

![](/img/user/Attachments/netexec.png)

Now it's getting very interesting. From netexec's results it is confirmed that 
1. smb null authentification is present -> `Null Auth: True`
2. SMB relay attacks is possible -> `signing: false`
3. EternalBlue can be used due to SMBv1 is on the target -> `SMBv1: True`

**Logging in with Default Creds**
`netexec smb 10.114.173.81 -u 'admin' -p 'admin'`

The login works however the results show that the I only get a guest login likely with low privileges.

![](/img/user/Attachments/defaultlogin.png)
## Exploitation

## Post-exploitation

## Pwnage

First flag: 
Second flag: 

## Attack Pattern Analysis (APA)