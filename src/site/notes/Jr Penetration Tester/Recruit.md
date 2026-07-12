---
{"dg-publish":true,"permalink":"/jr-penetration-tester/recruit/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup","#recruit","#jrpenetrationtester"],"dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup","#recruit","#jrpenetrationtester"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-12 09:39 +0200  
Nmap scan report for 10.113.170.194  
Host is up (0.021s latency).  
Not shown: 65532 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
53/tcp open  domain  
80/tcp open  http  
  
Nmap done: 1 IP address (1 host up) scanned in 14.68 seconds
```

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![nmap.png](/img/user/nmap.png)

### Hidden Directories

```bash
dirsearch -u http://TARGET
```

```
200 OK 
/config : Not found
/flag.txt : Not found
/login.php
/includes Not found
/register.php Not found
/uploads Not found 
/test Not found
/api/api
/api
/assets
/file.php "missing cv parameter" (point of entry?)
********
301 MOVED REDIRECT
/logout.php
/profile.php
/admin
/admin/index.php
/admin/upload.php
/mail
/dashboard.php
```


**File.php page**
![file.php.page.png](/img/user/file.php.page.png)
### The API 

```
The API is used to fetch for CVs. On the FAQ page /api.php it says /file.php?cv=<URL> is the target endpoint.
``` 

Furthermore: "The API supports fetching CVs from external URLs such as HTTP and HTTPS."

## Enumeration


Checking out **/main** endpoint reveals an email sent from HR to IT-support. 

Valid email format is found and confirmed 
```
it-support@recruit.thm
hr@recruit.thm
```

Furthermore, the mail reveals 2 things.
1. **hr** is a valid username for the login panel.
2. Administrator credentials are stored in the database

-----

## Exploitation



----

## Post-exploitation


------

## Attack Pattern Analysis (APA)