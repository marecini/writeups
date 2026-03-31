---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/brains/","tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"],"created":"2026-03-31T19:25:54.559+02:00","updated":"2026-03-31T20:48:21.204+02:00","dg-note-properties":{"tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"]}}
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

So, googling the error in the stack trace above led me to find 2 **authentication bypasses**. 
I quickly find github repo's listing available exploits which are ready to use python scripts.
![](/img/user/Attachments/teamcity-cve-search.png)

Link: https://github.com/yoryio/CVE-2024-27198

![](/img/user/Attachments/cve1.png)

Saving the script and running it creates a new user and apparently gives access to the page. 

An **Unexpected error** is seen once logged in.
![](/img/user/Attachments/unexpected_error.png)

A more detailed explanation of the vulnerabilities can be found from [Rapid7][()](https://www.rapid7.com/blog/post/2024/03/04/etr-cve-2024-27198-and-cve-2024-27199-jetbrains-teamcity-multiple-authentication-bypass-vulnerabilities-fixed/) 

![](/img/user/Attachments/rapid7-vuln-description.png)