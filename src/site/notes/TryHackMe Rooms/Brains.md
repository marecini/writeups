---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/brains/","tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"],"created":"2026-03-31T19:25:54.559+02:00","updated":"2026-03-31T20:18:11.608+02:00","dg-note-properties":{"tags":["#tryhackme","brains","offensivesecurity","ethicalhacking","writeup"]}}
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

fuzzing with ..

![697](/img/user/Attachments/dirsearch.png)

Source code for the login page reveals something interesting ![](/img/user/Attachments/revealingsourcecode.png)

```
<!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp --><!-- START EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension: name:resetPasswordLink: /plugins/reset-password/resetPasswordLink.jsp -->



<!-- END EXTENSION CONTENT jetbrains.buildServer.resetPassword.ResetPasswordPageExtension -->

<!-- START EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension: name:jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning: /forbiddenDomainWarning.jsp -->


<!-- END EXTENSION CONTENT jetbrains.buildServer.web.forbiddenDomains.ForbiddenDomainHeaderWarning$Extension -->

```
