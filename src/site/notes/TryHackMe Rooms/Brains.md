---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/brains/","tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"],"created":"2026-03-31T19:25:54.559+02:00","updated":"2026-03-31T21:23:01.829+02:00","dg-note-properties":{"tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"]}}
---


## Description

## Recon

First a full scan of the system 
![](/img/user/Attachments/firstnmap.png)

Now a more targeted scanning of the services running on the system 
![](/img/user/Attachments/nmap2.png)

![](/img/user/Attachments/loginpage.png)
reset pw page
![](/img/user/Attachments/reset-pw-page.png)

A lot of these endpoints are 302's so not really useful. Gobuster also reveals a bunch of redirections and 400-like error messages.. 

![697](/img/user/Attachments/dirsearch.png)

Source code for the login page reveals something interesting ![](/img/user/Attachments/revealingsourcecode.png)

```
<!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp --><!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp -->



<!-- END EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension -->

<!-- START EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension: name:jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning: /forbiddenDomainWarning.jsp -->


<!-- END EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension -->

```

So, googling the error in the stack trace shows not much. However googling the Teamcity version quickly led me to find 2 **authentication bypass cve's**

I quickly find github repo's listing available exploits which are ready to use python scripts.
![](/img/user/Attachments/teamcity-cve-search.png)

Link: https://github.com/yoryio/CVE-2024-27198

![](/img/user/Attachments/cve1.png)

Saving the script and running it creates a new user and  gives full access to the system. 

And once I run the script I am able to log into the panel

![](/img/user/Attachments/loginsuccess.png)

A more detailed explanation of the vulnerabilities can be found from [[Rapid7https://www.rapid7.com/blog/post/2024/03/04/etr-cve-2024-27198-and-cve-2024-27199-jetbrains-teamcity-multiple-authentication-bypass-vulnerabilities-fixed/\|Rapid7https://www.rapid7.com/blog/post/2024/03/04/etr-cve-2024-27198-and-cve-2024-27199-jetbrains-teamcity-multiple-authentication-bypass-vulnerabilities-fixed/]]

![](/img/user/Attachments/rapid7-vuln-description.png)


Let's generate a SSH key 

`ssh-keygen -t ed25319 -C "administrator@domain.com" -f ~/.ssh/id_brains
![](/img/user/Attachments/ssh-generated.png)
